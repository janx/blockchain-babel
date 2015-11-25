# On Abstraction

[Original post](https://blog.ethereum.org/2015/07/05/on-abstraction/) by Vitalik Buterin on July 5th, 2015

_Special thanks to Gavin Wood, Vlad Zamfir, our security auditors and others for some of the thoughts that led to the conclusions described in this post_

_特别感谢Gavin Wood, Vlad Zamfir, 我们的安全审计专家，和所有为本文中描述到的结论贡献了想法的人们。_

One of Ethereum’s goals from the start, and arguably its entire raison d’être, is the high degree of abstraction that the platform offers. Rather than limiting users to a specific set of transaction types and applications, the platform allows anyone to create any kind of blockchain application by writing a script and uploading it to the Ethereum blockchain. This gives an Ethereum a degree of future-proof-ness and neutrality much greater than that of other blockchain protocols: even if society decides that blockchains aren’t really all that useful for finance at all, and are only really interesting for supply chain tracking, self-owning cars and self-refilling dishwashers and playing chess for money in a trust-free form, Ethereum will still be useful. However, there still are a substantial number of ways in which Ethereum is not nearly as abstract as it could be.

以太坊项目最初的目标之一，甚至可说是她存在的全部意义，在于这个平台提供的高度抽象。相对于限制用户使用特定的交易类型和应用，这个平台允许任何人通过编写脚本然后上传到以太坊区块链的方式，来构建任何种类的区块链应用。这给以太坊带来了远超其它区块链协议的未来适应度和中立性：即使区块链最终被认为对于金融不是那么有用，即使区块链只被用于供应链追踪，"自有"汽车，自补充洗碗机或者是免信任的带赌注象棋游戏，以太坊依然有用武之地。然而，以太坊在许多方面还远没有抽象到她应有的程度。

## Cryptography
## 密码学

Currently, Ethereum transactions are all signed using the ECDSA algorithm, and specifically Bitcoin’s secp256k1 curve. Elliptic curve signatures are a popular kind of signature today, particularly because of the smaller signature and key sizes compared to RSA: an elliptic curve signature takes only 65 bytes, compared to several hundred bytes for an RSA signature. However, it is becoming increasingly understood that the specific kind of signature used by Bitcoin is far from optimal; ed25519 is increasingly recognized as a superior alternative particularly because of its simpler implementation, greater hardness against side-channel attacks and faster verification. And if quantum computers come around, we will likely have to move to Lamport signatures.

目前，以太坊中的交易都是通过[ECDSA算法](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm)(椭圆曲线数字签名算法)签名的，更具体地说采用了在比特币中使用的secp256k1曲线。椭圆曲线签名是现在最流行的一种签名算法，很大程度上是因为其相对于RSA算法更小的签名和密钥长度：一个椭圆曲线签名只需要65字节，而RSA签名则是数百字节。但是人们越来越认识到比特币使用的这种签名远不是最优方案，越来越多的人认为[ed25519](http://ed25519.cr.yp.to/ed25519-20110926.pdf)算法会是一个更好的选择，因为它有[更简单的实现](https://github.com/vbuterin/ed25519/blob/master/ed25519.py)，更好的旁路攻击抗性，以及更快的验证速度。如果量子计算机问世，我们则更可能转向[Lamport签名算法](https://bitcoinmagazine.com/6021/bitcoin-is-not-quantum-safe-and-how-we-can-fix/)。

One suggestion that some of our security auditors, and others, have given us is to allow ed25519 signatures as an option in 1.1. But what if we can stay true to our spirit of abstraction and go a bit further: let people use whatever cryptographic verification algorithm that they want? Is that even possible to do securely? Well, we have the ethereum virtual machine, so we have a way of letting people implement arbitrary cryptographic verification algorithms, but we still need to figure out how it can fit in.

我们的安全审计专家和一些朋友给出过的一个建议是，在1.1版本中加入ed25519签名算法作为可选项。但是我们是否能忠于我们的抽象精神再走远一步：让人们可以选择他们想要的任何密码学验证算法呢？有可能安全的做到吗？没错，我们有以太坊虚拟机，这让人们可以实现任意的密码学验证算法，但我们还需要弄明白**如何**让这些算法融入到系统中。

Here is a possible approach:

一个可能的方法是：

1. Every account that is not a contract has a piece of “verification code” attached to it.
2. When a transaction is sent, it must now explicitly specify both sender and recipient.
3. The first step in processing a transaction is to call the verification code, using the transaction’s signature (now a plain byte array) as input. If the verification code outputs anything nonempty within 50000 gas, the transaction is valid. If it outputs an empty array (ie. exactly zero bytes; a single \x00 byte does not count) or exits with an exception condition, then it is not valid.
4. To allow people without ETH to create accounts, we implement a protocol such that one can generate verification code offline and use the hash of the verification code as an address. People can send funds to that address. The first time you send a transaction from that account, you need to provide the verification code in a separate field (we can perhaps overload the nonce for this, since in all cases where this happens the nonce would be zero in any case) and the protocol (i) checks that the verification code is correct, and (ii) swaps it in (this is roughly equivalent to “pay-to-script-hash” in Bitcoin).

1. 每一个非合约账户都可以附带一段“验证程序代码”。
2. 交易发送出去的时候，必须显式的指明发送者和接受者。
3. 处理交易的第一步是调用验证程序，用交易的签名（现在是一个普通的字节数组了）作为输入。如果验证程序在50000 gas之内产生了任何非空的输出，这个交易就是有效的。如果输出为空（也就是说0个字节，一个`\x00`字节不算）或者程序抛异常退出，则交易无效。
4. 为了让没有以太币的用户也可以创建账户，我们让协议支持用户离线生成一段验证程序并且使用这段程序的哈希值作为地址。人们可以向这个地址发送资金。当用户第一次从这个账户发起交易的时候，他需要另外输入对应的验证程序代码（???），而协议会 (i)检查验证程序正确（对应哈希值), (ii)将其作为此账户的验证程序代码（大致上等价于比特币中的"[pay-to-script-hash](https://en.bitcoin.it/wiki/Pay_to_script_hash)"机制)。

This approach has a few benefits. First, it does not specify anything about the cryptographic algorithm used or the signature format, except that it must take up at most 50000 gas (this value can be adjusted up or down over time). Second, it still keeps the property of the existing system that no pre-registration is required. Third, and quite importantly, it allows people to add higher-level validity conditions that depend on state: for example, making transactions that spend more GavCoin than you currently have actually fail instead of just going into the blockchain and having no effect.

这个方法有几个好处。首先，它没有指定使用**任何**密码学算法或是签名格式，只要求这个算法最多消耗50000 gas（这个数值可以随时间上下调整）。其次，它保留了现有系统无需预先注册的性质。第三也是很重要的一点，它允许人们加入更高层面的依赖状态(译注: state, 指区块链上保存的数据)的有效性条件：例如一个花费超过你当前持有的GavCoin的交易直接是无效的，而不是进入区块链但不产生效果。

However, there are substantial changes to the virtual machine that would have to be made for this to work well. The current virtual machine is designed well for dealing with 256-bit numbers, capturing the hashes and elliptic curve signatures that are used right now, but is suboptimal for algorithms that have different sizes. Additionally, no matter how well-designed the VM is right now, it fundamentally adds a layer of abstraction between the code and the machine. Hence, if this will be one of the uses of the VM going forward, an architecture that maps VM code directly to machine code, applying transformations in the middle to translate specialized opcodes and ensure security, will likely be optimal – particularly for expensive and exotic cryptographic algorithms like zk-SNARKs. And even then, one must take care to minimize any “startup costs” of the virtual machine in order to further increase efficiency as well as denial-of-service vulnerability; together with this, a gas cost rule that encourages re-using existing code and heavily penalizes using different code for every account, allowing just-in-time-compiling virtual machines to maintain a cache, may also be a further improvement.

然而为了良好的实现这个方案虚拟机需要大量改动。现在的虚拟机被设计成能很好的处理256位的数字，很适合目前使用的哈希算法和椭圆曲线签名算法，但对于基于其它位数的算法不是最优的。此外，
