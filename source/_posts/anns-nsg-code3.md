---
title: C++计算召回率
date: 2019-08-12 22:00:22
tags:
- C/C++
- ANNS
- 源码阅读
categories:
- 近似最近邻搜索
mathjax: true
---

## 引言

近似最近邻搜索中评估算法的搜索性能经常用到召回率，召回率的计算公式一般为：
$$
Recall@K = \frac{R_1 \bigcap R_2}{K}
$$
其中，$R_1$ 为搜索算法返回的 $K$ 个元素组成的集合，$R_2$ 为查询点真实的 $K$ 个近邻点，$Recall@K$ 为返回 $K$ 个近邻点时的召回率。

## C++代码实现召回率的计算

```cpp
#include <iostream>
#include <fstream>
#include <vector>

void load_ivecs_data(const char* filename,	//从文件中读取ivecs格式的数据
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

void cal_recall(std::vector<std::vector<unsigned> > results, std::vector<std::vector<unsigned> > true_data, unsigned num, unsigned k) {//召回率计算
  float mean_acc = 0;
  for(size_t i = 0; i < num; i++) {
    float acc = 0;
    for(size_t j = 0; j < k; j++) {
      for(size_t m = 0; m < k; m++) {
        if(results[i][j] == true_data[i][m]) {
          acc++;
          break;
        }
      }
    }
    mean_acc += acc / k;
  }
  std::cout << "recall: " << mean_acc / num << std::endl;
}

int main(int argc, char** argv) {
  std::vector<std::vector<unsigned> > true_data;
  std::vector<std::vector<unsigned> > results;
  unsigned dim, num;
  load_ivecs_data(argv[1], true_data, num, dim);
  load_ivecs_data(argv[2], results, num, dim);
  cal_recall(results, true_data, num, dim);
  return 0;
}
```

## 参考文献

> [1]付聪, NSG : Navigating Spread-out Graph For Approximate Nearest Neighbor Search, https://github.com/ZJULearning/nsg, 2019.8.12.