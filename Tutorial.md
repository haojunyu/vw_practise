# 学习路线
本篇文章主要翻译Vowpal Wabbit的Tutorial，并且会以小白的角色按照教程来走一遍。我们知道Vowpal Wabbit是以最基本的逻辑回归算法起家的，而Philippe Adjiman的三篇博文[深入了解逻辑回归一二三][dive_into_logistic_regression]会从逻辑回归的角度来讲述Vowpal Wabbit的[输入格式](InputFormat.md)等有用选项。

## 版本特色
目前Vowpal Wabbit最新的版本是8.6，而docker镜像中安装的是8.5。下面将倒序各个版本的特色：
1. 8.5版本包含[稀疏模型，基线，优化探索][SparseModel_Baseline_OptimizedExploration]，[成本敏感主动学习][CostSensitiveActiveLearning]，[java接口][JavaInterface]和[决策服务JSON提取][DecisionServiceiJsonIngestion]
2. 8.3版本包含[上下文老虎机][ContextualBandits],[对数时间分类][LogarithmicTimeClassification]和[联合预测][JointPrediction]
3. 8.1版本包含[学习搜索结果，主动学习，C#库和决策服务][Learning2Search_ActiveLearning_C#Library_DecisionService]
4. 7.8版本包含[探索库][ExploreLib]，[多项式学习][PolynomialLearn]，[Hogwild模式][Hogwild]，[在线svm模型][OnlineSvm]，[学习搜索][Learning2Search]，[实体关系和依赖关系解析][EntityRelation&DependencyParsing]。
5. 7.4版本包含[降维和搜索系统][LearningReduction&SearchSystems]，[坚持和topK][Holdout&TopK]，[正则化梯度下降][NormalizedGradientDescent]，[主动学习接口][ActiveLearningInterface]。
6. 7.0版本涵盖了基础知识和最常见的选项。比如如何使用VW以及处理不同问题（二分类，回归，多分类，成本敏感多分类，离线基于上下文的多臂老虎机以及序列预测等问题）所对应的不同数据格式。
7. [6.1版本][vw6.1_tutorial]包含了[L-BFGS][lbfgs]，[集群并行学习][ClusterParallelLearning]，[主动学习][ActiveLearning]，[LDA主题模型][LatentDirichletAllocation]以及[VW的外行指南][VWForUninitiated]。

## 依次介绍
### 安装
第一步是下载Vowpal Wabbit。我们将使用github上的[VW主仓库][vw_master_github],当然另一个选择是使用稳定版本。使用下面命令行来获取源码并编译。
```bash
# 下载github中vowpal wabbit
git clone git://github.com/JohnLangford/vowpal_wabbit.git
# 编译
cd vowpal_wabbit && make
```
> **TIPS:**
> 在一些操作系统中，vowpal wabbit所依赖的库可能没有安装在标准的位置（一般Linux或者OS-X还算标准，在其他POSIX系统中就不一定了），此时可以在使用`make`之前使用`./autogen.sh && ./configure`来生成针对当前系统的Makefile文件。如果`configure`过程中报"Configure could not find version of library"的错误，可以使用`./configure --with-boost-libdir=/usr/lib/x86_64-linux-gnu`。如果`make`失败，你得需要安装VW依赖的库boost。

Vowpal Wabbit是依赖boost库的。基于Debian的Linux发行版（Debian，Ubuntu，Mint等）操作系统可以使用以下命令来安装：
```bash
sudo apt-get install libboost-program-options-dev libboost-python-dev zlib1g-dev
```
而对于苹果的OS-X系统可以通过编译源码安装，但强烈建议使用`brew`安装。

安装好依赖的boost库之后，就可以执行以下命令安装vowpal wabbit了。
```bash
# 生成vw可执行文件
pushd vowpalwabbit && make vw && popd
# 安装library
pushd library && make library_example && popd
# 测试vw和样例
pushd test && make test && popd
# 安装vw到系统路径
make install
```

### 使用样例
现在我们来创建一个有关房子在后面的十年需不需要新屋顶的数据集 `house_dataset` ,其内容如下：
```
0 | price:0.23 sqft:0.25 age:0.05 2006
1 2 'second_house' | price:0.18 sqrf:0.15 age:0.35 1976
0 1 0.5 'third_house' | price:0.53 sqft:0.32 age:0.87 1924
```
vw的输入格式详细可参考[输入格式](InputFormat.md)，这里简要提一下，其基本格式如下：
```
[label] [Importance] [Base] [Tag] |Namespace Features  |Namespace Feature ... |Namespace Features
Namespace=String[:Value]
Features=(String[:Value] )*
```
这里 `label` 是样本标签，像第一个样本的0（表示不需要新屋顶），第二个样本的1（表示需要新屋顶），`[]` 表示可有可无，预测时标签就是没有的。`Importance` 表示样本权重，比如第二个样本的2。`Base` 表示初步预测，比如第三个样本中的0.5。有时你有多个交互学习系统，并且想要预测一个偏移而不是一个绝对值时会用到。`Namespace` 表示特征空间，一定要紧靠 `|`，这个在做特征交叉时会用到。 `Features` 表示多个特征，每个特征包含特征名和特征值，并以 `:` 分隔。如果特征名为字符串，则会使用hash函数进行散列，比如比如 `price`，`sqrf`，`age`，而`2006` 直接使用2006这个索引。而特征值默认为1。


接着，我们可以使用如下命令进行模型学习
```bash
vw house_dataset
```
该命令会将 `vw` 执行的细节输出到终端上,如果想关闭执行细节或称诊断信息，可以使用 `--quiet` 参数。
```bash 
# 执行细节如下
Num weight bits = 18  # 散列位数
learning rate = 0.5 # 学习率
initial_t = 0 # 学习率衰减初始时间
power_t = 0.5 # 学习率衰减的力度
using cache_file = house_ds.cache # 使用缓存文件
ignoring text input in favor of cache input # 不会读源数据集
num sources = 1 # 代表数据来源数目

average  since         example        example  current  current  current
loss     last          counter         weight    label  predict features
0.000000 0.000000            1            1.0   0.0000   0.0000        5
0.666667 1.000000            2            3.0   1.0000   0.0000        5
```

由于 `vw` 支持对同一份数据集进行多轮训练，此外我们也会对同一份数据进行参数调优（使用相同的数据集但训练参数不同），这两种情况强烈推荐使用 `-c` 参数来在训练中使用缓存文件来加速训练，默认缓存文件名为 `house_dataset.cache` ，也可以使用 `--cache_file` 来指定缓存文件。而该文件的更新会随着原数据文件的更新而更新（数据集文件总比缓存文件新）。

作为一个在线学习的工具，`vw` 支持多种输入形式。这里我们只是使用样本文件的方式，诊断信息中通过 `num sources=1` 来体现，除此之外还有使用缓存文件，标准输入和tcp套接口。**在上述的列子中只有一个输入文件，但是你可以指定多个文件（目前不清楚如何指定）**。

`vw` 会对特征名进行散列，默认使用18位的散列函数，对上述的特征数目而言已经足够，你可以使用 `-b bits` 来设定位数。**18位是指最多$$2^{18}$$个特征么**


`vw` 默认的学习率设置为0.5，是因为在默认的更新参数（`--normalized` `--invariant` `--adaptive` 自适应）下表现较好。如果数据有噪声，则需要更大的数据或者通过多轮训练来达到好的预测效果。在训练这些较大的数据集过程中，学习率将会衰减为0。你可以使用 `-l rate`也会 来设置学习率。学习率越大，模型收敛得越快，也更可能过拟合，使得模型效果差。

学习率通常会随着时间的推移而衰减，你可以使用 `--initial_t time` 来指定最初的时间。该参数已经很少使用了。

另一个描述学习率的参数是学习率衰减的幅度，可以使用 `--power_t p` 来调节，其中 $$p \in [0,1]$$ 。$$p=0$$ 意味着学习率不会衰减，这在状态跟踪中会很有作用。$$p=1$$ 很极端，但对于IID数据集貌似是最佳的。 $$p=0.5$$ 是一个折中的选择。说明这一点的另一种方法是：对于输入特征和标签都不变的固定数据集采用高的学习率衰减（接近1）会有较高的收益；而对于在变动着的环境中学习训练，比如学习对抗一个不断改变游戏规则的对手，采用低的学习率衰减（接近0）会有较高的收益，因为模型可以对变化的环境较快的做出反应。对于很多情况， $$p=0.5$$ 是一个最好的选择。


紧接着这些参数的是一堆标题信息。`vw` 将输出一些实时执行信息。第一列是 `average loss`，用来计算渐进验证损失（随机梯度下降每次只使用小部分数据训练，这样无法做交叉验证，所以使用渐进验证，也就是使用小部分数据训练后再使用该数据进行验证，以此来看模型是否被优化）。这里要理解的关键是渐进验证损失会像测试集一样偏离，以此对于任何数据这是一个可靠的指标。




## 参考文献
1. [深入了解逻辑回归一二三][dive_into_logistic_regression]
2. [Vowpal Wabbit主仓库][vw_master_github]

[vw_master_github]: https://github.com/JohnLangford/vowpal_wabbit
[dive_into_logistic_regression]: http://www.philippeadjiman.com/blog/2017/12/09/deep-dive-into-logistic-regression-part-1/
[SparseModel_Baseline_OptimizedExploration]: https://github.com/VowpalWabbit/vowpal_wabbit/wiki/update.pdf
[CostSensitiveActiveLearning]: https://github.com/VowpalWabbit/vowpal_wabbit/wiki/cs_active.pdf
[JavaInterface]: https://github.com/VowpalWabbit/vowpal_wabbit/wiki/vw-jni-tutorial.pdf
[DecisionServiceJsonIngestion]: https://github.com/VowpalWabbit/vowpal_wabbit/wiki/microsoft-ds-nips.pptx
[ContextualBandits]: https://github.com/VowpalWabbit/vowpal_wabbit/wiki/intro_CBs.pdf
[LogarithmicTimeClassification]: https://github.com/VowpalWabbit/vowpal_wabbit/wiki/recalltree.pdf
[JointPrediction]: https://github.com/VowpalWabbit/vowpal_wabbit/wiki/l2s.pdf
[Learning2Search_ActiveLearning_C#Library_DecisionService]: https://github.com/VowpalWabbit/vowpal_wabbit/wiki/2015_tutorial.pdf
[ExploreLib]: https://github.com/VowpalWabbit/vowpal_wabbit/wiki/intro_2014.pdf
[PolynomialLearn]: https://github.com/VowpalWabbit/vowpal_wabbit/wiki/poly.pdf
[Hogwild]: https://github.com/VowpalWabbit/vowpal_wabbit/wiki/lrq_hogwild.pdf
[OnlineSvm]: https://github.com/VowpalWabbit/vowpal_wabbit/wiki/ksvm.pdf
[Learning2Search]: https://github.com/VowpalWabbit/vowpal_wabbit/wiki/learning2search_python.pdf
[EntityRelation&DependencyParsing]: https://github.com/VowpalWabbit/vowpal_wabbit/wiki/ER_dep_parse.pdf
[LearningReduction&SearnSystems]: https://github.com/VowpalWabbit/vowpal_wabbit/wiki/reductions_and_searn.pdf
[Holdout&TopK]: https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Zhen.pdf
[NormalizedGradientDescent]: https://github.com/VowpalWabbit/vowpal_wabbit/wiki/normalized.pdf
[ActiveLearningInterface]: https://github.com/VowpalWabbit/vowpal_wabbit/wiki/active_learning.pdf
[vw6.1_tutorial]: https://github.com/VowpalWabbit/vowpal_wabbit/wiki/v6.1_tutorial.pdf
[lbfgs]: https://github.com/VowpalWabbit/vowpal_wabbit/wiki/L-BFGS.pdf
[ClusterParallelLearning]: https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Cluster_parallel.pdf
[ActiveLearning]: https://github.com/VowpalWabbit/vowpal_wabbit/wiki/active.pdf
[LatentDirichletAllocation]: https://github.com/VowpalWabbit/vowpal_wabbit/wiki/lda.pdf
[VWForUninitiated]: http://zinkov.com/posts/2013-08-13-vowpal-tutorial
