# 说明文档
## 介绍

Vowpal Wabbit是一个机器学习系统。
它通过在线服务`online`,散列`hashing`,`allreduce`,`reductions`,`learning2search`,`active`和交互学习`interactive learning`等技术来推动机器学习的发展。

本书是对[Vowpal Wabbit Wiki][vw-wiki]的翻译而来，其中结合[Vowpal Wabbit源码][vw-src-8.5]对内容进行理解和印证。
而翻译这个机器学习系统的初衷是因为目前正在做着推荐系统结合机器学习方面的尝试，而作为一个成熟的机器学习系统，
在其工程和算法性能方面一定非常接近完美，有助于在这方面探索中的我能从中汲取经验和技巧。之所以选择Vowpal Wabbit，
一方面它从机器学习中最基础的逻辑回归入手，方便小白的我入手；另一方面它用C/C++编写核心，保证其算法的高性能，在此
基础上还支持python等高级语言，以此来保证其易用性。该思路和自己的技术栈完全吻合。



## 参考文献
1. [Vowpal Wabbit Wiki][vw-wiki]
2. [Vowpal Wabbit源码][vw-src-8.5]

[vw-wiki]: https://github.com/VowpalWabbit/vowpal_wabbit/wiki
[vw-src-8.5]: https://github.com/VowpalWabbit/vowpal_wabbit/tree/8.5.0
