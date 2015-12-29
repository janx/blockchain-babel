# Understanding Serenity, Part 2: Casper
# 理解Serenity 第二部分: Casper

[Original post](https://blog.ethereum.org/2015/12/28/understanding-serenity-part-2-casper/) by Vitalik Buterin, on December 28th, 2015

**Special thanks to Vlad Zamfir for introducing the idea of by-block consensus and convincing me of its merits, alongside many of the other core ideas of Casper, and to Vlad Zamfir and Greg Meredith for their continued work on the protocol.**

**特别感谢Vlad Zamfir，他提出了by-block???共识这个想法，并且说服我认同它的价值和Casper的许多核心想法；以及Vlad Zamfir和Greg Meredith，为他们在这个协议上的持续努力。**

In the last post in this series, we discussed one of the two flagship feature sets of Serenity: a heightened degree of abstraction that greatly increases the flexibility of the platform and takes a large step in moving Ethereum from “Bitcoin plus Turing-complete” to “general-purpose decentralized computation”. Now, let us turn our attention to the other flagship feature, and the one for which the Serenity milestone was originally created: the Casper proof of stake algorithm.

在这个系列的上一篇中，我们讨论了Serenity的两大旗舰特性之一：深度抽象，它将给平台带来极大的灵活性，使以太坊从“比特币加图灵完备”向“通用去中心化的计算”迈进一大步。现在，让我们把注意力转向另一个主要特性，也是当初设立Serenity里程碑的原因：Casper权益证明算法。

## Consensus By Bet
## 由投注达成共识

The keystone mechanism of Casper is the introduction of a fundamentally new philosophy in the field of public economic consensus: the concept of consensus-by-bet. The core idea of consensus-by-bet is simple: the protocol offers opportunities for validators to bet against the protocol on which blocks are going to be finalized. A bet on some block X in this context is a transaction which, by protocol rules, gives the validator a reward of Y coins (which are simply printed to give to the validator out of thin air, hence “against the protocol”) in all universes in which block X was processed but which gives the validator a penalty of Z coins (which are destroyed) in all universes in which block X was not processed.

Casper向公开经济学共识这个领域引入了一个根本上全新的理念作为自己的基础：通过投注产生共识。投注共识的核心思想很简单：为验证人(validator)提供**与协议**对赌哪个块会被最终确定(finalized)的机会。在这里对某个区块X的投注就是一笔交易，在**所有区块X被处理了的世界中**都会带给验证人Y个币的奖励（奖励是凭空“印”出来的，因而是“与协议”对赌），而在所有区块X没有被处理的世界中会对验证人收取Z个币的罚款（罚金被销毁）。

(译注：一个世界(world)在这里应理解为区块链未来的一种可能状态。)

The validator will wish to make such a bet only if they believe block X is likely enough to be processed in the universe that people care about that the tradeoff is worth it. And then, here’s the economically recursive fun part: the universe that people care about, ie. the state that users’ clients show when users want to know their account balance, the status of their contracts, etc, is itself derived by looking at which blocks people bet on the most. Hence, each validator’s incentive is to bet in the way that they expect others to bet in the future, driving the process toward convergence.

只有在相信区块X在人们在意的那个世界中被处理的可能性足够大时，这才是值得去做的交易，验证人才会愿意投注。然而，接下来就是经济上递归的有趣部分：人们在意的那个世界，也就是用户的客户端在用户想要知道他们的账户余额或是合约的状态时所展现的那个状态，**本身就是根据人们对哪个区块投注最多推导出来的**。因此，每一个验证人都具有根据他们所预期的其他人的投注情况进行投注的动机，驱使这个过程走向收敛。
