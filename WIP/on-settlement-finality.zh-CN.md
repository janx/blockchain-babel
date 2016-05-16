# On Settlement Finality

[Origin post](https://blog.ethereum.org/2016/05/09/on-settlement-finality/) by Vitalik Buterin on May 9th, 2016

*Special thanks to Tim Swanson for reviewing, and for further discussions on the arguments in his original paper on settlement finality.*

*特别感谢Tim Swanson的审阅，以及他关于结算确定性的观点的进一步讨论。*

Recently one of the major disputes in ongoing debate between public blockchain and permissioned blockchain proponents is the issue of settlement finality. One of the simple properties that a centralized system at least appears to have is a notion of “finality”: once an operation is completed, that operation is completed for good, and there is no way that the system can ever “go back” and revert that operation. Decentralized systems, depending on the specific nature of their design, may provide that property, or they may provide it probabilistically, within certain economic bounds, or not at all, and of course public and permissioned blockchains perform very differently in this regard.

结算的确定性问题是最近公有链与许可链之间的一个主要战场。看起来中心化的系统至少有一个优点，即所谓的“确定性”（"finality"）：操作一旦完成，就永远完成了，系统永不可能再回退回去撤销这个操作。而去中心化的系统，根据设计的不同，可能有这个性质，也可能只提供在一定激励上限内的概率性的确定性，甚至没有确定性???，在这一点上公有链和许可链是有很大区别的。

This concept of finality is particularly important in the financial industry, where institutions need to maximally quickly have certainty over whether or not the certain assets are, in a legal sense, “theirs”, and if their assets are deemed to be theirs, then it should not be possible for a random blockchain glitch to suddenly decide that the operation that made those assets theirs is now reverted and so their ownership claim over those assets is lost.

确定性的概念在金融业中尤其重要，因为机构们需要以最快的速度确认资产是否已经在法律意义上是“他们的”了。而如果资产是他们的了，那就不能容忍一个随机产生的区块链分叉能撤销这笔转让，使他们又失去对资产的所有权。

In one of his recent articles, Tim Swanson argues:

Tim Swanson（译注：Director of Market Research at R3）在他最近的[一篇文章](http://tabbforum.com/opinions/settlement-risks-involving-public-blockchains)中写到：

> Entrepreneurs, investors and enthusiasts claim that public blockchains are an acceptable settlement mechanism and layer for financial instruments. But public blockchains by design cannot definitively guarantee settlement finality, and as a result, they are currently not a reliable option for the clearing and settling of financial instruments.

> 企业家，投资者和区块链的支持者们声称公有链可以作为金融工具的结算层。但是公有链从设计上就无法保证结算的确定性，因此他们目前无法成为金融工具清算和结算的可靠选择。

Is this true? Are public blockchains completely incapable of any notion of settlement finality, is it the case, as some proof of work maximalists imply, that only proof of work can provide true finality and it’s permissioned chains that are a mirage, or is the truth even more nuanced and complex? In order to fully understand the differences between the finality properties that different blockchain architectures provide, we will have to dig into the depths of mathematics, computer science and game theory – that is to say, cryptoeconomics.

这样的说法是正确的吗？公有链真的无法提供任何形式的结算确定性？还是说像某些工作量证明神教教徒讲的那样，只有工作量证明才有真正的确定性，许可链的确定性才是幻象？或者真相比这些说法更为微妙复杂？为了彻底理解不同区块链架构所提供的确定性，我们需要数学，计算机科学以及博弈论 - 或者说密码经济学（cryptoeconomics）- 的帮助。

## Finality is always probabilistic
## 一切确定性都有概率性

First of all, a very important philosophical point to make is that there is no system in the world that offers truly 100% settlement finality in the literal sense of the term. If share ownership is recorded on a paper registry, then it is always possible for the registry to burn down, or for a hooligan to run into the registry, draw a “c” in front of every “1” to make it look like a “9”, and run out. Even without any malicious attackers, it is also possible that one day everyone who knows the registry’s location will be struck by lightning and die simultaneously. Centralized computerized registries have the same problems, and arguably an attack is even easier to pull off, at least if the security of the central bank of Bangladesh is any indication.

首先我们要明确一个重要概念，这个世界上没有一个系统可以提供100%严格意义上的结算确定性。如果我们把共有权利登记在纸上，这个记录可能会被烧掉，或者某天有个坏人闯入登记机构，在每个数字1前都画一笔'c'，变成一个9。即使没有坏人捣乱，某天所有知道这个记录保管地点的人同一时间都被雷劈死的事情也是有概率发生的（译注：不代表译者观点...）。因此中心化的登记系统也有同样的问题（译注：指finality），甚至可以认为此时攻击更容易实施 - 也许前不久的[孟加拉国央行风波](http://www.wsj.com/articles/bangladesh-central-bank-found-100-million-missing-after-a-weekend-break-1457653764)能给我们一些启示。

In the case of fully on-chain “digital bearer assets” where there is no ownership other than the chain itself, the only recourse is a community-driven hard fork. In the case of using blockchains (permissioned or public) as registries for ownership of legally registered property (land, stocks, fiat currency, etc), however, it is the court system that is the ultimate source of decision-making power regarding ownership. In these case that the registry does fail, the courts can do one of two things. First, it is possible that the attackers find some way to get their assets out of the system before they can respond. In this case, the total quantity of assets on the ledger and the total quantity of assets in the real world no longer match up; hence, it is a mathematical certainty that someone with a finalized balance of x will eventually instead have to make do with an actual balance of y < x.

对于完全在链上的“不记名数字资产”，其所有权完全由区块链本身决定，因而只能靠社区驱动的硬分叉来追索。如果只是将区块链（许可链或者公有链）用作合法所有资产（土地，股票等等）的登记表，那么对资产所有权拥有最终裁决的就是司法系统了。如果这个登记表出了问题，法院会面临两种情况。第一，如果攻击者已经在司法系统反应之前设法将他们的资产转移走了，那么登记表上记录的资产数量和现实世界中的资产数量会产生不一致，因此必然会有某个人，虽然他确定性的拥有数量为`x`的资产，但是必须面对**实际资产**只有`y and y < x`的现实。

But the courts also have another alternative. They are absolutely not required to look at the registry in its standard presentation and take the results literally; it is the job of physical courts to look at intent, and determine that the correct response to the “c” drawn in front of the “1” is an eraser, not putting up one’s hands and agreeing that uncle Billy is now rich. Here, once again, finality is not final, although this particular instance of finality reversion will be to society’s benefit. These arguments apply to all other tools used to maintain registries and attacks against them, including 51% attacks on both public and consortium blockchains, as well.

法院还有另外一个选择。他们完全可以无视问题登记表，拒绝以它的字面记载为准；法院的需要做的是查清来龙去脉，以一块橡皮作为应对1被改写为9的正确方式，而不是举手赞同问题数据并表示现在所有人都更富有了。此时我们再一次看到，确定性并不确定，只不过这一次我们对确定性的违反是为了社会福祉。这些观点可以如数用到其他所有用于维护和攻击登记表的方式上，包括公有链和联盟链上的51%攻击。

The practical relevance of the philosophical argument that all registries are fallible is strengthened by the empirical evidence presented to us by the experience of Bitcoin. In Bitcoin, there have so far been three instances in which a transaction has been reverted after a long time:

比特币的经验告诉我们“所有的登记表都会出错”这个理论观点可能比想象的更加实际。在比特币的历史中一共出现过三次在足够长的时间之后交易被撤销的事件：

* In 2010, an attacker managed to give themselves 186 billion BTC by exploiting an integer overflow vulnerability. This was fixed, but at the cost of reverting half a day’s worth of transactions.
* In 2013, the blockchain forked because of a bug that existed in one version of the software but not another version, leading to part of the network rejecting a chain that was accepted as dominant by the other part. The split was resolved after 6 hours.
* In 2015, roughly six blocks were reverted because a Bitcoin mining pool was mining invalid blocks without verifying them

* 2010年，攻击者利用一个整数溢出缺陷[给自己创造了一千八百亿的比特币](https://en.bitcoin.it/wiki/Incidents#Value_overflow)。这个问题最终被修复，代价是大约半天之内的交易被撤销。
* 2013年，由于某个只在特定版本中存在的[bug](https://bitcoinmagazine.com/articles/bitcoin-network-shaken-by-blockchain-fork-1363144448)，比特币产生了分叉，导致一半网络拒绝接受另一半网络认为正确的链。这次分叉持续了6个小时。
* 2015年，因为矿池产生了无效区块[又不进行验证](https://www.reddit.com/r/Bitcoin/comments/3c2cfd/psa_f2pool_is_mining_invalid_blocks/)，大约6个块被撤销。

Out of these three incidents, it is only in the case of the third that the underlying cause is unique to public chain consensus, as the reason why the mining pool was acting incorrectly was precisely due to a failure of the economic incentive structure (essentially, a version of the verifier’s dilemma problem). In the other two, the failure was the result of a software glitch – a situation which could have happened in a consortium chain as well. One could argue that a consistency-favoring consensus algorithm like PBFT would have prevented the second incident, but even that would have failed in the face of the first incident, where all nodes were running code containing the overflow vulnerability.

在这三次事件中，只有导致第三次的根本原因是公有链独有的：这些矿池的非正常行为正是由于经济激励设计的问题导致的（本质上等价于[验证者悖论](https://eprint.iacr.org/2015/702.pdf)）。在另外两次事件中，罪魁则是软件错误 - 一个同样可以发生在联盟链中的问题。有观点认为类似PBFT这样偏好一致性的共识算法可以避免第二类事件的发生，但即便如此，由**所有**节点软件自身溢出缺陷导致的第一类事件依然无解。

Hence, one can make a reasonably strong case that if one is actually interested in minimizing failure rates, there is a piece of advice which may be even more valuable than “switch from a public chain to a consortium chain”: run multiple implementations of the consensus code, and only accept a transaction as finalized if all of the implementations accept it (note that this is already standard advice that we give to exchanges and other high-value users building on the Ethereum platform). However, this is a false dichotomy: if one wants to truly be robust, and one agrees with the arguments put forward by consortium chain proponents that the consortium trust model is more secure, then one should certainly do both.

因此我们有理由相信，如果你**真的**希望减少系统故障率，那么一个比“从公有链转向联盟链”**更有**价值的建议是：运行多个版本的共识协议实现，当且仅当一个交易被**所有**实现都接受的时候才认为它被最终确定了（这正是我们给交易所以及其他以太坊上项目的建议）。公有链/联盟链是一个错误的对立：如果你想要真正的健壮性，而且认同联盟链支持者所说的联盟信任模型更安全的观点，那你应该两者都用。

## Finality in Proof of Work
## 工作量证明中的确定性
