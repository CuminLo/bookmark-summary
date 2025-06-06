# 套利，为乐趣和利润 --- Arbitrage for Fun and Profit
- URL: https://www.thinkingthoughts.dev/blog/arbitrage
- Added At: 2025-05-12 04:51:18
- [Link To Text](2025-05-12-套利，为乐趣和利润-----arbitrage-for-fun-and-profit_raw.md)

## TL;DR
作者开发了BaseBuster套利机器人研究MEV，详细解析了套利机制、路径搜索、状态管理等核心技术，虽未盈利但积累了宝贵经验。项目涉及闪电贷、REVM模拟等高级技术，完整代码已开源。

## Summary
1. **MEV探索背景**  
   - 作者2024年深入研究了MEV（最大可提取价值），对其隐秘的PVP性质产生兴趣  
   - 开发了名为[BaseBuster](https://github.com/Zacholme7/BaseBuster)的套利机器人，这是最常见的MEV机器人类型之一  

2. **套利基础概念**  
   - 传统金融中已存在数十年，通常由对冲基金主导  
   - 定义：捕捉不同金融场所之间的价格差异  
   - 示例：  
     - Uniswap ETH/USD价格为1/100  
     - SushiSwap ETH/USD价格为1/150  
     - 通过原子交易和闪电贷可实现0.5 ETH利润  

3. **套利机器人核心组件**  
   - **路径发现**：构建可套利的路径图或预计算路径  
   - **机会检测**：  
     - 使用图遍历算法  
     - 结合输出量模拟或链下计算  
   - **执行合约**：发现有效路径时执行交易的智能合约  

4. **BaseBuster架构**  
   - 历时6个月开发的L2套利机器人  
   - 虽未获得显著利润，但积累了宝贵经验  
   - 成功进入少数能完成套利交易的开发者行列  

5. **链状态管理**  
   - 关键：准确同步特定区块高度的池状态  
   - 开发了[PoolSync](https://github.com/Zacholme7/PoolSync)工具：  
     - 替代无法编译的amm-rs  
     - 可靠同步DeFi池状态  

6. **池选择策略**  
   - 需平衡高活跃度和充足流动性  
   - 多层过滤流程：  
     1. 通过Birdeye查询高交易量代币  
     2. 匹配基础/报价对均为高交易量代币的池  
     3. 使用REVM模拟滑点指标  
   - 滑点计算挑战：  
     - 需实际执行交换测量输出  
     - 通过访问列表检查器推断正确的余额槽位  

7. **路径搜索与选择**  
   - 两种方法：  
     1. 每区块执行图深度优先搜索  
     2. 搜索预计算路径  
   - 优化：仅检查最新区块中被触及的池  
   - 实现方式：  
     - 获取区块状态差异追踪  
     - 匹配索引路径快速定位相关路径  

8. **盈利能力计算**  
   - 混合方法：  
     1. 计算标准输入量的汇率  
     2. 通过简单乘法估算盈利能力  
     3. 对潜力路径进行精确模拟验证  
   - 示例：150 * 0.01 = 1.5 > 1 显示潜在利润  

9. **高级模拟技术演进**  
   - 四个发展阶段：  
     1. 初始：使用debug_traceCall（RPC瓶颈导致速度慢）  
     2. REVM集成：本地模拟但仍依赖RPC状态访问  
     3. 直接数据库集成：开发[NodeDB](https://github.com/Zacholme7/NodeDB)对接RethDB  
     4. 定制状态管理：仅维护必要存储槽的精简数据库  

10. **执行合约优化**  
    - 最小化状态读取  
    - 实现底层交换逻辑替代高级交换函数  
    - 特点：  
      - 使用Aave闪电贷提高资本效率  
      - 包含安全检查确保机会失效时回滚  

11. **项目总结**  
    - 虽未获得巨额利润，但获得了关于MEV、区块链架构和高性能系统设计的深刻见解  
    - 文章仅概述了系统设计，更多细节见代码库  
    - 欢迎直接联系作者探讨未解答的问题
