# 介绍

## 由来
Vowpal Wabbit项目是由雅虎发起，目前是微软研究院在维护的一个快速核外`out-of-core`学习系统。有两种方法可以实现快速学习算法：第一种是给一个慢算法加速，第二种是从本质上构建一个快速学习算法。而该项目属于第二种，并且它可以作为一个研究和实验的平台。
> **TIPS:**
> `out-of-core`翻译为核外，是指该学习系统能够处理超出计算机内存大小的数据。因为一般模型算法需要一次性加载所有训练数据到内存中，如果训练数据超出计算机内存大小，那么该模型算法就无法训练，而Vowpal Wabbit能够处理这样级别的数据。


