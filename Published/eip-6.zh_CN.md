# EIP 6

### Title
### Title

      EIP: 6
      Title: Renaming SUICIDE opcode
      Author: Hudson Jameson <hudson@hudsonjameson.com>
      Status: Draft
      Type: Standards Track
      Layer: Applications
      Created: 2015-11-22

      编号: 6
      标题: 重命名SUICIDE操作码
      作者: Hudson Jameson <hudson@hudsonjameson.com>
      状态: 草案
      类型: Standards Track
      层级: 应用
      创建日期: 2015-11-22

### Abstract
### 概要

The solution proposed in this EIP is to change the name of the `SUICIDE` opcode in Ethereum programming languages with `SELFDESTRUCT`.

这份EIP提议将以太坊编程语言中的`SUICIDE`操作码名字改为`SELFDESTRUCT`。

### Motivation
### 动机

Mental health is a very real issue for many people and small notions can make a difference. Those dealing with loss or depression would benefit from not seeing the word suicide in our a programming languages. By some estimates, 350 million people worldwide suffer from depression. The semantics of Ethereum's programming languages need to be reviewed often if we wish to grow our ecosystem to all types of developers.

精神健康是许多人都面临的真实问题，有时候细小的概念差别足以造成不同的结果。在我们的编程语言中避免suicide(自杀)这个词的出现，对于正在面对失去亲人或者抑郁症的人们是有益处的。有人估计，全世界有三亿五千万人正遭受抑郁的折磨。如果我们希望以太坊生态系统有各种各样的开发者加入，以太坊编程语言的语义需要更多的检查和反思。

An Ethereum security audit commissioned by DEVolution, GmbH and [performed by Least Authority](https://github.com/LeastAuthority/ethereum-analyses/blob/master/README.md) recommended the following:

在委托DEVolution, GmbH和[Least Authority](https://github.com/LeastAuthority/ethereum-analyses/blob/master/README.md)进行的一次以太坊安全审计中，提出了如下建议：

> Replace the instruction name "suicide" with a less connotative word like "self-destruct", "destroy", "terminate", or "close", especially since that is a term describing the natural conclusion of a contract.

> 将指令名"suicide"(自杀)替换为一个隐含意思更少的单词，例如"self-destruct"(自毁), "destroy"(毁灭), "terminate"(终止), 尤其是"close"(关闭), 因为它是描述合约自然结束的一个术语。

The primary reason for us to change the term suicide is to show that people matter more than code and Ethereum is a mature enough of a project to recognize the need for a change. Suicide is a heavy subject and we should make every effort possible to not affect those in our development community who suffer from depression or who have recently lost someone to suicide. Ethereum is a young platform and it will cause less headaches if we implement this change early on in it's life.

我们想要改变suicide这个词的最主要目的，是想表示人比代码更重要，以及以太坊是一个足够成熟的项目，可以认识到自身需要改变。自杀是一个沉重的话题，我们应该尽全力保护我们的开发者社区中那些受到抑郁困扰或者还沉浸在身边亲人自杀带来的痛苦中的人。以太坊是一个年轻的平台，越早改变，伤害越少。

### Implementation
### 实现

`SELFDESTRUCT` is added as an alias of `SUICIDE` opcode (rather than replacing it).

新增了`SELFDESTRUCT`作为`SUICIDE`操作码的别名（不是替换）。

https://github.com/ethereum/solidity/commit/a8736b7b271dac117f15164cf4d2dfabcdd2c6fd
https://github.com/ethereum/serpent/commit/1106c3bdc8f1bd9ded58a452681788ff2e03ee7c
