# A 101 Noob Intro to Programming Smart Contracts on Ethereum

[Original Post](http://consensys.github.io/developers/articles/101-noob-intro/) by Eva, With help from Consensys devs

_Some people say Ethereum is too logic-heavy and hard to use, but here’s a write-up to give you a feel for building smart contracts and applications with it. Tools, wallets, applications and the ecosystem are still in development and it’ll get easier!_

?? logic-heavy
_有人说以太坊既重逻辑又难用，因此我们(译注：指[Consensys](http://consensys.net), 下同)写了这篇文章来帮助大家学习如何利用以太坊编写智能合约和应用。这里所用到的工具，钱包，应用程序以及整个生态系统仍处于开发状态，它们将来会更加好用！_

* Part I is an overview of key terms and discusses Ethereum Clients and Smart Contract Languages.
* Part II discusses overall workflow and some current DApp Frameworks and Tools and
* Part III is a the Programming Part, a quick walkthrough of writing tests and building a DApp for a smart contract using Truffle.

* [第一部分](#第一部分. 概述)概述，讨论了关键概念，几大以太坊客户端以及写智能合约用到的编程语言。
* 第二部分讨论了总体的工作流程，以及目前流行的一些DApp框架和工具。
* 第三部分主要关于编程，我们将学习如何使用Truffle来为智能合约编写测试和构建DApp。

## Part I. Intro
## 第一部分. 概述

If you’re new to all this cryptocurrency stuff, including Bitcoin and how it works, check out the first couple chapters of Andreas Antonopoulos’ Bitcoin Book to dip your toe in the water. Then head over to the Ethereum Whitepaper.

如果你对诸如比特币以及其工作原理等密码学货币的概念完全陌生，我们建议你先看看Andreas Antonopoulos所著的[Bitcoin Book](https://github.com/aantonop/bitcoinbook)的头几章，然后读一下[以太坊白皮书](https://github.com/ethereum/wiki/wiki/White-Paper)。(译注：以太坊白皮书中文版请看 http://ethfans.org/posts/ethereum-whitepaper)

If you start getting into some murky sections and would rather build something to get familiar first, then just read on. You don’t have to understand all the crypto economic computer science to start building, and a lot of that paper is about Ethereum’s improvements over Bitcoin’s architecture.

如果你觉得白皮书中的章节太晦涩，也可以直接动手来熟悉以太坊。在以太坊上做开发并不要求你理解所有那些“密码经济计算机科学”(crypto economic computer science)，而白皮书的大部分是关于以太坊想对于比特币架构上的改进。(译者受不了了注：那你写这么多干嘛！#@*&*$~!!！)

### Starter Tutorials
### 新手教程

The official place to start is ethereum.org which has a starter tutorial with follow-up token and crowdsale tutorials. There’s also the official Solidity docs. Another good place to start with smart contracts (where I started) is dappsForBeginners, although it might be outdated.

[ethereum.org](http://ethereum.org)提供了官方的新手入门教程，以及一个代币合约和众筹合约的教程。合约语言Solidity也有[官方文档](https://ethereum.github.io/solidity/)。学习智能合约的另一份不错的资料（也是我的入门资料）是[dappsForBeginners](https://dappsforbeginners.wordpress.com/)，不过现在可能有些过时了。

The goal of this write-up is complement those tutorials and introduce some basic developer tools that make starting out with Ethereum, smart contracts and building DApps (decentralized apps) easier. And to try to explain the overall flow of what’s going on. This is from my (still-noob) perspective and with much help from the cool developers at ConsenSys.

这篇文章的目的是成为上述资料的补充，同时介绍一些基本的开发者工具，使入门以太坊，智能合约以及构建DApps(decentralized apps, 分布式应用)更加容易。我会试图按照我自己(依然是新手)的理解来解释工作流程中的每一步是在做什么，我也得到了ConsenSys酷酷的开发者们的许多帮助。

### Basic Concepts
### 基本概念

It’d be good to know some of these terms:
了解这些名词是一个不错的开始：

**Public Key Cryptography.** Alice has a public key and private key. She can use her private key to create a digital signature, and Bob can use Alice’s public key to verify that a signature is really from Alice’s private key, i.e., really from Alice. When you create an Ethereum or Bitcoin wallet the long ‘0xdf…5f’ address is a public key and the private key is stored somewhere. A Bitcoin wallet service like Coinbase stores your wallet’s complementary private key for you, or you can store it yourself. If you lose your private key for a wallet with real funds you’ll lose all your funds forever, so it’s good to back up your keys. It hurts to learn this the hard way! I’ve done it.

**公钥加密系统。** Alice有一把公钥和一把私钥。她可以用她的私钥创建数字签名，而Bob可以用她的公钥来验证这个签名确实是用Alice的私钥创建的，也就是说，确实是Alice的签名。当你创建一个以太坊或者比特币钱包的时候，那长长的`0xdf...5f`地址实质上是个公钥，对应的私钥保存某处。类似于Coinbase的在线钱包可以帮你保管私钥，你也可以自己保管。如果你弄丢了存有资金的钱包的私钥，你就等于永远失去了那笔资金，因此你最好对私钥做好备份。过来人表示：通过踩坑学习到这一点是非常痛苦的...

**Peer-to-Peer Networking.** Like BitTorrent, all Ethereum nodes are peers in a distributed network, there’s no centralized server. [In the future, there’ll be hybrid semi-centralized services for Ethereum as a convenience to users and developers, more on that later.]

**点对点网络。** 就像BitTorrent, 以太坊分布式网络中的所有节点都地位平等，没有中心服务器。(未来会有半中心化的混合型服务出现为用户和开发者提供方便，这我们后面会讲到。)

**Blockchain.** Like a global ledger or simple database of all transactions, the entire history of all transactions on the network.
**区块链。** 区块链就像是一个全球唯一的帐簿，或者说是数据库，记录了网络中所有交易历史。

**Ethereum Virtual Machine.** So you can write more powerful programs than on top of Bitcoin. It refers to the blockchain, what executes smart contracts, everything.
**以太坊虚拟机(EVM)。** 它让你能在以太坊上写出更强大的程序（比特币上也可以写脚本程序）。它有时也用来指以太坊区块链，负责执行智能合约以及一切。

**Node.** Using this to mean you can run a node and through it read and write to the Ethereum blockchain, i.e., use the Ethereum Virtual Machine. A full node has to download the entire blockchain. Light nodes are possible but in the works.

**节点。** 你可以运行节点，通过它读写以太坊区块链，也即使用以太坊虚拟机。完全节点需要下载整个区块链。轻节点仍在开发中。

**Miner.** A node on the network that mines, i.e., works to process blocks on the blockchain. You can see a partial list of live Ethereum miners here: stats.ethdev.com.

**矿工。** 挖矿，也就是处理区块链上的区块的节点。这个网页可以看到当前活跃的一部分以太坊矿工：[stats.ethdev.com](http://stats.ethdev.com)。

**Proof of Work.** Miners compete to do some math problem. The first one to solve the problem (the next block on the Blockchain) wins a reward: some ether. Every node then updates to that new block. Every miner wants to win the next new block so are incentivized to keep up to date and have the one true blockchain everybody else has, so the network always achieves consensus. [Note: Ethereum is planning to move to a Proof of Stake system without miners eventually, but that’s beyond noob scope.]

**工作量证明。** 矿工们总是在竞争解决一些数学问题。第一个解出答案的(算出下一个区块)将获得以太币作为奖励。然后所有节点都更新自己的区块链。所有想要算出下一个区块的矿工都有与其他节点保持同步，并且维护同一个区块链的动力，因此整个网络总是能达成共识。(注意：以太坊正计划转向没有矿工的权益证明系统(POS)，不过那不在本文讨论范围之内。)

**Ether.** Or ETH for short. It’s a real digital currency you can buy and use! Here’s a chart from one of several exchanges for it. At the time of writing, 1 ETH is worth about 65 cents in USD.

**以太币。** 缩写ETH。一种你可以购买和使用的真正的数字货币。这里是可以交易以太币的其中一家交易所的[走势图](https://poloniex.com/exchange#usdt_eth)。在写这篇文章的时候，1个以太币价值65美分。

**Gas.** Running and storing things on Ethereum costs small amounts of ether. Keeps things efficient.

**Gas. (汽油)** 在以太坊上执行程序以及保存数据都要消耗一定量的以太币，Gas是以太币转换而成。这个机制用来保证效率。

**DApp.** Decentralized App, what applications using smart contracts are called in the Ethereum community. The goal of a DApp is (well, should be) to have a nice UI to your smart contracts plus any extra niceties like IPFS (a neat way to store and serve stuff in a decentralized network, not made by Ethereum but a kindred spirit). While DApps can be run from a central server if that server can talk to an Ethereum node, they can also be run locally on top of any Ethereum node peer. [Take a minute: unlike normal webapps, DApps may not be served from a server. They may use the blockchain to submit transactions and retrieve data (important data!) rather than a central database. Instead of a typical user login system, users may be represented by a wallet addresses and keep any user data local. Many things can be architected differently from the current web.]

**DApp.** 以太坊社区把基于智能合约的应用称为去中心化的应用程序(Decentralized App)。DApp的目标是(或者应该是)让你的智能合约有一个友好的界面，外加一些额外的东西，例如IPFS（可以存储和读取数据的去中心化网络，不是出自以太坊团队但有类似的精神)。DApp可以跑在一台能与以太坊节点交互的中心化服务器上，也可以跑在任意一个以太坊平等节点上。(花一分钟思考一下：与一般的网站不同，DApp不能跑在普通的服务器上。他们需要提交交易到**区块链**并且从**区块链**而不是中心化数据库读取**重要**数据。相对于典型的用户登录系统，用户有可能被表示成一个钱包地址而其它用户数据保存在本地。许多事情都会与目前的web应用有不同架构。)

For another noob angle on some of the concepts above here’s a good read: Just Enough Bitcoin for Ethereum.

如果想看看从另一个新手视角怎么理解这些概念，请读[Just Enough Bitcoin for Ethereum](https://medium.com/@ConsenSys/time-sure-does-fly-ed4518792679)。
