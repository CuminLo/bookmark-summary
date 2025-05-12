Title: Arbitrage for Fun and Profit

URL Source: https://www.thinkingthoughts.dev/blog/arbitrage

Markdown Content:
My 2024 was characterized by diving deep into the rabbit hole of MEV (Maximal Extractable Value). I started knowing nothing about how it worked, but was instantly intrigued by its shadowy PVP nature. After extensive research, I decided to take a shot at writing an [arbitrage bot](https://github.com/Zacholme7/BaseBuster) - one of the most common forms of MEV bots. A quick look at the top of any block is sure to reveal several arbitrage transactions. Throughout this post, I’ll walk through my journey from complete novice to having a bot that can successfully land transactions in today’s brutal MEV ecosystem.

Background
----------

Arbitrage isn’t new - it has existed in traditional finance for decades, typically dominated by massive hedge funds that dig tunnels to shave milliseconds off their latency. For newcomers, arbitrage simply means capturing pricing discrepancies between different financial venues. Here’s a concrete example: imagine Uniswap is trading ETH/USD at 1/100 while SushiSwap is trading at 1/150. In this case, we can swap 1 ETH for 150 USD on SushiSwap, then immediately swap that 150 USD back for 1.5 ETH on Uniswap, pocketing a 0.5 ETH profit! Even better, this can be executed atomically with minimal capital using flash loans.

Core Components of an Arbitrage Bot
-----------------------------------

While specific implementations vary, arbitrage bots share several fundamental components:

1.  Path Discovery: You need a set of paths that you can arbitrage over. This can be structured as a graph or precomputed paths.
2.  Opportunity Detection: A system to process these paths and identify profitable arbitrage opportunities, typically using graph traversal algorithms combined with either amount-out simulations or off-chain calculations.
3.  Execution Contract: A smart contract that can execute your arbitrage transactions when a valid path is found.

### BaseBuster Architecture

BaseBuster is my L2 arbitrage bot that consumed six months of my life and countless hours of development. While my days of arbitrage have (temporarily) concluded and I didn’t make significant profits, I learned an immense amount and joined the relatively small group of developers who have successfully landed arbitrage transactions.

### Chain State Management

The foundation of any arbitrage bot is accurate chain state management. In this context, chain state refers to the current state of pools at a specific block height - something rarely discussed but absolutely critical. You can’t find arbitrage opportunities without knowing the exact state of the pools you’re targeting.

Initially, I planned to use [amm-rs](https://github.com/darkforestry/amms-rs) like most developers. However, when it failed to compile, I did the reasonable thing and fixe… spent five months building my own solution: [PoolSync](https://github.com/Zacholme7/PoolSync). This battle-tested application accurately syncs state from DeFi pools and has proven extremely reliable in production.

### Pool Selection Strategy

One hard-learned lesson was the crucial importance of pool selection. You need to strike a perfect balance between pools with high activity and those with sufficient liquidity. There’s no point in searching dead pools with no volume or liquidity.

My filtering process involved multiple layers:

1.  Querying Birdeye for top tokens by volume (assuming volume correlates with volatility)
2.  Matching these against the raw pool set to find pools where both base/quote pairs were high-volume tokens
3.  Filtering by slippage metrics using REVM for simulation

The slippage calculation was particularly tricky. Simple amount-out calculations don’t account for pool constraints - you need to perform actual swaps and measure the output. This requires finding the correct balance slots in contracts, which isn’t standardized across pools. I used an access list inspector with balance calls to deduce the proper slots.

```
let mut insp = AccessListInspector::default();

// Populate inspector via transact
let mut evm = Evm::builder()
    .with_db(&mut nodedb)
    .with_external_context(&mut insp)
    .modify_tx_env(|tx| {
        tx.caller = account;
        tx.transact_to = TransactTo::Call(token);
        tx.data = calldata.clone().into();
    })
    .append_handler_register(inspector_handle_register)
    .build();
let _ = evm.transact();
drop(evm);
let access_list = insp.access_list().0;

// Process access list to find balance slot
for item in &access_list {
    if item.address == token {
        // Case 1: Single storage key that's not implementation slot
        if item.storage_keys.len() == 1 && !item.storage_keys.contains(&implementation_slot) {
            slot_map.insert(token, item.storage_keys[0]);
            break;
        }

        // Case 2: Multiple storage keys - look for known slots first
        for &known_slot in &known_slots {
            if item.storage_keys.contains(&known_slot) {
                slot_map.insert(token, known_slot);
                break;
            }
        }

        // Case 3: Take first non-implementation slot if no known slots found
        if !slot_map.contains_key(&token) {
            for &slot in &item.storage_keys {
                if slot != implementation_slot {
                    slot_map.insert(token, slot);
                    break;
                }
            }
        }
    }
}
```

From there, you can perform full cycle swaps and only keep pools within a specified slippage threshold.

### Path Searching and Selection

For path discovery, you can either:

1.  Perform a DFS on a graph every block
2.  Search over precomputed paths

I chose the second approach. After establishing our pool set, we compute all possible arbitrage paths of a certain depth. The key optimization is only checking paths containing pools that were touched in the latest block - there’s no reason to check for arbitrage opportunities in untouched pools since their prices haven’t changed.

To implement this, we take a state diff trace of each block to identify touched contracts, then match these against our indexed paths to quickly find relevant paths to check.

```
pub async fn debug_trace_block<T: Transport + Clone, N: Network, P: Provider<T, N>>(
    client: Arc<P>,
    block_tag: BlockNumberOrTag,
    diff_mode: bool,
) -> Vec<BTreeMap<Address, AccountState>> {
    let tracer_opts = GethDebugTracingOptions {
        config: GethDefaultTracingOptions::default(),
        ..GethDebugTracingOptions::default()
    }
    .with_tracer(BuiltInTracer(PreStateTracer))
    .with_prestate_config(PreStateConfig {
        diff_mode: Some(diff_mode),
        disable_code: Some(false),
        disable_storage: Some(false),
    });
    let results = client
        .debug_trace_block_by_number(block_tag, tracer_opts)
        .await
        .unwrap();

    let mut post: Vec<BTreeMap<Address, AccountState>> = Vec::new();

    for trace_result in results.into_iter() {
        if let TraceResult::Success { result, .. } = trace_result {
            match result {
                GethTrace::PreStateTracer(PreStateFrame::Diff(diff_frame)) => {
                    post.push(diff_frame.post)
                }
                _ => warn!("Invalid trace"),
            }
        }
    }
    post
}
```

### Profitability Calculations

While you can simulate paths on-chain, off-chain calculations are generally faster and more efficient. The challenge lies in balancing speed with accuracy. We use a hybrid approach:

1.  Calculate rates for standard input amounts
2.  Use these rates to estimate profitability through simple multiplication
3.  Validate promising paths with more precise simulations

For example, if we see 1 ETH = 150 USD on one path and 1 USD = 0.01 ETH on the reverse path, multiplying these rates (150 \* 0.01 = 1.5) shows potential profit since the result exceeds 1.

### Advanced Simulation Techniques

Our simulation strategy evolved through several iterations:

1.  Initial Approach: Using debug\_traceCall (too slow due to RPC bottlenecks)
2.  REVM Integration: Local simulations, but still RPC-dependent for state access
3.  Direct Database Integration: Developed [NodeDB](https://github.com/Zacholme7/NodeDB) to interface directly with RethDB for REVM
4.  Custom State Management: Building a focused database with only relevant storage slots

The final approach minimizes RPC calls by maintaining only the essential state information needed for our specific arbitrage calculations.

### Contract

Our execution contract is optimized for minimal state reads. Instead of using high-level swap functions that access unnecessary state, I implemented direct, low-level swap logic. This allows precise control over which storage slots are accessed, making our simulations faster and more reliable.

The contract uses Aave flash loans for capital efficiency and includes safety checks to revert if an opportunity becomes unprofitable during execution.

Conclusion
----------

While my arbitrage journey didn’t result in massive profits, it provided invaluable insights into MEV, blockchain architecture, and high-performance systems design. This article was a relatively brief, high level overview of my system. There are many undiscussed design decisions and complexities that went into each area of the bot. The code is the best reference to understand more, but feel free to contact me directly with any lingering questions! The ecosystem continues to evolve, and I hope sharing these learnings helps others entering the space.
