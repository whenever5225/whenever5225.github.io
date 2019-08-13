---
title: C++读取ivecs格式数据
date: 2019-08-12 21:54:50
tags:
- C/C++
- ANNS
- 源码阅读
categories:
- 近似最近邻搜索
---

## 引言

近似最近邻搜索中通常会涉及到`fvecs`和`ivecs`格式的数据，其中，原始数据一般为`fvecs`格式的数据，查询结果一般为`ivecs`格式的。`ivecs`内部存储的主要是数据的`id`，数据类型为`unsigned`类型。就其内部数据结构而言，行数为查询点的个数，列数为对每个查询点查询返回个数再加1，因为每行的第一个位置存储的是对每个查询点查询返回个数。

可以通过程序来读取`ivecs`格式数据的内容，下面是用`c++`程序读取`ivecs`格式数据内容并输出其查询数据个数和对每个查询点查询返回个数。

## C++读取`ivecs`格式数据

```cpp
#include <iostream>
#include <fstream>
#include <vector>

void load_ivecs_data(const char* filename,
                 std::vector<std::vector<unsigned> >& results, unsigned &num, unsigned &dim) {
  std::ifstream in(filename, std::ios::binary);
  if (!in.is_open()) {
    std::cout << "open file error" << std::endl;
    exit(-1);
  }
  in.read((char*)&dim, 4);
  //std::cout<<"data dimension: "<<dim<<std::endl;
  in.seekg(0, std::ios::end);
  std::ios::pos_type ss = in.tellg();
  size_t fsize = (size_t)ss;
  num = (unsigned)(fsize / (dim + 1) / 4);
  results.resize(num);
  for (unsigned i = 0; i < num; i++) results[i].resize(dim);

  in.seekg(0, std::ios::beg);
  for (size_t i = 0; i < num; i++) {
    in.seekg(4, std::ios::cur);
    in.read((char*)results[i].data(), dim * 4);
  }
  in.close();
}

int main(int argc, char** argv) {
  std::vector<std::vector<unsigned> > true_load;
  unsigned dim, num;
  load_ivecs_data(argv[1], true_load, num, dim);
  for(size_t i = 0; i < num; i++) {
    for(size_t j = 0; j < dim; j++) {
      std::cout << true_load[i][j] << " ";
    }
    std::cout << std::endl;
  }
  std::cout << "result_num："<< num << std::endl << "result dimension：" << dim << std::endl;
  return 0;
}
```

## 参考文献

> [1]付聪, NSG : Navigating Spread-out Graph For Approximate Nearest Neighbor Search, https://github.com/ZJULearning/nsg, 2019.8.12.