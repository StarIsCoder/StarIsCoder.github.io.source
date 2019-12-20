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

除了递归能遍历二叉树以外，我们还可以用stack来实现二叉树的遍历

```java
public static void visitWithStackPreorder(TreeNode root) {
    Stack<TreeNode> stack = new Stack<>();
    stack.push(root);
    while (!stack.empty()) {
        TreeNode tmp = stack.pop();
        System.out.println(tmp.val);
        if (tmp.right != null) {
            stack.push(tmp.right);
        }
        if (tmp.left != null) {
            stack.push(tmp.left);
        }

    }
}

public static void visitWithStackInorder(TreeNode root) {
    Stack<TreeNode> stack = new Stack<>();
    stack.push(root);
    TreeNode tmp = stack.pop();
    while (!stack.empty() || tmp != null) {
        while (tmp != null) {
            stack.push(tmp);
            tmp = tmp.left;
        }
        tmp = stack.pop();
        System.out.println(tmp.val);
        tmp = tmp.right;
    }
}

public static void visitTreeWithStackPostorder(TreeNode root) {
    Stack<TreeNode> stack = new Stack<>();
    TreeNode cur;
    cur = root;
    while (cur != null || !stack.empty()) {
        if (cur != null) {
            stack.push(cur);
            cur = cur.left;
        } else {
            TreeNode tmp = stack.peek().right;
            //检查右边是否还有节点
            if (tmp == null) {
                tmp = stack.pop();
                System.out.println(tmp.val);
                while (!stack.empty() && tmp == stack.peek().right) {
                    tmp = stack.pop();
                    System.out.println(tmp.val);
                }
            } else {
                cur = tmp;
            }
        }
    }
}
```

除了stack还有一种叫Morris Traversal的遍历方式，这种方法连stack都不需要

```java
public static void morrisPreorderTraverse(TreeNode root) {
    TreeNode cur = root;
    while (cur != null) {
        if (cur.left == null) {
            System.out.println(cur.val);
            cur = cur.right;
        } else {
            TreeNode pre = cur.left;
            while (pre.right != null && pre.right != cur) {
                pre = pre.right;
            }

            if (pre.right == cur) {
                pre.right = null;
                cur = cur.right;
            } else {
                pre.right = cur;
                System.out.println(cur.val);
                cur = cur.left;
            }
        }
    }
}
```



参考文章：http://www.learn4master.com/algorithms/morris-traversal-inorder-tree-traversal-without-recursion-and-without-stack