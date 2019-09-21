---
title: 极度快速的近似最近邻搜索算法(EFANNA)学习笔记
date: 2019-09-21 15:18:08
tags:
- 论文阅读
- ANNS
categories:
- 近似最近邻搜索
mathjax: true
---

## 引言

极度快速的近似最近邻搜索算法(EFANNA)是[NSG](http://www.vldb.org/pvldb/vol12/p461-fu.pdf)的作者之前的一篇论文，这篇论文主要介绍用更快的方法建立KNN图并且建立一个高性能的KNN图索引。这种方法建KNN图时采用类似于Wei等人提出的方案([地址](https://www.researchgate.net/publication/221022023_Efficient_K-nearest_neighbor_graph_construction_for_generic_similarity_measures))，首先初始化一个KNN图，然后再使用NN-descent的方法精细化KNN图。该论文提出的方法改进了初始化KNN图的方式，使用更快的方法来初始化KNN图。在基于近似KNN图搜索时，大多方法都是随机选择入口点，该论文使用树结构而不是随机的方式来选择入口点。

## 简介

基于图的方法的两个问题

1. 贪婪过程往往会收敛到局部最优
2. 建立 $kNN$ 图非常耗时

尝试解决第一个问题的一个方法是不随机选择入口点，而是基于辅助结构(如[哈希方法](https://www.ncbi.nlm.nih.gov/pubmed/25330477)，树方法)为查询点提供更好的初始化入口点。

尝试解决第二个问题的很多方法是要创建近似的 $kNN$ 图，大都采用了 `divide-and-conquer`策略(分治策略)，这种策略主要分三步：

1. 划分整个数据集为多个小的子集
2. 在子集中暴力搜索获得许多重叠子图
3. 归并子图，用NN-expansion的方法细化图

该论文提出的EFANNA方法的索引包括两部分：

1. 多重随机分层结构
2. 近似 $kNN$ 图

在离线阶段，首先，EFANNA通过快速分层的方式多次划分数据集为许多子集；接着，自底向上创建近似 $kNN$ 图，在此过程中，EFANNA利用这些结构定位可能的最近邻居，并利用这些候选来更新图，最后采用改进了类似于NN-Descent的方法来精细化索引。

在在线阶段，首先在分层结构中搜索获取查询点的候选邻居（确定入口点），然后使用贪婪方法（NN-expansion）在近似 $kNN$ 图上执行查询。

## EFANNA索引构建之随机截断KD树构建算法

EFANNA索引的第一大部分为多个随机分层的结构，作者采用了随机截断的KD树（randomized truncated KD-tree）来实现这一结构，其实还有很多其它方法。作者在给定的数据集 $D$ 上建立了多个随机截断KD树。

### 主要思想

首先给要建立的随机截断KD树的叶子结点所能包含的数据点的个数限定一个上界 $K$ ，当输入的数据点的个数小于 $K$ 时直接返回，否则的话就进行划分，划分时随机选择一个维度，计算数据集在该维的平均值，然后根据平均值将数据集划分为两个子集，接着递归地对划分的两个子集建立随机截断KD树，直到数据集中点的个数小于 $K$。重复对数据集 $D$ 执行上述过程多次建立多个随机截断KD树。

### 伪代码分析

```c
//输入:
//数据集D;树的数量T;一个叶子结点中的数据点数K
Input:
the data set D, the number of trees T, the number of points in a leaf node K.
//输出:
//随机截断KD树集S
Output:
the randomized truncated KD-tree set S
//构建树的函数BUILDTREE，输入结点和数据点集
function BUILDTREE(Node,PointSet)
    //若数据点的个数小于K则直接返回
    if size of PointSet < K then
		return
	else
		//随机选择一个维度d
        Randomly choose dimension d.
        //计算数据集在该维的平均值mid
		Calculate the mean mid over PointSet on dimension d. 
        //根据平均值把数据集平分为两个子集LeftHalf和RightHalf
		Divide PointSet evenly into two subsets, LeftHalf and RightHalf, according to mid.
		//递归建立左子树
        BUILDTREE(Node.LeftChild, LeftHalf)
		//递归建立右子树
		BUILDTREE(Node.RightChild,RightHalf)
	end if
	return
end function
//迭代T次，建立T棵树
for all i = 1 to T do
    //对数据集D建立第i棵树
	BUILDTREE(Rooti,D)
    //添加第i棵树根到树集S
	Add Rooti to S.
end for
```

## EFANNA索引构建之近似kNN图构建算法

EFANNA索引的另一大部分就是近似kNN图了，作者采用了类似近似最近邻搜索的方法来建立近似kNN图，这个过程包括两个阶段：

1. kNN图初始化
2. 对初始kNN图进行细化

下面分别进行概括。

### kNN图初始化之分层分治算法

作者采用**分治**的策略来快速获得一个初始化的kNN图。得到一个初始化的kNN图无非就是给基数据集中的每个点初始化邻居，这个过程其实就是在已经建好的几个随机截断KD树上为数据点找邻居，在这里作者采用的方法不过是“分层分治”的策略。

<center>
    <img src="分层分治初始化图.png">
         图1 分层分治算法图解
</center>

#### 主要思想

对数据集中的每一个数据点，我们都要给它们初始化邻居，对某个数据点初始化邻居时，我们要在已经建立的**所有**树上以该数据点为查询点进行**分层分治**查找，也就是说在每棵树上都要查找一遍。下面只详细分析在一颗树上的**分层分治**过程，多棵树只是重复的过程。

为了便于分析，下面以图1的示例来说明。现在要给数据点 $q$ 找邻居（在树中找一些离它较近的数据点），从树根开始执行深度优先搜索进行查找，一直找到标号为`8`的叶子结点，显然 $q$ 也处于这个结点中（整棵树涵盖了所有的数据点），这个叶子结点中的数据点加入到数据点 $q$ 的邻居候选集中，此时我们处理的是最底层（深度为3）。标号为`1`、`2`和`4`的结点（非叶结点）是搜索的过程中遍历到的结点。接着该处理倒数第2层（深度为2）了，我们**只处理该层中遍历到的结点**，也就是结点`4`了，其它的标号为`5`、`6`和`7`的结点都没有遍历到。结点`4`的孩子结点分别为结点`8`和`9`，因为结点`8`已经遍历过了，此时我们**只处理没有遍历过的孩子结点**，因此，我们只处理结点`9`，对以结点`9`为根的子树进行深度优先搜索直到某个叶子结点，因为结点`9`本身就是叶子结点，因此返回的叶子结点也就是它了，把该叶子结点中的数据点加入到数据点 $q$ 的邻居候选集中。接着处理第1层，同理选中结点`2`，然后同理选中它的孩子结点`5`，对以结点`5`为根的子树执行深度优先搜索，将返回叶子结点`10`（上面的局限图可以看出结点`10`离 $q$ 更近），同样把该叶子结点中的数据点加入到数据点 $q$ 的邻居候选集中……

上面的那个过程可以一种处理到第0层，即根结点。但是，实际应用时这个过程是很耗时的，因为我们要对多个树重复上述过程，因此，都要处理到根结点是没必要的。在此，可以把能处理到的最小层设定为一个参数`Dep`，`Dep`具体值的设定可根据精度和速度的权衡来安排。

明白了上述过程，我们也就很容易明白作者给该算法命名为**分层分治**的原因了。

#### 伪代码分析

```c
//输入：
//数据集D;近似kNN图的k;树构建算法构建的随机截断KD树集S;处理的最小深度Dep(从叶子向上处理)
Input:
the data set D, the k in approximate kNN graph , the randomized truncated KD-tree set S built with Algorithm 2, the conquer-to depth Dep.
//输出：
//近似kNN图G
Output:
approximate kNN graph G.
//“分”阶段
//使用树构建算法建立多个随机截断KD树，得到树集S
%%Division step
Using Algorithm 2 to build tree, which leads to the input S
//“治”阶段
%% Conquer step
//近似kNN图G初始化为空
G = ∅
//对数据集中的每一个结点i
for all point i in D do
	Candidate pool C = ∅
	//对树集中的每一棵树t
	for all binary tree t in S do
        //用数据点i在树t上进行搜索直到叶子结点
		search in tree t with point i to the leaf node.
        //添加叶子结点中的所有数据点到候选集C
		add all the point in the leaf node to C.
        //记下叶子结点的深度d
		d = depth of the leaf node
		//当d大于给定的处理到的最小深度Dep执行下面的循环
		while d > Dep do
			d = d − 1
            //用数据点i在树t上执行深度优先搜索直到深度d，在深度为d的那层搜索到的非叶结点标记为N，它的还没被访问过的孩子结点标记为Sib
			Depth-first-search in the tree t with point i to depth d. Suppose N is the non-leaf node on the search path with depth d. Suppose Sib is the child node of N. And Sib is not on the search path of point i.
            //在以Sib为根的子树上用数据点i执行深度优先搜索，直到叶子结点，添加叶子结点中所有的数据点到候选集C
			Depth-first-search to the leaf node in the subtree of Sib with point i . Add all the points in the leaf node to C.
		end while
	end for
    //保留候选集C中离数据点i最近的K个数据点
	Reserve K closest points to i in C.
	//添加候选集C中的点到近邻图，作为数据点i的初始化邻居
	Add C to G.
end for
```

### 初始kNN图的精致化算法

这个过程就是对得到的初始化kNN图进行细化，使其更接近精确的kNN图，从而成为一个高质量的近似kNN图。这里有两种方法，分别为NN-expansion和NN-descent（两者区别详见），实践说明在构建kNN图方面NN-descent更有效，一句话概括它的思想就是**各邻居之间更可能彼此互为邻居**。

#### 主要思想

精致化算法是在初始kNN图的基础上进行的，精致化之后将得到结果图G。一开始先将结果图初始化为预先建好的初始kNN图，对数据集中的每一个点，它在G中会有一定量的邻居（此时就是初始kNN图中它的邻居），将它的这些邻居之间互相添加为各自的邻居（添加到G中），添加后将新添加的邻居标记为new，对数据集中的所有点都执行上述过程后，对每个点保留其最近的一定量个邻居（预先设定的上界），这便是第一次迭代的过程，也是相对简单的一次。

下面来看第二次迭代。对数据集中的每一个点，它它在G中会有一定量的邻居（此时的邻居有新添加标记为new的，也有初始kNN图中没标记为new的记其标记为old），将它的这些标记为new的邻居（遍历过之后取消new标记记其标记为old）之间互相添加为各自的邻居（同样添加到G中），添加后将新添加的邻居标记为new，不仅如此，对每个标记为new的邻居，还要将所有标记为old的邻居添加到它的邻居中，同时也添加反向边（标记为new的点也要添加到标记为old的点的邻居中），对数据集中的所有点都执行上述过程后，对每个点保留其最近的一定量个邻居（预先设定的上界）。

接下来的各次迭代就和第二次类似了，可以预先设置一个合适的迭代次数。

#### 伪代码分析

```c
//输入：
//初始化的近似kNN图Ginit;数据集D;最大迭代次数Imax;侯选池尺寸P;Gnew中的每个点的最大邻居数L
Input:
an initial approximate k-nearest neighbor graph Ginit, data set D, maximum iteration number Imax, Candidate pool size P, new neighbor checking num L.
//输出：
//结果近似kNN图G
Output:
an approximate kNN graph G.
iter = 0, G = Ginit
//Gnew记录每个点在上一次迭代中新添加的候选邻居集，初始化为Ginit
Graph Gnew records all the new added candidate neighbors of each point. Gnew = Ginit.
//Gold记录每个点在之前的迭代中添加的旧候选邻居集，初始化为空
Graph Gold records all the old candidate neighbors of each point at previous iterations. Gold = ∅
//Grnew记录本次迭代给NNnew中的点添加的NNnew中的点作为邻居
Graph Grnew records all the new added reverse candidate neighbors of each point.
//Grold记录本次迭代给NNold中的点添加的NNnew中的点作为邻居
Graph Grold records all the old reverse candidate neighbors of each point.
	//迭代次数小于最大迭代次数时执行循环
while iter < Imax do
    Grnew = ∅, Grold = ∅.
	for all point i in D do
        //数据点i在Gnew中的邻居集NNnew
		NNnew is the neighbor set of point i in Gnew.
        //数据点i在Gold中的邻居集NNold
		NNold is the neighbor set of of point i in Gold.
        //对NNnew中某个点j，将其它所有点添加到j在G中的条目中，并标记为新添加的候选邻居，将j添加到
		for all point j in NNnew do
            
			for all point k in NNnew do
				if j! = k then
					//计算数据点j和数据点k的距离
					calculate the distance between j and k.
                    //添加数据点k到G中j的条目，标记k为new
					add k to j’s entry in G. mark k as new.
                    //添加j到G和Grnew中k的条目，标记为new
					add j to k’s entry in G and Grnew.
					mark j as new.
				end if
			end for
			for all point l in NNold do
				calculate the distance between j and l.
                //添加l到G中j的条目，标记为old
				add l to j’s entry in G. mark l as old.
                //添加j到G和Grold中l的条目，标记为old
				add j to l’s entry in G and Grold.
				mark j as old.
			end for
		end for
	end for
	for all point i in D do
        //保留距离数据点i最近的P个邻居
		Reserve the closest P points to i in respective
		//作为结果图的条目
		entry of G.
	end for
	Gnew = Gold = ∅
	for all point i in D do
        //G中i的邻居集为NN
		l = 0. NN is the neighbor set of i in G.
		while l < L and l < P do
			j = NN[l].
            //如果j标记为new
			if j is marked as new then
				//添加j到Gnew中i的条目
				add j to i’s entry in Gnew.
                //l统计标记为new的邻居检查的次数
				l = l + 1.
			else
                //添加j到Gold中i的条目
				add j to i’s entry in Gold.
			end if
		end while
	end for
	Gnew = Gnew ∪ Grnew.
	Gold = Gold ∪ Grold
	iter = iter + 1
end while
```

## 在EFANNA索引上进行近似最近邻搜索

### EFANNA的搜索算法伪代码

```c
//输入:
//数据集D;查询向量q;要求的最近邻居数K;EFANNA索引(包括树集Stree和kNN图G);
//侯选池尺寸P;扩展因子(贪婪搜索时每次迭代保留的最大候选邻居数)E;迭代次数I.
Input:
data set D, query vector q, the number K of required nearest neighbors, EFANNA index (including tree set Stree and kNN graph G), the candidate pool size P, the expansion factor E, the iteration number I.
//输出:
//查询点的近似最近邻居
Output:
approximate nearest neighbor set ANNS of the query
iter = 0
NodeList = ∅
candidate set C = ∅
//每个叶子结点包含的数据点的最大个数记为Sleaf
suppose the maximal number of points of leaf node is Sleaf
//树的数量记为Ntree
suppose the number of trees is Ntree
//每棵树需要返回的最大叶子结点个数Nnode
then the maximal node check number is Nnode = P ÷ Sleaf ÷Ntree + 1
//遍历Stree中的树，对每一个树执行深度优先搜索，返回其中离查询最近的Nnode个叶子结点
//并添加到NodeList表中
for all tree i in Stree do
    Depth-first search i for top Nnode closest leaf nodes according to respective tree search criteria, add to NodeList
end for
//把NodeList中的叶子结点中的数据点添加到候选集C中
add the points belonging to the nodes in NodeList to C
//保留候选集C中离查询点最近的E个数据点
keep E points in C which are closest to q. 
//迭代I次贪婪搜索
while iter < I do
    candidate set CC = ∅
    //遍历候选集C中的数据点
    for all point n in C do
        //对于点n,它在kNN图中的邻居集为Sn
        Sn is the neighbors of point n based on G.
        //遍历Sn中的数据点
        for all point nn in Sn do
            //如果nn还没被检查
            if nn hasn’t been checked then
                //把nn添加到CC中
                put nn into CC.
            end if
        end for
    end for
    //把CC集中的点添加到候选集C中并保留离查询点最近的P个数据点
    move all the points in CC to C and keep P points in C which are closest to q.
    iter = iter + 1
end while
return ANNS as the closet K points to q in C.
```

注：作者的实验表明，迭代次数 $I=4$ 就足够了。

## 参考文献
>Fu C , Cai D . EFANNA : An Extremely Fast Approximate Nearest Neighbor Search Algorithm Based on kNN Graph[J]. 2016.