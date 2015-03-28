---
layout: post
title: "Expectation Propagation: Theory and Application"
date: 2014-01-01 22:04
comments: true
published: true
categories: 
---
##简介

第一次接触EP是10年在百度实习时，当时组里面正有计划把线上的CTR预估模型改成支持增量更新的版本，读到了微软一篇基于baysian的CTR预估模型的文章（见推荐阅读5），文章中没有给出推导的细节，自己也没有继续研究。今年在PRML中读Approximal inference这章对EP有了一些了解，同时参考了其它相关的一些资料，在这里和大家探讨。

####什么是期望传播
期望传播(Expectation Propagation): 基于**bayesian**的一种**近似**推断方法，常用于图模型中计算单个节点的边缘分布或者后验分布，属于message passing这一类推断方法。

####牛人
首先当然是Thomas Minka, 其在MIT读博期间提出了EP，并将EP作为博士论文课题在2001年发表。Minka毕业之后去了CMU教书，现在和Bishop一起在剑桥微软研究院。

其次是Kevin p. Murphy, 他是我做EP相关文献调研时发现的paper比较多的，我读到的一篇全文基本都是在推导Minka博士论文中一些公式的细节。btw Murphy 2013年出版了一本书，见推荐阅读2。

####中英文对照
下面是一些关键词的中英文对应 （由于相关的书籍文献基本都是英文的，有些词没有想到比较好的中文翻译，故保留英文）

截断高斯: Truncated Gaussian

置信传播: Belief Propagation （后面会简称BP）

期望传播: Expectation Propagation (后面会简称为EP)

消息传递: Message passing


##背景
EP本身的思想和方法都还是比较简单的，不过会涉及到一些背景知识，这边一并介绍。
###高斯、截断高斯
EP的核心思想之一是用指数族分布近似复杂分布，实际应用中通常选择高斯分布，所以多个高斯分布的乘积，相除，积分在EP应用过程中不可避免。

截断高斯是高斯分布在指定区间归一化后的结果，（所以其并不是一个高斯分布），EP本身并不和截断高斯直接相关，但是如果在分类问题中应用EP，对观察样本（0-1）建模方法通常是y=sign(f(x)>t), 和另一个高斯分布相乘之后即为截断高斯分布。（然后就需要计算其的均值方差，原因后面会提到）

我在另一篇文章[Gaussian and Truncated Gaussian](http://dongguo.me/blog/2013/12/02/gaussian-and-truncated-gaussian/)中介绍了比较多的细节，可以参考。

###指数族分布
指数族分布（exponential family distribution）有着非常好的特性，比如其有**充分统计量**，多个指数族分布的乘积依然是指数族分布，具体的介绍可以参见wikipedia, 介绍的非常全面，也可以参考PRML第2章。

由于指数族的良好特性，其常被拿去近似复杂的概率分布（EP和variance baysian都是）。由于EP中常常选择高斯分布，我们这边强调一下，高斯分布的充分统计量为: (x, x^2), 其中x为高斯分布的自变量。

###图模型
EP是贝叶斯流派的计算变量后验分布（或者说是边缘分布）的近似推断方法，通常都可以通过一个概率图模型来描述问题的生成过程（generation process），所以可以说图模型是EP的典型应用场景。

图模型在很多地方都有介绍，比如PRML第8章，在这里就不重复了。有1点提一下，一个图模型的联合分布（不管是有向图还是无向图）可以写成若干个因子的乘积，对于有向图每个因子是每个节点的条件分布（条件于其的所有直接相连的父节点），对于无限图每个因子是energy function。
这个特性在后面的置信传播算法会用到。

###factor graph
图模型中节点之间的关系通过边来表达，factor graph将这种节点之间的关系通过显式的节点（factor node）来表达，比如对于有向图，每个factor node就代表一个条件概率分布，图中的所有的信息都存在于节点上（variable nodes和factor nodes）。

后面的BP和EP都基于factor graph，可以认为factor graph使得图上的inference方法变得比较直观，另一个好处是factor graph屏蔽了有向图和无向图的差异。（有向图无向图都可以转变为factor graph）

更多了解可以看PRML第8章。

###置信传播
Belief Propagation (BP)又叫'sum-product'，是一种计算图模型上节点边缘分布的推断方法，属于消息传递方法的一种，非近似方法（基于其延伸的Loopy Belief propagation为近似推断方法）。
BP的核心为如下3点：

* **单个variable node边缘分布的计算**

<img src="/images/personal/research/ep-intro/marginal_dis_variable_node.png" width=400 align=top />

(注：上图来之PRML)

前面提到过图模型的联合分布可以分解为若干因子的乘积，每个因子对应一个factor node：

<img src="/images/personal/research/ep-intro/posterior_dis_variable_node_f1.png" width=500 align=top />

每个variable node的边缘分布为与其直接相连的factor nodes传递过来的message的乘积：

<img src="/images/personal/research/ep-intro/posterior_dis_variable_node_f2.png" width=700 align=top />

* **从factor node到variable node的消息传递**

<img src="/images/personal/research/ep-intro/message_factor_to_variable.png" width=400 align=bottom />

(注：上图来之PRML)

从factor node f传递到variable node x的message为：与f直接相连（除了x）的variable nodes传递到f的messages与f本身的乘积的积分（积分变量为与f直接相连的除x之外的所有variable nodes）：

<img src="/images/personal/research/ep-intro/message_factor_to_variable_f1.png" width=600 align=top />

* **从variable node到factor node的消息传递**

<img src="/images/personal/research/ep-intro/message_variable_to_factor.png" width=400 align=bottom />

(注：上图来之PRML)

从variable node x到factor node f的message为：与x直接相连的factor nodes（除f以外）传递到x的messages的乘积：

<img src="/images/personal/research/ep-intro/message_variable_to_factor_f1.png" width=400 align=top />
	
更多细节请参考PRML

###Moment matching
在实际的问题中，要么后验分布本身比较复杂（推荐阅读3中的Clutter example），要么最大化后验的计算比较复杂，要么破坏了具体算法的假设（比如EP要求图中的所有message都是指数族），所以常常会用（有良好性质的）指数族分布近似实际的概率分布。

<img src="/images/personal/research/ep-intro/moment_matching_1.png" width=600 align=top />
<img src="/images/personal/research/ep-intro/moment_matching_2.png" width=300 align=top />

用一个分布去近似另一个分布的常见方法是最小化KL散度:

<img src="/images/personal/research/ep-intro/moment_matching_3.png" width=600 align=top />
<img src="/images/personal/research/ep-intro/moment_matching_4.png" width=600 align=top />

我们发现通过最小化KL散度得到的‘最接近’p(x)的q(x)可以简单地通过匹配充分统计量的期望得到。

当q(x)为高斯分布的时候，我们知道其充分统计量u(x)=(x, x^2)，这时通过KL散度最小化近似分布近似的方法称为moment matching(匹配矩)

<img src="/images/personal/research/ep-intro/moment_matching_5.png" width=600 align=top />

为什么称为匹配矩呢，看看矩的定义就知道了：

<img src="/images/personal/research/ep-intro/moment_matching_6.png" width=350 align=top />

##期望传播方法-理论
EP的思想：在图模型中，用高斯分布近似每一个factors，然后'approximate each factor in turn in the context of all remaining facotrs'.

下面为具体的算法：
<img src="/images/personal/research/ep-intro/ep_def.png" width=800 align=top />
(注：本算法参考了PRML)

下面通过Minka博士论文中的例子‘clutter problem’来解释：每个观察样本以(1-w)的概率由高斯分布N(x|sita, I)生成，以w的概率由noise生成（同样也是高斯分布N(x|0, aI)），于是：
<img src="/images/personal/research/ep-intro/clutter_problem_1.png" width=400 align=top />
<img src="/images/personal/research/ep-intro/clutter_problem_2.png" width=200 align=top />

按照EP的思想，我们用一个单高斯q(sita)去近似混合高斯p(x|sita)
<img src="/images/personal/research/ep-intro/clutter_problem_3.png" width=400 align=top />

单高斯去近似混合高斯听起来效果一定不好，但实际上，由于EP在近似的时候乘了其他所有factors的高斯近似之后的上下文，考虑到很多个高斯分布相乘之后的方差一般都很小，所有实际上单高斯只需要在很小的区间近似好混合高斯即可。如下图：

<img src="/images/personal/research/ep-intro/clutter_problem_4.png" width=350 align=top />
<img src="/images/personal/research/ep-intro/clutter_problem_5.png" width=350 align=top />

(注：上面2张图来之PRML)

其中蓝色曲线为混合高斯（没有画完整），红色曲线为近似的单高斯，绿色曲线为‘其它所有factor的乘积’。


###EP怎么应用在message passing中：
在图模型中，所谓的'context of all remaining factors'就是当前节点之外所有节点和messages，所以EP在图模型中的使用方式为：和BP一样的方法计算message和marginal distribution，当某个factor或者marginal distribution不是高斯分布时，用高斯分布近似它。所以Minka认为EP也就是BP+moment matching。

由于每个factor以及variable node的边缘分布都是高斯分布（或被近似为高斯分布），所以EP的计算过程一般并不复杂。


##期望传播方法-应用
EP被广泛地应用在图模型的inference上，这边提一下微软的2个应用：Bing的CTR预估，XBOX游戏中player skill的评估。
###Bing的CTR预估
详细的推导及实验请参考：[Bayesian CTR prediction for Bing](http://dongguo.me/blog/2013/12/01/bayesian-ctr-prediction-for-bing/)
paper中称这个model为ad predictor，其在我的数据集上预估效果很不错，训练预测速度快，天然支持增量更新，主要的缺点就是模型不是稀疏的。如果你知道怎么自然地达到稀疏效果，请指教。

和其它算法的比较请参考：[Classification Models](http://dongguo.me/blog/2013/12/15/classification-models/)


###XBOX中player skill的评估
图模型和上一篇略有差异，推导过程差不多，paper中没有给出详细的推导过程，不过Murphy的新书中给出了，请参考推荐阅读2。

##一些小结
1. EP的通用性比较好，对于实际的问题，画出graph model和factor graph，就可以尝试用EP来进行inference；
2. 虽然应用EP时的推导过程略长（计算很多个message和marginal distribution），但是最终的整体的更新公式一般都非常简单，所以模型训练时间开销往往较小；
3. 为了使用EP，只能用高斯分布来建模，比如Bing的CTR预估那篇对每个feature的weight建模，只能假设服从高斯分布，相当于是2范数的正则化，不能达到稀疏模型的效果；
4. 在我的实验中，通过EP进行inference得到的模型预估效果不错，值得一试；


##推荐阅读
1. 机器学习保留书籍:[Pattern recognition and machine learning](http://research.microsoft.com/en-us/um/people/cmbishop/prml/) 第2,8,10章 (第2章看看高斯四则运算，指数族分布特性；第8章了解图模型基础，期望传播算法；第10章了解期望传播算法)
	
2. Murphy新书: [Machine Learning: A Probabilistic Perspective](http://www.cs.ubc.ca/~murphyk/MLbook/) 第22章 (本书相比PRML更加具体，第22章干脆包含了TrueSkill的详细推导步骤)

3. Minka的博士论文：[A family of algorithms for approximate Bayesian inference](http://qh.eng.ua.edu/e_paper/e_thesis/EPThesis.pdf) (想了解基本思想和理论看完前3节即可)

4. EP的应用之一：[TrueSkill: A Bayesian Skill Rating System](http://research.microsoft.com/pubs/74419/tr-2006-80.pdf) (文中并没有给出EP每一步的细节)

5. EP的应用之二：[Web-Scale Bayesian Click-Through Rate Prediction for Sponsored Search Advertising in Microsoft’s Bing Search Engine](http://research.microsoft.com/pubs/122779/adpredictor%20icml%202010%20-%20final.pdf) (CTR预估的应用比较吸引人，文章写得很棒，算法的效果也很好，只是干脆忽略的inference过程，有兴趣的同学可以参看我另一个文章，里面有一步一步推导的过程)

6. Minka整理的EP学习资料：[link](http://research.microsoft.com/en-us/um/people/minka/papers/ep/roadmap.html) (其中的包含了一个videolecture上他做的variance inference的talk值得一看)

7. 本文的PPT： [墙外](http://www.slideshare.net/guo_dong/expectation-propagation-researchworkshop), [墙内](http://pan.baidu.com/s/1kT3DtvL)


