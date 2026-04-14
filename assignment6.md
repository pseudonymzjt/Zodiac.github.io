# DSA Assignment #6: 🌲（1/3）

*Updated 2026-04-05 21:54 GMT+8*
 *Compiled by <mark>肖云天、生命科学学院</mark> (2026 Spring)*
{{date}}

## 1. 题目

### <font color='skyblue'>E94.二叉树的中序遍历</font>

dfs, stack, https://leetcode.cn/problems/binary-tree-inorder-traversal/

思路：



代码：

```python
# Definition for a binary tree node.  
# class TreeNode:  
#     def __init__(self, val=0, left=None, right=None):  
#         self.val = val  
#         self.left = left  
#         self.right = right  
class Solution:  
    def inorderTraversal(self, root: Optional[TreeNode]) -> List[int]:  
        res=[]  
        def dfs(node):  
            if not node:  
                return  
            dfs(node.left)  
            res.append(node.val)  
            dfs(node.right)  
        dfs(root)  
        return res
```



代码运行截图 <mark>（至少包含有"Accepted"）</mark>

![[Pasted image 20260409135209.png]]



### E108.将有序数组转换为二叉搜索树

https://leetcode.cn/problems/convert-sorted-array-to-binary-search-tree/
Given an integer array `nums` where the elements are sorted in **ascending order**, convert _it to a_ **_height-balanced_** _binary search tree_.

思路：
在leetcode中，所有的tree都是由root来代表的。
递归来做
代码：

```python
class Solution:  
    def sortedArrayToBST(self, nums: List[int]) -> Optional[TreeNode]:  
        def dfs(left,right):  
            if left>right: #如果left=right的话，应该执行创建节点并返回节点的操作。只有l>r无点可选时，才应该return None.  
                return None  
            mid=(left+right)//2  
            node = TreeNode(nums[mid])  
            node.left=dfs(left,mid-1)  
            node.right=dfs(mid+1,right)  
            return node  
        return dfs(0,len(nums)-1)
```



代码运行截图 <mark>（至少包含有"Accepted"）</mark>
![[Pasted image 20260409140737.png]]




### M102.二叉树的层序遍历

bfs, https://leetcode.cn/problems/binary-tree-level-order-traversal/
Given the `root` of a binary tree, return _the level order traversal of its nodes' values_. (i.e., from left to right, level by level).
思路：
**优化方法：**
考虑如何优化空间开销：如何不用哈希映射，并且只用一个变量 node 表示状态，实现这个功能呢？
我们可以用一种巧妙的方法修改广度优先搜索：
首先根元素入队
当队列不为空的时候
求当前队列的长度 s_ i
依次从队列中取 s_ i个元素进行拓展，然后进入下一次迭代
它和普通广度优先搜索的区别在于，普通广度优先搜索每次只取一个元素拓展，而这里每次取 s_ i个元素。在上述过程中的第 i 次迭代就得到了二叉树的第 i 层的 s_ i个元素。

代码：

```python
from collections import deque  
class Solution:  
    def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:  
        stack=[]  
        queue=deque()  
        queue.append((root,0))  
        res=[]  
        while queue:  
            node,floor=queue.popleft()  
            if not node:  
                continue  
            if node.left:  
                queue.append((node.left,floor+1))  
            if node.right:  
                queue.append((node.right,floor+1))  
            if queue and queue[0][1]==floor:  
                stack.append(node.val)  
            else:  
                stack.append(node.val)  
                res.append(stack[:])  
                stack.clear()  
        return res
```

%%optimized edition%%
~~~python
from collections import deque  
class Solution:  
    def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:  
  
        queue=deque()  
        queue.append(root)  
        res=[]  
        if not root:  
            return res  
        while queue:  
            size=len(queue)  
            level=[]  
            for _ in range(size):  
                node=queue.popleft()  
                level.append(node.val)  
                if node.left:  
                    queue.append(node.left)  
                if node.right:  
                    queue.append(node.right)  
            res.append(level[:])  
        return res
~~~


代码运行截图 <mark>（至少包含有"Accepted"）</mark>
![[Pasted image 20260410142507.png]]




### <font color='pink'>M1123.最深叶节点的最近公共祖先(LCA)</font>

dfs, https://leetcode.cn/problems/lowest-common-ancestor-of-deepest-leaves/


Given the `root` of a binary tree, return _the lowest common ancestor of its deepest leaves_.

Recall that:

- The node of a binary tree is a leaf if and only if it has no children
- The depth of the root of the tree is `0`. if the depth of a node is `d`, the depth of each of its children is `d + 1`.
- The lowest common ancestor of a set `S` of nodes, is the node `A` with the largest depth such that every node in `S` is in the subtree with root `A`.
思路：
Do a postorder traversal.
Then, if both subtrees contain a deepest leaf, you can mark this node as the answer (so far).
**The final node marked will be the correct answer.**
由于dfs具有从下向上的特性，因此上层的可行解一定会在下层的可行解之后被标出。最后标出的可行解就是最大的包含解的结构（左和右中都有最深叶）;如果再往上的话就不是最近的共同祖先了（是某一个正确解的父亲；一定只有一边有deepest leaf）

一战错误：
**没有比较深度**
后序遍历找最深叶子节点的LCA，**核心是比较左右子树的最大深度**。你的代码完全没有记录或比较深度信息。

**因此要判断左子树和右子树的深度：**
如果某一边更深的话，就说明所有的深节点都在这一边，答案应该是在这一边子树内部；
只有两边一样深的话，才说明深节点（深叶）在两边都有分布，是可行解。

代码：

```python
class Solution:  
    def lcaDeepestLeaves(self, root: Optional[TreeNode]) -> Optional[TreeNode]:  
        def dfs(node):  
            if not node:  
                return (None,0)  
            left,left_depth=dfs(node.left)  
            right,right_depth=dfs(node.right)  
            if left_depth > right_depth:  
                return (left,left_depth+1)#向上传递答案，node这个子树的LCA在子树的左子树内部，深度为l_d+1  
            elif right_depth > left_depth:  
                return (right,right_depth+1)  
            else:  
                return (node,right_depth+1)  
        return dfs(root)[0]
```



代码运行截图 <mark>（至少包含有"Accepted"）</mark>
![[Pasted image 20260413221813.png]]




### M07161: 森林的带度数层次序列存储

tree, http://cs101.openjudge.cn/practice/07161/
输入

输入数据的第一行包括一个正整数n，表示森林中非空的树的数目。  
随后的 n 行，每行给出一棵树的带度数的层次序列。  
树的节点名称为A-Z的单个大写字母。

输出

输出包括一行，输出对应森林的后**序**遍历序列。
思路：
在 Python 中，**默认参数只在函数定义时计算一次**，而不是每次调用时创建新的。这意味着：


```python
# 错误示例
class TreeNode:
    def __init__(self, children=[]):  # 这个 [] 只创建一次
        self.children = children
# 创建两个节点
node1 = TreeNode()
node2 = TreeNode()
node1.children.append(1)
print(node1.children)  # [1]
print(node2.children)  # [1] ← 被影响了！两个实例共享同一个列表！

```
正确做法：
```python

class TreeNode:
    def __init__(self, val=None, degree=None, children=None):
        self.val = val
        self.degree = degree
        self.children = children if children is not None else []  # 每次创建新列表
```

#### 为什么这样修复？

- `children=None` 是不可变对象，安全
    
- 在函数体内判断：如果没传 `children`，就创建一个**新的空列表**
    
- 每个实例都有自己的列表，互不干扰

**不要用可变对象（列表、字典、集合）作为默认参数！**
代码

```python
from collections import deque  
class TreeNode:  
    def __init__(self, val=None,degree=None,children=None):  
        self.val = val  
        self.children= children if children is not None else []  
        self.degree = degree  
def main():  
    n=int(input().strip())  
    res=[]  
    for _ in range(n):  
        ls=list(input().strip().split())  
        nodelist=[]  
        for i in range(0,len(ls),2):  
            nodelist.append(TreeNode(val=ls[i],degree=int(ls[i+1])))  
        if not nodelist:  
            print()  
            continue  
        q=deque()  
        q.append(nodelist[0])  
        i=1  
        while i<len(nodelist) :  
            if not q:  
                q.append(nodelist[i])  
                i+=1  
            if len(q [0].children)<q[0].degree:  
                q[0].children.append(nodelist[i])  
                q.append(nodelist[i])  
                i += 1  
            else:  
                q.popleft()  
  
        def dfs(node):  
  
            if not node:  
                return  
            for child in node.children:  
                dfs(child)  
            res.append(node.val)  
            return  
        dfs(nodelist[0])  
    print(*res)  
if __name__ == '__main__':  
    main()
```



<mark>（至少包含有"Accepted"）</mark>
![[Pasted image 20260414143757.png]]




### M27928: 遍历树

 adjacency list邻接表, dfs, http://cs101.openjudge.cn/practice/27928/
implementation
思路：
请你对输入的树做遍历。遍历的规则是：遍历到每个节点时，按照该节点和所有子节点的值从小到大进行遍历.
临接表在处理这道题时还是挺便利的！


代码

```python
  
a=dict()  
n=int(input().strip())  
occurred_child=set()  
occurred_node=set()  
for _ in range(n):  
    sample=list(map(int,input().strip().split()))  
    a[sample[0]]=sample[1:] if len(sample)>1 else []  
    for i in range(len(sample)):  
        occurred_node.add(sample[i])  
        if i>=1:  
            occurred_child.add(sample[i])  
root=(occurred_node-occurred_child).pop()  
#find the root 👆  
res=[]  
def dfs(node):  
    if not a[node]:  
        res.append(node)  
        return  
    stack=a[node][:]  
    stack.append(node)  
    stack.sort()  
    for n in stack:  
        if n in a[node]:  
            dfs(n)  
        else:  
            res.append(n)  
    return  
dfs(root)  
for __ in res:  
    print(__)
```



<mark>（至少包含有"Accepted"）</mark>
![[Pasted image 20260414151307.png]]




## 2. 学习总结和个人收获

==如果发现作业题目相对简单，有否寻找额外的练习题目，如“数算2026spring每日选做”、LeetCode、Codeforces、洛谷等网站上的题目。==

1、2为dfs
3为bfs,而且由于是一层有多个node, 我们可以采取一次pop出多个值进入stack之后统一return
4题虽然作为LCA有很多种既有方法，但是这里使用的dfs+深度比较已经是一个O(n)的好解法了
5、6题处理输入麻烦一点点，但是其他还好
注意了__init__ 部分中不要将`[]`作为default变量（会指向同一个列表）而是保证每次创建时会单独
创建一个空列表。
正在练习阅读英文题干。
![[Pasted image 20260414152414.png]]
换一个字体？
在不要求左孩子、右孩子的区分时，一般都用邻接表或矩阵来表示。
