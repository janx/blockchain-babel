
Ethereum Dev Update 2015 / Week 41

以太坊开发更新 2015年/第41周

Posted by Taylor Gerring on October 13th, 2015.

In an effort to bring the community more information about the going-ons at Ethereum, we’re planning to release semi-weekly updates of project progress. I hope this provides some high-level information without having to live on GitHub.

为了让社区更好地了解以太坊的开发进度，我们计划每半周对开发进度做一次更新。我希望这能为社区成员提供有用信息。

The Go and C++ teams have daily standups and are making regular progress along with several other improvements around the ecosystem. Concurrent with the, the teams are gearing up for DEVCON1, preparing talks and presentations along with finalizing more details in the coming weeks. If you haven’t already registered, visit https://devcon.ethereum.org/ to join the fun next month in London. Now on to the updates…

Go和C++开发团队有每日站立会议（daily standups），围绕着生态系统进行正常开发。现在开发团队正在为以太坊开发者会议（DEVCON1)做准备，准备演讲和展示。如果你还没有登记参加会议，在这里https://devcon.ethereum.org/登记参加将于下个月在伦敦举行的会议。

Mist

我们希望在下周发布Geth或者Eth客户端的图形用户界面。开发团队正在修改bug，为不同的平台提供二进制文件。

Hoping to have GUI wallet next week with chocie of Geth or Eth client. The team is spending time polishing remaining bugs and bundling binaries for the various platforms.

Light Client

轻客户端
团队正在开发LES子协议，构建原型。第一版只是一个开始，团队会继续开发新特性，从而使得轻客户端可以更加可靠地支持移动和嵌入设备。为提高同步区块速度所做的工作将为轻客户端提供帮助。

The LES subprotocol being developed and prototyped into the clients. First release is only a start and work will continue on light-client features into the future, more robustly supporting mobile and embedded scenarios. Separately, work on fast-syncing will help aid in this transition.

Solidity/Mix

许多改进已经被推送到Christian的browser-solidity项目。Mix用户界面也在不断优化中。

Improvements to passing certain types in Solidity has been added along with improving exception handling throughout. Multiple improvements have been pushed to Christian’s browser-solidity project. Continued refinement is also being put into the Mix UI.

cpp-ethereum

C++团队目前正在开发1.0版，希望尽快推出候选版本。重构错误报告机制和

The team is currently working towards 1.0 release and hoping to have new RC available soon. Refactoring of the error reporting mechanism (“Exceptions”) and the ability for libraries to access storage of the calling contract by passing storage reference types is now possible. Additionally, the build system being overhauled with a switch to Jenkins.

go-etheruem

Go团队正在开发Geth 1.3版，这个版本主要的改进是内部API的改进和提速，为将来的新特性做好准备。团队为Window端的编译 能够加快区块同步的代码已经被加入到客户端中，使得全节点客户端可以更快地与网络同步。

Continuing working towards Geth 1.3 release, which will consist primarily of internal API improvements and speedups in preparation for future features. More work being put into Windows builds, hoping to have source-built scripts available soon. Fast-sync code being added to client, allowing full-nodes to syncronize to the network much quicker.

