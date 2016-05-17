# On Settlement Finality

[Origin post](https://blog.ethereum.org/2016/05/09/on-settlement-finality/) by Vitalik Buterin on May 9th, 2016

*Special thanks to Tim Swanson for reviewing, and for further discussions on the arguments in his original paper on settlement finality.*

*特别感谢Tim Swanson的审阅，以及他关于结算确定性的观点的进一步讨论。*

Recently one of the major disputes in ongoing debate between public blockchain and permissioned blockchain proponents is the issue of settlement finality. One of the simple properties that a centralized system at least appears to have is a notion of “finality”: once an operation is completed, that operation is completed for good, and there is no way that the system can ever “go back” and revert that operation. Decentralized systems, depending on the specific nature of their design, may provide that property, or they may provide it probabilistically, within certain economic bounds, or not at all, and of course public and permissioned blockchains perform very differently in this regard.

结算的确定性问题是最近公有链与许可链之间的一个主要战场。看起来中心化的系统至少有一个优点，即所谓的“确定性”（"finality"）：操作一旦完成，就永远完成了，系统永不可能再回退回去撤销这个操作。而去中心化的系统，根据设计的不同，可能有这个性质，也可能只提供在一定激励范围内的概率性确定，甚至没有确定性，在这一点上公有链和许可链是有很大区别的。

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

Technically, a proof of work blockchain never allows a transaction to truly be “finalized”; for any given block, there is always the possibility that someone will create a longer chain that starts from a block before that block and does not include that block. Practically speaking, however, financial intermediaries on top of public blockchains have evolved a very practical means of determining when a transaction is sufficiently close to being final for them to make decisions based on it: waiting for six confirmations.

从技术的观点看，工作量证明区块链上的交易永远也不会最终确定，对于任何一个区块，随时都存在着冒出一条更长的始于它的父块又不包含它的分叉的可能。然而在现实中，公有链上的金融服务已经演化出了一种非常实用的方法来判断一个交易是否足够近似确定：等6个确认。

The probabilistic logic here is simple: if an attacker has less than 25% of network hashpower, then we can model an attempted double spend as a random walk that starts at -6 (meaning “the attacker’s double-spend chain is six blocks shorter than the original chain”), and at each step has a 25% chance of adding 1 (ie. the attacker makes a block and inches a step closer) and an 75% chance of subtracting 1 (ie. the original chain makes a block). We can determine the probability that this process will ever reach zero (ie. the attacker’s chain overtaking the original) mathematically, via the formula (0.25 / 0.75)^6 ~= 0.00137 – smaller than the transaction fee that nearly all exchanges charge. If you want even greater certainty, you can wait 13 confirmations for a one-in-a-million chance of the attacker succeeding, and 162 confirmations for a chance so small that the attacker is literally more likely to guess your private key in a single attempt. Hence, some notion of de-facto finality even on proof-of-work blockchains does in fact exist.

这里的概率计算很简单：如果攻击者拥有的算力不到25%，我们便可以用随机漫步来描述双花攻击，随机漫步从-6开始（意味着攻击者的链比原来的链短了6个区块）。通过公式`(0.25 / 0.75)^6 ~= 0.00137`，我们可以计算出这个随机过程达到0的概率（即攻击者的链超过原来的链的概率），小于几乎所有交易所的手续费率（译注：攻击成本）。如果你想要更高的确定性，可以等待13个确认让攻击者只有百万分之一的成功机会，或者等待162个确认使得攻击者的机会比直接猜出你的私钥还低。因此，即使在基于工作量证明的区块链上**也存在**一定程度的确定性。

However, this probabilistic logic assumes that 75% of nodes behave honestly (at lower percentages like 60% a similar argument can be made but more confirmations are required). There is now also an economic debate to be had: is that assumption likely to be true? There are arguments that miners can be bribed, eg. through a P + epsilon attack, to all follow an attacking chain (a practical way of executing such a bribe may be to run a negative-fee mining pool, possibly advertising a zero fee and quietly providing even higher revenues to avoid arousing suspicion). Attackers may also try to hack into or disrupt the infrastructure of mining pools, an attack which can potentially be done very cheaply as the incentive for security in proof of work is limited (if a miner gets hacked, they lose only their rewards for a few hours; their principal is safe). And, last but not least, there is what Swanson has elsewhere called the “Maginot Line” attack: throw a very large amount of money at the problem and simply bring more miners in than the rest of the network combined.

问题是，我们的计算假设了75%的节点是靠谱的（对于更低的比例，例如60%，会有近似的结论但是需要更多的确认数）。这个前提能在我们的激励模型中成立吗？攻击者可以贿赂矿工都选择攻击者的链（一个比较实际的贿赂方式是运行一个负费率的矿池，或者名义上零费率另外补贴收益率来避免招人怀疑），[P + epsilon attack](https://blog.ethereum.org/2015/01/28/p-epsilon-attack/)就是一个思路。攻击者还可以尝试黑进矿池或者破坏矿池基础设施，这在对工作量证明的安全保护缺少激励的环境下实施成功的概率不小（如果矿工被黑，他们只会损失一段时间内的奖励，本金是安全的）。最后还有Swanson说的所谓的“马其诺防线”攻击：投入巨量的金钱制造出比全网更高的算力进行简单碾压。

## Finality in Casper
## Casper中的确定性

The Casper protocol is intended to offer stronger finality guarantees than proof of work. First, there is a standard definition of “total economic finality”: it takes place when 2/3 of all validators make maximum-odds bets that a given block or state will be finalized. This condition offers very strong incentives for validators to never try to collude to revert the block: once validators make such maximum-odds bets, in any blockchain where that block or state is not present, the validators lose their entire deposits. As Vlad Zamfir put it, imagine a version of proof of work where if you participate in a 51% attack your mining hardware burns down.

Casper尝试提供比工作量证明更高的确定性保证。首先Casper对“经济上完全确定”（total economic finality）有标准定义：当大于等于2/3的验证人以最大概率投注一个区块或者说状态会最终确定的时候。在这个定义下验证人有非常强的激励不去合谋推翻这个区块：一旦验证人作出了最大概率的投注，在任何一个不包含这个区块的分叉中验证人都会失去他们全部的保证金。正如Vlad Zamfir指出的，你可以把Casper想象成一个参与51%的攻击会导致你的矿机被烧毁的工作量证明变种。

Second, the fact that validators are pre-registered means that there is no possibility that somewhere else out there there are some other validators making the equivalent of a longer chain. If you see 2/3 of validators placing their entire stakes behind a claim, then if you see somewhere else 2/3 of validators placing their entire stakes behind a contradictory claim, that necessarily implies that the intersection (ie. at least 1/3 of validators) will now lose their entire deposits no matter what happens. This is what we mean by “economic finality”: we can’t guarantee that “X will never be reverted”, but we can guarantee the slightly weaker claim that “either X will never be reverted or a large group of validators will voluntarily destroy millions of dollars of their own capital”.

其次，由于成为验证人需要事先申请，这意味着不可能存在另外的验证人在悄悄的制造另一条更长的链。如果你看到2/3的验证人将他们的全部本钱压倒了某个块上，然后又发现有2/3的验证人对另一个矛盾的块做了相同的事，那么这只能说明这两组验证人的交集（即至少1/3的验证人）将失去他们的全部保证金。这便是所谓的“经济上的确定性”：我们无法保证“X永远不会被撤销”，但我们可以保证的是一个稍弱的说法，“X要么永远不被撤销，要么有一大群验证人自愿的销毁他们自己价值数百万美元的本金”。

Finally, even if a double-finality event does take place, users are not forced to accept the claim that has more stake behind it; instead, users will be able to manually choose which fork to follow along, and are certainly able to simply choose “the one that came first”. A successful attack in Casper looks more like a hard-fork than a reversion, and the user community around an on-chain asset is quite free to simply apply common sense to determine which fork was not an attack and actually represents the result of the transactions that were originally agreed upon as finalized.

最后，即使双重确定（double-finality）的事件真的发生了，用户也无需被迫接受有更多投注支持的分叉。相反，用户可以自行决定追随哪个分叉，一个完全可行的简单策略是接受“先来的那个”。Casper中的一次成功攻击更像是一次硬分叉而不是回退，而持有链上资产的用户社区可以自由的基于常识选择那条不是攻击者制造的，而是包含真正应该被确定的交易的分叉。

## Law and Economics
## 法律与经济

However, these stronger protections are nevertheless economic. And this is where we get to the next part of Swanson’s argument:

可惜这些强有力的保证依然是经济上的。正如Swanson在他的下一个观点中说到的：

> Thus, if the market value of a native token (such as a bitcoin or ether) increases or decreases, so too does the amount of work generated by miners who compete to receive the networks seigniorage and expend or contract capital outlays in proportion to the tokens marginal value. This then leaves open the distinct possibility that, under certain economic conditions, Byzantine actors can and will successfully create block reorgs without legal recourse.

> 因而，如果原生代币（例如比特币或是以太币）的市场价值上涨或下跌，矿工为竞争筑币权和手续费而产生的工作量以及合约的成本价值也会随之变动。这就留下了一种可能，在特定的经济环境下，恶意节点可以成功的促成链上区块的重组。

There are two versions of this argument. The first is a kind of “law maximalist” viewpoint that “mere economic guarantees” are worthless and purely in some philosophical sense legal guarantees are the only kind of guarantees that count. This stronger version is obviously false: in many cases, the primary or only kind of punishment that the law metes out for malfeasance is fines, and fines are themselves nothing more than a “mere economic incentive”. If mere economic incentives are good enough for the law, at least in some cases, then they ought to be good enough for settlement architectures, at least in some cases.

这个观点有两个版本。一种来自“极端法律主义者”，他们认为“区区经济保证”没什么价值，只有理论意义，法律保障才是唯一有效力的保证。这样的观点显然不对：在很多情况下，法律能对不法行为给予的主要或者唯一惩罚便是罚款，而罚款正是一种“区区经济激励”。如果区区经济激励能被法律所用，那么至少在某些场合，它们也能被结算系统所用。

The second version of the argument is much more simple and pragmatic. Suppose that, in the current situation where the total value of all existing ether is $700 million, you calculate that you need $30 million of mining power to successfully conduct a 51% attack, and once Casper launches you predict that there will be a staking participation rate of 30%, and so finality reversion will carry a minimum cost of $700 million * 30% * 1/3 = $70 million (if you are willing to reduce your tolerance to validators dropping offline to 1/4, then you can increase the finality threshold to 3/4, and thereby increase the size of the intersection to 1/2 and thereby get an even higher security margin at $105 million). If you are trading $10 million worth of equities, and you intend to do this for only two months, then this is almost certainly fine; the public blockchain’s economic incentives will do quite a fine job of disincentivizing malfeasance and any attack will not be nearly worth the trouble.

第二个版本要简单和务实的多。假设现在所有以太币的价值加起来是7亿美元，你发现进行一次成功的51%攻击需要价值3千万美元的算力，而一旦Casper上线你预计资本参与率将是30%，因此要撤销一个确定交易需要的最低成本是 7亿美元 * 30% * 1/3 = 7千万美元（如果你愿意把对验证人在线率的容忍度降低到1/4，则可以换来3/4的确定性阈值，从而把作恶资本的交集增加到1/2，使攻击的最低成本上升为1亿500万美元）。如果你要交易价值为1亿美元的证券，期限是两个月，那么不会有什么大问题；公有链的经济激励设计可以很好的防止恶意行为，任何的攻击都不会划算。

Now, suppose that you intend to trade $10 million worth of equities, but you are going to commit to using the Ethereum public blockchain as the base infrastructure layer for five years. Now, you have much less certainty. The value of ether could be the same or higher, or it could be near-zero. The participation rate in Casper could go up to 50%, or it could drop to 10%. Hence, it’s entirely possible that the cost of a 51% attack will drop, say to even below $1 million. At that point, conducting a 51% attack in order to earn profits through some market manipulation attack is entirely possible.

现在假设你同样交易价值1亿美元的证券，但是会在5年的时间内用以太坊公有链作为底层。此时你拥有的确定性要少得多。以太币的价格可能不变，更高，或者归零。Casper共识的资本参与率可能上升到50%，也可能掉到10%。因此，发动一次51%攻击所需的成本完全可能下降到，例如，一百万美元以下。在那个时刻，通过51%攻击配合一些市场操纵来盈利是完全可行的。

A third case is an even more obvious one: what if you want to trade $100 billion worth of equities? Now, the cost of attacking the public blockchain is peanuts compared to the potential profits from a market manipulation attack; hence, the public blockchain is completely unsuitable for the task.

还有一种情况更简单：如果你要交易价值1千亿美元的证券呢？此时攻击公有链的成本想对于市场操纵可获得的收益来说可以忽略不计，因此此时公有链完全不适用。

It is worth noting that the cost of an attack is not quite as simple to estimate as was shown above. If you bribe existing validators to carry out an attack, then the math applies. A more realistic scenario, however, would involve buying coins and using those deposits to attack; this would have a cost of either $105 million or $210 million depending on the finality threshold. The act of buying coins may also affect the price. The actual attack, if imperfectly planned, will almost certainly result in even greater losses than the theoretical minimum of 1/3 or 1/2, and the amount of revenue that can be earned from an attack will likely be much less than the total value of the assets. However, the general principle remains the same.

需要指出的是实际上攻击成本的计算比上面的例子要更复杂。如果你要通过贿赂现有的验证人来发动攻击，这些计算没问题。现实中更可能发生的是，你需要购买代币来发动攻击，根据确定性的阈值不同这会使成本变为1亿500万或是2亿1千万美元。购买代币这个操作本身又会影响代币价格。攻击本身，如果计划得有瑕疵，必然会使实际处罚比例大于理论上的最小值1/3或1/2，导致攻击所能获得的收益要远小于预期。但基本的原理依然是成立的。

Hence, all in all, the weaker argument, that for high-value assets the economic security margin of public blockchains is too low, is entirely correct and depending on the use case is a completely valid reason for financial institutions to explore private and consortium chains.

于是我们可以说，这个观点的弱化版本，也就是说**公有链的经济安全边界对于高价值的资产来说太低了**，是完全正确的，金融机构探索在特定场景中使用私有链和联盟链是完全合理的。

## Censorship Resistance, and other Practical Concerns
## 防审查以及其他考量

Another concern that is raised is the issue that public blockchains are censorship resistant, allowing anyone to send transactions, whereas financial institutions have the requirement to be able to limit which actors participate in which systems and sometimes what form that participation takes. This is entirely correct. One counter-point that can be raised is that public blockchains, and particularly highly generalizeable ones such as Ethereum, can serve as base layers for systems that do carry these restrictions: for example, one can create a token contract that only allows transactions which transfer to and from accounts that are in a specific list or are approved by an entity represented by a specific address on the chain. The rebuttal that is made to this counter-point elsewhere is that such a construction is unnecessarily Rube-Goldbergian, and one may as well just create the mechanism on a permissioned chain in the first place – otherwise one is paying the costs of censorship-resistance and independence from the traditional legal system that public chains provide without the benefits. This argument is reasonable, although it is important to point out that it is an argument about efficiency, and not fundamental possibility, so if benefits of public chains not connected to censorship resistance (eg. lower coordination costs, network effect) prove to dominate then it is not an absolute knockdown.

对公有链的另一担心是其防审查的性质，即任何人都可以发起交易，而金融机构总是有需要去限制系统中的参与者或是参与形式。这也是完全正确的。对此有一种针对性的观点是公有链尤其是类似以太坊这样高度通用化的区块链，可以作为实现这些限制的系统的底层：例如，你可以创建一个只允许白名单内的账户参与或是只有代表某机构的账户才有管理权限的代币合约。对此观点的反驳是这样的设计完全是没有必要的复杂，相较于直接在许可链上实现这些机制，你需要承担公有链为做到防审查和独立于传统司法体系而付出的成本，同时放弃其好处。这样的观点是合理的，但是需要指出的是这个关于效率的反驳，而不是关于可能性的。所以如果除了防审查之外的好处（例如，更低的协调成本，网络效应）足够大，这个理由会变得需要商榷。

There are other efficiency concerns. Because public blockchains must maintain a high degree of decentralization, the node software must be able to be run on standard consumer laptops; this puts strains on transaction throughput that do not exist to the same extent on a permissioned network, where one can simply require all nodes to run on 64-core servers with very high-speed internet connections. In the future, the intention is certainly for innovations in sharding to alleviate these concerns on the public chain, and if implementation goes as planned then in half a decade’s time there will be no limit to the scaling throughput of public chains as long as you parallelize enough and add enough nodes to the network, although even still there will always inevitably remain at least some efficiency and thus cost differential between public and permissioned chains.

除此之外还有另外的效率考量。由于公有链必须保持高度的去中心化，节点软件必须能够在标准的消费级电脑上运行。这就给交易吞吐量设置了一个在许可链中不存在的限制，在许可链中我们可以轻松要求所有节点都运行在64核相互之间以高速网络连接的服务器上。在未来，我们需要通过诸如分片的创新来缓解公有链的这个问题，如果进行顺利的话在5年左右的时间内公有链的吞吐量将可以无限扩充，只要并行程度足够高，网络中有足够多的节点。但即使这样公有链和许可链之间依然不可避免的会存在一些效率和成本的差异。

The final technical concern is latency. Public chains run between thousands of consumer laptops on the public internet, whereas permissioned chains run between a much smaller number of nodes with fast internet connections, which may even be located physically close to each other. Hence, the latency, and hence time-to-finality, of permissioned chains will inevitably be lower than of public chains. Unlike concerns about efficiency, this is a problem that can never be made negligible because of technological improvements: as much as we might wish it to, Moore’s law does not make the speed of light become twice as fast every two years, and no matter how many optimizations get made there will always be a differential between networks made out of many arbitrarily located nodes and networks made out of a possibly colocated few nodes, and the difference between the two will always be quite visible to the human eye.

最后一个技术考量是时延。公有链运行于成千上万的通过公开网络连接的消费级电脑之上，而许可链则运行在通过高速网络相连的数量少得多的一组节点上，这些节点甚至可能物理上非常接近。因此许可链的交易时延，也就是最终确定所需要的时间，必然低于公有链。与效率问题不同，这是一个永远无法通过技术进步解决的问题：与我们希望的相悖，摩尔定律无法让光速每隔两年翻倍。无论我们做多少优化，一个节点位置任意的网络和节点相邻的网络之间终究会有差异，而且这个差异基本上是肉眼可见。

At the same time, public blockchains of course have many advantages in their own right, and there are likely many use cases for which the legal, business development and trust costs of setting up a consortium chain for some application are so high that it will be much simpler to just throw it on the public chain, and a large part of what makes the public chain valuable is in fact its ability to allow users to build applications regardless of how socially well-connected they are: even a 14-year-old can code up a decentralized exchange, publish it to the blockchain, and others can evaluate and use the application based on its own merits. Some developers just don’t have the connections to put together a consortium, and public chains play a crucial role in serving these developers. The cross-application synergies that can so easily organically emerge in public chains are another important benefit. Ultimately, we may see the two ecosystems evolving to serve different constituencies over time, although even still they share many challenges in scalability, security and privacy, and can benefit greatly by working together.

与此同时，公有链当然有它自己的优点，也许在很多场景下运行联盟链所需的法律，商务和信任成本会高到让人选择公有链。而公有链的很大一块价值在于任何人，无论拥有什么样的社会资源，都能在上面构建应用：一个14岁的孩子也可以实现一个去中心化的交易所部署到区块链上，其他人可以评估这个应用并根据自己的需求使用。许多开发者不具备发起一个联盟的能力，而公有链对于这一类开发者来说非常关键。公有链的另一个重要优点是可以轻松实现的跨应用的协同。最终，我们将看到这两种体系随着时间进化，服务于不同的人群，而它们也面临着包括扩展性，安全性和隐私保护在内的共有挑战，因此双方都可以从合作中获益。
