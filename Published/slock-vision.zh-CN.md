# Slock.it - Decentralizing the Emerging Sharing Economy
# Slock - 区块锁，去中心化的共享经济

*[区块锁Slock](http://slock.it)是本届[DEVCON](https://www.youtube.com/watch?v=49wHQoJxYPo)上关注度最高的项目之一。本文原文于DEVCON之前发表在[slock.it官方博客](http://slock.it/index.php?c=20150901_vision#0_blog)上。*

For this first blog post, I will describe the vision we have for Slock.

在这第一篇博文中，我想要描述一下Slock的愿景。

With Uber, Airbnb, and others leading the way, we have to ask ourselves, “Is this how we want to build the sharing economy?” Monopolistic companies that take extraordinary fees and have full control of the market? I have to admit, I use them and appreciate the progress they are making in bringing awareness to the sharing economy, but I would prefer a system that would allow me to deal directly with the owner without any third-parties skimming off the top. But how do we know whether we can trust the other party without an intermediary? Who handles a secure money transfer? How do we handover keys (I have used Airbnb in NYC with the key under the door mat… is that really the best we can do)? How do owners and renters find each other?

随着Uber, Airbnb以及其它风口上公司的不断前进，我们不禁要问问自己，这就是我们想要的共享经济吗？收取着高额手续费，完全控制了市场的垄断性公司？我必须承认，我也是他们的用户，而且很感激他们为共享经济所带来的关注，但是我更想要的是一个让我可以不被剥削直接与产权人打交道的系统。问题是在没有中间人的情况下我们如何信任对方呢？谁来保证转账安全？我们要怎样交付钥匙（我曾经在纽约用过一次Airbnb，从门口地毯下摸到了钥匙... 这是最佳的解决方案吗）？如何让产权人和租赁人找到对方？

These problems can now be solved using smart contracts on a blockchain. A blockchain is a decentralized cryptographically-secured database, which allows anyone in the world to access it and send transactions to it, safely changing its state without having to get permission! Slock uses Ethereum (ethereum.org) for that. Ethereum expands upon the blockchain idea with the concept of self-enforcing smart contracts. Users of Slock interact with each other through this contract: https://github.com/slockit/smart-contract/blob/master/BasicSlock.sol and because of the rules in the Ethereum protocol, this contract will always do exactly what it is programmed to do and cannot be cheated (I will write another post covering the details of the Slock contract). So we, as the creators of Slock, do not need to run servers or handle transactions. We don't need to transfer money or manage the handover of keys. All of this will be handled by the Ethereum blockchain.

这些问题都可以由区块链上的智能合约来解决。区块链是一个去中心化的由密码学保证安全的数据库，世界上的任何人都可以访问，在上面发送交易，安全的改变其状态而无需事先获得许可！Slock使用以太坊作为区块链层。以太坊扩展了区块链技术，加入了“自实施”的智能合约概念。Slock的用户即通过这个智能合约互相交互： https://github.com/slockit/smart-contract/blob/master/BasicSlock.sol，由以太坊协议定下的规则保证这份合约永远会精确按照程序代码执行而不会作弊（我将会另外写一篇文章来解释Slock合约的细节）。我们作为Slock的创造者，也不需要运行什么服务器或者处理交易。我们不需要进行转账或者交付钥匙。所有这些都会由以太坊区块链处理。

When someone purchases a Slock, it will be connected to the Slock smart contract in the Ethereum blockchain and controlled by it. The owner of a Slock can set a deposit amount and a price for renting his property, and the user will pay that deposit through a transaction to the Ethereum blockchain (without us), thereby getting permission to open and close that smart lock through their smart phone. The deposit will be locked in the Ethereum blockchain until the user decides to return the virtual key by sending another transaction to the Ethereum blockchain. Then the contract will be automatically enforced. The deposit will be sent back to the user minus the price for the rental which will be automatically sent to the owner of the Slock. All of this happens without any assistance from a third-party!

当有人购买一把区块锁时，它会连接到以太坊区块链上的区块锁智能合约并受其控制。区块锁的所有者可以给区块锁所控制的资产设置预付款额度和租赁价格，用户则通过以太坊区块链发送一笔交易（不经过我们）支付预付款，获得通过智能手机控制锁打开或者关闭的使用权。在用户决定通过以太坊发送另外一笔交易归还这把虚拟钥匙之前，这笔预付款会被锁定在以太坊区块链上。收到归还钥匙的交易之后，区块锁智能合约会自动执行，将租赁费用转账给出租人，将余款返还给租赁人。一切这些都在没有第三方协助的情况下进行！

Since the data for each Slock is publically available on the blockchain, any web service can scan it and offer matchmaking. The current model of giving all power to one company because they have all the data will be broken. The data will now be there for any blockchain explorer to use.

由于所有区块锁的数据都可以在区块链上找到，任何人都可以扫描区块链信息并提供配对服务。一家公司因为拥有数据而掌握所有权力的现有模式将被打破。数据将可以被任何区块链浏览者使用。

Therefore, Slock can be described as an open platform for others to use and build upon. It can be used in a completely decentralized fashion, connecting the renters directly to the owners of the property they are renting.

因此，区块锁Slock可以说是一个开放平台，供所有人使用和开发。它以一种完全去中心化的方式运作，将租赁人和产权人直接连接起来。

What exactly are we doing? First, we will build the foundation for this platform by deploying the smart contract on the Ethereum blockchain and selling the hardware that makes use of it. Second, we will provide services such as a "Slock Blockchain Explorer", a way to use traditional payment methods (credit cards, PayPal, etc.) to buy ether which will be used in the contract, and customized solutions for various businesses (hotels, bike outlets, tool rentals...). However, users do not need to rely on our services to use the underlying platform.

那么我们具体要怎样做？首先，我们会构建这个平台的基础，把区块锁智能合约部署到以太坊区块链上，销售使用这个合约的硬件产品。第二，我们会提供包括“区块锁浏览器”在内的一些服务，帮助用户通过传统支付手段（信用卡，PayPal, 等等）来购买使用平台需要的以太币，同时为各种不同的场景（宾馆，自行车出租，工具出租...）提供定制方案。而且用户要使用底层的平台都不需要依赖我们。

We will offer hardware and software to make use of this system and encourage others to build upon this standard as well. Imagine the possibility to efficiently and securely rent out your flat, house, bike, car, washing machine, lawn mower, etc. just by locking it with a Slock. Contract enforcement and payments will all be handled automatically.

我们会提供使用这个系统所需的硬件和软件，也鼓励其他人基于这个平台来提供服务。想象一下，仅仅通过安装区块锁Slock，就可以安全高效的出租你的公寓，房子，自行车，汽车，洗衣机，剪草机等等等等合，而合约的履行和支付都会被自动处理，未来会是怎样的可能？

Christoph Jentzsch
