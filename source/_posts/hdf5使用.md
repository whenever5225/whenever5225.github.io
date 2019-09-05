---
title: python读取HDF5格式文件（HDF5格式文件的结构,附高速下载gist数据集）
date: 2019-09-05 21:34:39
tags:
- HDF5
- python
categories:
- HDF5
---

做近似最近邻搜索的实验时，需要用到一些开源数据集，这些数据集的格式并不都是统一的，常见的有**fvecs**&**ivecs**格式，**hdf5**格式。能够读取和保存这些格式的文件是实验的基础，另一方面，测试不同的数据集时，可能需要将一种格式转换成另一种格式。

因为实验代码是以载入**fvecs**格式数据写的，需要将**hdf5**格式转换成**fvecs**&**ivecs**格式可以用python很方便的读取**hdf5**格式的数据集，在此基础上转换为**fvec**&**ivecs**格式。

## hdf5格式的结构

一个hdf5格式的文件是一个包含两个对象的容器，一个是**数据集**（datasets），另一个是**组**（groups）。数据集的结构非常类似于Numpy的**array**，组的结构非常类似于python的字典，它像一个文件夹一样，它可以包含数据集和其它的组。总结起来：**组像字典一样工作，数据集像NumPy数组一样工作**。

拿hdf5格式数据集**gist-960-euclidean.hdf5**为例（[下载地址](http://ann-benchmarks.com/gist-960-euclidean.hdf5)），整个文件有一个**根组**，就是下图的`"/"`。

<center>
<img src="hdf5结构1.png">
</center>

根组下有四个键，分别为**distances**、**neighbors**、**test**和**train**，类比于上图中的**A**、**B**和**C**。**distances**对应的是shape为**(1000, 100)**的数据集（类比于Numpy的**array**），为每个查询向量最近的100个向量距该查询向量的距离，数据类型为**float32**；**neighbors**对应的是shape为**(1000, 100)**的数据集，为每个查询向量最近的100个向量，数据类型为**int32**；**test**对应的是shape为**(1000000, 960)**的数据集，这是基数据（原始数据），一共1000000个向量，每个向量的维度为960维，数据类型为**float32**；**train**对应的是shape为**(1000, 960)**的数据集，只是查询数据，一共1000个向量，向量的维度为960维，数据类型为**float32**。

## 读取转换过程

安装**h5py**模块。我的python是用**anaconda**装的，可以用下面命令安装。

```shell
conda install h5py
```

读取**hdf5**格式文件

```python
import h5py

f = h5py.File('gist-960-euclidean.hdf5', 'r')	#gist-960-euclidean.hdf5为hdf5格式的文件
print(list(f.keys()))	#展示所有的键
```

## 参考文献

> [1]GZKPeng, HDF5 的介绍以及在python中的应用, https://blog.csdn.net/zkp_987/article/details/79852236, 2019.9.1.
>
> [2]python中使用hdf5格式文件的文档：http://docs.h5py.org/en/stable/quick.html