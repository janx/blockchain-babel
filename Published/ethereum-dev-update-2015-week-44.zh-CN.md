# Ethereum Dev Update 2015 / Week 44
# 以太坊开发更新 2015年第44周

[Original Posted](https://blog.ethereum.org/2015/11/02/ethereum-dev-update-2015-week-44/) by Taylor Gerring on November 2nd, 2015.

With DEVCON1 a mere week away, the teams are excited and preparing to share all the great tools and technology the ecosystem has to offer. There will be hundreds of developers and dozens of talks including Nick Szabo, Vitalik Buterin, and Marley Gray from Microsoft. Tickets are limited and we anticipate selling out, so if you haven’t already registered, head on over to https://devcon.ethereum.org to secure your spot now!

离DEVCON1只剩下不到一周，开发团队已经按耐不住兴奋，要与大家分享以太坊生态中的各种激动人心的工具和科技了。参与本次大会的将有包括Nick Szabo, Vitalik Buterin以及来自微软的Marley Gray在内的演讲嘉宾以及数百名开发者。门票有限，即将售磬，如果你还没有订票的话赶快访问[https://devcon.ethereum.org](https://devcon.ethereum.org)吧！

## cpp-ethereum

TurboEthereum released 1.0.0 of the C++ tools. Work has been done on state trie pruning and upgrading network protocols (PV 62 & 63). Several Mix bugs have been fixed and the team is working on improving on the build/release process.

C++工具集TurboEthereum发布了1.0.0版本。这个版本包括了状态树(state trie)剪枝方面的工作以及网络协议升级(PV 62 & 63)，解决了一些Mix上的bug。目前C++团队正忙于改进构建与发布流程。

## go-ethereum

Geth preparing 1.3.0 release and continuing to build new features such as RPC with push support and Natural Language Specification (NatSpec), allowing users to be be prompted with friendlier confirmation messages. Filter search optimizations have been added, speeding up access to historical logs.

Geth正在准备1.3.0版本，这个版本将提供包括支持PUSH的远程调用，自然语言规范(NatSpec), 以及用户友好提示信息在内的一系列新功能。优化了的过滤搜索加快了对历史日志的读取。

## pyethereum

Pyethapp is continuing to move forward, with some upgrades to user services and RPC efficiency this week.

Python客户端Pyethapp本周的变化在用户服务和远程调用效率的改进。

## Swarm

Work towards Swarm accounting & incentive mechanisms being finalized along with a whitepaper on results of research. Initial integration of Swarm with IPFS as data store are beginning in the clients.

Swarm审计和激励机制的设计基本完成，研究结果将会以白皮书形式发布。将Swarm与IPFS集成作为客户端数据存储层的工作已经展开。

## Light client/mobile

New network protocols supporting fast sync are rapidly maturing and will be basis for future light clients, helping new users bootstrap into the network within just a few minutes.

支持快速同步的新网络协议已经越来越成熟，这将是未来轻客户端的基础，可以帮助新用户在数分钟之内同步整个网络。

## Mist

Wallet Beta 3 available soon with contract deployment and custom token support. Both the Go and C++ clients are integrated, giving users a choice of consensus implementations.

支持智能合约部署和自定义代币的新钱包测试版很快会发布。用户可以自由选择用Go或者C++客户端作为共识底层实现。

## Solidity

First proof-of-concept of formal verification for Solidity contracts now available with why3 tool. Additional attention has been given to internal types like tuples. Updated documentation online at https://ethereum.github.io/solidity/docs/home/

通过一个叫做Why3的工具我们得到了Solidity合约[形式化证明](https://forum.ethereum.org/discussion/3779/formal-verification-for-solidity-contracts)的首个概念原型(POC)。下一步工作会增加类似tuple的更多内部类型。[Solidity最新的文档已放到网上](https://ethereum.github.io/solidity/docs/home/)。
