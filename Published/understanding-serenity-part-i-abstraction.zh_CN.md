# Understanding Serenity, Part I: Abstraction
# 理解Serenity 第一部分: 深度抽象

[Origin post](https://blog.ethereum.org/2015/12/24/understanding-serenity-part-i-abstraction/) by Vitalik Buterin, on December 24th, 2015

**For a long time we have been public about our plans to continue improving the Ethereum protocol over time and our long development roadmap, learning from our mistakes that we either did not have the opportunity to fix in time for 1.0 or only realized after the fact. However, the Ethereum protocol development cycle has started up once again, with a Homestead release coming very soon, and us quietly starting to develop proof-of-concepts for the largest milestone that we had placed for ourselves in our development roadmap: Serenity.**

**我们已经公开继续改进以太坊协议的计划和长期开发路线图相当长一段时间了，这个做法也是来自于从1.0版本发布之前或者事后没有能及时处理的错误中学到的经验。不管怎样，以太坊核心协议的周期性开发已经重新启动，Homestead阶段很快就要到来，我们也已经悄悄开始开发一个概念原型(PoC)，目标是[开发路线图](https://blog.ethereum.org/2015/03/03/ethereum-launch-process/)中最大的里程碑: Serenity.**

Serenity is intended to have two major feature sets: abstraction, a concept that I initially expanded on in this blog post here, and Casper, our security-deposit-based proof of stake algorithm. Additionally, we are exploring the idea of adding at least the scaffolding that will allow for the smooth deployment over time of our scalability proposals, and at the same time completely resolve parallelizability concerns brought up here – an instant very large gain for private blockchain instances of Ethereum with nodes being run in massively multi-core dedicated servers, and even the public chain may see a 2-5x improvement in scalability. Over the past few months, research on Casper and formalization of scalability and abstraction (eg. with EIP 101) have been progressing at a rapid pace between myself, Vlad Zamfir, Lucius Greg Meredith and a few others, and now I am happy to announce that the first proof of concept release for Serenity, albeit in a very limited form suitable only for testing, is now available.

Serenity会有两大主要特性：深度抽象，一个我最早在[这里](http://ethfans.org/topics/109)展开讨论过的特性，和Casper，基于保证金的权益证明(PoS)算法。此外，我们也在探索平滑的部署[可伸缩性](https://www.youtube.com/watch?v=-QIt3mKLIYU)(scalability)[改进](https://blog.ethereum.org/2015/04/05/blockchain-scalability-chain-fibers-redux/)的方法，至少是一个脚手架，同时完全解决[这里](http://www.multichain.com/blog/2015/11/smart-contracts-slow-blockchains/)对并行性的担忧 - 运行在私有链环境下，多核CPU专有服务器之上的以太坊节点性能将会有立竿见影的巨大提升，甚至公有链的可伸缩性也能看到2到5倍的提升。在过去的几个月中，Casper的研究和对可伸缩性与抽象改进的形式化工作(eg. [EIP101](https://github.com/ethereum/EIPs/issues/28))都在快速推进，参与者有我, Vlad Zamfir, Lucius Greg Meredith和其他一些人。现在我很高兴的宣布，Serenity阶段的第一个概念原型, 尽管能做的事情还非常有限仅仅可以用于测试，[已经完成](http://github.com/ethereum/pyethereum/blob/serenity/)。

The PoC can be run by going into the ethereum directory and running python test.py (make sure to download and install the latest Serpent from https://github.com/ethereum/serpent, develop branch); if the output looks something like this then you are fine:

在`ethereum`目录下运行`python test.py`就可以把概念原型跑起来（别忘了先从https://github.com/ethereum/serpent下载和安装最新的Serpent develop分支），如果看到这样的输出就对了：

```
vub@vub-ThinkPad-X250 15:01:03 serenity/ethereum: python test.py
REVERTING 940534 gas from account 0x0000000000000000000000000000000000000000 to account 0x98c78be58d729dcdc3de9efb3428820990e4e3bf with data 0x
Warning (file "casper.se.py", line 74, char 0): Warning: function return type inconsistent!
Running with 13 maximum nodes
Warning (file "casper.se.py", line 74, char 0): Warning: function return type inconsistent!
Warning (file "casper.se.py", line 74, char 0): Warning: function return type inconsistent!
Length of validation code: 57
Length of account code: 0
Joined with index 0
Length of validation code: 57
Length of account code: 0
Joined with index 1
Length of validation code: 57
```

This is a simulation of 13 nodes running the Casper+Serenity protocol at a 5-second block time; this is fairly close to the upper limit of what the client can handle at the moment, though note that (i) this is python, and C++ and Go will likely show much higher performance, and (ii) this is all nodes running on one computer at the same time, so in a more “normal” environment it means you can expect python Casper to be able to handle at least ~169 nodes (though, on the other hand, we want consensus overhead to be much less than 100% of CPU time, so these two caveats combined do NOT mean that you should expect to see Casper running with thousands of nodes!). If your computer is too slow to handle the 13 nodes, try python test.py 10 to run the simulation with 10 nodes instead (or python test.py 7 for 7 nodes, etc). Of course, research on improving Casper’s efficiency, though likely at the cost of somewhat slower convergence to finality, is still continuing, and these problems should reduce over time. The network.py file simulates a basic P2P network interface; future work will involve swapping this out for actual computers running on a real network.

在每5秒一个块，使用Casper+Serenity协议的条件下，这个程序模拟了的13个节点的运行；这个模拟已经很接近当前客户端能处理的上限了，不过要注意：(i) 这是python写的，C++和Go的实现很可能会有更好的表现，以及(ii) 所有这些节点都同时运行在一台电脑上，所以你有理由相信在一个更“正常”的环境中python版本的Casper可以处理大约169个节点（不过，从另一方面来说，我们希望共识的开销远低于100%的CPU占用，因此这两点注意并不表示你可以期待Casper和上千个节点一起工作！）。如果你的电脑太慢处理不了13个节点，试试用`python test.py 10`来模拟10个节点（或者`python test.py 7`来模拟7个节点，你懂的）。当然，改进Casper效率的研究仍在继续，虽然这改进可能会以减慢终局性(finality)的收敛为代价，这些问题都会逐渐解决。`network.py`文件模拟了一个基本的P2P网络接口，接下来的工作会把它替换成运行在真实网络上的真实计算机。

The code is split up into several main files as follows:

程序代码被分割到几个主要文件中：

* serenity_blocks.py – the code that describes the block class, the state class and the block and transaction-level transition functions (about 2x simpler than before)
* serenity_transactions.py – the code that describes transactions (about 2x simpler than before)
* casper.se.py – the serpent code for the Casper contract, which incentivizes correct betting
* bet.py – Casper betting strategy and full client implementation
* ecdsa_accounts.py – account code that allows you to replicate the account validation functionality available today in a Serenity context
* test.py – the testing script
* config.py – config parameters
* vm.py – the virtual machine (faster implementation at fastvm.py)
* network.py – the network simulator

* `serenity_blocks.py` - 描述block类，state类，以及block和transaction级别状态转移函数的代码（比以前的版本简单一倍）。
* `serenity_transactions.py` - 描述transaction的代码（比之前简单一倍）。
* `casper.se.py` - Casper合约的serpent实现，激励正确的下注行为。
* `bet.py` - Casper的下注逻辑和完整的客户端实现。
* `ecdsa_accounts.py` - 账户相关代码，让你可以在Serenity上模拟当前的账户验证逻辑。
* `test.py` - 测试脚本
* `config.py` - 参数配置
* `vm.py` - 虚拟机（`fastvm.py`提供了一个更快的实现）
* `network.py` - 网络模拟

For this article, we will focus on the abstraction features and so serenity_blocks.py, ecdsa_accounts.py and serenity_transactions.py are most critical; for the next article discussing Casper in Serenity, casper.se.py and bet.py will be a primary focus.

在这篇文章中，我们只讨论深度抽象的特性，因此关键文件是`serenity_blocks.py`, `ecdsa_accounts.py`和`serenity_transactions.py`；在下一篇讨论Casper的文章中，`casper.se.py`和`bet.py`将会是焦点。

## Abstraction and Accounts
## 账户的抽象

Currently, there are two types of accounts in Ethereum: externally owned accounts, controlled by a private key, and contracts, controlled by code. For externally owned accounts, we specify a particular digital signature algorithm (secp256k1 ECDSA) and a particular sequence number (aka. nonce) scheme, where every transaction must include a sequence number one higher than the previous, in order to prevent replay attacks. The primary change that we will make in order to increase abstraction is this: rather than having these two distinct types of accounts, we will now have only one – contracts. There is also a special “entry point” account, 0x0000000000000000000000000000000000000000, that anyone can send from by sending a transaction. Hence, instead of the signature+nonce verification logic of accounts being in the protocol, it is now up to the user to put this into a contract that will be securing their own account.

目前以太坊中有两种类型的账户：外部拥有的账户，由私钥控制，和合约账户，由代码控制。对于外部拥有的账户，我们指定了一种特别的数字签名算法(secp256k1椭圆曲线签名)和一个序号体系(nonce)，要求每一个交易都必须包含一个比前一个交易序号大1的数字，目的是防止重放攻击(replay attacks)。我们为提高抽象程度而做的主要改动是：不再有两种不同类型的账户，而是统一为一种 - 合约账户。将会存在一个特殊的“入口”账户，`0x0000000000000000000000000000000000000000`，**任何人**都可以从这个账户发起交易。因此，协议中将不再包含签名+nonce的账户验证逻辑，用户必须用合约来保护他们自己的账户。

The simplest kind of contract that is useful is probably the ECDSA verification contract, which simply provides the exact same functionality that is available right now: transactions pass through only if they have valid signatures and sequence numbers, and the sequence number is incremented by 1 if a transaction succeeds. The code for the contract looks as follows:

最简单有效的此类合约可能要属椭圆曲线数字签名验证合约了，它可以提供和现在完全一样的功能：只有拥有有效签名和序号的交易才能通过验证，而且序号在交易成功后增加1。这个合约的代码如下：

```
# We assume that data takes the following schema:
# bytes 0-31: v (ECDSA sig)
# bytes 32-63: r (ECDSA sig)
# bytes 64-95: s (ECDSA sig)
# bytes 96-127: sequence number (formerly called "nonce")
# bytes 128-159: gasprice
# bytes 172-191: to
# bytes 192-223: value
# bytes 224+: data

# Get the hash for transaction signing
~mstore(0, ~txexecgas())
~calldatacopy(32, 96, ~calldatasize() - 96)
~mstore(0, ~sha3(0, ~calldatasize() - 64))
~calldatacopy(32, 0, 96)
# Call ECRECOVER contract to get the sender
~call(5000, 1, 0, 0, 128, 0, 32)
# Check sender correctness; exception if not
if ~mload(0) != 0x82a978b3f5962a5b0957d9ee9eef472ee55b42f1:
    ~invalid()
# Sequence number operations
with minusone = ~sub(0, 1):
    with curseq = self.storage[minusone]:
        # Check sequence number correctness, exception if not
        if ~calldataload(96) != curseq:
            ~invalid()
        # Increment sequence number
        self.storage[minusone] = curseq + 1
# Make the sub-call and discard output
with x = ~msize():
    ~call(msg.gas - 50000, ~calldataload(160), ~calldataload(192), 160, ~calldatasize() - 224, x, 1000)
    # Pay for gas
    ~mstore(0, ~calldataload(128))
    ~mstore(32, (~txexecgas() - msg.gas + 50000))
    ~call(12000, ETHER, 0, 0, 64, 0, 0)
    ~return(x, ~msize() - x)
```

This code would sit as the contract code of the user’s account; if the user wants to send a transaction, they would send a transaction (from the zero address) to this account, encoding the ECDSA signature, the sequence number, the gasprice, destination address, ether value and the actual transaction data using the encoding specified above in the code. The code checks the signature against the transaction gas limit and the data provided, and then checks the sequence number, and if both are correct it then increments the sequence number, sends the desired message, and then at the end sends a second message to pay for gas (note that miners can statically analyze accounts and refuse to process transactions sending to accounts that do not have gas payment code at the end).

这段代码就是用户账户的合约代码；如果用户要从自己的账户发送一个交易，他们会先从地址0发送一个交易到这个账户，交易数据会像上述代码所示那样包括ECDSA签名，序号，gas价格，目标地址，数额和真正的交易数据。合约代码会检查对交易gas限制和数据的签名，然后检查交易序号，如果两者都没有问题就把保存的序号加一，发送所需的消息，然后发送另一条消息支付gas费用作为结束（注意，矿工可以静态分析账户的合约代码，如果交易的账户合约最后没有支付gas可以拒绝处理）。

An important consequence of this is that Serenity introduces a model where all transactions (that satisfy basic formatting checks) are valid; transactions that are currently “invalid” will in Serenity simply have no effect (the invalid opcode in the code above simply points to an unused opcode, immediately triggering an exit from code execution). This does mean that transaction inclusion in a block is no longer a guarantee that the transaction was actually executed; to substitute for this, every transaction now gets a receipt entry that specifies whether or not it was successfully executed, providing one of three return codes: 0 (transaction not executed due to block gas limit), 1 (transaction executed but led to error), 2 (transaction executed suceessfully); more detailed information can be provided if the transaction returns data (which is now auto-logged) or creates its own logs.

Serenity的这个改动有一个很重要的后果，系统中的**所有交易（只要满足基本的格式）都是有效的**。现阶段无效的交易在Serenity中将仅仅是没有作用（上例中的`invalid`是一个尚未使用的操作码，它会使程序执行立即退出）。这意味着交易被打包进区块不再是交易会被真正执行的保证，作为弥补，每一笔交易都会有一条收据记录（receipt entry），通过返回码指明它是否成功执行：**0**表示由于gas限制交易没有执行，**1**表示交易执行了但是出错，**2**表示交易成功执行。收据记录还可以提供更多的信息，例如交易的返回值（现在有自动日志记录）或者自己创建的日志，

The main very large benefit of this is that it gives users much more freedom to innovate in the area of account policy; possible directions include:

这个改动最主要的好处是用户可以在账户策略这个领域自由的创新了。可能的方向包括：

* Bitcoin-style multisig, where an account expects signatures from multiple public keys at the same time before sending a transaction, rather than accepting signatures one at a time and saving intermediate results in storage
* Other elliptic curves, including ed25519
* Better integration for more advanced crypto, eg. ring signatures, threshold signatures, ZKPs
* More advanced sequence number schemes that allow for higher degrees of parallelization, so that users can send many transactions from one account and have them included more quickly; think a combination of a traditional sequence number and a bitmask. One can also include timestamps or block hashes into the validity check in various clever ways.
* UTXO-based token management – some people dislike the fact that Ethereum uses accounts instead of Bitcoin’s “unspent transaction output” (UTXO) model for managing token ownership, in part for privacy reasons. Now, you can create a system inside Ethereum that actually is UTXO-based, and Serenity no longer explicitly “privileges” one over the other.
* Innovation in payment schemes – for some dapps, “contract pays” is a better model than “sender pays” as senders may not have any ether; now, individual dapps can implement such models, and if they are written in a way that miners can statically analyze and determine that they actually will get paid, then they can immediately accept them (essentially, this provides what Rootstock is trying to do with optional author-pays, but in a much more abstract and flexible way).
* Stronger integration for “ethereum alarm clock”-style applications – the verification code for an account doesn’t have to check for signatures, it could also check for Merkle proofs of receipts, state of other accounts, etc

* **比特币风格的多重签名**，账户要求交易同时具有多个私钥的签名，而不是一次接收一个签名，然后把中间结果临时存放在区块链里。
* **其他的椭圆曲线**, 包括ed25519。
* **更好的集成先进的加密算法**，例如环状签名(ring signature)， 门限签名(threshold signature), 零知识证明等等。
* **更先进的序号方案**, 支持更高程度的并行化，让用户可以同时从一个账户发出多个交易，并且更快的把这些交易打包。想想传统的序号和位掩码(bitmask)如果结合会怎样。我们也可以通过各种机智的方法在有效性检查中利用时间戳或是区块的hash值。
* **基于UTXO的代币管理**, 有些用户处于隐私的元应，不喜欢以太坊使用账户而不是比特币的“未使用的交易输出”(unspent transaction output, UTXO)模型来管理代币的所有权。现在你可以在以太坊中建立一个事实上基于UTXO的系统了，并且Serenity不再显式的特殊对待其中某一个。
* **支付方案的创新**，对于某些dapp, “由合约支付费用”可能比“由使用者支付”更有用，使用者可能没有以太币。现在dapp就可以实现这个支付模型了。如果矿工能对它的代码做静态分析并确信他们可以得到报酬，矿工就会接受这种交易（本质上，我们实现了[Rookstock想要通过可选的作者支付(author-pay)想做的事情](http://www.rootstock.io/)，但是是通过一种更抽象和灵活的方法）。
* **与“以太坊闹钟”之类的应用更好的集成**，账户的检验代码不一定要检查签名，也可以检查收据的Merkle proof，或是其他账户的状态，等等。

In all of these cases, the primary point is that through abstraction all of these other mechanisms become much easier to code as there is no longer a need to create a “pass-through layer” to feed the information in through Ethereum’s default signature scheme; when no application is special, every application is.

提出这些场景是为了说明最主要的论点，通过抽象所有这些另外的机制都可以更加容易的实现，不再需要创造一个“传递层”来把信息喂给以太坊默认的签名体系。当没有应用是特殊的时候，每个应用都特殊。

One particular interesting consequence is that with the current plan for Serenity, Ethereum will be optionally quantum-safe; if you are scared of the NSA having access to a quantum computer, and want to protect your account more securely, you can personally switch to Lamport signatures at any time. Proof of stake further bolsters this, as even if the NSA had a quantum computer and no one else they would not be able to exploit that to implement a 51% attack. The only cryptographic security assumption that will exist at protocol level in Ethereum is collision-resistance of SHA3.

一个特别有意思的结果是：在Serenity的设计下，以太坊将具有可选的量子安全性。如果你害怕NSA秘密拥有一台量子计算机，想要把自己的账户变得更安全，你随时可以个人切换使用Lamport签名。转换到PoS机制进一步巩固了安全性，即使世界上只有NSA有量子计算机他们也不能利用这一点来实施51%攻击。在以太坊的协议层唯一剩下的密码学安全假设只有SHA3的抵御碰撞（collision-resistance）的性质了。

As a result of these changes, transactions are also going to become much simpler. Instead of having nine fields, as is the case right now, transactions will only have four fields: destination address, data, start gas and init code. Destination address, data and start gas are the same as they are now; “init code” is a field that can optionally contain contract creation code for the address that you are sending to.

这些改变的另一个结果是，交易数据会变得更加简单。交易将会用四个字段取代现在的九个：目标地址，数据，初始gas和初始化代码。目标地址，数据和初始gas和现在一样，“初始化代码”是一个可选用于保存目标地址账户合约创建代码的字段。

The reason for the latter mechanic is as follows. One important property that Ethereum currently provides is the ability to send to an account before it exists; you do not need to already have ether in order to create a contract on the blockchain before you can receive ether. To allow this in Serenity, an account’s address can be determined from the desired initialization code for the account in advance, by using the formula `sha3(creator + initcode) % 2**160` where creator is the account that created the contract (the zero account by default), and initcode is the initialization code for the contract (the output of running the initcode will become the contract code, just as is the case for CREATEs right now). You can thus generate the initialization code for your contract locally, compute the address, and let others send to that address. Then, once you want to send your first transaction, you include the init code in the transaction, and the init code will be executed automatically and the account created before proceeding to run the actual transaction (you can find this logic implemented here).

需要这个机制的原因如下。目前以太坊有一个重要的性质，是允许往一个还不存在的账户转账。为了在区块链上创建一个合约接受以太币，你并不需要事先持有任何以太币。为了在Serenity中实现这个性质，我们让账户地址能事先从它的初始化代码中推导出来，通过公式`sha3(creator + initcode) % 2**160`。这里`creator`是创建这个合约的账户（默认是地址为0的账户），而`initcode`就是合约的初始化代码（这段代码的运行结果会成为账户合约的代码，正如现在的CREATE一样）。因此你可以在本地先生成这段初始化代码，计算它的地址，然后让其他人往这个地址转账。然后一旦你想要从这个地址发出第一笔交易，你可以在交易里面包含这段初始化代码，它就会在真正的交易执行之前，被自动执行并创建账户（这段逻辑的实现代码在[这里](https://github.com/ethereum/pyethereum/blob/42a448be003e07db18cd94159a61f3a361aff57e/ethereum/serenity_blocks.py#L253)）。

## Abstraction and Blocks
## 区块的抽象

Another clean separation that will be implemented in Serenity is the complete separation of blocks (which are now simply packages of transactions), state (ie. current contract storage, code and account balances) and the consensus layer. Consensus incentivization is done inside a contract, and consensus-level objects (eg. PoW, bets) should be included as transactions sent to a “consensus incentive manager contract” if one wishes to incentivize them.

Serenity中将实现的另一个干净的分离是将区块（仅仅是一堆交易），状态（例如合约的存储区，代码和账户余额）和共识层完全分开。共识激励在合约内部实现，而如果你希望激励的话，共识级别的对象（例如PoW, 赌注）应该被包含在发往“共识激励管理合约”的交易中。

This should make it much easier to take the Serenity codebase and swap out Casper for any consensus algorithm – Tendermint, HoneyBadgerBFT, subjective consensus or even plain old proof of work; we welcome research in this direction and aim for maximum flexibility.

这应该会让你更容易用其它的共识算法 - Tendermint, [HoneyBadgeBFT](https://github.com/amiller/HoneyBadgerBFT), [subjective consensus](http://arxiv.org/abs/1501.06238)甚至是普通的PoW - 替换掉Serenity代码中的Casper。我们非常欢迎这个方向的研究，并且希望能做到最大的灵活。

## Abstraction and Storage
## 存储的抽象

Currently, the “state” of the Ethereum system is actually quite complex and includes many parts:

目前，以太坊系统的“状态”数据实际上相当复杂，包括这些部分：

* Balance, code, nonce and storage of accounts
* Gas limit, difficulty, block number, timestamp
* The last 256 block hashes
* During block execution, the transaction index, receipt tree and the current gas used

* 余额，代码，nonce，和账户存储区
* Gas上限，难度，块高度，时间戳
* 最后256个块的hash值
* 在执行区块内代码时，需要保存交易索引，收据树(receipt tree, receipt是EVM中的一个概念)和当前消耗的gas。

These data structures exist in various places, including the block state transition function, the state tree, the block header and previous block headers. In Serenity, this will be simplified greatly: although many of these variables will still exist, they will all be moved to specialized contracts in storage; hence, the ONLY concept of “state” that will continue to exist is a tree, which can mathematically be viewed as a mapping {address: {key: value} }. Accounts will simply be trees; account code will be stored at key "" for each account (not mutable by SSTORE), balances will be stored in a specialized “ether contract” and sequence numbers will be left up to each account to determine how to store. Receipts will also be moved to storage; they will be stored in a “log contract” where the contents get overwritten every block.

这些数据结构存在于许多地方，包括块状态转移函数，状态树，区块头和前一个区块头中。在Serenity里面这些将被大幅简化：虽然许多数据仍然会存在，但他们会被转移到特殊的合约中去；因此，**唯一的**”“状态”将以一棵树的形式继续存在，数学上可以看作是形如`{address: {key: value}}`的映射。账户将是一些树，账户合约代码会被存放在主键(key)为`""`的地方（`SSTORE`不可以修改），余额会存在特别的“以太币合约”中，而序号将由每一个帐号自己决定如何保存。收据也将被转移到合约存储区，他们会被保存在一个内容在每个区块都会被覆盖的“日志合约”中。

This allows the State object in implementations to be simplified greatly; all that remains is a two-level map of tries. The scalability upgrade may increase this to three levels of tries (shard ID, address, key) but this is not yet determined, and even then the complexity will be substantially smaller than today.

这样代码实现中的状态对象可以极大的简化。现在只剩下一个两级的trie了。可伸缩性方面的升级可能要求增加为三级trie（分片ID, 地址，主键），这还没有确定，但即使是这样复杂性也远低于现在。

Note that the move of ether into a contract does NOT constitute total ether abstraction; in fact, it is arguably not that large a change from the status quo, as opcodes that deal with ether (the value parameter in CALL, BALANCE, etc) still remain for backward-compatibility purposes. Rather, this is simply a reorganization of how data is stored.

需要注意，把以太币转移进一个合约管理并不是以太币抽象的全部。事实上，一个有争议的看法是相对于现状这并不是一个很大的进步，因为为了向前兼容那些和以太币相关的操作码（带value参数的`CALL`，`BALANCE`等等）依然保留着。某种程度上说，这只是数据存放的一次重组。

## Future Plans
## 未来的计划

For POC2, the plan is to take abstraction even further. Currently, substantial complexity still remains in the block and transaction-level state transition function (eg. updating receipts, gas limits, the transaction index, block number, stateroots); the goal will be to create an “entry point” object for transactions which handles all of this extra “boilerplate logic” that needs to be done per transaction, as well as a “block begins” and “block ends” entry point. A theoretical ultimate goal is to come up with a protocol where there is only one entry point, and the state transition function consists of simply sending a message from the zero address to the entry point containing the block contents as data. The objective here is to reduce the size of the actual consensus-critical client implementation as much as possible, pushing a maximum possible amount of logic directly into Ethereum code itself; this ensures that Ethereum’s multi-client model can continue even with an aggressive development regime that is willing to accept hard forks and some degree of new complexity in order to achieve our goals of transaction speed and scalability without requiring an extremely large amount of ongoing development effort and security auditing.

在第二个概念原型中，我们计划让抽象更进一步。目前区块和交易级别的状态转移函数依然有相当的复杂性（例如更新收据，gas限制，交易索引，区块高度，状态根节点），我们的目标是为交易创建一个“入口”对象来处理所有这些每一个交易都需要的额外的“样板逻辑”，以及“块开始”和“块结束”的入口。理论上的终极目标，是找到一个只有一个入口点的协议，这样状态转移函数只需要从零地址发送一条包含区块内容数据的消息给入口点即可。这样做的目的是尽可能的减少**客户端实现的共识关键部分（consensus-critical client implementation）**的大小，把尽可能多的逻辑推到以太坊自身上去。这样即使为了达到我们对交易速度和可伸缩性的目标，我们需要采用一个接受硬分叉和一定的新复杂度的激进开发制度，也能够个确保以太坊的多重客户端形态可以持续而无需大量额外开发工作和安全审计。

In the longer term, I intend to continue producing proof-of-concepts in python, while the Casper team works together on improving the efficiency and proving the safety and correctness of the protocol; at some point, the protocol will be mature enough to handle a public testnet of some form, possibly (but not certainly) with real value on-chain in order to provide stronger incentives for people to try to “hack” Casper they way that we inevitably expect that they will once the main chain goes live. This is only an initial step, although a very important one as it marks the first time when the research behind proof of stake and abstraction is finally moving from words, math on whiteboards and blog posts into a working implementation written in code.

长期来看，我打算继续在python上开发概念原型，Casper团队则共同改进协议的效率，并证明它的安全性和正确性。在某个时刻，这个协议将成熟到足以处理一个公开的某种形式的测试网络，其上可能会有真实的价值，为人们找出Casper的漏洞提供激励，就像一条真正的链不可避免的遭受的那样。这只是第一步，不过是非常重要的一步，它标志着我们对于权益证明和深度抽象的研究终于从谈话，白板上的数学公式和博客文章变成了能工作的代码。

The next part of this series will discuss the other flagship feature of Serenity, the Casper consensus algorithm.

这个系列的下一篇文章将会讨论Serenity的另一个旗舰特性，Casper共识算法。
