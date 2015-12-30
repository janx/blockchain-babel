# Understanding Serenity, Part 2: Casper
# 理解Serenity 第二部分: Casper

[Original post](https://blog.ethereum.org/2015/12/28/understanding-serenity-part-2-casper/) by Vitalik Buterin, on December 28th, 2015

**Special thanks to Vlad Zamfir for introducing the idea of by-block consensus and convincing me of its merits, alongside many of the other core ideas of Casper, and to Vlad Zamfir and Greg Meredith for their continued work on the protocol.**

**特别感谢Vlad Zamfir，他提出了by-block???共识这个想法，并且说服我认同它的价值和Casper的许多核心想法；以及Vlad Zamfir和Greg Meredith，为他们在这个协议上的持续努力。**

In the last post in this series, we discussed one of the two flagship feature sets of Serenity: a heightened degree of abstraction that greatly increases the flexibility of the platform and takes a large step in moving Ethereum from “Bitcoin plus Turing-complete” to “general-purpose decentralized computation”. Now, let us turn our attention to the other flagship feature, and the one for which the Serenity milestone was originally created: the Casper proof of stake algorithm.

在这个系列的上一篇中，我们讨论了Serenity的两大旗舰特性之一：深度抽象，它将给平台带来极大的灵活性，使以太坊从“比特币加图灵完备”向“通用去中心化的计算”迈进一大步。现在，让我们把注意力转向另一个主要特性，也是当初设立Serenity里程碑的原因：Casper权益证明算法。

## Consensus By Bet
## 投注共识

The keystone mechanism of Casper is the introduction of a fundamentally new philosophy in the field of public economic consensus: the concept of consensus-by-bet. The core idea of consensus-by-bet is simple: the protocol offers opportunities for validators to bet against the protocol on which blocks are going to be finalized. A bet on some block X in this context is a transaction which, by protocol rules, gives the validator a reward of Y coins (which are simply printed to give to the validator out of thin air, hence “against the protocol”) in all universes in which block X was processed but which gives the validator a penalty of Z coins (which are destroyed) in all universes in which block X was not processed.

Casper向公开经济学共识这个领域引入了一个根本上全新的理念作为自己的基础：投注共识。投注共识的核心思想很简单：为验证人(validator)提供**与协议**对赌哪个块会被最终确定(finalized)的机会。在这里对某个区块X的投注就是一笔交易，在**所有区块X被处理了的世界中**都会带给验证人Y个币的奖励（奖励是凭空“印”出来的，因而是“与协议”对赌），而在所有区块X没有被处理的世界中会对验证人收取Z个币的罚款（罚金被销毁）。

(译注：一个世界(world)在这里应理解为区块链未来的一种可能状态。)

The validator will wish to make such a bet only if they believe block X is likely enough to be processed in the universe that people care about that the tradeoff is worth it. And then, here’s the economically recursive fun part: the universe that people care about, ie. the state that users’ clients show when users want to know their account balance, the status of their contracts, etc, is itself derived by looking at which blocks people bet on the most. Hence, each validator’s incentive is to bet in the way that they expect others to bet in the future, driving the process toward convergence.

只有在相信区块X在人们关心的那个世界中被处理的可能性足够大时，这才是值得去做的交易，验证人才会愿意投注。然而，接下来就是经济上递归的有趣部分：人们关心的那个世界，也就是用户的客户端在用户想要知道他们的账户余额或是合约的状态时所展现的那个状态，**本身就是根据人们对哪个区块投注最多推导出来的**。因此，每一个验证人都具有根据他们所预期的其他人的投注情况进行投注的动机，驱使这个过程走向收敛。

A helpful analogy here is to look at proof of work consensus – a protocol which seems highly unique when viewed by itself, but which can in fact be perfectly modeled as a very specific subset of consensus-by-bet. The argument is as follows. When you are mining on top of a block, you are expending electricity costs E per second in exchange for receiving a chance p per second of generating a block and receiving R coins in all forks containing your block, and zero rewards in all other chains:

一个有帮助的类比是工作量证明共识 - 它本身看起来非常独特，实际上可以成为投注共识的一个特别子模型。理由如下：当你基于一个块挖矿时，你是在花费每秒`E`的电力成本换取每秒`p`的出块概率，并且在**所有包含你的出块的分叉中**获得`R`个币，在其它分叉中分文不得。

![PoW as submodel](understanding-serenity-part-ii-casper/pow-as-submodel.png)

Hence, every second, you receive an expected gain of `p*R-E` on the chain you are mining on, and take a loss of E on all other chains; this can be interpreted as taking a bet at `E:p*R-E` odds that the chain you are mining on will “win”; for example, if p is 1 in 1 million, R is 25 BTC ~= $10000 USD and E is $0.007, then your gains per second on the winning chain are 0.000001 * 10000 - 0.007 = 0.003, your losses on the losing chain are the electricity cost of 0.007, and so you are betting at 7:3 odds (or 70% probability) that the chain you are mining on will win. Note that proof of work satisfies the requirement of being economically “recursive” in the way described above: users’ clients will calculate their balances by processing the chain that has the most proof of work (ie. bets) behind it.

因此，每一秒钟，在你挖矿的链上你可以获得`p*R-E`的期望收益，在其它链上遭受`E`的损失；因此你的挖矿选择可以理解为下注赌你所在的链有`E:p*R-E`的相对概率(odds)胜出。比如，假设p等于百万分之一，R是25个币约等于10000美元，而E是0.007美元，则你在胜出链上每秒钟的期望收益是`0.000001 * 10000 - 0.007 = 0.003`，你在失败链上的损失是`0.007`的电力成本，因此你是在赌自己挖矿的链有7:3的相对概率（或者说70%的概率）胜出。注意工作量证明满足上面所说的经济上递归的要求：用户的客户端通过处理拥有最大工作量的那条链来计算其账户余额。

Consensus-by-bet can be seen as a framework that encompasses this way of looking at proof of work, and yet also can be adapted to provide an economic game to incentivize convergence for many other classes of consensus protocols. Traditional Byzantine-fault-tolerant consensus protocols, for example, tend to have a concept of “pre-votes” and “pre-commits” before the final “commit” to a particular result; in a consensus-by-bet model, one can make each stage be a bet, so that participants in the later stages will have greater assurance that participants in the earlier stages “really mean it”.

投注共识可以看作是包含了以特定方式看待的工作量证明的一个框架，也适合为其他多种类型的共识协议提供能促进收敛的经济博弈。例如传统的[拜占庭容错](https://en.wikipedia.org/wiki/Byzantine_fault_tolerance)共识协议中，通常在结果最后"commit"之前还有"pre-votes"和"pre-commits"的概念；在投注共识的模型下，我们可以把每一阶段都变成投注，这样后面阶段的参与者就有更大把握相信前面阶段的参与者“真的是这个意思”。

It can also be used to incentivize correct behavior in out-of-band human consensus, if that is needed to overcome extreme circumstances such as a 51% attack. If someone buys up half the coins on a proof-of-stake chains, and attacks it, then the community simply needs to coordinate on a patch where clients ignore the attacker’s fork, and the attacker and anyone who plays along with the attacker automatically loses all of their coins. A very ambitious goal would be to generate these forking decisions automatically by online nodes – if done successfully, this would also subsume into the consensus-by-bet framework the underappreciated but important result from traditional fault tolerance research that, under strong synchrony assumptions, even if almost all nodes are trying to attack the system the remaining nodes can still come to consensus.

投注共识还可以用于激励链外人类共识(out-of-band human consensus)的正确行为，如果为了克服类似51%攻击的极端情况有需要的话。假设有人购买了一条PoS链上超过一半的币，并进行攻击，那么社区只需要协商出一个让客户端忽略攻击者分叉的补丁，就能自动让攻击者和他的小伙伴们失去他们所有的币。一个极有野心的目标是让在线节点可以自动的产生这种分叉决定 - 如果能成功实现，传统容错研究中的一个被低估但却重要的结论 - 在强同步假设下，即使几乎所有节点都在尝试攻击系统，剩下的节点[依然可以达成共识](http://research.microsoft.com/en-us/um/people/lamport/pubs/byz.pdf) - 也可以被纳入投注共识的框架中。

In the context of consensus-by-bet, different consensus protocols differ in only one way: who is allowed to bet, at what odds and how much? In proof of work, there is only one kind of bet offered: the ability to bet on the chain containing one’s own block at odds `E:p*R-E`. In generalized consensus-by-bet, we can use a mechanism known as a scoring rule to essentially offer an infinite number of betting opportunities: one infinitesimally small bet at 1:1, one infinitesimally small bet at 1.000001:1, one infinitesimally small bet at 1.000002:1, and so forth.

在投注共识的情境中，不同的共识协议只在一件事情上有区别：谁，可以以什么赔率，投多少注？工作量证明只提供了一种盘口：投注胜出链有`E:p*R-E`的相对概率包含你自己出的块。在广义的投注共识中，根据一种被称为[评分规则(scoring rule)](https://en.wikipedia.org/wiki/Scoring_rule)的机制我们本质上可以提供无限多的盘口：在1:1上压极小的一注，在1.000001:1上也压极小注，在1.000002:1上也压极小注，如此继续。

![scoring rule](understanding-serenity-part-ii-casper/scoring-rule.png)

One can still decide exactly how large these infinitesimal marginal bets are at each probability level, but in general this technique allows us to elicit a very precise reading of the probability with which some validator thinks some block is likely to be confirmed; if a validator thinks that a block will be confirmed with probability 90%, then they will accept all of the bets below 9:1 odds and none of the bets above 9:1 odds, and seeing this the protocol will be able to infer this “opinion” that the chance the block will be confirmed is 90% with exactness. In fact, the revelation principle tells us that we may as well ask the validators to supply a signed message containing their “opinion” on the probability that the block will be confirmed directly, and let the protocol calculate the bets on the validator’s behalf.

参与者可以自己决定在每一个概率等级上的这些极小边际投注到底是多大，但大体上这个技术让我们能打探出验证人认为某个块会被确认的相当精确的概率读数。如果验证人认为一个块有90%的概率会被确认，那么他们就会接受所有相对概率低于9:1的投注，拒绝相对概率高于9:1的投注，而协议就能基于这一点准确推断出这个块会被确认的概率是90%。事实上，[显示原理(revelation principle)](https://en.wikipedia.org/wiki/Revelation_principle)告诉我们可以直接要求验证人提供包含他们对某个块被确认概率的“意见”的签名消息，让协议代表验证人计算投注。

![scoring rule 2](understanding-serenity-part-ii-casper/scoringrule.png)

Thanks to the wonders of calculus, we can actually come up with fairly simple functions to compute a total reward and penalty at each probability level that are mathematically equivalent to summing an infinite set of bets at all probability levels below the validator’s stated confidence. A fairly simple example is s(p) = p/(1-p) and f(p) = (p/(1-p))^2/2 where s computes your reward if the event you are betting on takes place and f computes your penalty if it does not.

<small>多亏了神奇的微积分，我们可以得到在每一个概率等级上计算总奖励和总惩罚的简单函数，计算结果在数学上等价于根据验证人表达的信心在所有概率等级上形成的投注的无限集合的总和。一个简单的例子是`s(p) = p(1-p)`和`f(p) = (p/(1-p))^2/2`，这里`s`计算如果你投注的事件发生能获得的奖励，`f`计算如果没有发生你受到的惩罚。<small>
