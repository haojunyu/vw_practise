# 介绍

## 由来
Vowpal Wabbit项目是由雅虎发起，目前是微软研究院在维护的一个快速核外`out-of-core`学习系统。
> **TIPS:**
> `out-of-core`翻译为核外，是指该学习系统能够处理超出计算机内存大小的数据。因为一般模型算法需要一次性加载所有训练数据到内存中，如果训练数据超出计算机内存大小，那么该模型算法就无法训练，而Vowpal Wabbit能够处理这样级别的数据。

有两种方法可以实现快速学习算法：第一种是给一个慢算法加速，第二种是从本质上构建一个快速学习算法。而该项目属于第二种，并且它可以作为一个研究和实验的平台。

目前有很多优化求解算法，最常见的就是对损失函数使用梯度下降算法`Gradient Descent`求解，代码应该很容易使用。它唯一的外部依赖就是`boost库`，一般默认安装。

## 特色
Vowal Wabbit有如下强大的功能：
1. 灵活的[输入格式](InputFormat.md)
该学习算法的输入格式比预期的要灵活得多。它可以包含由自由格式的文本构成的特征，甚至支持命名空间。

2. 高效的运算速度
该学习算法非常快，跟一些在线的算法有的一拼。它能够处理数据千亿级$10^12$特征的学习问题，此外对于[RCV1样例](ExampleRCV1.md)，它比Leon Bottou的[svmsgd算法][svmsgd]在执行时间上快3倍。
> **TIPS:**
> wall clock execution time是指真实世界的执行时间，wall clock是指墙上的时钟，这里是为了和微处理的脉冲时钟作区分。
3. 扩展性强
这个特性和快速不一样，是指执行该学习算法所消耗的内存大小和训练数据无关。这就意味着该学习算法在训练模型的时候不需要将训练集一次性全部加载到内存。此外，因为该算法使用了[散列技巧][hash_trick]，导致特征集的大小也与训练集的数据量无关。
> **TIPS:**
> 原文为the size of the set of features is bounded independent of the amount of training data using the hashing trick.这里感觉特征集的大小还是和训练集有关的。

4. 方便做交叉特征
特征子集可以内部配对以便算法在交叉特征上也是线性的。这对排序问题很有用。David Grangier在PAMIR代码中使用类似对的技巧。在将特征提供给学习算法之前扩展特征的方法可以是计算和空间密集，这主要取决于处理方式。


## 科研成果
Vowpal Wabbit也是一个前沿研究的工具。2007年它发布了第一版，只包含散列`hashing`，缓存`caching`和在线学习`online learning`等功能。至此，许多算法对它的设计产生了影响，包括如下：
1. Kai-Wei Chang, He He, Hal Daumé III, John Langford, Stephane Ross, [A Credit Assignment Compiler for Joint Prediction](https://arxiv.org/abs/1406.1837), NIPS 2016.
2. Kai-Wei Chang, Akshay Krishnamurthy, Alekh Agarwal, Hal Daumé III, John Langford [Learning to Search Better Than Your Teacher](https://arxiv.org/abs/1502.02206), ICML 2015.
3. Alekh Agarwal, Olivier Chapelle, Miroslav Dudik, John Langford, [A Reliable Effective Terascale Linear Learning System](https://arxiv.org/abs/1110.4198), 2011.
4. M. Hoffman, D. Blei, F. Bach, [Online Learning for Latent Dirichlet Allocation](https://www.di.ens.fr/~fbach/mdhnips2010.pdf), in Neural Information Processing Systems (NIPS) 2010.
5. Alina Beygelzimer, Daniel Hsu, John Langford, and Tong Zhang [Agnostic Active Learning Without Constraints](https://arxiv.org/abs/1006.2588) NIPS 2010.
6. John Duchi, Elad Hazan, and Yoram Singer, [Adaptive Subgradient Methods for Online Learning and Stochastic Optimization](https://people.eecs.berkeley.edu/~jduchi/projects/DuchiHaSi10.html), JMLR 2011 & COLT 2010.
7. H. Brendan McMahan, Matthew Streeter, [Adaptive Bound Optimization for Online Convex Optimization](https://arxiv.org/abs/1002.4908), COLT 2010.
8. Nikos Karampatziakis and John Langford, [Importance Weight Aware Gradient Updates](https://arxiv.org/abs/1011.1576) UAI 2010.
9. Kilian Weinberger, Anirban Dasgupta, John Langford, Alex Smola, Josh Attenberg, [Feature Hashing for Large Scale Multitask Learning](https://arxiv.org/pdf/0902.2206.pdf), ICML 2009.
10. Qinfeng Shi, James Petterson, Gideon Dror, John Langford, Alex Smola, and SVN Vishwanathan, [Hash Kernels for Structured Data](http://hunch.net/~jl/projects/hash_reps/hash_kernels/hashkernel.pdf), AISTAT 2009.
11. John Langford, Lihong Li, and Tong Zhang, [Sparse Online Learning via Truncated Gradient](http://hunch.net/~jl/projects/interactive/sparse_online/paper_sparseonline.pdf), NIPS 2008.
12. Leon Bottou, [Stochastic Gradient Descent](http://leon.bottou.org/projects/sgd), 2007.
13. Avrim Blum, Adam Kalai, and John Langford [Beating the Holdout: Bounds for KFold and Progressive Cross-Validation](http://hunch.net/~jl/projects/prediction_bounds/progressive_validation/coltfinal.pdf). COLT99 pages 203-208.
14. Nocedal, J. (1980). "Updating Quasi-Newton Matrices with Limited Storage". Mathematics of Computation 35: 773–782.

[svmsgd]: https://leon.bottou.org/projects/sgd
[hash_trick]: http://hunch.net/~jl/projects/hash_reps/index.html

