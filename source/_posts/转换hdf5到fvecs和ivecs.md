---
title: python读取HDF5格式文件并保存为fvecs和ivecs格式（将HDF5格式gist数据集转换为fvecs和ivecs格式）
date: 2019-09-08 15:18:22
tags:
- HDF5
- python
categories:
- HDF5
---

## 引言

[网站](http://corpus-texmex.irisa.fr/)的gist数据集下载很慢，尝试多次都失败了，科学上网也不行。github上（[地址](https://github.com/erikbern/ann-benchmarks/)）提供了很多`hdf5`格式的数据集，其中也包括gist，下载速度很快。但是实验的代码处理的数据格式是`fvecs`和`ivecs`格式的，需要将`hdf5`格式的数据进行转换。

## 代码示例1（转换成`fvecs`格式）

```python
import h5py
import struct

def load_hdf5_file(filename):
    f = h5py.File(filename, 'r')
    print('done !')
    return f

def to_fvecs(filename, data):
    with open(filename, 'wb') as fp:
        for y in data:
            d = struct.pack('I', y.size)	#将y.size以C++的unsigned int的形式写入二进制文件
            fp.write(d)
            for x in y:
                a = struct.pack('f', x)	#将x以C++的float的形式写入二进制文件
                fp.write(a)

if __name__ == "__main__":
    f = load_hdf5_file('/home/tdlab/dataset/gist-960-euclidean.hdf5')
    to_fvecs('gist_base.fvecs', f['train'])
    to_fvecs('gist_query.fvecs', f['test'])
```

## 代码示例2（转换成`ivecs`格式）

```python
import h5py
import struct

def load_hdf5_file(filename):
    f = h5py.File(filename, 'r')
    print('done !')
    return f

def to_ivecs(filename, data):
    with open(filename, 'wb') as fp:
        for y in data:
            d = struct.pack('I', y.size)
            fp.write(d)
            for x in y:
                a = struct.pack('I', x)
                fp.write(a)

if __name__ == "__main__":
    f = load_hdf5_file('/home/tdlab/dataset/gist-960-euclidean.hdf5')
    to_ivecs('gist_groundtruth.ivecs', f['neighbors'])
```