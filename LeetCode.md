# 算法归类

## 1、双指针

采用p,q，借助栈，可以完成（）的匹配，接雨水、算容量；

其中，借助栈的时候，可以将**下标**存放在**栈**里面。





## 2、回溯+剪枝

有些用**动态规划**比较方便！！！不存在**超时**或者堆栈导致**内存限制**等问题。



### 2.1查找单词是否存在

1、确定起点，

 2、对每个起点做四个方向的判断 ，（二层循环,判断采用递归重复调用）**check(i, j ,0 )** 从（i, j）位置开始的word[0]及以后是否都相等

3、剪枝，有一个不相等，一定不相等，匹配完了要退出；



### 2.2 [二叉树中和为某一值的路径](https://leetcode.cn/problems/er-cha-shu-zhong-he-wei-mou-yi-zhi-de-lu-jing-lcof/)

1、递归遍历，先访问，遍历左子树，再遍历右子树

2、终止条件，指针为空

3、何时存储数据，当它和最后一个要求的数相等时，且为叶子节点，存储

```c++
int x=p->val;
a.push_back(x);

if(p->left==nullptr && p->right==nullptr && target==x )  
{
    ans.push_back(a);
}

find(p->left,target-x,a);
find(p->right,target-x,a);
a.pop_back(); 
```

### 2.3 [字符串的排列](https://leetcode.cn/problems/zi-fu-chuan-de-pai-lie-lcof/)

1、确定递归公式，包含重复操作的是什么，以及递归出口（为什么是考虑n，因为每次找一个字母，找几次就是出口）

2、怎样的情况调用递归

3、借助外在条件

```C++
//伪代码
void find(const string s,vector<bool> &visited,string &a,int n)//引用传递，保持状态还原
{
    if(n==0) //递归出口，存储数据
    {
        ans.insert(a);
        return;
    }

    for(int i=0;i<s.size();i++) //因为是排列，所以遍历每个字母开头
    {
        if(!visited[i])//但是因为字母用过了不能重复，借助是否访问过来判断
        {
            a.push_back(s[i]);
            visited[i]=true;
            find(s,visited,a,n-1); //递归调用，求得下一个字母，下个字母不能重复
            visited[i]=false; //递归结束后还原状态，以便for循环下一次状态初始化
            a.pop_back();
        }
    }
}
```

## 3、递归

输入一个整数数组，判断该数组是不是某**二叉搜索树**的**后序遍历**结果。如果是则返回 `true`，否则返回 `false`。假设输入的数组的任意两个数字都互不相同。（左小右大）



单调栈实现：https://leetcode.cn/problems/er-cha-sou-suo-shu-de-hou-xu-bian-li-xu-lie-lcof/solution/dan-diao-zhan-si-lu-c-shi-xian-ji-bai-10-msjk/

```c++
class Solution {
public:
    bool solve(vector<int>& postorder,int i,int j,int root)
    {
        if(i==j)  return true;//只有一个元素时，结束
        //i,j为左闭右开
        int x=i;
        while(x<j && postorder[x]<=root)
        {
            x++;
        }
        int mid=x;
        while( x<j && postorder[x]>root)
        {
            x++;
        }
        // cout<<x<<" "<<j<<endl;
        if(x!=j) return false; 	//不满足所有的左小右大
        else
        {
            bool f1,f2;
            // cout<<i<<" "<<mid<<" "<<postorder[mid-1]<<endl;
            if(mid-1>=i)
                f1=solve(postorder,i,mid-1,postorder[mid-1]); //注意边界值，谨防越界
            else f1=true;//否则是空树

            // cout<<mid<<" "<<j<<" "<<postorder[j]<<endl;
            if(j-1>=mid)
                f2=solve(postorder,mid,j-1,postorder[j-1]);
            else f2=true;
            if( f1 && f2 )   //左右子树都符合条件，那么成立
                return true;
        }
        return false;  
    }
    bool verifyPostorder(vector<int>& postorder) {
        int len=postorder.size();
        if(len==0)  return true; //考虑树空的情况
        return solve(postorder,0,len-1,postorder[len-1]);
    }
};
```

