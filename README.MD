# Dynamic-CNN-Sentence-Classification-TF
本代码是论文： A Convolutional Neural Network for Modelling Sentences”基于TensorFlow的实现方法：
论文地址：https://arxiv.org/abs/1404.2188
本代码部分注解：
1、数据集及处理方法
本文仿真的是论文的第二个实验，使用的数据集是TREC。该数据集是QA领域用于分类问题类型的。其中问题主要分为6大类别，比如地理位置、人、数学信息等等，这里使用one-hot编码表明其类别关系。其包含5452个标记好的训练集和500个测试集。每个样本数据如下所示，以冒号分隔，前面标示类别，后面为问题：
NUM:date When did Hawaii become a state ?

2、接下来介绍数据处理函数，这部分写在dataUtils.py文件中。其实和之前写的也大都差不多，都是读取文件中的句子和标签、进行PADDING、构建vocabulary、将句子转换成单词索引以方便embedding层进行转化为词向量。代码入下，已经注释的很清楚，不再进行过多介绍。

3、模型构建：
参照论文中所介绍的DCNN模型进行仿真，但是这里有三个细节未按照论文中的要求进行设置：
卷积层的宽卷积这里使用的是tensorflow中的“SAME”卷积模式，该模式是保证输入与输出的维数相同，而不是按照l+m-1的输出。其实宽卷积也有很多形式，就是padding的个数不同导致输出的维数也不尽相同。“SAME”模式也是宽卷积的一种。
文中所说的动态k，这里因为只是用了两层卷积，所以直接指定了k1和top_k的值，并未实现k的动态计算
k-max pooling算法也未按照文中所说的保留k个最大值的相对顺序，因为tf中的top_k函数只能返回最大的k个值何其索引，然后就懒得再去进行排序了==
模型构建类照例写在model.py文件中，其实就是两个conv_fold_k-max-pooling层加一个full-connected层。

4、这里主要介绍一下top_k()函数的用法，该函数的返回结果最大的k个值values，和其对应的索引位置indices。当输入input是一维的时候直接返回k个最大值，当其是一个高维tensor时，返回最后一个维度上的所有的k个值。就是如果一个输入是[100,200,300,400]的tensor，k取10，那么返回结果就是[100,200,300,10]的tensor，可以参考这篇文章进行具体调试方便理解。
5、模型训练：
上面完成了模型搭建的任务，接下来要做的工作就是训练模型。这部分代码在train.py文件中。也是老套路，先进行数据集的读入和转换工作，然后接下来定义输入的placeholder以及网络中要用的weight，b等参数。然后初始化DCNN类并调用DCNN函数完成模型的搭建。接下来定义cost、predict、acc等需要衡量的指标。然后就可以sess.run了。同样加上一堆的summary，以方便我们在tensorboard中观察训练过程。

6、训练结果：
训练结果

看了训练结果之后，明显的感觉到了过拟合是什么概念==但是挑了一些参数感觉还是过拟合的，修复效果并不明显。学习率，dropout都试了，还是会过拟合，不过相比上次仿真的那篇论文来说效果已经好很多了，毕竟在训练集上的准确率已经分分钟达到了100%。 
下面我们看几个从tensorboard上面接下来的图片： 
这里写图片描述 
这里写图片描述

http://blog.csdn.net/liuchonge/article/details/67644305