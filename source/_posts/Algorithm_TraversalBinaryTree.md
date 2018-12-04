---
title: Traversal Binary Tree
date: 2018/12/4
categories: Algorithm
---

遍历二叉树主要有三种方式
 - 中序(Inorder)
 - 前序(Preorder)
 - 后序(Postorder)

假设有一个这样的二叉树
     5
   /   \
  1     3
 / \   / \
4   2 6   7
 - 前序：5 - 1 - 4 - 2 - 3 - 6 - 7
 - 后序：4 - 2 - 1 - 6 - 7 - 3 - 5
 - 中序：4 - 1 - 2 - 5 - 6 - 3 - 7

然后看下如何用代码实现这些,可以看到区别在于是先遍历还是先处理节点上的数据。
```java
private static void printPostorder(List list, TreeNode node) {
    if (node == null) return;
    printPostorder(list, node.left);
    printPostorder(list, node.right);
    list.add(node.val);
}

private static void printPreorder(List list, TreeNode node) {
    if (node == null) return;
    list.add(node.val);
    printPreorder(list, node.left);
    printPreorder(list, node.right);
}

private static void printInorder(List list, TreeNode node) {
    if (node == null) return;
    printInorder(list, node.left);
    list.add(node.val);
    printInorder(list, node.right);
}
```