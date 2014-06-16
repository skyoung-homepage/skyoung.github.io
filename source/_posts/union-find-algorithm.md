---
title: 利用不相交集实现等价元素的聚类
date: '2013-08-09'
categories: ['Computer Vision']
tags: [OpenCV, Union find]

---


说到聚类，相信大家最先想到的应该是k-means，但是我们知道k-means必须指定聚类的个数，而且聚类初始点的选取也很大影响最后聚类的效果。虽然有一些方法（如k-means++）可以设置较为合理的初始聚类点，但仍然需要指定聚类的个数，但有时我们并不知道一堆数据中有多少个类，这样就使得聚类变得没法下手。这里介绍一种利用一种树型数据结构——不相交集，实现的数据聚类方法。

不相交集（[disjoint-set](http://en.wikipedia.org/wiki/Union_find)），也叫并査集（union-find）是保持一组不相交的动态集合。给定一系列数据，我们首先假设每个元素是一个集合，通过集合的查找合并实现等价元素的合并，从而实现相同类别的元素的聚类。这里两个元素等价的标准是由自己设定，取决于具体应用。例如目标检测中滑窗检测目标时用于聚合方框，其等价标准就是方框的重叠率的大小；再比如图像处理中联通区域标记的应用，其等价标准就是两个点是否联通。
<!--more-->
###OpenCV中并查算法的实现

OpenCV中的partition函数实现了这一数据分类的功能，具体的函数声明如下：
```
template<typename _Tp, class _EqPredicate>
int partition (const vector< _Tp > &  vec, vector< int > &  labels, _EqPredicate  predicate =_EqPredicate())
```

其中，

- **vec**是输入数据集
- **lables**是每个元素对应的标记类别
- **predicate**是等价标准，这是一个指向函数对象的指针，和C++中泛型算法中的谓词函数是一样，可以使用普通的bool函数做判断，也可以使用函数对象，即写个类，里面包含bool operator()(const _Tp& a, const _Tp& b)。


下面做个简单的示例：
当使用bool函数的时候，调用如下，***注意equivalence没有括号***
```
bool equivalence(const int &a, const int &b)
{
	return a == b;
}

cv::partition(data,labels,equaliance);
```

当使用函数对象的时候，调用如下，这次***equivalence有括号***，如果需要传入参数的话，可以在类里面写个私有变量，传入即可。
```
class equivalence
{
public:
	bool operator()(const int &a, const int &b)
	{
		return a == b;
	}
};

cv::partition(data,labels,equaliance());
```

在介绍partition源代码之前，先大体讲一下不相交集数据结构的一种较快的实现方式——不相交集森林。用有根树来表示一个集合，树上的每个节点都是集合中的元素，每棵树代表一个集合。通过引入两种启发式策略，可以实现目前已知最快的并查算法。具体的介绍可以参考[Thomas H. Cormen](http://www.cs.dartmouth.edu/~thc/)等人的《算法导论》第二十一章的内容。此外除了opencv中有这个算法的实现外，[Boost](http://www.boost.org/)中也有相关的实现，可以参考[Disjoint Sets](http://www.boost.org/doc/libs/1_54_0/libs/disjoint_sets/disjoint_sets.html)。

####两种启发式策略：

**按秩合并**：是指两棵树在合并的时候秩小的树的根指向值大的树的根，当两棵树秩相同的时候，任选一棵树指向另一棵树，同时秩加一。这里秩是指树的高度。

**压缩路径**：是指使查找路径上的每个节点都直接指向根节点，但该操作不改变秩的大小，因为秩还要用到树的合并中。

OpenCV中partition的实现在opencv_core工程下的operations.hpp文件下。下面贴出核心代码，并作了详细注释：
```
template<typename _Tp, class _EqPredicate>
int partition( const vector<_Tp>& _vec, vector<int>& labels,
          _EqPredicate predicate=_EqPredicate())
{
	int i, j, N = (int)_vec.size();
   	const _Tp* vec = &_vec[0];
	
	//存放每个节点的父节点位置index
   	const int PARENT=0;
	//存放节点秩的index
   	const int RANK=1;
    
	//每个节点有两个元素：父节点和节点的秩
   	vector<int> _nodes(N*2);
   	int (*nodes)[2] = (int(*)[2])&_nodes[0];
    	// The first O(N) pass: create N single-vertex trees
	//初始时，把每个元素看成一个集合，即一棵树
    	for(i = 0; i < N; i++)
   	{
	    //初始时，父节点为-1，秩为0
    	    nodes[i][PARENT]=-1;
   	    nodes[i][RANK] = 0;
    	}
	
    	// The main O(N^2) pass: merge connected components
   	for( i = 0; i < N; i++ )
   	{
   	    int root = i;

   	    // find root
	    //找的节点所在树的根节点
   	    while( nodes[root][PARENT] >= 0 )
   	        root = nodes[root][PARENT];
	
  	    for( j = 0; j < N; j++ )
    	    {
		//对除i元素以外的其他元素，判断两元素是否等价
 	        if( i == j || !predicate(vec[i], vec[j]))
   	            continue;
  	        int root2 = j;
			
		    //找到该等价元素的所在树的根节点
    	        while( nodes[root2][PARENT] >= 0 )
   	            root2 = nodes[root2][PARENT];
			
		    //判断两个元素根节点是否相同，即两元素是否是同一集合
   	        if( root2 != root )
   	        {
   	            // unite both trees
		        //如果不是同一集合，则按秩合并
   	            int rank = nodes[root][RANK], rank2 = nodes[root2]	[RANK];
		        //秩小的树的根指向秩大的树的根
		        //如果秩相等，就任意选一根指向另一根，同时秩加一
   	            if( rank > rank2 )
    	                nodes[root2][PARENT] = root;
				
   	            else
   	            {
   	                nodes[root][PARENT] = root2;
   	                nodes[root2][RANK] += rank == rank2;
	                root = root2;
    	            }
		        //判定此时的root是树的根节点
   	            assert( nodes[root][PARENT] < 0 );
	                int k = j, parent;

                // compress the path from node2 to root
			        // 压缩j处节点的路径，沿着j处的节点向上逐一指向根节点
                while( (parent = nodes[k][PARENT]) >= 0 )
                {
                    nodes[k][PARENT] = root;
                    k = parent;
	        }

                // compress the path from node to root
		        //压缩i处节点的路径，沿着i处节点向上逐一指向根节点
                k = i;
                while( (parent = nodes[k][PARENT]) >= 0 )
                {
                    nodes[k][PARENT] = root;
                    k = parent;
                }
            }
	}
    }

    // Final O(N) pass: enumerate classes
    labels.resize(N);
    int nclasses = 0;

    //对每个元素找到它的根节点，然后重新利用rank，
    //把rank赋值为集合的类别label
    for( i = 0; i < N; i++ )
    {
        int root = i;
	    //查找根节点
        while( nodes[root][PARENT] >= 0 )
	         root = nodes[root][PARENT];
        // re-use the rank as the class label
        if( nodes[root][RANK] >= 0 )
	        //此处设为负值是为了表明该节点的rank已经赋值为label了
            nodes[root][RANK] = ~nclasses++;
	    //root相同的节点赋值同样的label
        labels[i] = ~nodes[root][RANK];
    }
	
    return nclasses;
}
```

###总结

相比传统的k-means聚类方法，该方法具有不需要指定聚类个数的优势。针对要分类的数据集，如果你事先知道元素间相似性的判断准则，像重叠矩形框的聚合，那么用并查算法是比较方便的。我的理解是k-means的聚类方法不需要知道元素确切的属于同一类元素的相似性准则，而是通过该元素与每个集合的中心点的距离远近判定元素的集合归属。
