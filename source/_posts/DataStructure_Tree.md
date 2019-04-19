---
title: Tree
date: 2019/04/16
categories: DataStructure
---

## 二叉查找树（BST）

二叉树是一棵树，其中每个节点都不能多于两个儿子。其平均深度为O(√N)。

二叉查找树是有特殊性质的二叉树，它的任意一个节点X，X的所有左儿子都比X小，X的所有右儿子都比X大。BST的平均深度是O(logN)。

以下是常用api的简单实现，BST大部分的处理都是采用递归的方式：

### contains

由于BST的性质，只需要和节点X比较，如果比X大，则继续从X的右儿子寻找，否则从X的左儿子寻找

```java
public boolean contains(int data) {
    return contains(data, root);
}

public boolean contains(int data, Node root) {
    if (root == null) {
        return false;
    }
    if (data > root.data)
        return contains(data, root.right);
    else if (data < root.data)
        return contains(data, root.left);
    else
        return true;
}
```

### insert

思路上和contains差不多，沿着树并根据节点大小查找，这儿并不处理重复值的情况。

```java
public Node insert(int data) {
    return insert(data, root);
}

private Node insert(int data, Node root) {
    if (root == null) {
        new Node(data);
    }
    if (data > root.data)
        root.right = insert(data, root.right);
    else if (data < root.data)
        root.left = insert(data, root.left);
    else
        ;
    return root;
}
```

### remove

删除的情况比较复杂，如果节点X是树叶，那么直接删除即可，如果X只有一个儿子那么重新调整下父节点的链接。

![](/assets/DataStructure_Tree/remove_node_one_son_before.png)

![](/assets/DataStructure_Tree/remove_node_one_son_after.png)

如果X有两个儿子的话可以用右子树中最小的节点代替X，由于最小的节点只有一个儿子因此再remove一次会比较容易。如图删除2节点，那么先去2节点的右子树中找到最小的value为3，用3去替代2，再去删除3（这个比较容易）

![](/assets/DataStructure_Tree/remove_node_two_son_before.png)

```java
void deleteKey(int key) { 
    root = deleteRec(root, key); 
} 

Node deleteRec(Node root, int key) { 
    if (root == null)  return root; 
    if (key < root.key) 
        root.left = deleteRec(root.left, key); 
    else if (key > root.key) 
        root.right = deleteRec(root.right, key); 
    else { 
        if (root.left == null) 
            return root.right; 
        else if (root.right == null) 
            return root.left; 
        root.key = minValue(root.right); 
        root.right = deleteRec(root.right, root.key); 
    } 
    return root; 
} 
```



还有种策略叫做懒惰删除，就是在执行删除操作的时候该元素依然留在树种，只是将其标记为删除。



## AVL树

如果向一个BST插入一个已经排序好了的集合，那么将有可能只有左子树和右子树，这样的话就和链表没有区别，因此引入了一个带有平衡条件的BST，条件就是左右子树的高度最多差1。并且所有的树操作都可以以时间O(logN)来执行。

下图就不是AVL树，因为节点2的高度是2，而节点8的高度是0，差为2。

![](/assets/DataStructure_Tree/wrong_avl.png)

对于AVL树来说，插入或者删除都有可能破坏AVL树的平衡性，由于每个节点最多两个儿子，因此一共有四种可能性破坏平衡条件：

1. 左 - 左

2. 左 - 右

3. 右 - 左

4. 右 - 右


其中1、4是对称情形，2、3是对称情形

### 单旋转

#### 左 - 左

```
         z                                      y 
        / \                                   /   \
       y   T4      Right Rotate (z)          x      z
      / \          - - - - - - - - ->      /  \    /  \ 
     x   T3                               T1  T2  T3  T4
    / \
  T1   T2
```

#### 右 - 右

```
  z                                y
 /  \                            /   \ 
T1   y     Left Rotate(z)       z      x
    /  \   - - - - - - - ->    / \    / \
   T2   x                     T1  T2 T3  T4
       / \
     T3  T4
```

### 双旋转

#### 左 - 右

```
     z                               z                           x
    / \                            /   \                        /  \ 
   y   T4  Left Rotate (y)        x    T4  Right Rotate(z)    y      z
  / \      - - - - - - - - ->    /  \      - - - - - - - ->  / \    / \
T1   x                          y    T3                    T1  T2 T3  T4
    / \                        / \
  T2   T3                    T1   T2
```

#### 右 - 左

```
   z                            z                            x
  / \                          / \                          /  \ 
T1   y   Right Rotate (y)    T1   x      Left Rotate(z)   z      y
    / \  - - - - - - - - ->     /  \   - - - - - - - ->  / \    / \
   x   T4                      T2   y                  T1  T2  T3  T4
  / \                              /  \
T2   T3                           T3   T4
```

左 - 左的实现：

```java
Node rotateWithLeftChild(Node y) {
    Node x = y.left;
    y.left = x.right;

    // Perform rotation
    x.right = y;

    // Update heights
    y.height = max(height(y.left), height(y.right)) + 1;
    x.height = max(height(x.left), y.height) + 1;

    // Return new root
    return x;
}
```

左- 右的实现：先将其变为左 - 左的情况，在执行左 - 左的旋转

```java
private Node doubleWithLeftChild(Node k3) {
    k3.left = rotateWithRightChild(k3.left);
    return rotateWithLeftChild(k3);
}
```

为了检查是否需要执行平衡操作需要在Node增加一个成员变量height

```java
class Node {
    int key, height;
    Node left, right;

    Node(int d) {
        key = d;
        height = 1;
    }
}
```

因此，最后平衡的操作实现：

```java
private Node balance(Node t) {
    if (t == null) {
        return t;
    }

    if (height(t.left) - height(t.right) > 1) {
        if (height(t.left.left) >= height(t.left.right)) {
            t = rotateWithLeftChild(t);
        } else {
            t = doubleWithLeftChild(t);
        }
    } else {
        if (height(t.right) - height(t.left) > 1) {
            if (height(t.right.right) >= height(t.right.left)) {
                t = rotateWithRightChild(t);
            } else {
                t = doubleWithRightChild(t);
            }
        }
    }
    t.height = Math.max(height(t.left), height(t.right)) + 1;
    return t;
}
```



### 伸展树

假设一棵树存了百万级别的key，但其中只有20%是经常用到的，如果用AVL树的话时间为O(logN)，但是用伸展树的可能只需要O(1)。伸展树的基本思想是当一个节点被访问后，它会经过一系列的旋转被推到root上，这样下次再访问这个节点的话会更快，当然如果由于节点X被推到root上，而导致其他的节点依然处于较深的位置，这样并没有提升多少效率，因此伸展树也需要能平衡这棵树。