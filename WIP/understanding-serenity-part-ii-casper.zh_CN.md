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

Casper向公开经济学共识这个领域引入了一个根本上全新的理念作为自己的基础：投注共识。投注共识的核心思想很简单：为验证人(validator)提供**与协议**对赌哪个块会被finalized的机会。在这里对某个区块X的投注就是一笔交易，在**所有区块X被处理了的世界中**都会带给验证人Y个币的奖励（奖励是凭空“印”出来的，因而是“与协议”对赌），而在所有区块X没有被处理的世界中会对验证人收取Z个币的罚款（罚金被销毁）。

(译注：一个世界(world)在这里应理解为区块链未来的一种可能状态。finalize/finality字典解释为最终确定，定案/终局性等等，在区块链中可理解为这个块所在的链被永久承认为正确的链，这个区块不可能再被revert。这里保持原文不做翻译。)

The validator will wish to make such a bet only if they believe block X is likely enough to be processed in the universe that people care about that the tradeoff is worth it. And then, here’s the economically recursive fun part: the universe that people care about, ie. the state that users’ clients show when users want to know their account balance, the status of their contracts, etc, is itself derived by looking at which blocks people bet on the most. Hence, each validator’s incentive is to bet in the way that they expect others to bet in the future, driving the process toward convergence.

只有在相信区块X在人们关心的那个世界中被处理的可能性足够大时，这才是值得去做的交易，验证人才会愿意投注。然而，接下来就是经济上递归的有趣部分：人们关心的那个世界，也就是用户的客户端在用户想要知道他们的账户余额或是合约的状态时所展现的那个状态，**本身就是根据人们对哪个区块投注最多推导出来的**。因此，每一个验证人都具有根据他们所预期的其他人的投注情况进行投注的动机，驱使这个过程走向收敛。

A helpful analogy here is to look at proof of work consensus – a protocol which seems highly unique when viewed by itself, but which can in fact be perfectly modeled as a very specific subset of consensus-by-bet. The argument is as follows. When you are mining on top of a block, you are expending electricity costs E per second in exchange for receiving a chance p per second of generating a block and receiving R coins in all forks containing your block, and zero rewards in all other chains:

一个有帮助的类比是工作量证明共识 - 它本身看起来非常独特，实际上可以成为投注共识的一个特别子模型。理由如下：当你基于一个块挖矿时，你是在花费每秒`E`的电力成本换取每秒`p`的出块概率，并且在**所有包含你的出块的分叉中**获得`R`个币，在其它分叉中分文不得。

![PoW as submodel](understanding-serenity-part-ii-casper/pow-as-submodel.png)

Hence, every second, you receive an expected gain of `p*R-E` on the chain you are mining on, and take a loss of E on all other chains; this can be interpreted as taking a bet at `E:p*R-E` odds that the chain you are mining on will “win”; for example, if p is 1 in 1 million, R is 25 BTC ~= $10000 USD and E is $0.007, then your gains per second on the winning chain are 0.000001 * 10000 - 0.007 = 0.003, your losses on the losing chain are the electricity cost of 0.007, and so you are betting at 7:3 odds (or 70% probability) that the chain you are mining on will win. Note that proof of work satisfies the requirement of being economically “recursive” in the way described above: users’ clients will calculate their balances by processing the chain that has the most proof of work (ie. bets) behind it.

因此，每一秒钟，在你挖矿的链上你可以获得`p*R-E`的期望收益，在其它链上遭受`E`的损失；因此你的挖矿选择可以理解为下注赌你所在的链有`E:p*R-E`的相对概率(odds)胜出。比如，假设p等于百万分之一，R是25个币约等于10000美元，而E是0.007美元，则你在胜出链上每秒钟的期望收益是`0.000001 * 10000 - 0.007 = 0.003`，你在失败链上的损失是`0.007`的电力成本，因此你是在赌自己挖矿的链有7:3的相对概率（或者说70%的概率）胜出。注意工作量证明满足上面所说的经济上递归的要求：用户的客户端通过处理拥有最大工作量的那条链来计算其账户余额。

Consensus-by-bet can be seen as a framework that encompasses this way of looking at proof of work, and yet also can be adapted to provide an economic game to incentivize convergence for many other classes of consensus protocols. Traditional Byzantine-fault-tolerant consensus protocols, for example, tend to have a concept of “pre-votes” and “pre-commits” before the final “commit” to a particular result; in a consensus-by-bet model, one can make each stage be a bet, so that participants in the later stages will have greater assurance that participants in the earlier stages “really mean it”.

投注共识可以看作是包含了以特定方式看待的工作量证明的一个框架，也适合为其他多种类型的共识协议提供能促进收敛的经济博弈。例如传统的[拜占庭容错](https://en.wikipedia.org/wiki/Byzantine_fault_tolerance)共识协议中，通常在对某个结果进行最后"commit"之前还有"pre-votes"和"pre-commits"的概念；在投注共识的模型下，我们可以把每一阶段都变成投注，这样后面阶段的参与者就有更大把握相信前面阶段的参与者“真的是这个意思”。

It can also be used to incentivize correct behavior in out-of-band human consensus, if that is needed to overcome extreme circumstances such as a 51% attack. If someone buys up half the coins on a proof-of-stake chains, and attacks it, then the community simply needs to coordinate on a patch where clients ignore the attacker’s fork, and the attacker and anyone who plays along with the attacker automatically loses all of their coins. A very ambitious goal would be to generate these forking decisions automatically by online nodes – if done successfully, this would also subsume into the consensus-by-bet framework the underappreciated but important result from traditional fault tolerance research that, under strong synchrony assumptions, even if almost all nodes are trying to attack the system the remaining nodes can still come to consensus.

投注共识还可以用于激励链外人类共识(out-of-band human consensus)的正确行为，如果为了克服类似51%攻击的极端情况有需要的话。假设有人购买了一条PoS链上超过一半的币，并进行攻击，那么社区只需要协商出一个让客户端忽略攻击者分叉的补丁，就能自动让攻击者和他的小伙伴们失去他们所有的币。一个极有野心的目标是让在线节点可以自动的产生这种分叉决定 - 如果能成功实现，传统容错研究中的一个被低估但却重要的结论 - 在强同步假设下，即使几乎所有节点都在尝试攻击系统，剩下的节点[依然可以达成共识](http://research.microsoft.com/en-us/um/people/lamport/pubs/byz.pdf) - 也可以被纳入投注共识的框架中。

In the context of consensus-by-bet, different consensus protocols differ in only one way: who is allowed to bet, at what odds and how much? In proof of work, there is only one kind of bet offered: the ability to bet on the chain containing one’s own block at odds `E:p*R-E`. In generalized consensus-by-bet, we can use a mechanism known as a scoring rule to essentially offer an infinite number of betting opportunities: one infinitesimally small bet at 1:1, one infinitesimally small bet at 1.000001:1, one infinitesimally small bet at 1.000002:1, and so forth.

在投注共识的情境中，不同的共识协议只在一件事情上有区别：谁，可以以什么赔率，投多少注？工作量证明只提供了一种赌局：投注胜出链有`E:p*R-E`的相对概率包含你自己出的块。在广义的投注共识中，依据一种被称为[评分规则(scoring rule)](https://en.wikipedia.org/wiki/Scoring_rule)的机制我们本质上可以提供无限多种赌局：在1:1上压极小的一注，在1.000001:1上也压极小注，在1.000002:1上也压极小注，如此继续。

![scoring rule](understanding-serenity-part-ii-casper/scoring-rule.png)

One can still decide exactly how large these infinitesimal marginal bets are at each probability level, but in general this technique allows us to elicit a very precise reading of the probability with which some validator thinks some block is likely to be confirmed; if a validator thinks that a block will be confirmed with probability 90%, then they will accept all of the bets below 9:1 odds and none of the bets above 9:1 odds, and seeing this the protocol will be able to infer this “opinion” that the chance the block will be confirmed is 90% with exactness. In fact, the revelation principle tells us that we may as well ask the validators to supply a signed message containing their “opinion” on the probability that the block will be confirmed directly, and let the protocol calculate the bets on the validator’s behalf.

参与者依然可以选择在每一个概率等级上的这些极小边际投注的确切大小，但大体上这个技术让我们能打探出验证人认为某个块会被确认的相当精确的概率读数。如果验证人认为一个块有90%的概率会被确认，那么他们就会接受所有相对概率低于9:1的赌局，拒绝相对概率高于9:1的赌局，而协议就能基于这一点准确得出这个块会被确认的概率是90%的“看法”。事实上，[显示原理(revelation principle)](https://en.wikipedia.org/wiki/Revelation_principle)告诉我们可以要求验证人直接给出他们对某个块被确认概率的“看法”的签名消息，让协议代表验证人计算投注。

![scoring rule 2](understanding-serenity-part-ii-casper/scoringrule.png)

Thanks to the wonders of calculus, we can actually come up with fairly simple functions to compute a total reward and penalty at each probability level that are mathematically equivalent to summing an infinite set of bets at all probability levels below the validator’s stated confidence. A fairly simple example is s(p) = p/(1-p) and f(p) = (p/(1-p))^2/2 where s computes your reward if the event you are betting on takes place and f computes your penalty if it does not.

<small>多亏了神奇的微积分，我们可以得到在每一个概率等级上计算总奖励和总惩罚的简单函数，计算结果在数学上等价于根据验证人信心在所有概率等级上形成的投注的无限集合的总和。一个简单的例子是`s(p) = p(1-p)`和`f(p) = (p/(1-p))^2/2`，这里`s`计算如果你投注的事件发生能获得的奖励，`f`计算如果没有发生你受到的惩罚。<small>

A key advantage of the generalized approach to consensus-by-bet is this. In proof of work, the amount of “economic weight” behind a given block increases only linearly with time: if a block has six confirmations, then reverting it only costs miners (in equilibrium) roughly six times the block reward, and if a block has six hundred confirmations then reverting it costs six hundred times the block reward. In generalized consensus-by-bet, the amount of economic weight that validators throw behind a block could increase exponentially: if most of the other validators are willing to bet at 10:1, you might be comfortable sticking your neck out at 20:1, and once almost everyone bets 20:1 you might go for 40:1 or even higher. Hence, a block may well reach a level of “de-facto complete finality”, where validators’ entire deposits are at stake backing that block, in as little as a few minutes, depending on how brave the validators are (and how much the protocol incentivizes them to be).

投注共识的广义形式有一个重要优点。在工作量证明中，给定区块背后的“经济权重”仅仅随着时间线性增加：如果一个块有6个确认，那么要撤销它只需要花费矿工大约6倍于出块奖励（在均衡态下）的成本，而如果一个块有600个确认，那么撤销它的成本就是出块奖励的600倍。在广义的投注共识中，验证人在一个块上投入的经济权重可以指数级增加：如果其他大多数验证人愿意以10:1下注，你可能会想冒险以20:1下注；而一旦几乎所有人都增加到20:1，你可能会跳到40:1或者更高。因此，一个块很可能在几分钟之内，取决于验证人有多少勇气（以及协议提供的激励大小），就达到一种“准完全finality”的状态，这种状态下验证人的所有保证金都成为了支持这个块的投注。

> 50000-foot view summary: the blockchain is a prediction market on itself.
> **在50000英尺的高度看：区块链是一个关于自身的预测市场。**


## Blocks, Chains and Consensus as Tug of War
## 区块，链，共识与拔河

Another unique component of the way that Casper does things is that rather than consensus being by-chain as is the case with current proof of work protocols, consensus is by-block: the consensus process comes to a decision on the status of the block at each height independently of every other height. This mechanism does introduce some inefficiencies – particularly, a bet must register the validator’s opinion on the block at every height rather than just the head of the chain – but it proves to be much simpler to implement strategies for consensus-by-bet in this model, and it also has the advantage that it is much more friendly to high blockchain speed: theoretically, one can even have a block time that is faster than network propagation with this model, as blocks can be produced independently of each other, though with the obvious proviso that block finalization will still take a while longer.

Casper的另一个独特之处在于它的共识是**按块达成的(by-block)**而不是像工作量证明那样**按链达成的(by-chain)**：共识过程在某个高度上对区块状态的决策是独立于其它所有高度的。这个机制确实会导致一定程度的低效 - 特别是，一次投注必须表达验证人对于每一个高度上区块的意见，而不能仅是链的头部区块 - 但是在这个模型下为投注共识实现投注策略会十分简单，而且它还有个优点是对高速区块链友好：理论上，这个模型中的出块时间甚至可以比网络传播时间还要块，因为区块可以相互独立的被制造出来，虽然有个明显的附带条件，即区块的**最终确认**依然要一段时间。

In by-chain consensus, one can view the consensus process as being a kind of tug-of-war between negative infinity and positive infinity at each fork, where the “status” at the fork represents the number of blocks in the longest chain on the right side minus the number of blocks on the left side:

在按链达成的共识中，我们可以把共识过程看作是正无穷和负无穷在每个分叉点进行的拔河游戏，每次分叉的“状态值”的含义是了右手边最长链的区块数减去左边最长链的区块数：

![tug of war](understanding-serenity-part-ii-casper/bchain_tugofwar.png)

Clients trying to determine the “correct chain” simply move forward starting from the genesis block, and at each fork go left if the status is negative and right if the status is positive. The economic incentives here are also clear: once the status goes positive, there is a strong economic pressure for it to converge to positive infinity, albeit very slowly. If the status goes negative, there is a strong economic pressure for it to converge to negative infinity.

为了找出“正确的链”我们只需从起源块（genesis block）开始前进，在每一个分叉处，如果状态值是负数则往左边移动，反之向右边移动。这里的经济激励很明显：一旦状态值变正，则说明有很强的经济压力迫使它走向正无穷，尽管过程很缓慢。如果状态值变负，则有很强的经济压力迫使它走向负无穷。

Incidentally, note that under this framework the core idea behind the GHOST scoring rule becomes a natural generalization – instead of just counting the length of the longest chain toward the status, count every block on each side of the fork:

顺便说一句，在这个框架下，[GHOST评分规则](https://eprint.iacr.org/2013/881.pdf)的核心思想变成了一种自然的范化：与其只通过最长链的长度来计算状态值，不如计算分叉单边所有块的数量：

![tug of war 2](understanding-serenity-part-ii-casper/bchain_tugofwar2.png)

In by-block consensus, there is once again the tug of war, though this time the “status” is simply an arbitrary number that can be increased or decreased by certain actions connected to the protocol; at every block height, clients process the block if the status is positive and do not process the block if the status is negative. Note that even though proof of work is currently by-chain, it doesn’t have to be: one can easily imagine a protocol where instead of providing a parent block, a block with a valid proof of work solution must provide a +1 or -1 vote on every block height in its history; +1 votes would be rewarded only if the block that was voted on does get processed, and -1 votes would be rewarded only if the block that was voted on does not get processed:

在按块达成的共识中，同样有这个拔河游戏，不过此时“状态值”仅仅是通过协议的某些操作可以任意增加或者减少的数字。在每一个高度上，如果状态值为正，客户端就处理这个块，如果为负则不处理。注意虽然工作量证明当前是按链达成共识的，但并非必须如此：我们很容易可以想象一个协议，其中一个包含有效工作量证明的区块不引用父区块，而是给它自身历史上的所有区块投一张+1或者-1票。+1票只有在所投的区块被处理时才获得奖励，而-1票只在所投的区块没有被处理时才得到奖励：

![pow score](understanding-serenity-part-ii-casper/pow-score.png)

Of course, in proof of work such a design would not work well for one simple reason: if you have to vote on absolutely every previous height, then the amount of voting that needs to be done will increase quadratically with time and fairly quickly grind the system to a halt. With consensus-by-bet, however, because the tug of war can converge to complete finality exponentially, the voting overhead is much more tolerable.

不过，在工作量证明中这样的设计无法很好的工作，原因很简单：如果你需要对**每一个**历史高度进行投票，那么需要进行的投票数量会随着时间的平方增长，很快就能让系统宕机。而在投注共识中，由于拔河游戏可以以指数级速度收敛到完全finality，投票的开销要容易承受的多。

One counterintuitive consequence of this mechanism is the fact that a block can remain unconfirmed even when blocks after that block are completely finalized. This may seem like a large hit in efficiency, as if there is one block whose status is flip-flopping with ten blocks on top of it then each flip would entail recalculating state transitions for an entire ten blocks, but note that in a by-chain model the exact same thing can happen between chains as well, and the by-block version actually provides users with more information: if their transaction was confirmed and finalized in block 20101, and they know that regardless of the contents of block 20100 that transaction will have a certain result, then the result that they care about is finalized even though parts of the history before the result are not. By-chain consensus algorithms can never provide this property.

这个机制的一个反直觉结果是，一个块可以在其后的块都完全finalize之后依然处于未确认的状态。这看起来像是一个很大的效率问题，因为如果其上有10个块的一个区块状态一直反复变化的话，每一次变动都会导致重新计算这10个块的状态转移，但其实在按链达成的共识中在链与链之间会发生完全相同的事情，而按块的共识实际上可以提供更多的信息给用户：如果他们的交易在第20101个块被确认和finalize，而且他们知道不管第20100个块的内容是什么，这笔交易都有一个确定的结果，那么他们关心的这个结果就是finalized，即使结果前的部分历史还不是。按链达成的共识算法永远也不会有这个性质。


## So how does Casper work anyway?
## 那么Casper到底是如何工作的？



