---
layout: default
title:  "左式堆的合并"
date:   2016-03-11 17:50:00
categories: main
---

二叉堆对于合并操作是困难的，因为需要把一个数组拷贝到另一个数组。

左式堆可以**高效的地支持合并**操作, 左式堆与二叉树之间唯一区别是，左式堆不是平衡的，可能非常趋向不平衡。

```C
// 左式堆的结构
typedef struct TreeNode {
    element_t element;
    struct TreeNode *left;
    struct TreeNode *right;
    int npl;
} LeftistHeap;
```
任一节点X的零路径长Npl(X)定义为从X到一个没有两个儿子的节点的最短路径。
左式堆的性质：对于堆中每一个节点X，左儿子的零路径长至少与右儿子的零路径长一样大，即

```C
X->left->npl >= X->right->npl;
```

可以这样理解
```C
求Npl
int Npl(LeftistHeap X) { 
    if (!X) // 节点为NULL时，Npl为-1；
        return -1;
    else if (X->left && X->right) // 节点左右儿子都存在时，返回右儿子的零路径（根据性质）
        return X->right->npl + 1;
    else 
        return X->right->npl; // 只有左儿子或者没有左右儿子时返回右儿子的零路径
}
```
有一个左式树的定理
> 在右路径上有r个节点的左式树必然有 $2^r - 1$ 个节点 


首先右路径是这棵树的最右边的那一条路径（不是右子树的节点，摔，在这里犯混了），我们用递归的思想去验证，
考虑最右边的节点，**根据性质一棵树的右节点必定有左节点与之匹配**，所以这棵树至少有 `r` 个（可能左兄弟还有左儿子）左儿子，
所以树上至少有 $2^r - 1$个节点。


现在开始思考左式堆的合并操作了，那么肯定就要用到它的性质
> X->left->npl >= X->right->npl; 

既然每次都是左边大于等于右边，如果左子树的深度远大于右子树，那么对左子树的处理会很复杂，
所以对每次合并操作都是**对树的右节点进行操作，朝着右路径进行**，总结起来就是两棵树先从树根开始比较，值更大的节点成为值更小的节点的右儿子，
同时满足小根堆的特点（儿子比父节点的值更大，反之则反），终止条件是一棵树到达了两棵树的右路径的最后一个节点，中间过程有左儿子npl小于右儿子npl
的时候，交换左右儿子。

```C
// 合并左式堆的驱动例程（就是控制该把哪棵树合并到另外一棵树下）

LeftistHeap Merge(LeftistHeap h1, LeftistHeap h2) {
    if (!h1) // 当一个树为空树时返回另外一棵树
        return h2;
    if (!h2)
        return h1;
    if (h1->element < h2->element) // 把值大的节点所在的树合并到值小的节点所在树
        return merge(h1, h2);
    else 
        return merge(h2, h1);
}

// 合并的实际例程
static LeftistHeap merge(LeftistHeap h1, LeftistHeap h2) {
    if (!h1->left) 
        h1->left = h2; // 为满足左式堆性质
    else {
        h1->right = Merge(h1->right, h2); // 朝值小的节点所在树的右路径方向进行
        if (h1->left->npl < h1->right->npl) // 当左子树的npl小于右子树的npl时，交换左右儿子
            swap(h1->left, h1->right);
        h1->npl = h1->right->npl + 1;   // 更新当前节点的npl
    }
    return h1;
}
```
左式堆的插入操作为创建一个节点即只有一个根节点的树与需要插入的树进行Merge操作。
删除操作，最小（大）的元素处在树根的位置，只需要把根节点删除就可以，然后对左右子树进行Merge操作。

// 前几天看到网上的很多博客都一个样子，定下心来自己理解后，写下这篇博客, 没有把所有代码都贴上来，把主要的思想和部分代码写在这里。

如果有问题或者哪里错误的可以联系：definezxh@163.com
