# Serenity PoC2
# Serenity概念原型 乙

[Original post](https://blog.ethereum.org/2016/03/05/serenity-poc2/) by Vitalik Buterin on March 5th, 2016.

**After an additional two months of work after the release of the first python proof of concept release of Serenity, I am pleased to announce that Serenity PoC2 is now available. Although the release continues to be far from a testnet-ready client, much less a production-ready one, PoC2 brings with it a number of important improvements. First and foremost, the goal of PoC2 was to implement the complete protocol, including the basic corner cases (slashing bets and deposits), so as to make sure that we have a grasp of every detail of the protocol and see it in action even if in a highly restricted test environment. This goal has been achieved. Whereas PoC1 included only the bare minimum functionality needed to make Casper and EIP 101 run, PoC2 includes essentially the full Casper/Serenity protocol, EIP 101 and 105 included.**

**在Serenity的第一个python版本概念原型发布过去两周之后，今天我很高兴地宣布Serenity概念原型乙(Serenity PoC2)已经[完成了](http://github.com/ethereum/pyethereum/tree/serenity)！虽然这个版本依然离能在测试网络上运行客户端很遥远，更不要说和生产可用版本的差距，它也是包含了一系列重要的改进。概念原型乙最重要的目标是实现一个完整的协议，覆盖基本的边界情况(重复投注和区块的裁剪)，以便确保我们掌握了协议的每一个细微之处，并可以在测试环境中（即便是高度受限的环境）观测其实际运行情况。这个目标已经达到了。相对于仅包含支撑Casper和EIP101运行之最小功能集合的概念原型甲，概念原型乙实现了完整的Casper/Serenity协议，包括EIP101和105。**

*译注: EIP是[以太坊改进提案(Ethereum Improvement Proposal)](https://github.com/ethereum/EIPs/)的缩写。*

The specific features that can be found in PoC2 that were not available in PoC1 are as follows:

在概念原型乙中新增的具体特性如下：

* EIP 105 implementation – EIP 105 is the “sharding scaffolding” EIP, which will allow processing Ethereum transactions to be somewhat parallelized, and will set the stage for a later sharding scheme (which is yet to be determined). It uses the binary tree sharding mechanism described here to allow transactions to specify an “activity range” which restricts the addresses that transaction execution can touch, guaranteeing that sets of transactions with disjoint activity ranges can be processed in parallel. It also introduces SSTOREEXT and SLOADEXT opcodes to allow contracts to access storage of the same address in other shards (provided that the target shard is within the activity range); this mechanism essentially means that the binary shard tree serves as a super-contract sharding mechanism and a sub-contract sharding mechanism at the same time.

* **EIP 105的实现** - EIP 105是“分片脚手架”提案，目的是让以太坊上交易的处理可以在一定程度上并行化，并且为更后面阶段的分片(sharding)设计（尚未确定）打下基础。概念原型乙使用[这里](https://github.com/ethereum/EIPs/issues/53)描述的二叉树分片机制，使交易可以自行指定一个“活动范围”，从而限制交易在执行中可以访问的地址范围，保证指定了不同活动范围的交易可以并行的被处理。它还引入了`SSTOREEXT`和`SLOADEXT`两个操作码，使得合约可以操作其它分片中相同地址上的存储（前提是目标分片在同一个地址范围中）。因此这个二叉分片树实际上同时实现了超合约分片机制和次合约分片机制。

* Gas checking – the algorithm that pattern-matches a transaction to make sure that it correctly pays gas. Currently, this is accomplished by only accepting transactions going to accounts that have a particular piece of “mandatory account code“, which gives the account holder freedom to specify two pieces of code: the checker code and the runner code. Checker code is meant to perform quick checks such as signature and nonce verification; the pattern-matching algorithm gives a maximum of 250,000 gas for the checker code to run. Runner code is meant to perform any expensive operations that the transaction needed to carry out (eg. calling another contract with more than 250,000 gas). The main practical consequence of this is that users will be able to pay for gas directly out of contracts (eg. multisig wallets, ring signature mixers, etc) and will not need to separately always have a small amount of ETH in their primary account in order to pay for gas – as long as the gas payment from the contract is made within 250,000 gas all is good.

* **Gas检验** - 实现了通过对交易数据进行模式匹配来确保其正确支付Gas费用的算法。现在的做法是只接受发往特定类型账户的交易，此类账户必须包含一段特定的“[规定账户代码](https://github.com/ethereum/pyethereum/blob/serenity/ethereum/mandatory_account_code.py)”，规定代码允许账户所有者自由选择两段代码：checker code和runner code. Check code用来做快速检查，例如签名和nonce校验，它会分配到250,000个gas的费用上限。Runner code用来执行交易处理所需的开销大的操作（例如，使用超过250,000个gas来调用另外一个合约）。这个改动的主要实际用途，是用户现在可以让合约（例如多重签名钱包，环签名混币等）直接支付gas费用，而无需再自己持有一个保留少量以太币的账户用来支付gas费用 - 唯一的条件是合约支付费用的代码本身的执行费用不超过250,000个gas.

* Ring signature mixer – part of the test.py script now includes creating an instance of a ring signature verification contract which is designed as a mixer: five users send their public keys in alongside a deposit of 0.1 ETH, and then withdraw the 0.1 ETH specifying the address with a linkable ring signature, simultaneously guaranteeing that (i) everyone who deposited 0.1 ETH will be able to withdraw 0.1 ETH exactly once, and (ii) it’s impossible to tell which withdrawal corresponds to which deposit. This is implemented in a way that is compliant with the gas checker, providing the key advantage that the transaction withdrawing the 0.1 ETH does not need to be sent from an additional account that pays gas (something which a ring signature implementation on top of the current ethereum would need to do, and which causes a potential privacy leak at the time that you transfer the ETH to that account to pay for the gas); instead, the withdrawal transaction can simply be sent in by itself, and the gas checker algorithm can verify that the signature is correct and that the mixer will pay the miner a fee if the withdrawal transaction gets included into a block.

* **环签名混币合约** - test.py代码包含了一个环签名验证合约实例的创建，可以用来混币：五个用户各自发送0.1个以太币和公钥给这个合约，然后各自通过可关联环签名(linkable ring signature)提走0.1个以太币，这样就同时做到了 i) 每个存入0.1个以太币的用户都可以提走0.1个以太币，且 ii) 你无法得知哪一笔提取对应哪一笔存入。这个实现能通过Gas检验，因此有一个非常大的优点，也就是无需一个额外的账户来支付提取以太币的交易费用（如果在当前的以太坊上实现相同的环签名合约则需要这个额外的账户，这会导致潜在的隐私泄漏问题），相反的，你可以直接发出提取费用的交易，Gas检验代码会确认交易签名正确，然后混币合约就会在该交易打包进区块链时自己向矿工支付费用。

* More precise numbers on interest rates and scoring rule parameters – the scoring rule (ie. the mechanism that determines how much validators get paid based on how they bet) is now a linear combination of a logarithmic scoring rule and a quadratic scoring rule, and the parameters are such that: (i) betting absolutely correctly immediately and with maximal “bravery” (willingness to converge to 100% quickly) on both blocks and stateroots will get you an expected reward of ~97.28 parts per billion per block, or 50.58% base annual return, (ii) there is a penalty of 74 parts per billion per block, or ~36.98% annual, that everyone pays, hence the expected net return from betting perfectly is ~22 parts per billion per block, or ~10% annual. Betting absolutely incorrectly (ie. betting with maximum certainty and being wrong) on any single block or state root will destroy >90% of your deposit, and betting somewhat incorrectly will cause a much less extreme but still negative return. These parameters will continue to be adjusted so as to make sure that realistic validators will be able to be reasonably profitable.

* **更明确的利率和评分规则参数** - 现在的评分规则（根据验证人投注行为来分配其所获得回报的机制）是对数曲线和二次曲线评分规则的线性组合，其参数设置为：i) 在区块和状态树根上的投注都完全正确且有最大“勇气值”（代表快速达到100%收敛的意愿）的投注将获得大约每个块十亿分之97.28的回报，即50.58%的年回报率，ii) 在每一个块，向所有验证人收取十亿分之74的罚金，即大约36.98%的负年利率，因此完美投注的净回报期望是大约十亿分之22, 即大约10%的年回报率。在任何一个区块或是状态树根上的完全错误投注（带有最高确定度的错误投注）会毁掉超过90%的保证金，一定程度上的错误投注造成的损失会小得多，但依然会使回报变成负数。这些参数会持续被改进，以保证现实参与的验证人能得到合理的报酬。

* More precise validator induction rules – maximum 250 validators, minimum ether amount starts off at 1250 ETH and goes up hyperbolically with the formula min = 1250 * 250 / (250 - v) where v is the current active number of validators (ie. if there are 125 validators active, the minimum becomes 2500 ETH, if there are 225 validators active it becomes 12500 ETH, if there are 248 validators active it becomes 156250 ETH). When you are inducted, you can make bets and earn profits for up to 30 million seconds (~1 year), and after that point a special penalty of 100 parts per billion per block starts getting tacked on, making further validation unprofitable; this forces validator churn.

* **更明确的验证人就职规则** - 最多250个验证人，所需的最低保证金从1250个以太币起步，按公式`min = 1250 * 250 / (250 - v)`以双曲线速率增长，`v`是当前有效验证人数量（例如，假设现在有125位有效验证人，最低保证金就是2500个以太币，如果有225位有效验证人则需要12500个以太币，如果有248位有效验证人所需的最低保证金是156250个以太币）。当你加入后，你最长可以在三千万秒（大约一年）的时间内进行投注并获得回报，这个时间点之后将会启动一个特殊的惩罚机制，每个块收取十亿分之100的罚金，使得此时的验证工作没有利润，以此强迫验证人变动。

* New precompiles including ECADD and ECMUL (critical for ring signatures), MODEXP, RLP descent and the “gas deposit contract” (a mechanism used in the mandatory account code to pay for gas; theoretically it could be written in EVM code if need be but there may be efficiency concerns with that)

* **新的预编译合约** 包括`ECADD`, `ECMUL`（对于环签名很关键）, `MODEXP`, RLP decode和gas存入合约（规定账户代码用来支付gas费用的一种机制，理论上如果需要可以在EVM里面实现，但可能会有效率问题）。

* Rearchitecting of LOG and CREATE as precompiles – the opcodes still exist for backwards compatibility purposes, but they simply call the precompile addresses. This is a further move in the direction of “abstraction”.

* **将LOG和CREATE重构为预编译合约** - 出于向后兼容的原因依然保留这些操作码，但是它们现在只是简单的调用预编译的合约。这是往“抽象”发展的其中一步。

* New mechanism for betting directly on state roots

* 直接对状态树根进行投注的新机制

* Logic for detecting and slashing double bets and double blocks

* 检测和删除双重投注和双重出块的逻辑

* Logic for coming to consensus at a height even if a validator produced multiple blocks at that height

* 当验证人在某一高度产出多个块时依然可以达成共识的处理逻辑

The protocol decisions made here are by no means final; many of them are still actively being debated within the research channels. The next few rounds of PoC releases will thus move toward creating something resembling a Serenity node implementation, alongside a proper p2p networking layer, with the eventual goal of running a Serenity testnet between multiple computers; at the same time, our research team will continue hammering away at the finer details of the protocol and make sure that every single protocol decision is made correctly and well justified.

到目前为止协议中的一切决定都不是最终确定的，关于这些细节的争论依然在我们的研究聊天室中继续。接下来的概念原型将朝着Serenity节点实现的方向推进，包括一个合适的P2P网络层，最终的目标是运行起一个多计算机构成的Serenity测试网络。同时，我们的研究团队将继续锤炼协议细节，确保协议的每一个部分都是正确且合理的。

Additionally, we will be coming out with more accessible materials on the Casper protocol specification and design rationale in the next few weeks, covering both the broad consensus-by-bet concept as well as specific design decisions ranging from validator induction rules to betting mechanisms and block proposer selection.

接下来几周我们会发布关于Casper协议规格和设计理念的更多资料，覆盖从广义投注共识概念到具体设计决定，例如验证人参与规则，投注机制和区块提案人选择，的各个方面。
