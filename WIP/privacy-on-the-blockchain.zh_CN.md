# Privacy on the Blockchain
# 区块链上的隐私

[Original post](https://blog.ethereum.org/2016/01/15/privacy-on-the-blockchain/) by Vitalik Buterin, January 15th, 2016

Blockchains are a powerful technology, as regular readers of the blog already likely agree. They allow for a large number of interactions to be codified and carried out in a way that greatly increases reliability, removes business and political risks associated with the process being managed by a central entity, and reduces the need for trust. They create a platform on which applications from different companies and even of different types can run together, allowing for extremely efficient and seamless interaction, and leave an audit trail that anyone can check to make sure that everything is being processed correctly.

**正如本博客的固定读者已经肯定的那样，区块链是一种强大的技术。它可以把大量的人际互动变成代码，并以一种可以极大增加可靠性的方式执行，从而去除与中心化方式执行相伴的商业和政治风险，降低我们对信任的需要。它创造了一个可以运行不同公司甚至不同类型应用的平台，由此带来极高的效率和无缝对接，同时会保留可审计的记录，任何人都可以查阅并确信一切都以正确的方式被处理。**

However, when I and others talk to companies about building their applications on a blockchain, two primary issues always come up: scalability and privacy. Scalability is a serious problem; current blockchains, processing 3-20 transactions per second, are several orders of mangitude away from the amount of processing power needed to run mainstream payment systems or financial markets, much less decentralized forums or global micropayment platforms for IoT. Fortunately, there are solutions, and we are actively working on implementing a roadmap to making them happen. The other major problem that blockchains have is privacy. As seductive as a blockchain’s other advantages are, neither companies or individuals are particularly keen on publishing all of their information onto a public database that can be arbitrarily read without any restrictions by one’s own government, foreign governments, family members, coworkers and business competitors.

然而，当我和其他人与公司商谈如何在区块链上打造他们的应用时，经常会遇到两个问题：可伸缩性和隐私。可伸缩性是一个严重的问题；当前的区块链技术每秒钟只能处理3-20个交易，这比运行一个主流支付系统或是金融市场所需要的处理能力小了好几个数量级，更是远小于去中心化的论坛或是物联网的全球微支付平台。幸运的是，[解决方案是存在的](https://docs.google.com/presentation/d/1CjD0W4l4-CwHKUvfF5Vlps76fKLEC6pIwu1a_kC_YRQ/edit#slide=id.p)，我们也正在积极的[制定一个实现它的路线图](https://www.reddit.com/r/ethereum/comments/40u54x/eip_105_serenity_binary_sharding_plus_contract/)。区块链的另一个主要问题则是隐私。与区块链其他优点的巨大吸引力不同，没有公司或者是个人想要把他们的信息发布到一个可以被自己的政府，外国政府，家庭成员，同事和竞争对手不受节制任意读取的公开数据库之上。

Unlike with scalability, the solutions for privacy are in some cases easier to implement (though in other cases much much harder), many of them compatible with currently existing blockchains, but they are also much less satisfying. It’s much harder to create a “holy grail” technology which allows users to do absolutely everything that they can do right now on a blockchain, but with privacy; instead, developers will in many cases be forced to contend with partial solutions, heuristics and mechanisms that are designed to bring privacy to specific classes of applications.

与可伸缩性不同的是，隐私问题的解决方案在某些场景下很容易实现（不过在其他场景下则要难的多），而且基本与现有的区块链技术兼容，但是这些方案远不能满足用户需求。想要找到既能实现用户在当前区块链上能做的一切事情又能保护隐私的技术“圣杯”非常难。于是，大多数时候开发者被迫满足于局部解决方案，试错和某种机制为特定类型的应用提供隐私保护。

## The Holy Grail
## 圣杯 (The Holy Grail)

First, let us start off with the technologies that are holy grails, in that they actually do offer the promise of converting arbitrary applications into fully privacy-preserving applications, allowing users to benefit from the security of a blockchain, using a decentralized network to process the transactions, but “encrypting” the data in such a way that even though everything is being computed in plain sight, the underlying “meaning” of the information is completely obfuscated.

首先，让我们从技术的“圣杯”开始，它们可以把任意应用都变成具有完全隐私保护的应用，让用户既能享受到区块链使用去中心化网络处理交易的安全性，又可以将数据“加密”，虽然所有数据都会在众目暌暌下被处理，但数据所包含的内在“意义”是被**彻底混淆**的。

The most powerful technology that holds promise in direction is, of course, cryptographically secure obfuscation. In general, obfuscation is a way of turning any program into a “black box” equivalent of the program, in such a way that the program still has the same “internal logic”, and still gives the same outputs for the same inputs, but it’s impossible to determine any other details about how the program works.

在这个方向上最有希望的当然是加密安全混淆技术。大体上说，混淆是指把一个程序转换成一个“黑盒”运行的等价程序，这个新程序保留了原有的“内在逻辑”，对于同样的输入会给出同样的输出，但其他人没有办法找出程序的运行细节。

![obfuscation](privacy-on-the-blockchain/obfuscation.png)

Think of it as “encrypting” the wires inside of the box in such a way that the encryption cancels itself out and ultimately has no effect on the output, but does have the effect of making it absolutely impossible to see what is going on inside.

*我们可以把混淆想象成对盒子内部的线路进行“加密”，最后这些加密以某种方式相互抵消，对于输出不产生影响，但是副作用是使得盒子内部的工作机制变得难以辨别。*

Unfortunately, absolutely perfect black-box obfuscation is mathematically known to be impossible; it turns out that there is always at least something that you can get extract out of a program by looking at it beyond just the outputs that it gives on a specific set of inputs. However, there is a weaker standard called indistinguishability obfuscation that we can satisfy: essentially, given two equivalent programs that have been obfuscated using the algorithm (eg. x = (a + b) * c and x = (a * c) + (b * c)), one cannot determine which of the two outputs came from which original source. To see how this is still powerful enough for our applications, consider the following two programs:

不幸的是，我们已经知道绝对完美的黑盒混淆在数学上是[不可能的](https://www.iacr.org/archive/crypto2001/21390001.pdf)；你总是可以通过检查特定输入产生的输出和其他数据从程序中提取出**某些东西**。然而，我们可以做到满足一个被称为[不可辨别混淆](https://eprint.iacr.org/2013/451.pdf)的弱一点的标准：给出两个通过该算法混淆过的**等价**程序（例如，`x = (a + b) * c`和`x = (a * c) + (b * c)`），我们无法分辨哪个混淆版本对应于哪个原始版本。下面让我们来看看为什么这个性质依然足够有用。考虑下面两段程序：

1. `y = 0`
2. `y = sign(privkey, 0) - sign(privkey, 0)`

One just returns zero, and the other uses an internally contained private key to cryptographically sign a message, does that same operation another time, subtracts the (obviously identical) results from each other and returns the result, which is guaranteed to be zero. Even though one program just returns zero, and the other contains and uses a cryptographic private key, if indistinguishability is satisfied then we know that the two obfuscated programs cannot be distinguished from each other, and so someone in possession of the obfuscated program definitely has no way of extracting the private key – otherwise, that would be a way of distinguishing the two programs. That’s some pretty powerful obfuscation right there – and for about two years we’ve known how to do it!

一个直接返回0, 另一个使用包含在内部的私钥对消息做加密签名，重复一次同样的操作，然后把两个（显然相等的）签名相减并返回结果，这个结果肯定也是0. 虽然一个程序直接返回0, 而另一个**包含并且使用了加密私钥**，通过不可辨别的性质我们就知道没有人能区分混淆过的两段程序，因此手上拿到这两段混淆程序的人必然无法从中提取出私钥 - 否则，这就是区分这两个程序的方法。这是一种相当有用的混淆 - 而且两年前我们就知道如何实现了！
