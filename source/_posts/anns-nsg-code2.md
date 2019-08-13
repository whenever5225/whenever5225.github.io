---
title: C++读取fvecs格式数据（SIFT1M数据集的结构）
date: 2019-08-12 22:00:18
tags:
- C/C++
- ANNS
- 源码阅读
categories:
- 近似最近邻搜索
mathjax: true
---

## 引言

sift1M是一个近似最近邻搜索（ANNS）的数据集，它可用于评估ANNS的性能。它包含3个向量子集，分别为：

* 基矢量：执行搜索的矢量
* 查询向量 
* 学习向量：查找特定方法中涉及的参数

此外，它以预先计算的k个最近邻居及其平方欧式距离的形式为每个集合提供真值。
每个向量取 $ 4+d\times 4B$ ，其中 $ d$ 是维数，$B$ 是字节，具体如下：

| 域值 | 域值类型 | 描述     |
| ---- | :------: | -------- |
| d    |   int    | 向量维度 |
| d*4B |  float   | 向量分量 |

## C++读取`.fvecs`格式数据

```cpp
#include <iostream>
#include <fstream>

void load_data(char* filename, float*& data, unsigned& num, unsigned& dim) { 
  std::ifstream in(filename, std::ios::binary);	//以二进制的方式打开文件
  if (!in.is_open()) {
    std::cout << "open file error" << std::endl;
    exit(-1);
  }
  in.read((char*)&dim, 4);	//读取向量维度
  in.seekg(0, std::ios::end);	//光标定位到文件末尾
  std::ios::pos_type ss = in.tellg();	//获取文件大小（多少字节）
  size_t fsize = (size_t)ss;
  num = (unsigned)(fsize / (dim + 1) / 4);	//数据的个数
  data = new float[(size_t)num * (size_t)dim];

  in.seekg(0, std::ios::beg);	//光标定位到起始处
  for (size_t i = 0; i < num; i++) {
    in.seekg(4, std::ios::cur);	//光标向右移动4个字节
    in.read((char*)(data + i * dim), dim * 4);	//读取数据到一维数据data中
  }
  for(size_t i = 0; i < num * dim; i++) {	//输出数据
    std::cout << (float)data[i];
    if(!i) {
        std::cout << " ";
        continue;
    }
    if(i % (dim - 1) != 0) {
      std::cout << " ";
    }
    else{
      std::cout << std::endl;
    }
  }
  in.close();
}

int main(int argc, char** argv) {
  float* data_load = NULL;
  unsigned points_num, dim;
  load_data(argv[1], data_load, points_num, dim);
  std::cout << "points_num："<< points_num << std::endl << "data dimension：" << dim << std::endl;
  return 0;
}
```

## 参考文献

> [1]付聪, NSG : Navigating Spread-out Graph For Approximate Nearest Neighbor Search, https://github.com/ZJULearning/nsg, 2019.8.12.