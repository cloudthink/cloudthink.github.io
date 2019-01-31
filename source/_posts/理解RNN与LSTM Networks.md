---
title: 理解RNN与LSTM神经网络
date: 2019-01-21 18:12:45
tags:
recommended_posts: true

---

## Recurrent Neural Networks
人类不会每时每刻都开始思考。当你阅读这篇文章时，你会根据你对之前单词的理解来理解每个单词。你不会立刻忘记之前读的所有东西，然后再从头开始思考。你的想法有持久性。

传统的神经网络无法做到这一点，这似乎是其一个主要的缺点。例如，假设您想要对电影中每个时间点发生的事件进行分类。目前尚不清楚传统神经网络如何利用其对电影中先前事件的记录来影响之后发生的事情。<!--more-->

循环神经网络解决了这个问题。它们是带有循环的网络，允许信息持续存在。
![递归神经网络具有循环](https://img-blog.csdnimg.cn/20190130141517792.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RhdGFwYWQ=,size_16,color_FFFFFF,t_70#pic_center =130x180)
<center>Recurrent Neural Networks的循环结构</center>
在上图中，一块神经网络A查看一些输入xt并输出一个值ht。循环允许信息从网络的一个步骤传递到下一个步骤。

这个循环使得循环神经网络看起来有点神秘。但是，如果你再想一下，事实证明它们与普通的神经网络并没有什么不同。可以将循环神经网络视为同一网络的多个副本，每个副本都将消息传递给后继者。如果把RNN神经网络展开：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190130143144185.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RhdGFwYWQ=,size_16,color_FFFFFF,t_70#pic_center =530x180)
                                             <center>展开的RNNs网络</center>

这种类似链的性质表明，递归神经网络与序列和列表密切相关。它们是用于此类数据的神经网络的自然架构。

他们肯定会被使用！在过去几年中，将RNN应用于各种问题取得了令人难以置信的成功：语音识别，语言建模，翻译，图像字幕......这个列表还在继续。我将讨论使用RNNs可以实现的惊人壮举，以及Andrej Karpathy的[博客文章](http://karpathy.github.io/2015/05/21/rnn-effectiveness/)，但他们真的很棒。

这些成功的关键在于使用“LSTM”，这是一种非常特殊的递归神经网络，对于许多任务而言，它比标准版本好得多。几乎所有基于递归神经网络的令人兴奋的结果都是用它们实现的。这篇文章将探讨这些LSTM。



## 长期依赖（Long-Term Dependencies）问题

RNN的一个吸引力是他们可能能够将先前信息连接到当前任务，例如使用先前的视频帧可能推动对当前帧的理解。如果RNN可以做到这一点，它们将非常有用。但他们可以吗？这取决于很多依赖因素。

有时，我们只需要查看最近的信息来执行当前任务。例如，考虑一种语言模型，试图根据之前的单词预测下一个单词。如果我们试图预测“the clouds are in the [sky]”的最后一个词，我们并不需要任何其他的上下文 - 很明显下一个词将是天空。在这种情况下，如果相关信息与所需信息之间的差距很小，则RNN可以学习使用过去的信息。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190130145357869.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RhdGFwYWQ=,size_16,color_FFFFFF,t_70#pic_center =430x180)

但也有一些情况需要更多的背景。考虑尝试预测文本中的最后一个词“我I grew up in France… I speak fluent French.” 最近的信息表明，下一个词可能是一种语言的名称，但如果我们想缩小具体是哪种语言的范围，我们需要从更进一步的背景来看，法国的背景。相关信息与需要变得非常大的点之间的差距是完全可能的。

不幸的是，随着差距的扩大，RNNs无法学习连接信息。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190130155041274.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RhdGFwYWQ=,size_16,color_FFFFFF,t_70#pic_center =500x200)
理论上，RNNs绝对能够处理这种“长期依赖性”。人类可以仔细挑选参数来解决这种形式的玩具问题。遗憾的是，在实践中，RNN似乎无法学习它们。 [Hochreiter（1991）\[德国\]](http://people.idsia.ch/~juergen/SeppHochreiter1991ThesisAdvisorSchmidhuber.pdf)和[Bengio 等人（1994）](http://www-dsi.ing.unifi.it/~paolo/ps/tnn-94-gradient.pdf)对该问题进行了深入探讨。他找到了一些为什么它可能很困难的根本原因。

值得庆幸的是，LSTM没有这个问题！

## LSTM Networks
长短期内存网络 - 通常只称为“LSTM” - 是一种特殊的RNN，能够学习长期依赖性。 它们是由[Hochreiter＆Schmidhuber（1997）](http://www.bioinf.jku.at/publications/older/2604.pdf)介绍的，并且在下面的工作中被许多人精炼和推广。他们在各种各样的问题上工作得非常好，现在被广泛使用。

LSTM明确旨在避免长期依赖性问题。长时间记住信息实际上是他们的默认行为，而不是他们难以学习的东西！

所有递归神经网络都具有神经网络重复模块链的形式。在标准RNN中，该重复模块将具有非常简单的结构，例如单个tanh层。


<center>The repeating module in a standard RNN contains a single layer</center>

LSTM也具有这种类似链的结构，但重复模块具有不同的结构。RNNs只有一个神经网络层，而LSTM只有四个。以一种非常特殊的方式进行交互。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190130153608998.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RhdGFwYWQ=,size_16,color_FFFFFF,t_70#pic_center =600x180)
<center>The repeating module in an LSTM contains four interacting layers</center>
不要担心发生了什么的细节。我们将逐步介绍LSTM图。现在，让我们试着对我们将要使用的符号感到满意。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190130153738310.png)
在上图中，每一行都携带一个整个向量，从一个节点的出到其他节点的输入。粉色圆圈表示逐点运算，如矢量加法，而黄色框表示神经网络层。行合并表示连接，而行分叉表示其内容被复制，副本将转移到不同的位置。
## The Core Idea Behind LSTMs
LSTM的关键是单元状态，水平线贯穿图的顶部。

电池状态有点像传送带。 它直接沿着整个链运行，只有一些次要的线性交互。 信息很容易沿着它不变地流动。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190130160057487.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RhdGFwYWQ=,size_16,color_FFFFFF,t_70)

LSTM具有删除或添加信息到细胞状态的功能，由门(gates)这种结构对这两种操作进行精确的调节。

门是一种控制信息能否通过的方式。 它们由sigmoid形神经网络层和逐点乘法运算组成。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190130160313901.png#pic_center =100x100)

sigmoid层输出0到1之间的数字，描述每个组件应该通过多少信息。 值为零意味着“不让任何东西通过”，而值为1则意味着“让一切都通过！”

LSTM具有三个这样的门，用于保护和控制细胞的状态。

## Step-by-Step LSTM Walk Through



译自[http://colah.github.io](http://colah.github.io/posts/2015-08-Understanding-LSTMs/)
