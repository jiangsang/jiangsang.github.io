---
layout: post
title: "数据结构中的一些递归运用"
categories:
 - 数据结构
tags:
 - 递归
 - 二叉树
---

### 递归的两个必要条件

- 存在限制条件，当满足这个条件时，递归便不再继续。
- 每次递归调用之后越来越接近这个限制条件。
<!-- more -->
### 获取二叉树的深度

```c
int getDepth(BTNode *p)
{
    int LD,RD;
    if(p==NULL)
        return 0;
    else
    {
        LD=getDepth();//返回左子树深度
        RD=getDepth();//返回右子树深度
        return (LD>RD? LD:RD)+1;//返回左右子树深度的最
    }
}
```

### 查找二叉树中结点值为key的个数

```c
int searchkey(BTNode *p,int key)
{
    if(p!=NULL)
    {
        if(p->data==key)//若找到了相等的结点值,结点数+1，+1操作是递归关键
            return searchkey(p->lchild,key)+searchkey(p->rchild,key)+1;
        else
            searchkey(p->lchild,key)+searchkey(p->rchild,key);//未找到继续遍历子结点
    }
    return 0;
}
```

### 后序遍历求中序表达式的值

```c
int comp(BTNode *p)
{
    int A,B;
    if(p!=NULL)
    {
        if(p->lchild!=NULL&&p->rchild!=NULL)
        {
            A=comp(p->lchild);
            B=comp(p->rchild);
            return op(A,B,p->data);//计算A、B子树的值,p->data为运算符
        }
        else
            return p->data-'0';//将字符型数字转为整型数字
    }
    else
        return 0;
}
int op(int a,int b,char c)//四则运算函数
{
    if(c=='+')
        return a+b;
    else if(c=='-')
        return a-b;
    else if(c=='*')
        return a*b;
    else if(c=='/')
        return a/b;
}
```


### 二叉树中，输出根结点到每个叶结点的路径（采用二叉链存储结构）

```c
int i;
int top=0;
char pathstack[Maxsize];//保存路径的栈
void allpath(BTNode *p)
{
    if(p!=NULL)
    {
        pathstack[top]=p->data;
        ++top;
        if(p->lchild==NULL&&p->rchild==NULL)//若当前结点为叶子结点，则进行路径打印
        {
            for(i=0;i<top;++i)
                printf("%s",pathstack[i]);
        }
        allpath(p->lchild);//递归左孩子
        allpath(p->rchild);//递归右孩子
        --top;//所访问结点出栈
    }
}
```

###  获得二叉搜索树中的众数

```c
void inOrder(TreeNode* root, TreeNode*& pre, int& curTimes, 
    int& maxTimes, vector<int>& res){
    if (!root) return;
    inOrder(root->left, pre, curTimes, maxTimes, res);
    if (pre)
        curTimes = (root->val == pre->val) ? curTimes + 1 : 1;
    if (curTimes == maxTimes)
        res.push_back(root->val);
    else if (curTimes > maxTimes){
        res.clear();
        res.push_back(root->val);
        maxTimes = curTimes;
    }
    pre = root;
    inOrder(root->right, pre, curTimes, maxTimes, res);
}
vector<int> findMode(TreeNode* root) {
    vector<int> res;
    if (!root) return res;
    TreeNode* pre = NULL;
    int curTimes = 1, maxTimes = 0;
    inOrder(root, pre, curTimes, maxTimes, res);
    return res;
}
```

> 思路：二叉搜索树的中序遍历是一个升序序列，逐个比对当前结点(root)值与前驱结点（pre)值。更新当前节点值出现次数(curTimes)及最大出现次数(maxTimes)，更新规则：若`curTimes=maxTimes`,将root->val添加到结果向量(res)中；若`curTimes>maxTimes`，清空res,将root->val添加到res,并更新maxTimes为curTimes。

