1. Hierarchical Softmax的缺点与改进

在讲基于Negative Sampling的word2vec模型前，我们先看看Hierarchical Softmax的的缺点。的确，使用霍夫曼树来代替传统的神经网络，可以提高模型训练的效率。
但是如果我们的训练样本里的中心词w是一个很生僻的词，那么就得在霍夫曼树中辛苦的向下走很久了。能不能不用搞这么复杂的一颗霍夫曼树，将模型变的更加简单呢？
Negative Sampling就是这么一种求解word2vec模型的方法，它摒弃了霍夫曼树，采用了Negative Sampling（负采样）的方法来求解，下面我们就来看看Negative Sampling的求解思路。

2. 基于Negative Sampling的模型概述

既然名字叫Negative Sampling（负采样），那么肯定使用了采样的方法。
比如我们有一个训练样本，中心词是w,它周围上下文共有2c个词，记为context(w)。由于这个中心词w,的确和context(w)相关存在，因此它是一个真实的正例.
通过Negative Sampling采样，我们得到neg个和w不同的中心词wi,i=1,2,..neg，这样context(w)和wi就组成了neg个并不真实存在的负例。利用这一个正例和neg个负例，我们进行二元逻辑回归，得到负采样对应每个词wi对应的模型参数θi，和每个词的词向量。
从上面的描述可以看出，Negative Sampling由于没有采用霍夫曼树，每次只是通过采样neg个不同的中心词做负例，就可以训练模型，因此整个过程要比Hierarchical Softmax简单。

3.Negative Sampling负采样方法

现在我们来看看如何进行负采样，得到neg个负例。word2vec采样的方法并不复杂，如果词汇表的大小为V,那么我们就将一段长度为1的线段分成V份，每份对应词汇表中的一个词。
当然每个词对应的线段长度是不一样的，高频词对应的线段长，低频词对应的线段短。每个词w的线段长度由下式决定：

![14](https://github.com/wonderfultina/word2vec/blob/master/images/13.png)

在word2vec中，分子和分母都取了3/4次幂如下：

![14](https://github.com/wonderfultina/word2vec/blob/master/images/14.png)


在采样前，我们将这段长度为1的线段划分成M等份，这里M>>V，这样可以保证每个词对应的线段都会划分成对应的小块。而M份中的每一份都会落在某一个词对应的线段上。
在采样的时候，我们只需要从M个位置中采样出neg个位置就行，此时采样到的每一个位置对应到的线段所属的词就是我们的负例词。

![14](https://github.com/wonderfultina/word2vec/blob/master/images/15.png)

在word2vec中，M取值默认为108。

4. 基于Negative Sampling的模型梯度计算

Negative Sampling也是采用了二元逻辑回归来求解模型参数，通过负采样，我们得到了neg个负例(context(w),wi)i=1,2,..neg。为了统一描述，我们将正例定义为w0。
在逻辑回归中，我们的正例应该期望满足：

![14](https://github.com/wonderfultina/word2vec/blob/master/images/16.png)

我们的负例期望满足：

![14](https://github.com/wonderfultina/word2vec/blob/master/images/17.png)

我们期望可以最大化下式：

![14](https://github.com/wonderfultina/word2vec/blob/master/images/18.png)

利用逻辑回归和上一节的知识，我们容易写出此时模型的似然函数为：

![14](https://github.com/wonderfultina/word2vec/blob/master/images/19.png)


此时对应的对数似然函数为：

![14](https://github.com/wonderfultina/word2vec/blob/master/images/20.png)


和Hierarchical Softmax类似，我们采用随机梯度上升法，仅仅每次只用一个样本更新梯度，来进行迭代更新得到我们需要的xwi,θwi,i=0,1,..neg, 这里我们需要求出xw0,θwi,i=0,1,..neg的梯度。


首先我们计算θwi的梯度：

![14](https://github.com/wonderfultina/word2vec/blob/master/images/21.png)

同样的方法，我们可以求出xw0的梯度如下：

![14](https://github.com/wonderfultina/word2vec/blob/master/images/22.png)

有了梯度表达式，我们就可以用梯度上升法进行迭代来一步步的求解我们需要的xw0,θwi,i=0,1,..neg。
