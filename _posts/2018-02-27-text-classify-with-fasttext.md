---
layout: post
title: 文本分类系列：使用 Fasttext 进行中文文本分类
date: 2018-02-27
categories: machine learning
tags: machine learning
---

## 0. 说明

本文主要参考：[文本分类（六）：使用fastText对文本进行分类--小插曲](http://blog.csdn.net/lxg0807/article/details/52960072)，加入一些个人注解；

## 1. 环境 & 数据

### 环境准备

```
# 安装 python 版本的 fasttext 和 jieba 分词工具
pip install fasttext jieba
```

### 数据

**原始数据**：清华的新闻数据：[下载地址](http://thuctc.thunlp.org/message)

**分词后数据**：文章：[文本分类（六）：使用fastText对文本进行分类--小插曲](http://blog.csdn.net/lxg0807/article/details/52960072) 提供了一个整理好的数据，[news_fasttext_train.txt](http://pan.baidu.com/s/1jH7wyOY)，[news_fasttext_test.txt](http://pan.baidu.com/s/1slGlPgx)，不过该数据对每个类别取前 10000 篇文章做训练，把之后最多 10000 篇文章作为测试集；本文使用的是原始数据，直接把所有数据随机打乱，然后按照 8:2 的比例切分为训练集和测试集；

## 2. 分词 & 去停用词

为方便处理，首先把原始文件夹名称改为英文，然后分词，去停用词等；停用词可以参考：[dongxiexidian/Chinese](https://github.com/dongxiexidian/Chinese/tree/master/dict)，这里使用的是哈工大的停用词表；

```
import os
import sys
import jieba

stopwords_set = set()
basedir = './THUCNews'
dir_list = ['affairs', 'constellation', 'economic', 'edu', 'ent', 'fashion',
            'game', 'home', 'house', 'lottery', 'public', 'science', 'sports', 'stock']

# 分词结果文件
data = open("news.dat.seg", 'w')

# 停用词文件
with open("./stopwords_hit.dat.utf8", 'r') as infile:
    for line in infile:
        stopwords_set.add(line.strip().decode("utf-8").encode("utf-8"))

for _dir in dir_list:
    indir = basedir + "/" + _dir + "/"
    files = os.listdir(indir)
    for _file in files:
        filepath = indir + _file

        with open(filepath, 'r') as fr:
            text = fr.read()

        text = text.decode("utf-8").encode("utf-8")
        
        # 使用结巴分词
        seg_text = jieba.cut(text.replace("\t", " ").replace("\n", " "))
        outline = " ".join(seg_text)
        outline = " ".join(outline.split())

        # 去停用词，这几步也可以省略
        # outline_list = outline.split(" ")
        # outline_list_filter = [item for item in outline_list if item.encode("utf-8") not in stopwords_set]
        # outline = " ".join(outline_list_filter)

        outline = outline.encode("utf-8") + "\t__label__" + _dir + "\n" 

        data.write(outline)
        data.flush()

data.close()

```

## 3. 分类 & 效果评估

先把所有文件随机打乱，然后按照 8:2 划分训练集和测试集，其中训练集大小为：668860，测试集大小为：167215

```
cat news.dat.seg | shuf >news.dat.seg.shuf
cat news.dat.seg.shuf | head -n 167215 >news.dat.seg.shuf.test
cat news.dat.seg.shuf | tail -n 668860 >news.dat.seg.shuf.train
```

训练代码：

```
#-*- coding:utf8 -*-
import logging
import fasttext

logging.basicConfig(format='%(asctime)s : %(levelname)s : %(message)s', level=logging.INFO)

classifier = fasttext.supervised("news.dat.seg.shuf.train", "news.dat.seg.shuf.model", label_prefix="__label__")
result = classifier.test("news.dat.seg.shuf.test")
print result.precision
print result.recall
```

### 结果：

未去停用词：准确率和召回率：0.94819244685

去停用词：0.948820380947

从结果中看，停用词对最终结果几乎没有影响，原因可能在于 fasttext 模型在训练的时候会自动去除对分类没有影响的词语（停用词）。

### 计算每个类别对应的准确率，召回率和 F-score

```
#-*- coding:utf-8 -*-

import logging
import fasttext

logging.basicConfig(format='%(asctime)s : %(levelname)s : %(message)s', level=logging.INFO)

classifier = fasttext.load_model('news.dat.seg.shuf.model.bin', label_prefix='__label__')

labels_right = []
texts = []

with open('news.dat.seg.shuf.test', 'r') as testfile:
    for line in testfile:
        line = line.decode("utf-8").rstrip()
        labels_right.append(line.split("\t")[1].replace("__label__", ""))
        # predict 的时候，输入的是 list，每一个元素是一个要预测的实例；
        texts.append(line.split("\t")[0])

# 预测结果为二维形式，输出每一个类别的概率，按概率从大到小排序
labels_predict = [e[0] for e in classifier.predict(texts)]

text_labels = list(set(labels_right))
text_predict_labels = list(set(labels_predict))

print text_labels
print text_predict_labels

A = dict.fromkeys(text_labels, 0)           # 预测正确的各个类的数目
B = dict.fromkeys(text_labels, 0)           # 测试集中各个类的数目
C = dict.fromkeys(text_predict_labels, 0)   # 预测结果中各个类的数目

for i in range(0, len(labels_right)):
    B[labels_right[i]] += 1
    C[labels_predict[i]] += 1

    if labels_right[i] == labels_predict[i]:
        A[labels_right[i]] += 1

print A
print B
print C

# 计算正确率，召回率，以及 F-score
for key in B:
    try:
        r = float(A[key]) / float(B[key])
        p = float(A[key]) / float(C[key])
        f = p * r * 2 / (p + r)

        # 类别左对齐，占 15 个字符（为了美观）
        print "%-15s p:%.6f\t r:%f\t f:%f" % (key, p, r, f)
    except:
        print "error:", key, "right:", A.get(key, 0), "real:", B.get(key, 0), "predict:", C.get(key,0)
```

## 参考文献：

- [文本分类（六）：使用fastText对文本进行分类--小插曲](http://blog.csdn.net/lxg0807/article/details/52960072)(备注：主要参考文献)
- [结巴中文分词](https://github.com/fxsjy/jieba)
- [Library for fast text representation and classification.](https://github.com/facebookresearch/fastText)
- [fasttext python 包地址](https://pypi.python.org/pypi/fasttext)
