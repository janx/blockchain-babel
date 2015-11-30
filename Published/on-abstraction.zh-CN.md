# On Abstraction

[Original post](https://blog.ethereum.org/2015/07/05/on-abstraction/) by Vitalik Buterin on July 5th, 2015

_Special thanks to Gavin Wood, Vlad Zamfir, our security auditors and others for some of the thoughts that led to the conclusions described in this post_

_特别感谢Gavin Wood, Vlad Zamfir, 我们的安全审计专家，和所有为本文中描述到的结论贡献了想法的人们。_

One of Ethereum’s goals from the start, and arguably its entire raison d’être, is the high degree of abstraction that the platform offers. Rather than limiting users to a specific set of transaction types and applications, the platform allows anyone to create any kind of blockchain application by writing a script and uploading it to the Ethereum blockchain. This gives an Ethereum a degree of future-proof-ness and neutrality much greater than that of other blockchain protocols: even if society decides that blockchains aren’t really all that useful for finance at all, and are only really interesting for supply chain tracking, self-owning cars and self-refilling dishwashers and playing chess for money in a trust-free form, Ethereum will still be useful. However, there still are a substantial number of ways in which Ethereum is not nearly as abstract as it could be.

以太坊项目最初的目标之一，甚至可说是她存在的全部意义，在于这个平台提供的高度抽象。相对于限制用户使用特定的交易类型和应用，这个平台允许任何人通过编写脚本然后上传到以太坊区块链的方式，来构建任何种类的区块链应用。这给以太坊带来了远超其它区块链协议的未来适应度和中立性：即使区块链最终被认为对于金融不是那么有用，即使区块链只被用于供应链追踪，"自有"汽车，自补充洗碗机或者是免信任的带赌注象棋游戏，以太坊依然有用武之地。然而，以太坊在许多方面还远没有抽象到她应有的程度。

## Cryptography
## 密码学签名

Currently, Ethereum transactions are all signed using the ECDSA algorithm, and specifically Bitcoin’s secp256k1 curve. Elliptic curve signatures are a popular kind of signature today, particularly because of the smaller signature and key sizes compared to RSA: an elliptic curve signature takes only 65 bytes, compared to several hundred bytes for an RSA signature. However, it is becoming increasingly understood that the specific kind of signature used by Bitcoin is far from optimal; ed25519 is increasingly recognized as a superior alternative particularly because of its simpler implementation, greater hardness against side-channel attacks and faster verification. And if quantum computers come around, we will likely have to move to Lamport signatures.

目前，以太坊中的交易都是通过[ECDSA算法](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm)(椭圆曲线数字签名算法)签名的，更具体地说采用了在比特币中使用的secp256k1曲线。椭圆曲线签名是现在最流行的一种签名算法，很大程度上是因为其相对于RSA算法更小的签名和密钥长度：一个椭圆曲线签名只需要65字节，而RSA签名则是数百字节。但是人们越来越认识到比特币使用的这种签名远不是最优方案，越来越多的人认为[ed25519](http://ed25519.cr.yp.to/ed25519-20110926.pdf)算法会是一个更好的选择，因为它有[更简单的实现](https://github.com/vbuterin/ed25519/blob/master/ed25519.py)，更好的旁路攻击抗性，以及更快的验证速度。如果量子计算机问世，我们则更可能转向[Lamport签名算法](https://bitcoinmagazine.com/6021/bitcoin-is-not-quantum-safe-and-how-we-can-fix/)。

One suggestion that some of our security auditors, and others, have given us is to allow ed25519 signatures as an option in 1.1. But what if we can stay true to our spirit of abstraction and go a bit further: let people use whatever cryptographic verification algorithm that they want? Is that even possible to do securely? Well, we have the ethereum virtual machine, so we have a way of letting people implement arbitrary cryptographic verification algorithms, but we still need to figure out how it can fit in.

我们的安全审计专家和一些朋友给出过的一个建议是，在1.1版本中加入ed25519签名算法作为可选项。但是我们是否能忠于我们的抽象精神再走远一步：让人们可以选择他们想要的任何密码学签名算法呢？有可能安全的做到吗？没错，我们有以太坊虚拟机，这让人们可以实现任意的密码学签名算法，但我们还需要弄明白**如何**让这些算法融入到系统中。

Here is a possible approach:

一个可能的方法是：

1. Every account that is not a contract has a piece of “verification code” attached to it.
2. When a transaction is sent, it must now explicitly specify both sender and recipient.
3. The first step in processing a transaction is to call the verification code, using the transaction’s signature (now a plain byte array) as input. If the verification code outputs anything nonempty within 50000 gas, the transaction is valid. If it outputs an empty array (ie. exactly zero bytes; a single \x00 byte does not count) or exits with an exception condition, then it is not valid.
4. To allow people without ETH to create accounts, we implement a protocol such that one can generate verification code offline and use the hash of the verification code as an address. People can send funds to that address. The first time you send a transaction from that account, you need to provide the verification code in a separate field (we can perhaps overload the nonce for this, since in all cases where this happens the nonce would be zero in any case) and the protocol (i) checks that the verification code is correct, and (ii) swaps it in (this is roughly equivalent to “pay-to-script-hash” in Bitcoin).

1. 每一个非合约账户都可以附带一段“验证程序代码”。
2. 交易发送出去的时候，必须显式的指明发送者和接受者。
3. 处理交易的第一步是调用验证程序，用交易的签名（现在是一个普通的字节数组了）作为输入。如果验证程序在50000 gas之内产生了任何非空的输出，这个交易就是有效的。如果输出为空（也就是说0个字节，一个`\x00`字节不算）或者程序抛异常退出，则交易无效。
4. 为了让没有以太币的用户也可以创建账户，我们让协议支持用户离线生成一段验证程序并且使用这段程序的哈希值作为地址。人们可以向这个地址发送资金。当用户第一次从这个账户发起交易的时候，他需要另外输入对应的验证程序代码（或许可以重用nonce字段来实现, 因为可以知道这种情况下nonce都应该是0），而协议会 (i)检查验证程序正确（对应哈希值), (ii)将其作为此账户的验证程序代码（大致上等价于比特币中的"[pay-to-script-hash](https://en.bitcoin.it/wiki/Pay_to_script_hash)"机制)。

This approach has a few benefits. First, it does not specify anything about the cryptographic algorithm used or the signature format, except that it must take up at most 50000 gas (this value can be adjusted up or down over time). Second, it still keeps the property of the existing system that no pre-registration is required. Third, and quite importantly, it allows people to add higher-level validity conditions that depend on state: for example, making transactions that spend more GavCoin than you currently have actually fail instead of just going into the blockchain and having no effect.

这个方法有几个好处。首先，它没有指定使用**任何**密码学算法或是签名格式，只要求这个算法最多消耗50000 gas（这个数值可以随时间上下调整）。其次，它保留了现有系统无需预先注册的性质。第三也是很重要的一点，它允许人们加入更高层面的依赖状态(译注: state, 指区块链上保存的数据)的有效性条件：例如一个花费超过你当前持有的GavCoin的交易直接是无效的，而不是进入区块链但不产生效果。

However, there are substantial changes to the virtual machine that would have to be made for this to work well. The current virtual machine is designed well for dealing with 256-bit numbers, capturing the hashes and elliptic curve signatures that are used right now, but is suboptimal for algorithms that have different sizes. Additionally, no matter how well-designed the VM is right now, it fundamentally adds a layer of abstraction between the code and the machine. Hence, if this will be one of the uses of the VM going forward, an architecture that maps VM code directly to machine code, applying transformations in the middle to translate specialized opcodes and ensure security, will likely be optimal – particularly for expensive and exotic cryptographic algorithms like zk-SNARKs. And even then, one must take care to minimize any “startup costs” of the virtual machine in order to further increase efficiency as well as denial-of-service vulnerability; together with this, a gas cost rule that encourages re-using existing code and heavily penalizes using different code for every account, allowing just-in-time-compiling virtual machines to maintain a cache, may also be a further improvement.

然而为了良好的实现这个方案虚拟机需要大量改动。现在的虚拟机被设计成能很好的处理256位的数字，很适合目前使用的哈希算法和椭圆曲线签名算法，但对于非256位的算法不是最优的。此外，无论虚拟机设计的多么好，本质上它终究是在字节码和机器码中间加了一层。因此，如果这是未来的虚拟机需要处理的场景，一个可以将虚拟机字节码直接映射为机器码，并且在此过程中通过特殊转换保证安全性的架构可能是最好的 - 尤其适合zk-SNARKs这类费时又奇特的算法。在此之上，我们还需要注意减少虚拟机的“启动成本”，以进一步提高效率，同时避免拒绝服务攻击(denial-of-service vulnerability)；最后，增加一条鼓励重用已有代码和重罚给每一个账户使用不同代码的gas消耗规则，会有利于虚拟机做即时编译优化，也是进一步优化的手段。

## The Trie
## The Trie

(译注：Trie是一种数据结构，又可称作前缀树或是字典树，本文中保留不译。)

Perhaps the most important data structure in Ethereum is the Patricia tree. The Patricia tree is a data structure that, like the standard binary Merkle tree, allows any piece of data inside the trie to be securely authenticated against a root hash using a logarithmically sized (ie. relatively short) hash chain, but also has the important property that data can be added, removed or modified in the tree extremely quickly, only making a small number of changes to the entire structure. The trie is used in Ethereum to store transactions, receipts, accounts and particularly importantly the storage of each account.

以太坊中最终要的数据结构也许就是[Patricia Tree](https://github.com/ethereum/wiki/wiki/Patricia-Tree)了。(译注：Patricia tree是基于trie的一种数据结构。这里Vitalik谈论的实际上是Merkle Patricia Tree, Patricia Tree的一种变体。下同。) Patricia Tree是一种类似标准Merkle二叉树的数据结构，可以通过对数大小(也就是说，相对很小)的哈希链可靠的验证验证树中任意数据，更重要的是还能非常迅速的进行插入，删除和更新操作，这些操作仅对树做很小的变动。以太坊用这种Trie来保存交易，交易收据，账户信息和账户持久数据。

One of the often cited weaknesses of this approach is that the trie is one particular data structure, optimized for a particular set of use cases, but in many cases accounts will do better with a different model. The most common request is a heap: a data structure to which elements can quickly be added with a priority value, and from which the lowest-priority element can always be quickly removed – particularly useful in implementations of markets with bid/ask offers.

这种实现常被提及的一个弱点是，Trie是一种很特殊的数据结构，为一组特殊的场景优化，而很多时候另外的数据结构更适合。最常见的需求是堆（heap）：一种可以快速插入带有权重节点的数据结构，支持快速的移除权重最低的元素 - 这个性质在实现交易撮合的时候特别有用。

Right now, the only way to do this is a rather inefficient workaround: write an implementation of a heap in Solidity or Serpent on top of the trie. This essentially means that every update to the heap requires a logarithmic number of updates (eg. at 1000 elements, ten updates, at 1000000 elements, twenty updates) to the trie, and each update to the trie requires changes to a logarithmic number (once again ten at 1000 elements and twenty at 1000000 elements) of items, and each one of those requires a change to the leveldb database which uses a logarithmic-time-updateable trie internally. If contracts had the option to have a heap instead, as a direct protocol feature, then this overhead could be cut down substantially.

当前，得到一个堆的唯一方法是一种很没效率的变通(workaround)：用Solidity或是Serpent语言[在trie的基础上实现一个堆](https://github.com/ethereum/serpent/blob/develop/examples/cyberdyne/heap.se). 这代表着对heap的每一次更新操作都会带来对下层trie的对数级次更新（eg. 对于1000个元素的堆，带来10次更新，对于1000000个元素的堆，20次更新），而对于下层trie的每次更新操作又需要更新对数级个元素（也就是同样的，对于1000个元素的trie，需要更新10个元素，对于1000000个元素的trie, 需要更新20个），最后每一个trie元素的更新又导致下层的leveldb数据库去更新其内部的，更新操作同样需要对数时间复杂度的，trie. 如果以太坊提供原生实现的堆做为选项供合约使用，这种开销可以极大的降低。

One option to solve this problem is the direct one: just have an option for contracts to have either a regular trie or a heap, and be done with it. A seemingly nicer solution, however, is to generalize even further. The solution here is as follows. Rather than having a trie or a treap, we simply have an abstract hash tree: there is a root node, which may be empty or which may be the hash of one or more children, and each child in turn may either be a terminal value or the hash of some set of children of its own. An extension may be to allow nodes to have both a value and children. This would all be encoded in RLP; for example, we may stipulate that all nodes must be of the form:

解决此问题的一个直接方法是：给合约提供是使用标准trie或是堆的选项，结束。然而进一步通用化看上去会是更好的解决方案。该方案如下。与其使用trie或者堆，我们仅提供一个抽象哈希树：它有一个根节点，可以是空的或是其一个或多个子节点的哈希值，而每一个子节点又可以是终点或是其子节点的哈希值。允许节点既有自己的值又有子节点可以是一个扩展。这些都会编码为RLP格式（译注: Recursive Length Prefix, 以太坊使用的序列化格式），比如，我们可以规定所有节点都必须表示为：

```
[val, child1, child2, child3....]
```

Where val must be a string of bytes (we can restrict it to 32 if desired), and each child (of which there can be zero or more) must be the 32 byte SHA3 hash of some other node. Now, we have the virtual machine’s execution environment keep track of a “current node” pointer, and add a few opcodes:

这里`val`必须是一个字节串（如有需要可以限制长度为32），每一个childX（可以有0个或是多个）必须是某个子节点的SHA3散列值。然后我们让虚拟机执行环境记住一个“当前节点”的指针，并增加一些指令：


* GETVAL: pushes the value of the node at the current pointer onto the stack
* SETVAL: sets the value at the of the node at the current pointer to the value at the top of the stack
* GETCHILDCOUNT: gets the number of children of the node
* ADDCHILD: adds a new child node (starting with zero children of its own)
* REMOVECHILD: pops off a child node
* DESCEND: descend to the kth child of the current node (taking k as an argument from the stack)
* ASCEND: ascend to the parent
* ASCENDROOT: ascend to the root node

* `GETVAL`: 将当前节点的值压入栈
* `SETVAL`: 将当前节点的值设为栈顶值
* `GETCHILDCOUNT`: 获取子节点数量
* `ADDCHILD`: 增加一个子节点（从0个子节点开始）
* `REMOVECHILD`: 弹出一个子节点
* `DESCEND`: 下指到当前节点的第k个子节点（参数k从栈中取得）
* `ASCEND`: 上指到父节点
* `ASCENDROOT`: 上指到根节点

Accessing a Merkle tree with 128 elements would thus look like this:

一个包含128个元素的Merkle Tree的读取看起来就像这样：

```
def access(i):
    ~ascendroot()
    return _access(i, 7)

def _access(i, depth):
    while depth > 0:
        ~descend(i % 2)   
        i /= 2
        depth -= 1
    return ~getval()
```

Creating the tree would look like this:

树的构造看起来像这样：

```
def create(vals):
    ~ascendroot()
    while ~getchildcount() > 0:
        ~removechild()
    _create(vals, 7)

def _create(vals:arr, depth):
    if depth > 0:
        # Recursively create left child
        ~addchild()
        ~descend(0)
        _create(slice(vals, 0, 2**(depth - 1)), depth - 1)
        ~ascend()
        # Recursively create right child
        ~addchild()
        ~descend(1)
        _create(slice(vals, 2**(depth - 1), 2**depth), depth - 1)
        ~ascend()
    else:
        ~setval(vals[0])
```


Clearly, the trie, the treap and in fact any other tree-like data structure could thus be implemented as a library on top of these methods. What is particularly interesting is that each individual opcode is constant-time: theoretically, each node can keep track of the pointers to its children and parent on the database level, requiring only one level of overhead.

显然，无论是trie, 堆还是其它任何类似树的数据结构，都可以实现为基于这些指令的库。这里最有意思的是每一条指令都是常数时间复杂度的：理论上，每个节点都可以记下指向其子节点和父节点的数据库层的指针，只需增加一层开销。

However, this approach also comes with flaws. Particularly, note that if we lose control of the structure of the tree, then we lose the ability to make optimizations. Right now, most Ethereum clients, including C++, Go and Python, have a higher-level cache that allows updates to and reads from storage to happen in constant time if there are multiple reads and writes within one transaction execution. If tries become de-standardized, then optimizations like these become impossible. Additionally, each individual trie structure would need to come up with its own gas costs and its own mechanisms for ensuring that the tree cannot be exploited: quite a hard problem, given that even our own trie had a medium level of vulnerability until recently when we replaced the trie keys with the SHA3 hash of the key rather than the actual key. Hence, it’s unclear whether going this far is worth it.

然而这个方案也有其缺陷。特别需要注意的是如果我们失去对树的结构的控制，我们也就失去了做优化的能力。目前大多数以太坊客户端，包括C++, Go和Python，都有一个高层缓存，可以将涉及存储层多次读写的交易中的读写操作优化到常数时间复杂度。如果trie不再是标准数据结构，类似的优化将成为不可能。此外，每一个不同的trie都需要对应的gas消耗规则，以及确保其不被攻击利用的机制：这是个非常难的问题，即使是我们自己的trie实现，在最近用key的SHA3散列值代替实际值做key的改动之前，也是一个中等级别的漏洞。因此，目前还不清楚这个抽象是否值得去做。

# Currency
# 货币

It’s well-known and established that an open blockchain requires some kind of cryptocurrency in order to incentivize people to participate in the consensus process; this is the kernel of truth behind this otherwise rather silly meme:

一个公开的区块链需要密码学货币来激励用户参与共识过程，这已经是一个被广泛接受的观点了，正如这张搞笑图片所表达的：

![bitcoin-is-not-useful.jpg](on-abstraction/btc-not-useful.jpg)

However, can we create a blockchain that does not rely on any specific currency, instead allowing people to transact using whatever currency they wish? In a proof of work context, particularly a fees-only one, this is actually relatively easy to do for a simple currency blockchain; just have a block size limit and leave it to miners and transaction senders themselves to come to some equilibrium over the transaction price (the transaction fees may well be done as a batch payment via credit card). For Ethereum, however, it is slightly more complicated. The reason is that Ethereum 1.0, as it stands, comes with a built-in gas mechanism which allows miners to safely accept transactions without fear of being hit by denial-of-service attacks; the mechanism works as follows:

问题是，我们是否可以创造一个不依赖任何**特定货币**的区块链，让人们可以用任何他们想用的货币进行交易呢？在工作量证明，尤其是只有交易费奖励的环境下，这是相对容易实现的。只需要定下一个区块大小限制，然后让矿工和交易发送者自己就交易价格达到某种均衡即可（交易费用或许可以通过信用卡批量支付）。对于以太坊来说则有些复杂。原因是目前的以太坊1.0版本引入了一个燃料机制(gas)，来帮助矿工可以安全的接受交易而无须担心拒绝服务攻击的风险。这个机制是这样工作的：

1. Every transaction specifies a max gas count and a fee to pay per unit gas.
2. Suppose that the transaction allows itself a gas limit of N. If the transaction is valid, and takes less than N computational steps (say, M computational steps), then it pays M steps worth of the fee. If the transaction consumes all N computational steps before finishing, the execution is reverted but it still pays N steps worth of the fee.

1. 每一个交易都指定好gas消耗上限，以及每单位gas的价格。
2. 假设这个交易的gas上限是N. 如果交易是有效的，而且只用了不到N的计算量（设用掉的计算量为M），那么它要支付计算量M对应的手续费。如果交易在处理结束之前用光了N单位的计算量，执行被回滚，而且依然要支付计算量N对应的手续费。

This mechanism relies on the existence of a specific currency, ETH, which is controlled by the protocol. Can we replicate it without relying on any one particular currency? As it turns out, the answer is yes, at least if we combine it with the “use any cryptography you want” scheme above. The approach is as follows. First, we extend the above cryptography-neutrality scheme a bit further: rather than having a separate concept of “verification code” to decide whether or not a particular transaction is valid, simply state that there is only one type of account – a contract, and a transaction is simply a message coming in from the zero address. If the transaction exits with an exceptional condition within 50000 gas, the transaction is invalid; otherwise it is valid and accepted. Within this model, we then set up accounts to have the following code:

这个机制依赖于受协议控制的特定货币ETH（以太币）的存在。我们是否可以不依赖任何特定货币实现这个机制呢？答案是可以的，至少我们可以把它和上文提到的“使用任意密码学算法”的方案相结合。方法如下。首先，我们稍微扩展一下上面的具有密码学算法中立性的方案：与其用一个新概念“验证程序”来决定某个交易是否有效，不如只保留一种账户类型 - 合约账户，此时交易可以简单看作是从零地址发出的一条消息。如果交易执行在50000 gas的额度下异常中止，它就是无效的；否则就是有效的并且被接受。在这个模型中，我们的账户遵循以下规则：

1. Check if the transaction is correct. If not, exit. If it is, send some payment for gas to a master contract that will later pay the miner.
2. Send the actual message.
3. Send a message to ping the master contract. The master contract then checks how much gas is left, and refunds a fee corresponding to the remaining amount to the sender and sends the rest to the miner.

1. 检查交易是否正确。如果不是，退出。如果是，向一个主合约（master contract）支付一些gas费用，主合约之后会支付矿工。
2. 发送实际的消息。
3. 发消息通知主合约。然后主合约会检查还有多少gas剩下，把余额退还给交易发起人，其余的支付给矿工。

Step 1 can be crafted in a standardized form, so that it clearly consumes less than 50000 gas. Step 3 can similarly be constructed. Step 2 can then have the message provide a gas limit equal to the transaction’s specified gas limit minus 100000. Miners can then pattern-match to only accept transactions that are of this standard form (new standard forms can of course be introduced over time), and they can be sure that no single transaction will cheat them out of more than 50000 steps of computational energy. Hence, everything becomes enforced entirely by the gas limit, and miners and transaction senders can use whatever currency they want.

第一步可以有一个标准形式，以便清楚的表明所需的gas不到50000。第三步也可以这样构造。第二步中消息的gas消耗上限可以等于交易指定上限减去100000。然后矿工们就可以只接受与标准形式模式匹配的交易（当然新的标准形式可以随着时间慢慢增加），而且他们也能确定没有任何一笔交易可以消耗超过50000单位的计算量。于是，所有东西都通过gas上限来控制，而矿工和交易发起人可以使用他们喜欢的任何货币。

One challenge that arises is: how do you pay contracts? Currently, contracts have the ability to “charge” for services, using code like this registry example:

一个冒出来的挑战是：如果为合约使用进行支付呢？目前，合约可以为提供的服务“收费”，代码类似下面的域名注册示例：

```
def reserve(_name:bytes32):
    if msg.value > 100 * 10**18:
        if not self.domains[_name].owner:
            self.domains[_name].owner = msg.sender
```

如果有多种货币，就没有一个清晰的机制可以把一条消息和它的支付联系起来。不过有两种常用设计模式可以做为替代。第一种是一种“收据”接口：当你支付货币给他人时，你可以要求货币合约保存付款人和支付数额信息。类似于`registrar.reserve("blahblahblah.eth")`的代码则会变成：

```
gavcoin.sendWithReceipt(registrar, 100 * 10**18)
registrar.reserve("blahblahblah.eth")
```

The currency would have code that looks something like this:

这个货币的合约会有类似这样的代码：

```
def sendWithReceipt(to, value):
    if self.balances[msg.sender] >= value:
        self.balances[msg.sender] -= value
        self.balances[to] += value
        self.last_sender = msg.sender
        self.last_recipient = to
        self.last_value = value

def getLastReceipt():
    return([self.last_sender, self.last_recipient, self.value]:arr)
```

And the registrar would work like this:

注册代码则是这样的：

```
def reserve(_name:bytes32):
    r = gavcoin.getLastReceipt(outitems=3)
    if r[0] == msg.sender and r[1] == self and r[2] >= 100 * 10**18:
        if not self.domains[_name].owner:
            self.domains[_name].owner = msg.sender
```

Essentially, the registrar would check the last payment made in that currency contract, and make sure that it is a payment to itself. In order to prevent double-use of a payment, it may make sense to have the get_last_receipt method destroy the receipt in the process of reading it.

基本上注册合约会检查货币合约中的最后一笔支付，确保支付对象是自己。为了防止重复使用同一笔支付，在读收据数据时让`get_last_receipt`方法销毁收据也许是有必要的。

The other pattern is to have a currency have an interface for allowing another address to make withdrawals from your account. The code would then look as follows on the caller side: first, approve a one-time withdrawal of some number of currency units, then reserve, and the reservation contract attempts to make the withdrawal and only goes forward if the withdrawal succeeds:

另一个设计模式是让货币合约提供一个允许其它地址从你的账户取款的接口。从调用者的角度来看代码会是这样：首先，批准一笔一定数额的一次性取款，然后调用`reserve`，然后注册合约尝试取款，如果成功则继续处理：

```
gavcoin.approveOnce(registrar, 100)
registrar.reserve("blahblahblah.eth")
```

And the registrar would be:

注册合约会变成：

```
def reserve(_name:bytes32):
    if gavcoin.sendCoinFrom(msg.sender, 100, self) == SUCCESS:
        if not self.domains[_name].owner:
            self.domains[_name].owner = msg.sender
```

The second pattern has been standardized at the Standardized Contract APIs wiki page.

第二种设计模式已经被标准化并记录在[标准合约API文档](https://github.com/ethereum/wiki/wiki/Standardized_Contract_APIs)中。

# Currency-agnostic Proof of Stake
# 货币无关的权益证明

The above allows us to create a completely currency-agnostic proof-of-work blockchain. However, to what extent can currency-agnosticism be added to proof of stake? Currency-agnostic proof of stake is useful for two reasons. First, it creates a stronger impression of economic neutrality, which makes it more likely to be accepted by existing established groups as it would not be seen as favoring a particular specialized elite (bitcoin holders, ether holders, etc). Second, it increases the amount that will be deposited, as individuals holding digital assets other than ether would have a very low personal cost in putting some of those assets into a deposit contract. At first glance, it seems like a hard problem: unlike proof of work, which is fundamentally based on an external and neutral resource, proof of stake is intrinsically based on some kind of currency. So how far can we go?

上述方案给我们带来了一个完全与特定货币无关的基于工作量证明的区块链。可是在权益证明机制中特定货币无关性能做到何种程度呢？基于两个原因，与特定货币无关的权益证明机制会很有用。首先，它给人的经济中立的感觉更强烈，这会让已经存在的利益团体更容易接受，因为它不在倾向于一群特定的精英人群（比特币持有者，以太币持有者，等等）。第二，它会增加共识保证金的数量，因为持有以太币之外的数字资产的个人能以极低的成本把一部分资产转入保证金合约。粗看起来，这似乎是个很难解决的问题：与工作量证明从根本上以外部和中立的资源为基础不同，权益证明机制本质上基于内部的某种货币。这种条件下我们能走多远呢？

The first step is to try to create a proof of stake system that works using any currency, using some kind of standardized currency interface. The idea is simple: anyone would be able to participate in the system by putting up any currency as a security deposit. Some market mechanism would then be used in order to determine the value of each currency, so as to estimate the amount of each currency that would need to be put up in order to obtain a stake depositing slot. A simple first approximation would be to maintain an on-chain decentralized exchange and read price feeds; however, this ignores liquidity and sockpuppet issues (eg. it’s easy to create a currency and spread it across a small group of accounts and pretend that it has a value of $1 trillion per unit); hence, a more coarse-grained and direct mechanism is required.

第一步是尝试定义某种标准化的货币合约接口，来创造一个支持任意货币的权益证明系统。想法很简单：任何人可以使用任意货币作为保证金来参与系统。然后通过某种市场机制来决定每一种货币的价值有多少，用来估算以某货币作为保证金的话需要多少数量。一个简单的近似方案是维护一个链上的去中心化交易所并且读取其价格数据，然而这种方案忽略了流动性和“马甲”问题（例如，容易创建一个货币并将其分配个一小群人，然后把它的价格造假到一万亿美元）。因此，我们需要一个更粗粒度和直接的机制。

To get an idea of what we are looking for, consider David Friedman’s description of one particular aspect of the ancient Athenian legal system:

为了更好的了解我们所追求的目标，让我们来看看[David Friedman](https://en.wikipedia.org/wiki/David_D._Friedman)对古代雅典人法律体系的[一段描述](http://daviddfriedman.com/Academic/Course_Pages/Legal_Systems_Very_Different_13/Book_Draft/Systems/AthenianChapter.html)：

> The Athenians had a straightforward solution to the problem of producing public goods such as the maintainance of a warship or the organizing of a public festival. If you were one of the richest Athenians, every two years you were obligated to produce a public good; the relevant magistrate would tell you which one.
> “As you doubtless know, we are sending a team to the Olympics this year. Congratulations, you are the sponsor.”
> Or
> “Look at that lovely trireme down at the dock. This year guess who gets to be captain and paymaster.”
> Such an obligation was called a liturgy. There were two ways to get out of it. One was to show that you were already doing another liturgy this year or had done one last year. The other was to prove that there was another Athenian, richer than you, who had not done one last year and was not doing one this year.
> This raises an obvious puzzle. How, in a world without accountants, income tax, public records of what people owned and what it was worth, do I prove that you are richer than I am? The answer is not an accountant’s answer but an economist’s—feel free to spend a few minutes trying to figure it out before you turn the page.
> The solution was simple. I offer to exchange everything I own for everything you own. If you refuse, you have admitted that you are richer than I am, and so you get to do the liturgy that was to be imposed on me.

> 雅典人对于诸如维护战舰或是组织公众节日这样的公众货物(public goods)的制造有一套直接的解决方案。如果你是最富有的雅典人之一，每两年就有义务制造一件公众货物，而相关的治安官会告诉你是哪一件。
> “正如你知道的，我们今年要派一队人去参加奥林匹克运动会。恭喜你，你是赞助人。”
> 或者是
> “看看码头那些可爱的三列桨战船！你猜今年由谁来做船长和军需官？”
> 这样的义务被称为liturgy, 有两种方式可以摆脱它。一种是证明你今年已经负责了另外一样liturgy或是去年做过了。另一种是证明有另外一个比你还富有的雅典人，去年和今年都没有做任何事情。
> 显然这里有个问题。在一个没有会计，所得税，公开财产和价值记录的年代，我要如何证明你比我富有？答案来自经济学家而不是会计 - 在你翻页之前可以花几分钟看你能不能想到。
> 解决方法很简单。我提议用我拥有的一切交换你拥有的一切。如果你拒绝，就等于承认你比我富有，因此你就要去负责本来分配给我的liturgy.

Here, we have a rather nifty scheme for preventing people that are rich from pretending that they are poor. Now, however, what we are looking for is a scheme for preventing people that are poor from pretending that they are rich (or more precisely, preventing people that are releasing small amounts of value into the proof of stake security deposit scheme from pretending that they are staking a much larger amount).

此时我们有了一个非常聪明的方法可以用来防止富人假装自己很穷。但是我们要追求的是一个防止穷人假装自己很富的方法（更准确的说，防止只向权益证明系统缴纳了一小笔保证金的人假装他们缴纳了一大笔钱）。

A simple approach would be a swapping scheme like that, but done in reverse via a voting mechanic: in order to join the stakeholder pool, you would need to be approved by 33% of the existing stakeholders, but every stakeholder that approves you would have to face the condition that you can exchange your stake for theirs: a condition that they would not be willing to meet if they thought it likely that the value of your stake actually would drop. Stakeholders would then charge an insurance fee for signing stake that is likely to strongly drop against the existing currencies that are used in the stake pool.

一个简单方法类似上面的交换做法，只不过通过投票机制反过来进行：为了加入参与人池，必须有超过33%的参与人批准，但每一个批准了的参与人都必须面对一个条件 - 你可以用你的保证金交换他们的。如果他们觉得你的保证金价值会下跌，他们不会愿意接受这样的条件。此时参与人可能会为批准这样一笔相对现有保证金池可能暴跌的保证金收取保险费。

This scheme as described above has two substantial flaws. First, it naturally leads to currency centralization, as if one currency is dominant it will be most convenient and safe to also stake in that currency. If there are two assets, A and B, the process of joining using currency A, in this scheme, implies receiving an option (in the financial sense of the term) to purchase B at the exchange rate of A:B at the price at the time of joining, and this option would thus naturally have a cost (which can be estimated via the Black-Scholes model). Just joining with currency A would be simpler. However, this can be remedied by asking stakeholders to continually vote on the price of all currencies and assets used in the stake pool – an incentivized vote, as the vote reflects both the weight of the asset from the point of view of the system and the exchange rate at which the assets can be forcibly exchanged.

这个方案有两处明显缺陷。第一，它会导致货币中心化的自然产生，因为一旦一种货币成为主流它就会成为最方便和安全的保证金选择。如果有两种资产，A和B，使用A参与共识，在这个方案下，意味着买入了一份以参与时A:B价格为准的B的买入期权（[金融意义](https://en.wikipedia.org/wiki/Option_%28finance%29)上的），而这份期权自然会有成本（可以通过[Black-Scholes模型](http://en.wikipedia.org/wiki/Black%E2%80%93Scholes_model)在估算）。直接使用B参与共识会更简单（译注：这里假定了B是主流货币）。不过，这一点可以通过要求参与人持续的对池中所有货币和资产的价格进行投票来补救 - 带激励的投票，因为投票既反映了从系统角度看资产的权重，也反映了资产可以被强制交易的价格。

A second, more serious flaw, however, is the possibility of pathological metacoins. For example, one can imagine a currency which is backed by gold, but which has the additional rule, imposd by the institution backing it, that forcible transfers initiated by the protocol “do not count”; that is, if such a transfer takes place, the allocation before the transfer is frozen and a new currency is created using that allocation as its starting point. The old currency is no longer backed by gold, and the new one is. Athenian forcible-exchange protocols can get you far when you can actually forcibly exchange property, but when one can deliberately create pathological assets that arbitrarily circumvent specific transaction types it gets quite a bit harder.

然而第二个，也是更严重的缺陷，在于存在病态的“超级”币(译注：metacoin, 但此处meta并不是“元”的意思）的可能性。例如，我们可以想象一种由黄金背书的货币，做背书的机构定了一条额外的规则，说共识协议发起的强制转账“不算数”，也就是说，如果这样的转账发生了，转账前的分配会被冻结，一种新的货币会基于冻结的分配产生。老的货币不再由黄金背书，新的才是。雅典人的强制交换协议只在你实际上能强制交换财产时才可行，在有人故意创造可以专断的绕过特定交易类型的病态资产时就变得很难实现。

Theoretically, the voting mechanism can of course get around this problem: nodes can simply refuse to induct currencies that they know are suspicious, and the default strategy can tend toward conservatism, accepting a very small number of currencies and assets only. Altogether, we leave currency-agnostic proof of stake as an open problem; it remains to be seen exactly how far it can go, and the end result may well be some quasi-subjective combination of TrustDavis and Ripple consensus.

理论上，投票机制应该可以绕过这个问题：节点们只要拒绝接受他们认为可疑的货币就好，而且默认的策略应该倾向于保守主义，只接受一小部分货币和资产。综上所述，我们将与特定货币无关的权益证明机制留做未解决的问题，我们还不知道在这条路上能走多远，结果也可能是TrustDavis和Ripple共识协议的某种“类主观”（quasi-subjective）组合。

# SHA3 and RLP
# SHA3和RLP

Now, we get to the last few parts of the protocol that we have not yet taken apart: the hash algorithm and the serialization algorithm. Here, unfortunately, abstracting things away is much harder, and it is also much harder to tell what the value is. First of all, it is important to note that even though we have shows how we could conceivably abstract away the trees that are used for account storage, it is much harder to see how we could abstract away the trie on the top level that keeps track of the accounts themselves. This tree is necessarily system-wide, and so one can’t simply say that different users will have different versions of it. The top-level trie relies on SHA3, so some kind of specific hashing algorithm there must stay. Even the bottom-level data structures will likely have to stay SHA3, since otherwise there would be a risk of a hash function being used that is not collision-resistant, making the whole thing no longer strongly cryptographically authenticated and perhaps leading to forks between full clients and light clients.

现在我们还没解决的只剩下协议中最后的几个部分了：哈希算法和序列化算法。不幸的是，要把它们抽象掉挺难，而且很难说能得到什么好处。首先，需要注意的是虽然我们展示了把用于账户存储的树结构抽象掉的可能方法，但是很难看出要怎么把顶层的保存账户本身的trie抽象掉。这是一个系统级别的树，因此不能简单的说让不同的用户有它不同的版本。这个顶层的trie依赖于SHA3, 因而这里必须保留某种特定的哈希算法。甚至底层的数据结构也很可能会继续使用SHA3，否则的话会有所选择的哈希函数不能抵御碰撞攻击的风险，让整个系统不再具有强密码学保证，而这也许会导致完整客户端和轻客户端之间的分叉。

RLP is similarly unavoiable; at the very least, each account needs to have code and storage, and the two need to be stored together some how, and that is already a serialization format. Fortunately, however, SHA3 and RLP are perhaps the most well-tested, future-proof and robust parts of the protocol, so the benefit from switching to something else is quite small.

RLP同样很难避免；最起码的，每个账户都需要存储代码和数据，而这两者又需要通过某种方式存在一起，这就需要一种序列化格式。幸运的是，SHA3和RLP也许是协议中测试最充分，最能适应未来变化，最强壮的部分，因此把它们替换掉能得到的好处非常小。
