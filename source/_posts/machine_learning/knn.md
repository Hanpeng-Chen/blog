---
title: K-近邻(KNN)算法
urlname: knn
date: 2019-03-21 17:04:17
tags:
  - KNN
  - 机器学习
  - 分类算法
categories:
  - 机器学习
---
K-近邻（KNN，K-Nearest Neighbor）算法是一种基本分类与回归方法，在机器学习分类算法中占有相当大的地位，既是最简单的机器学习算法之一，也是基于实例的学习方法中最基本的，又是最好的文本分类算法之一。

我们本篇文章只讨论分类问题的KNN算法。

## KNN算法概述
KNN是通过测量不同特征值之间的距离进行分类。

KNN算法思路：如果一个样本在特征空间中的k各最相似（即特征空间中最近邻）的样本中的大多数属于某一个类别，则该样本也属于该类别。其中k一个是不大于20的整数。KNN算法中，所选择的邻居都是已经正确分类的对象。

KNN算法的输入为实例的特征向量，对应于特征空间的点；输出为实例的类别，可以取多类。

KNN算法实际上利用训练数据集对特征向量空间进行划分，并作为其分类的模型。k值的选择、距离度量和分类决策规则是KNN算法的三个基本要素。


## KNN算法工作原理
 
KNN算法工作原理可描述为：
1、假设有一个带有标签的训练样本集，其中包含每条数据与所属分类的对应关系。

2、输入没有带标签的新数据后，计算新数据与训练样本集中每条数据的距离。

3、对求得的所有距离进行升序排序。

4、选取k个与新数据距离最小的训练数据。

5、确定k个训练数据所在类别出现的频率。

6、返回k个训练数据中出现频率最高的类别作为新数据的分类。

 
### KNN算法距离计算方式
在KNN中，通过计算对象间距离来作为各个对象之间的相似性指标，这里的距离计算一般采用欧式距离或者是曼哈顿距离，两种距离的计算公式如下图所示：
![Alt](/images/articles/2019/20190321knn-1.png#pic_center)

### KNN算法特点：

* 优点：精度高、对异常值不敏感、无数据输入假定

* 缺点：计算复杂度高、空间复杂度高

* 适用数据范围：数值型和标称型

 

## KNN算法demo实例

python（python3.6）实现一个KNN算法的简单demo

``` python
# -*- coding: utf-8 -*-
import numpy as np
from collections import Counter
import os
 
# 创建样本数据集
def createDataSet():
    group = np.array([[1.0, 1.1], [1.0, 1.0], [0, 0.1], [0.1, 0]])
    labels = ['A', 'A', 'B', 'B']
    return group, labels
 
# 近邻算法
def classify0 (inX, dataSet, labels, k):
    diffMat = np.tile(inX,(dataSet.shape[0],1)) - dataSet #待分类的输入向量与每个训练数据做差
    distance = ((diffMat ** 2).sum(axis=1)) ** 0.5 #欧氏距离
    sortDistanceIndices = distance.argsort() #从小到大的顺序，返回对应索引值
    votelabel = []
    for i in range(k):
        votelabel.append(labels[sortDistanceIndices[i]])
    Xlabel = Counter(votelabel).most_common(1)
    return Xlabel[0][0]
 
if __name__ == "__main__":
    # # 创建数据集和 k-近邻算法
    group, labels = createDataSet()
    # 新数据为[0, 0], k=3
    label = classify0([0, 0], group, labels, 3)
    print(label)
```

执行上面代码，在控制台打印输出  B，即为数据[0, 0]的分类。

### 方法说明

上面python代码中有几个方法在这里简单说明一下：

* Counter(votelabel).most_common(1)：求votelabel中出现次数最多的元素

* np.tile(A,B)：若B为int型：在列方向上将A重复B次 若B为元组(m,n):将A在列方向上重复n次，在行方向上重复m次

* sum(axis=1)：函数的axis参数，axis=0:按列相加；axis=1:按行的方向相加，即每行数据求和

* argsort：将数组的值按从小到大排序后，输出索引值