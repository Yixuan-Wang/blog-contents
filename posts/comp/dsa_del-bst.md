---
title: 二叉搜索树的删除结点操作
date: 2020-09-18T10:48:00+08:00
category: comp
tags:
  - dsa
keywords:
  - bst
math: true
---

二叉搜索树删除结点的操作实际上是一种旋转。

<!-- more -->

  <div style="text-align:center;">
  <strong><del>生活在二叉树上</del></strong> <br/>
  P市一辣鸡
  </div>

用二叉树的数据结构体现存取增减最优都能达到$O(\log n)$的超越性，保持婞直却又不拘泥于所谓“顺序表二分查找”的单向度形象。这便是二叉搜索树为我们提供的理想期望数据结构范式。~~生活在二叉树上——始终热爱全序——升上根结点~~。

## 二叉搜索树

回归正题。数据结构与算法 B 课程期末考试考到了二叉搜索树删除结点，自觉答得不够清晰完整，这里补充上，算是填补一下遗憾。

二叉搜索树是**中序遍历权值序列单调不降**（或不增）的二叉树。任意一个结点，左子的权值（若存在，下同）小于等于它，右子的权值大于等于它（反过来当然也可以）。而二叉堆则是任意一个结点，左子和右子权值都大于等于它（或小于等于）。

所以二叉搜索树任意一棵左右子树也都是二叉搜索树。特别无聊地，空树也是二叉搜索树。

## 删除结点

为了维护二叉搜索树的顺序关系，删除一个结点$o$以后，要用左子树$L$最大的结点$l_{r}$（也就是**最右下的结点**）替换$o$。如果没有左子树，也就是图 b 的情况，直接把$R$变为父结点$p$的子树即可。

<div style="display:flex;flex-wrap:wrap;">
<pre style="flex: 1 1;">
[a]
 p
  \
   o
  / \
 L   R
</pre>
&nbsp;
<pre style="flex: 1 1;">
[b]
 p
  \
   o
    \
     R  
</pre>
</div>

```python
# 二叉搜索树类的 delete 方法

def delete(self, key):
    current = self._root # current 根结点，设为当前结点
    fa = None # fa[ther] 当前结点的父结点

    order = self.order # 全序关系函数 order(a, b) -> Bool，二叉搜索树满足 order(左子，自己) == True
    equal = self.equal # 相等关系函数 equal(a, b) -> Bool

    while current != None:
        if equal(key, current.key): # 当前结点就是目标结点
            if current.lch == None: # l[eft]ch[ild] 左子 左子为空
                if fa == None: self._root = current.rch # 若要删除的是根结点，且没有左子树
                elif fa.lch == current: fa.lch = current.rch # 当前是父结点的左子，用当前的右子替换当前结点
                else: fa.rch = current.rch # 当前是父结点右子，同上
            else:
                rightmost = current.lch # rightmost 左子树最大结点，初始化为左子
                faRightmost = current # fa[ther-of-]Rightmost 左子树最大结点的父结点，初始化为当前结点

                while rightmost.rch != None: faRightmost, rightmost = rightmost, rightmost.rch
                # 循环找最右下结点

                if faRightmost == current: faRightmost.lch = rightmost.lch # 当前结点左子的右子树为空，用左子的左子替代左子
                else: faRightmost.rch = rightmost.lch # 最大结点用左子替代

                rightmost.lch = current.lch # 替换最右下结点的子结点引用
                rightmost.rch = current.rch
                if fa == None: self._root = rightmost # 若当前结点是根节点，不用改变父的子结点引用
                elif fa.lch == current: fa.lch = rightmost # 替换当前结点父结点的子结点引用
                else: fa.rch = rightmost
                # 左子树上最大结点替代当前结点

            del current # 删除当前结点
            return
        elif order(key, current.key): fa, current = current, current.lch # 找目标结点
        else: fa, current = current, current.rch
```

## 旋转

课程里没有提及旋转的概念。后来一位前 OIer 朋友（[\@FYH](https://www.cnblogs.com/FYH-SSGSS/)）给我科普，以上删除结点的操作其实涉及到了旋转。

对于以下两棵二叉搜索树：

<div style="display:flex;flex-wrap:wrap;">
<pre style="flex: 1 1;">
[a]
    3*
   / \
  2   4
 /     \
1       5
</pre>
&nbsp;
<pre style="flex: 1 1;">
[b]
 1
  \   
   2
    \
     3*
      \
       4
        \
         5
</pre>
</div>

存储的数据是完全相同的，但是树 a 中搜索的最坏时间复杂度达到$O(\log n)$，树 b 中搜索则达到了$O(n)$。二叉树构造得越“平衡”，在其上进行操作的性能就越好。为了使 b 变得更平衡，可以以标 `*` 的 3 号结点为新的根结点，将它的祖先结点**旋转**为它的左子树，变成树 a。

同样地，二叉搜索树删除结点操作也可以视为以待删除结点左子树的最大结点为中心，将待删除结点的左子树旋转为它的左子树，再将待删除结点的右子树接在旋转的中心上。

## 参考资料

- OI Wiki，[二叉搜索树](https://oi-wiki.org/ds/bst/)
- 裘宗燕，数据结构与算法：Python 语言描述，2018
