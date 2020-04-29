---
title: 文本挖掘预处理之TF-IDF
urlname: TF-IDF-1
tags:
  - TF-IDF
categories:
  - 机器学习
abbrlink: 21059
date: 2019-03-25 22:10:10
---

TF-IDF（Term Frequency-Inverse Document Frequency）即“词频-反文档频率”，主要由TF和IDF两部分组成。TF-IDF是一种用于资讯检索与资讯探勘的常用加权技术，是一种统计方法，用于评估一个词对于一个文件集或一个语料库中的其中一份文件的重要程度。字词的重要程度与它在文件中出现的次数成正比，但同时与它在语料库中出现的频率成反比。


**TF——词频：**一个词在文章中出现的次数。

在计算词频时，需要注意停用词的过滤。什么是停用词：在文章中出现次数最多的“的”、“是”、“在”等最常用词，但对结果毫无帮助，必须过滤的词。

TF计算有两种方式，具体公式如下：
<!-- ![Alt](/images/articles/2019/20190325TF-IDF-1.png#pic_center) -->
<!-- ![Alt](/images/articles/2019/20190325TF-IDF-2.png#pic_center) -->
{% qnimg machine_learning/20190325TF-IDF-1.png %}
{% qnimg machine_learning/20190325TF-IDF-2.png %}

**IDF——反文档频率：**一个词在所有文章中出现的频率。如果包含这个词的文章越少，IDF越大，则说明词具有很好的类别区分能力。计算公式如下：
<!-- ![Alt](/images/articles/2019/20190325TF-IDF-3.png#pic_center) -->
{% qnimg machine_learning/20190325TF-IDF-3.png %}

将TF和IDF相乘，就得到一个词的TF-IDF值，某个词对文章的重要性越高，该值越大，于是排在前面的几个词，就是这篇文章的关键词。
<!-- ![Alt](/images/articles/2019/20190325TF-IDF-4.png#pic_center) -->
{% qnimg machine_learning/20190325TF-IDF-4.png %}

## TF-IDF总结：

**优点：**简单快速，结果比较符合实际情况。

**缺点：**单纯以“词频”做衡量标准，不够全面，有时重要的词可能出现的次数不多。

 

## 用python实现TF-IDF的计算
将下图所示的已经分好词的文章作为语料库，计算101it.seg.cln.txt中的TF-IDF。
<!-- ![Alt](/images/articles/2019/20190325TF-IDF-5.png#pic_center) -->
{% qnimg machine_learning/20190325TF-IDF-5.png %}

具体python代码实现如下：

``` python
# -*- coding: utf-8 -*-
import os
import math
 
# 要计算TF-IDF的文章路径
file_path = './data/101it.seg.cln.txt'
# 语料库目录路径
data_dir_path = './data'
 
# 获取文章内容
def read_content(file):
    content = open(file, 'r', encoding='UTF-8')
    return content
 
# 计算IDF
def calculate_idf(dir_path):
    all_word_set = set()
    article_list = []
    article_count = 0
    for fd in os.listdir(dir_path):
        article_count += 1
        file = dir_path + '/' + fd
        content = read_content(file)
        content_set = set()
        for line in content:
            word_tmp = line.strip().split(' ')
            for word in word_tmp:
                word = word.strip()
                all_word_set.add(word)
                content_set.add(word)
        article_list.append(content_set)
 
    idf_dict = {}
    for word in all_word_set:
        count = 0
        for article in article_list:
            if word in article:
                count += 1
        idf_dict[word] = math.log(float(article_count)/(float(count) + 1.0))
 
    return idf_dict
 
# 计算TF
def calculate_tf(file):
    content = read_content(file)
    word_set = set()
    word_dict = {}
    word_count = 0
    # 计算词频和文章总词数
    for line in content:
        word_tmp = line.strip().split(' ')
        for word in word_tmp:
            word = word.strip()
            if word not in word_dict:
                word_dict[word] = 1
            else:
                word_dict[word] += 1
            word_count += 1
            word_set.add(word)
    # 计算TF
    for tmp in word_set:
        word_dict[tmp] = float(word_dict[tmp])/float(word_count)
    return word_dict
 
 
if __name__ == "__main__":
    idf_dict = calculate_idf(data_dir_path)
    tf_dict = calculate_tf(file_path)
    tfidf_dict = {}
    for key in tf_dict:
        tfidf_dict[key] = tf_dict[key] * idf_dict[key]
    print(tfidf_dict)
```

## TF-IDF应用：
TF-IDF有下面几个应用，具体的实现后续文章再给大家介绍：

* 提取文章的关键词

* TF-IDF结合余弦相似度找相似文章

* 给文章自动生成摘要