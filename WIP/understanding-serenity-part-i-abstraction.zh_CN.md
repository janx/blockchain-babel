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
## 抽象和账户

Currently, there are two types of accounts in Ethereum: externally owned accounts, controlled by a private key, and contracts, controlled by code. For externally owned accounts, we specify a particular digital signature algorithm (secp256k1 ECDSA) and a particular sequence number (aka. nonce) scheme, where every transaction must include a sequence number one higher than the previous, in order to prevent replay attacks. The primary change that we will make in order to increase abstraction is this: rather than having these two distinct types of accounts, we will now have only one – contracts. There is also a special “entry point” account, 0x0000000000000000000000000000000000000000, that anyone can send from by sending a transaction. Hence, instead of the signature+nonce verification logic of accounts being in the protocol, it is now up to the user to put this into a contract that will be securing their own account.

目前以太坊中有两种类型的账户：外部拥有的账户，由私钥控制，和合约账户，由代码控制。对于外部拥有的账户，我们指定了一种特别的数字签名算法(secp256k1椭圆曲线签名)和一个序号体系(nonce)，要求每一个交易都必须包含一个比前一个交易序号大1的序号，目的是防止重放攻击(replay attacks)。我们为提高抽象程度而做的主要改动是：不再有两种不同类型的账户，而是统一为一种 - 合约账户。将会存在一个特殊的“入口”账户，`0x0000000000000000000000000000000000000000`，**任何人**都可以从这个账户发起交易。因此，协议中将不再包含签名+序号的账户验证逻辑，用户必须用合约来保护他们自己的账户。

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

这段代码就是用户账户的合约代码；如果用户要发送一个交易，他们会发送一个交易（从地址0）到这个账户，交易数据会像上述代码所示那样包括ECDSA签名，序号，gas价格，目标地址，数额和真正的交易数据。合约代码会检查对交易gas限制和数据的签名，然后检查交易序号，如果两者都没有问题就把保存的序号加一，发送所需的消息，然后发送另一条消息支付gas费用作为结束（注意，矿工可以静态分析账户的合约代码，拒绝处理发往最后没有支付gas代码的账户的交易）。
