# A 101 Noob Intro to Programming Smart Contracts on Ethereum

[Original Post](http://consensys.github.io/developers/articles/101-noob-intro/) by Eva, With help from Consensys devs

_Some people say Ethereum is too logic-heavy and hard to use, but here’s a write-up to give you a feel for building smart contracts and applications with it. Tools, wallets, applications and the ecosystem are still in development and it’ll get easier!_

?? logic-heavy
_有人说以太坊既重逻辑又难用，因此我们(译注：指[Consensys](http://consensys.net), 下同)写了这篇文章来帮助大家学习如何利用以太坊编写智能合约和应用。这里所用到的工具，钱包，应用程序以及整个生态系统仍处于开发状态，它们将来会更加好用！_

* Part I is an overview of key terms and discusses Ethereum Clients and Smart Contract Languages.
* Part II discusses overall workflow and some current DApp Frameworks and Tools and
* Part III is a the Programming Part, a quick walkthrough of writing tests and building a DApp for a smart contract using Truffle.

* [第一部分](#第一部分. 概述)概述，讨论了关键概念，几大以太坊客户端以及写智能合约用到的编程语言。
* [第二部分](#第二部分. DApp框架，工具以及工作流程)讨论了总体的工作流程，以及目前流行的一些DApp框架和工具。
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

### Ethereum Clients, Smart Contract Languages
### 以太坊客户端，智能合约语言

You don’t have to run an Ethereum node to write and deploy smart contracts. See Browser-based IDEs and APIs below. But if you’re learning, run an Ethereum node, it’s good to get to know as a basic component and not hard to set up.

编写和部署智能合约并不要求你运行一个以太坊节点。下面有列出[基于浏览器的IDE和API](#browser_ides)。但如果是为了学习的话，还是应该运行一个以太坊节点，以便理解其中的基本组件，何况运行节点也不难。

#### Clients for Running an Ethereum Node
#### 运行以太坊节点可用的客户端

Ethereum has several different client implementations (meaning ways to run a node to interact with the Ethereum network) including C++, Go, Python, Java, Haskell, etc. Why? Different strokes for different folks (like how the Haskell one is supposedly mathematically verifiable), and it improves the security and ecosystem of Ethereum to have so many. There’s also a gui-based client in development, AlethZero.

以太坊有许多不同语言的客户端实现（即多种与以太坊网络交互的方法），包括C++, Go, Python, Java, Haskell等等。为什么需要这么多实现？不同的实现能满足不同的需求（例如Haskell实现的目标是可以被数学验证），能使以太坊更加安全，能丰富整个生态系统。

At the time of writing, I’ve been using geth, the Go language one (go-ethereum) and on other days a tool called testrpc that uses the Python client, pyethereum. The later examples will involve those tools.

在写作本文时，我使用的是Go语言实现的客户端geth ([go-ethereum](http://ethereum.github.io/go-ethereum/))，其他时候还会使用一个叫testrpc的工具, 它使用了Python客户端[pyethereum](https://github.com/ethereum/pyethereum)。后面的例子会用到这些工具。

[Sidebar: I’ve also tried the C++ one and still use its ethminer component for mining along with geth as the node, so different pieces can work together. Sidebar on Mining: Mining can be fun, sort of like having a houseplant you tend to, and another way to learn about the ecosystem….even if the price of ETH right now is not worth the local electricity costs of mining, that may change. Especially if you start building cool DApps and Ethereum becomes more popular.]

注: 我曾经使用过C++的客户端，现在仍然在用其中的ethminer组件和geth配合挖矿，因此这些不同的组件是可以一起工作的。
关于挖矿：挖矿很有趣，有点像精心照料你的室内盆栽，同时又是一种了解整个系统的方法。虽然以太币现在的价格可能连电费都补不齐，但以后谁知道呢。人们正在创造许多酷酷的DApp, 可能会让以太坊越来越流行。

Interactive Console. Once you have a node using one of the clients, you can sync with the blockchain, create wallets and send and receive real ether. One way to do that with geth is through the JavaScript console. Another way is via JSON RPC (remote procedure calls) using a command like cURL for getting stuff via URLs. However the goal of this article is to walk you through a DApp development scenario so let’s just move on. But these tools are good to remember for debugging, configuring nodes and using a wallet via command line.

**交互式控制台。** 客户端运行起来后，你就可以同步区块链，建立钱包，收发以太币了。使用geth的一种方式是通过[Javascript控制台](https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console)（JavaScript console, 类似你在chrome浏览器里面按F12出来的那个，只不过是跑在终端里）。此外还可以使用类似cURL的命令通过[JSON RPC](https://github.com/ethereum/wiki/wiki/JSON-RPC)来与客户端交互。本文的目标是带大家过一边DApp开发的流程，因此这块就不多说了。但是我们应该记住这些命令行工具是调试，配置节点，以及使用钱包的利器。

Running a node on a test network. If you install a client like geth and run it on the live network, it will take a while to download the entire blockchain and sync with the network. (You can check that it’s synced by seeing that you have the latest block which is listed at the top of stats.ethdev.com and comparing that number to the block number output by your client node’s logs.)

However to run smart contracts on the live network you’d have to cough up some real ether. Instead there are ways to run clients on a local testnet for for free. They won’t download the full blockchain and will create a private instance of the Ethereum network with its own blockchain, so are faster to use for development.

**在测试网络运行节点。** 如果你在正式网络运行geth客户端，下载整个区块链与网络同步会需要相当时间。（你可以通过比较节点日志中打印的最后一个块号和[stats.ethdev.com](http://stats.ethdev.com)上列出的最新块来确定是否已经同步。) 另一个问题是在正式网络上跑智能合约需要实实在在的以太币。在测试网络上运行节点的话就没有这个问题。此时也不需要同步整个区块链，创建一个自己的私有链就勾了，对于开发来说更省时间。

testrpc. You can run a test network using geth, or another fast way of getting a testnet running is using testrpc. Testrpc will create a bunch of pre-funded accounts for you that will be listed when it starts up. It’s is also super fast, so easier to develop and test with. You can start with testrpc, then when your contracts are in good shape, move to geth on a testnet, which can be started by specifying a networkid like: geth --networkid “12345”. Here’s the testrpc repo but I’ll review everything you need to install again in the tutorial part later.

**testrpc.** 用geth可以创建一个测试网络，另一种更快的创建测试网络的方法是使用testrpc. Testrpc可以在启动时帮你创建一堆存有资金的测试账户。它的运行速度也更快因此更适合开发和测试。你可以从testrpc起步，然后随着合约慢慢成型，转移到geth创建的测试网络上 - 启动方法很简单，只需要指定一个networkid：`geth --networkid "12345"`。这里是[testrpc的代码仓库](https://github.com/ConsenSys/eth-testrpc)，下文我们还会再讲到它。

Let’s talk about programming languages next, then we can dive into actually coding stuff.

接下来我们来谈谈可用的编程语言，之后就可以开始真正的编程了。

#### Programming Languages for Smart Contracts
#### 写智能合约用的编程语言

Just use Solidity. To write smart contracts there are a few different languages: Solidity, which is like JavaScript and has .sol as a file extension, Serpent, Python-like with extension .se, and a 3rd, LLL, based on Lisp. Serpent was popular a while back but Solidity is the most popular right now and more robust, so just use Solidity. You prefer Python? Use Solidity.

**用Solidity就好。** 要写智能合约有好几种语言可选：有点类似Javascript的Solidity, 文件扩展名是`.sol`. 和Python接近的Serpent, 文件名以`.se`结尾。还有类似Lisp的LLL。Serpent曾经流行过一段时间，但现在最流行而且最稳定的要算是Solidity了，因此用Solidity就好。听说你喜欢Python? 用Solidity。

solc Compiler. After writing a contract in Solidity, use solc to compile it. It’s from the C++ libraries (different implementations complementing each other again) which can be installed here. [If you don’t want to install solc you can also just use a browser-based compiler like the Solidity real-time compiler or Cosmo, but the programming part later on will assume you have solc installed.]

**solc编译器。** 用Solidity写好智能合约之后，需要用solc来编译。它是一个来自C++客户端实现的组件（又一次，不同的实现产生互补），[这里](https://github.com/ethereum/webthree-umbrella/wiki)是安装方法。如果你不想安装solc也可以直接使用基于浏览器的编译器，例如[Solidity real-time compiler](https://chriseth.github.io/browser-solidity/)或者[Cosmo](http://cosmo.to/)。后文有关编程的部分会假设你安装了solc。

[Note: Ethereum’s libraries are undergoing active development and sometimes things get out of sync with new versions. Make sure you have the latest dev version, or a stable version. Ask in one of the Ethereum Gitter’s on Github or forums.ethereum.org what version people are using if things that used to work stop working.]

注意：以太坊正处于积极的开发中，有时候新的版本之间会有不同步。确认你使用的是最新的dev版本，或者稳定版本。如果遇到问题可以去以太坊项目对应的Gitter聊天室或者[forums.ethereum.org](http://forums.ethereum.org)上问问其他人在用什么版本。

web3.js API. Once a Solidity contract is compiled with solc and sent to the network, you can call it using the Ethereum web3.js JavaScript API and build web apps that interact with contracts. (No need to install this yet, read up on DApp Frameworks below first.)

**web3.js API.** 当Solidity合约编译好并且发送到网络上之后，你可以使用以太坊的[web3.js JavaScript API](https://github.com/ethereum/wiki/wiki/JavaScript-API)来调用它，构建能与之交互的web应用。

Those are the basic Ethereum tools for coding smart contracts and interacting with them to build DApps.

以上就是在以太坊上编写智能合约和构建与之交互的DApp所需的基本工具。

## Part II. DApp Frameworks, Tools and Workflow
## 第二部分. DApp框架，工具以及工作流程
