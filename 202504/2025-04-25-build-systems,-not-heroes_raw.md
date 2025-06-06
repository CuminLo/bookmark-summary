Title: Build Systems, Not Heroes

URL Source: https://vitonsky.net/blog/2024/10/11/system-approach/

Published Time: Fri, 11 Oct 2024 13:21:00 GMT

Markdown Content:
October 11, 20245 minutes to read

Enterprise programming is the management of system complexity. The main goals of most enterprise projects are to minimize bugs, ensure scalability, and release as soon as possible. These goals are unreachable in projects where people rely on individual skills rather than on a system-based approach.

![Image 1: There Will Be Blood (2007)](https://vitonsky.net/files/60ef73f7a29ac2859e6a6e8a77fb58ad36d73d524f31b0a9c9e0f811c3526f3a.jpg)

Maybe you have seen videos where a worker [does their job exceptionally well](https://www.youtube.com/watch?v=YmkPoiEcgJM) on an [assembly line](https://www.youtube.com/watch?v=kUFdnMUb0rY), [oil rig](https://www.youtube.com/shorts/CSgEkjrGlbs?feature=share), or at a construction site. That is an ideal demonstration of individual skills. It looks impressive, but the problem is that if these workers will be under bus tomorrow, the same work would not be done as efficiently as before. This is because the workflow itself is not efficient, and only certain individuals are performing at a high level.

In these videos featuring "super-efficient" individuals, there may not seem to be any problems from a business perspective. These workers are likely paid the same as others but perform N times better, making them a great investment. However, it becomes a problem if the project tries to scale or if a manager promises certain deliverables based on current performance, and then the "super-efficient" worker leaves.

The system is more than just rules and people who follow them. It is based on the idea that individuals are weak and vulnerable to human factors (like the [bus factor](https://en.wikipedia.org/wiki/Bus_factor), [fundamental attribution error](https://en.wikipedia.org/wiki/Fundamental_attribution_error), etc.), but the problem can be minimized with a set of rules that everyone must follow.

For example, instead of trusting that programmers won’t push to the master branch, the repository owner can enable an automatic rejection for any attempts to push directly to the master branch, except through a pull request. This is a great example of a common practice that prevents potential problems. Programmers may not want to push to the master branch, but they could do it accidentally, as they are only human. I have personally caught myself doing this at times.

To be efficient in achieving goals, a team must always rely on system, not individuals. Individual efficiency is important and should be encouraged, but it must not be expected. Otherwise, you will have inconsistent quality, false expectations, no sustainability, and your business will struggle once your top programmer leaves for another job. This is why businesses must prioritize a system-based approach over trusting individuals.

So, how to think in system terms? Operate at a high level!

*   Improve workflows, not individuals. Make the process the lever to apply effort.
*   Identify problems that occur systematically, introduce processes to resolve them, and **prioritize process over people**.

Processes work like tests in programming. When you find a bug, it’s not enough to just fix it because the same bug could appear next week when someone else makes changes to the code. To properly fix a bug, the programmer must add a test to ensure the issue will caught if it will reproduce again.

It’s the same with processes. The process must solidify the problem-solving decision. The system should be able to function even if every person on the project is replaced with someone new. Therefore, the output of people’s work must be artifacts: solutions like code, collected data, and knowledge, such as documentation, decision logs, rules, and policies.

Here are some examples of typical system solutions in programming:

*   Make decisions based solely on written conversations, with an overview of all possible solutions, including all known positive and negative aspects of each.
*   Require material evidence, like measurements, to support any statement.
*   Use programming languages with static types and aggressively configured code analysis tools.
*   Set strict rules for linters to ban most dangerous language features and force programmers to write boring and obvious code.
*   Set up extensive testing and analysis jobs in pull request pipelines to detect and reject bad code as frequently as possible.
*   Make the code review process mandatory and hold those who approve pull requests accountable for quality, not the author of the changes.
*   Require programmers to add tests for any code changes.
*   Require programmers to use only immutable values.

The idea behind all these rules is to automate problem avoidance. Instead of relying on the belief that programmers on your project are disciplined specialists who don’t make mistakes, you can force them to write code in a way that makes it difficult to merge pull requests with bugs.

If you have strong enough rules, you don’t need to worry about the skills of the programmers. If they can pass through your barriers, then the code is fine, or the rules aren’t strong enough and need to be improved.

You may face resistance within your team when you introduce processes that allow you to rely on the system instead of individuals. Some programmers want to be experts on the project because it’s hard to fire an expert when the project relies on them. So, the system approach is an existential threat to them.

Once, we had performance issues due to unnecessary re-renders in a complex React project. Our investigation revealed that the problem was related to memoization, a fairly common issue. One of the solutions was to introduce a rule to memoize everything returned by hooks, to automatically prevent future problems with memoization. This idea was straightforward, simple, and stupid (that is good). However, we had a long discussion with one programmer who insisted it was a bad idea because of "premature optimization", the fact that React docs does not have recommendations about it, and "nobody does it that way". In my opinion, this was a clear case of a programmer resisting the system approach because he wanted to spend time fixing the same problems over and over and pretend to be working hard.
