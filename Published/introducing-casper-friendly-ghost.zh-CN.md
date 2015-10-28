# Introducing Casper “the Friendly Ghost”

# 友善的小精灵Casper - 以太坊未来的POS协议

[Origin Post](https://blog.ethereum.org/2015/08/01/introducing-casper-friendly-ghost/)

Posted by Vlad Zamfir on August 1st, 2015.

**Hi everyone – Vlad here. I’ve been working on the analysis and specification of  “proof-of-stake” blockchain architecture since September 2014. While Vitalik and I haven’t agreed on all of the details of the spec, we do have consensus on many properties of the proof-of-stake protocol that will likely be implemented for the Serenity release! It is called Casper “the friendly ghost” because it is an adaptation of some of the principles of the GHOST (Greedy Heaviest-Observed Sub-Tree) protocol for proof-of-work consensus to proof-of-stake. This blog post (my first one!) shares properties that are likely to be true of Casper’s implementation in the Serenity release. Formal verification and simulation of Casper’s properties is under way, and will be published eventually – in the meantime, please enjoy this high-level, informal discussion!  : )**

**大家好，我是Vlad. 2014年9月份我开始了研究和设计以太坊POS(proof-of-stake, 权益证明)架构的工作。目前Vitalik和我对于Serenity阶段的POS协议应该长什么样已经有了许多共识，只剩一些细节方面的分歧。我们称它为友善的幽灵Casper(Casper the friendly ghost)，因为它实际上是GHOST(Greedy Heaviest-Observed Sub-Tree), 一种POW协议的POS变种。接下来的文章介绍的一些特性非常可能在Serenity阶段成为现实。Casper的形式化验证和模拟正在进行中，结果会在将来发布。接下来先开始我们的非正式介绍和讨论吧！: )**

## Security-deposit based security and authentication

## 基于保证金的安全和共识鉴别 (Security-deposit based security and authentication)

Casper is a security-deposit based economic consensus protocol. This means that nodes, so called “bonded validators”, have to place a security deposit (an action we call “bonding”) in order to serve the consensus by producing blocks. The protocol’s direct control of these security deposits is the primary way in which Casper affects the incentives of validators. Specifically, if a validator produces anything that Casper considers “invalid”, their deposit are forfeited along with the privilege of participating in the consensus process. The use of security deposits addresses the “nothing at stake” problem; that behaving badly is not expensive. There is something at stake, and bonded validators who misbehave in an objectively verifiable manner will lose it.

Casper是一种基于保证金的经济激励共识协议(security-deposit based economic consensus protocol)。协议中的节点，作为“锁定保证金的验证人(bonded validators)”，必须先缴纳保证金(这一步叫做锁定保证金，"bonding")才可以参与出块和共识形成。Casper共识协议通过对这些保证金的直接控制来约束验证人的行为。具体来说就是，如果一个验证人作出了任何Casper认为“无效”的事情，他的保证金将被罚没，出块和参与共识的权利也会被取消。保证金的引入解决了"nothing at stake"，也就是经典POS协议中做坏事的代价很低的问题。现在有了代价，而且被客观证明做错事的验证人将会付出这个代价。

Very notably, a validator’s signature is only economically meaningful so long as that validator currently has a deposit. This means that clients can only rely on signatures from validators that they know are currently bonded. Therefore, when clients receive and authenticate the state of the consensus, their authentication chain ends in the list of currently-bonded validators. In proof-of-work consensus, on the other hand, the authentication chain ends in the genesis block – as long as you know the genesis block you can authenticate the consensus. Here, as long as you know the set of currently-bonded validators, you can authenticate the consensus. A client who does not know the list of currently bonded validators must authenticate this list out-of-band. This restriction on the way in which the consensus is authenticated solves the “long range attack” problem by requiring that everyone authenticate the consensus against current information.

我们容易发现，只有在验证人当前已缴纳保证金的情况下他的签名才有意义(economically meaningful)。这代表客户端只能依赖他们知道的锁定保证金的验证人的签名。因此当客户端接收和鉴別共识数据时，*共识认可的链必须起源于出自当前锁定保证金的验证人的块*。在POW协议中共识认可的链则是起源于创世块 - 只要你知道创世块的数据你就可以鉴别出共识认可的链。这里，只要你知道当前锁定保证金的验证人，你就可以鉴别出共识认可的链。不知道当前锁定保证金的验证人列表的客户端必须先通过另外的信道获取这个列表。这个限制通过要求所有人用当前信息鉴别共识解决了“远程攻击(long range attack)”问题。

The validator list changes over time as validators place deposits, lose their deposits, unbond, and get unbonded. Therefore, if clients are offline for too long, their validator list will no longer be current enough to authenticate the consensus. In the case that they are online sufficiently often to observe the validator set rotating, however, clients are able to securely update their validator list. Even in this case, clients must begin with an up-to-date list of currently-bonded validators, and therefore they must authenticate this list out-of-band at least once.

验证人列表随着验证人保证金不断的锁定，罚没，解锁而变动。如果客户端离线过长时间，它的验证人列表就会由于过时而不能用来鉴别共识。如果客户端经常在线，则能够与最新的验证人列表保持同步，但问题是在第一次同步之前，客户端还是需要从其他信道获取最新锁定保证金的验证人列表。

This “out-of-band authentication only necessarily once” property is what Vitalik calls weak subjectivity. In this context information is said to be “objective” if it can be verified in a protocol-defined manner, while it is “subjective” if it must be authenticated via extra-protocol means. In weakly subjective consensus protocols, the fork-choice rule is stateful, and clients must initialize (and possibly sometimes renew) the information that their fork-choice rule uses to authenticate the consensus. In our case, this entails identifying the currently bonded validators (or, more probably a cryptographic hash of the validator list).

这个“需要从其他信道鉴别共识至少一次”的性质，正是[Vitalik所说的“弱主观性(weak subjectivity)”](https://blog.ethereum.org/2014/11/25/proof-stake-learned-love-weak-subjectivity/)。在我们的上下文中，如果信息可以在协议之内被验证，则可称之为“客观的”；如果信息必须依赖协议外的手段才可验证，则称为“主观的”。在弱主观性共识协议中，*分叉选择规则*是有状态的，因此客户端必须初始化（有些时候是更新）这个状态才能鉴别共识。在这里，这个状态被用来辨认当前锁定保证金的验证人（更精确的说法可能是当前验证人列表的密码学哈希）。

## Gambling on Consensus

## 下注共识 (Gambling on Consensus)

Casper makes validators bet a large part of their security deposits on how the consensus process will turn out. Moreover, the consensus process “turns out” in the manner in which they bet: validators are made to bet their deposits on how they expect everyone else to be betting their deposits. If they bet correctly, they earn their deposit back with transaction fees and possibly token issuance upon it – if on the other hand they do not quickly agree, they re-earn less of their deposit. Therefore through iterated rounds of betting validator bets converge.

Casper要求验证人将保证金中的大部分对共识结果进行下注。而共识结果又通过验证人的下注情况形成：验证人必须猜测其他人会赌哪个块胜出，同时也下注这个块。如果赌对了，他们就可以拿回保证金外加交易费用，也许还会有一些新发的货币；如果下注没有迅速达成一致，他们只能拿回部分保证金。因此数个回合之后验证人的下注分布就会收敛。

Moreover, if validators change their bets too dramatically, for example by voting with a high probability on one block after voting with a very high probability on another, then they are severely punished. This guarantees that validators bet with very high probabilities only when they are confident that the other validators will also produce high probability bets. Through this mechanism we guarantee that their bets never converge to a second value after converging upon a first, as long as there there is sufficient validator participation.

此外如果验证人过于显著的改变下注，例如先是赌某个块有很高概率胜出，然后又改赌另外一个块有高概率胜出，他将被严惩。这条规则确保了验证人只有在非常确信其他人也认为某个块有高概率胜出时才以高概率下注。我们通过这个机制来确保不会出现下注先收敛于一个结果然后又收敛到另外一个结果的情况，只要验证人足够多。

Proof-of-work consensus is also a betting scheme: miners bet that their block will be part of the heaviest chain; if they eventually prove to be correct, they receive tokens – whereas if they prove to be incorrect, they incur electricity costs without compensation. Consensus is secured as long as all miners are betting their hashing power on the same chain, making it the blockchain with the most work (as a direct result of and as preempted by their coordinated betting). The economic cost of these proof-of-work bets add up linearly in the number of confirmations (generations of descendant blocks), while, in Casper, validators can coordinate placing exponentially growing portions of their security deposits against blocks, thereby achieving maximum security very quickly.

POW共识同样可以理解为是一个下注机制：矿工选择一个块基于它进行挖矿，也就是赌这个块会成为主链的一部分；如果赌对了，他可以收到奖励，而如果赌错了，他会损失电费。只要所有的矿工都将他们的算力下注到同一条链上，使这条链拥有最多的工作量（即时算力下注的预测，也是算力下注的结果），共识就是安全的。POW中算力赌注的经济价值随着确认数的增加线性增长，而在Casper中验证人可以通过协调使下注比例指数增长，从而使共识快速达到最大安全。

## By-height Consensus

## 高度级共识

Validators bet independently on blocks at every height (i.e. block number) by assigning it a probability and publishing it as a bet. Through iterative betting, the validators elect exactly one block at every height, and this process determines the order in which transactions are executed. Notably, if a validator ever places bets with probabilities summing to more than 100% at a time for a given height, or if any are less than 0%, or if they bet with more than 0% on an invalid block, then Casper forfeits their security deposit.

验证人对每一个高度上的每一个候选块独立下注，给每个块指定一个胜出概率并公布。通过反复下注，对于每个高度验证人会选出唯一一个胜出块，这个过程也决定了交易(transaction)执行的顺序。如果一个验证人在某个高度公布的概率分布总和大于100%，或者公布了小于0%的概率，或者对一个无效块指定了大于0%的概率，Casper将罚没他的保证金。

## Transaction Finality

## 交易最终确认(Transaction Finality)

When every member of a supermajority of bonded validators (a set of validators who meet a protocol-defined threshold somewhere between 67% and 90% of bonds) bets on a block with a very high (say, > 99.9%) probability, the fork-choice rule never accepts a fork where this block does not win, and we say that the block is final. Additionally, when a client sees that every block lower than some height H is final, then the client will never choose a fork that has a different application state at height H – 1 than the one that results from the execution of transactions in these finalized blocks. In this eventuality, we say that this state is finalized.

当锁定保证金的验证人中的绝大多数（满足协议定义阈值的一群验证人：保证金比例达到67%到90%之间某个百分比）以非常高的概率（例如，> 99.9%）下注某个块时，任何不包含这个块的分叉都不可能胜出，此时我们说这个块已最终确认(final)。此外，如果客户端发现所有小于高度H的块都已最终确认，那么此客户端永远不能接受一个在高度H - 1的状态和顺序执行这些完全块得到的状态不一样的分叉。这种情况下我们说这个状态(H - 1高度的状态)已最终确认。

There are therefore two relevant kinds of transaction finality: the finality of the fact that the transaction will be executed at a particular height (which is from finality of its block, and therefore priority over all future blocks at that height), and the finality of the consensus state after that transaction’s execution (which requires finality of its block and of unique blocks at all lower heights).

因此这里有相关的两种交易的最终确认：交易在特定高度被执行的最终确认（也就是对应块的最终确认，早于所有此高度之后的交易执行），以及交易执行后状态的最终确认（需要对应块和所有低于此高度的块被最终确认）。

## Censorship Resistance

## 防审查（Censorship Resistance）

One of the largest risks to consensus protocols is the formation of coalitions that aim to maximize the profits of their members at the expense of non-members. If Casper’s validators’ revenues are to be made up primarily of transaction fees, for example, a majority coalition could censor the remaining nodes in order to earn an increased share of transaction fees. Additionally, an attacker could bribe nodes to exclude transactions affecting particular addresses – and so long as a majority of nodes are rational, they can censor the blocks created by nodes who include these transactions.

共识协议最大的威胁之一是矿工形成以损害非成员利益为代价最大化成员获利的联盟。如果Casper中验证人的收入主要由手续费构成，一个多数联盟就能够通过过滤其它节点的出块来获取更大利益。不仅如此，攻击者还可以贿赂节点来剔除特定地址发出的交易，只要多数节点是理性的，他们就能够联合起来过滤掉没有剔除指定交易的块。

To resist attacks conducted by majority coalitions, Casper regards the consensus process as a cooperative game and ensures that each node is most profitable if they are in a coalition made up of 100% of the consensus nodes (at least as long as they are incentivized primarily by in-protocol rewards). If p% of the validators are participating in the consensus game, then they earn f(p) ≤ p% of the revenues they would earn if 100% of the validators were participating, for some increasing function f.

为了抵御多数派联盟攻击，Casper将共识过程看作一个[合作博弈](https://en.wikipedia.org/wiki/Cooperative_game)，确保每一个节点只有在由所有节点组成的联盟中才能获得最大利益（至少在当利益主要由协议内奖励构成的情况下如此）。如果**p%**的验证人参与了共识博弈，那么他们将得到**f(p) ≤ p%**的收益；如果有100%的验证人参与则能获得更多回报。

More specifically, Casper punishes validators for not creating blocks in a protocol-prescribed order. The protocol is aware of deviations from this order, and withholds transaction fees and deposits from validators accordingly. Additionally, the revenue made from betting correctly on blocks is linear (or superlinear) in the number of validators who are participating in at that height of the consensus game.

更具体的说，Casper会惩罚那些不按照协议规定顺序出块的验证人。协议会注意到出块顺序的偏离，并且扣下对应验证人的保证金和手续费。此外，通过下注赢得的收益与参与共识博弈的验证人数量成线性（或者超线性）关系。

## Will there be more transactions per second?

## 吞吐量（每秒可处理的交易数）会增加吗？

Most probably, yes, although this is due to the economics of Casper rather than due to its blockchain architecture. However, Casper’s blockchain does allow for faster block times than is possible with proof-of-work consensus.

答案很可能是确定的，而原因与其说是Casper的区块链架构不如说是Casper上的经济学。不过Casper的区块链设计确实支持了比POW共识更快的出块间隔。

Validators will likely be earning only transaction fees, so they have a direct incentive to increase the gas limit, if their validation server can handle the load. However, validators also have reduced returns from causing other, slower validators to fall out of sync, so they will allow the gas limit to rise only in a manner that is tolerable by the other validators. Miners investing in hardware primarily purchase more mining rigs, while validators investing in hardware primarily upgrade their servers so they can process more transactions per second. Miners also have an incentive to reinvest in more powerful transaction processing, but this incentive is much weaker than their incentive to purchase mining power.

验证人的收入很可能只有交易费用，因此他们有增加Gas上限的直接动机，只要他们的服务器负荷的了。但是如果因此造成其它处理能力比较弱的验证人跟不上节奏，他们的收入会变少。所以验证人只会接受大家都能承受的Gas上限上涨。矿工(Miners)投资硬件的方式是购买更多的矿机（算力），而验证人的投资硬件的方式则会是升级服务器以获取更高的吞吐量。矿工也有购买能获得更高吞吐量的硬件的动力，但是这比购买算力的动力弱得多。

Security-deposit-based proof-of-stake is very light-client friendly relative to proof-of-work. Specifically, light clients do not need to download block headers to have full security in authenticating the consensus, or to have full economic assurances of valid transaction execution. This means that a lot of consensus overhead affects only the validators, but not the light clients, and it allows for lower latency without causing light clients to lose the ability to authenticate the consensus.

相对于POW, 在基于保证金的POS上实现轻客户端更加方便。具体来说，轻客户端无需下载区块头来获得共识鉴别的安全性，或是交易执行有效性的经济性保证。大部分的共识开销只会影响验证人，不会影响轻客户端。轻客户端也可以在保留鉴别共识能力的前提下实现低延迟。

## Recovery from netsplits

## 网络分区的恢复

Casper is able to recover from network partitions because transactions in non-finalized blocks can be reverted. After a partition reconnects, Casper executes transactions from blocks that received bets on the partition with higher validator participation. In this manner, nodes from either side of the partition agree on the state of the consensus after a reconnection and before validators are able to replace their bets. Validator bets converge to finalize the blocks in the partition that had more validator participation, with very high probability. Casper will very likely process the losing transactions from losing blocks after the ones from winning blocks, although it is still to be decided whether validators will have to include these transactions in new blocks, or if Casper will execute them in their original order, himself.

由于未最终确认的区块中的交易可以被撤销，Casper具有在网络分区后恢复的能力。在分区重新连上后，Casper会执行具有更高验证人参与度的分区上获得下注的区块中的交易。这样在重连之后验证人重新下注前，分区双方就对共识状态达成了一致。验证人的下注会收敛并以高概率最终确认具有更高验证人参与度的分区上的某个块。Casper很可能会在产生胜出块之后再处理淘汰块中的交易，不过我们还没有决定到底是由验证人把这些交易纳入新块，还是由Casper按照原来的顺序处理。

## Recovery from mass crash-failure

## 大规模崩溃的恢复

Casper is able to recover from the crash-failure of all but one node. Bonded validators can always produce and place bets on blocks on their own, although they always make higher returns by coordinating on the production of blocks with a larger set of validators. In any case, a validator makes higher returns from producing blocks than from not producing blocks at all. Additionally, bonded validators who appear to be offline for too long will be unbonded, and new bonders subsequently will be allowed to join the validation set. Casper can thereby potentially recover precisely the security guarantees it had before the mass crash-failure.

即使在全网只剩一个节点没有崩溃的极端情况下Casper也有能力恢复。锁定保证金的验证人总是独立的对候选块下注，虽然下注和多数验证人一致的话回报会更高。在任何时候，验证人从出块获得的回报总是比不出块要高。如果锁定保证金的验证人过长时间没有上线，他的保证金会被解锁，新的验证人会接替他的位置。因此Casper的安全程度可以恢复到和大规模崩溃之前一样。

## What is Casper, in non-economic terms?

## 抛开经济学术语的解释

Casper is an eventually-consistent blockchain-based consensus protocol. It favours availability over consistency (see the CAP theorem). It is always available, and consistent whenever possible. It is robust to unpredictable message delivery times because nodes come to consensus via re-organization of transactions, after delayed messages are eventually received. It has an eventual fault tolerance of 50%, in the sense that a fork created by >50% correct nodes scores higher than any fork created by the remaining potentially-faulty validators. Notably, though, clients cannot be certain that any given fork created with 51% participation won’t be reverted because they cannot know whether some of these nodes are Byzantine. Clients therefore only consider a block as finalized if it has the participation of a supermajority of validators (or bonded stake).

Casper是一个基于区块链的最终一致性共识协议。想对于一致性(C)它更倾向于可用性(A)（[见CAP理论](https://en.wikipedia.org/wiki/CAP_theorem)）。它总是可用，并且尽可能迅速的达成一致。它健壮到足以容忍不可预计的消息传输延迟，因为节点可以在收到延迟的消息之后通过重新组织交易达成共识。由大于50%的正常节点建立的分叉总是比剩下的有潜在问题的节点建立的分叉权重更高，因此它对不超过50%的网络失效有最终容错性(eventual fault tolernace)。需要注意的是客户端无法知道一个有51%的验证人参与的分支会不会被放弃，因为此时不知道这些验证人中是否有恶意节点。一个块只有在**绝大多数(supermajority)**验证人（或者说保证金）参与共识的情形下才能被认为是最终确认(final)的。

## What is it like to be a bonded validator?

## 验证人有什么责任？

As a bonded validator, you will need to securely sign blocks and place bets on the consensus process. If you have a very large deposit, you will probably have a handful of servers in a custom multisig arrangement for validation, to minimize the chance of your server misbehaving or being hacked. This will require experimentation and technical expertise.

作为一个锁定保证金的验证人，你需要对块进行签名以及在共识过程中下注。如果你缴纳了一大笔保证金，你也许应该部署一个由多台服务器组成的多重签名环境来做验证工作，以减少服务器异常或是被黑导致的风险。这种方案需要反复实验和技术专家的帮助。

The validator should be kept online as reliably and as much as possible, for it to maximize its profitability (or for otherwise it will be unprofitable). It will be very advisable to buy DDoS protection. Additionally, your profitability will depend on the performance and availability of the other bonded validators. This means that there is risk that you cannot directly mitigate, yourself. You could lose money even if other nodes don’t perform well – but you will lose more money yet if you don’t participate at all, after bonding. However, additional risk also often means higher average profitability – especially if the risk is perceived but the costly event never occurs.

为了最大化收益，验证人需要尽可能的保持在线和服务稳定。DDoS防护服务是非常必要的。你的收益率还取决于其他验证人的处理性能和可用性，也就是说这里存在一个你无法直接化解的风险。如果其他节点表现不行你也会遭受损失！但是此时如果你决定完全不参与共识你会损失更多。然而额外的风险通常意味着更高的回报，尤其是当风险已经被认识到而永远不会发生的时候。

## What is it like to be an application or a user?

## 应用和普通用户有什么好处？

Applications and their users benefit a lot from the change from proof-of-work consensus to Casper. Lower latency significantly improves the user’s experience. In normal conditions transactions finalize very quickly. In the event of network partitions, on the other hand, transactions are still executed, but the fact that they can potentially still be reverted is reported clearly to the application and end-user. The application developer therefore still needs to deal with the possibility of forking, as they do in proof-of-work, but the consensus protocol itself provides them with a clear measure of what it would take for any given transaction to be reverted.

应用和它们的用户可以从POW转向Casper的变化中获得许多好处。低延迟确认可以极大的改善用户体验。一般情况下交易很快就能最终确认。如果有网络分区发生，交易依然会被执行，而交易有被撤销的可能性这一情况会被清楚的报告给应用和其用户。应用的开发者依然需要处理分叉的情况，和使用POW协议时一样，但这里共识协议会给出一个对交易撤销可能性的清楚估量。

## When can we hear more?

## 什么时候有更多消息？

Stay tuned! We’ll be sure to let you know more of Casper’s specification over the next months, as we come to consensus on the protocol’s details. In addition, you can look forward to seeing simulations, informal and formal specification, formal verification, and implementations of Casper! But please, be patient: R&D can take an unpredictable amount of time!  : )

请保持关注！随着协议细节的确定，接下来的几个月我们肯定会发布关于Casper的更多信息。大家可以期待下模拟测试，非正式的和形式化的标准，形式化的验证，以及Casper的实现！谢谢大家的耐心，研发需要的时间总是难以预测！: )
