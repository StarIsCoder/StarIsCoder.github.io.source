---
title:  Search  Algorithm
date: 2019/03/15
categories: Algorithm
---

## 二分法
二分查找有两种，递归和循环。

时间复杂度：

- O(Logn)

```java
 private int binarySearchIteration(int input[], int target) {
        int l = 0;
        int r = input.length - 1;
        while (l <= r) {
            int tmp = (l + r) / 2;
            int value = input[tmp];
            if (value == target) {
                return tmp;
            }
            if (value > target)
                r = tmp - 1;
            else
                l = tmp + 1;
        }
        return -1;
    }

    private int binarySearchRecursive(int input[], int target) {
        return binarySearchRecursive(input, 0, input.length - 1, target);
    }

    private int binarySearchRecursive(int input[], int l, int r, int target) {
        if (l <= r) {
            int tmp = (l + r) / 2;
            if (input[tmp] == target) return tmp;
            if (input[tmp] > target)
                return binarySearchRecursive(input, l, tmp - 1, target);
            else
                return binarySearchRecursive(input, tmp + 1, r, target);
        }
        return -1;
    }
```

这个二分法是假设数组中没有重复的元素，如果有重复的元素并且希望能找到第一个或者最后一个index的话，这个算法就需要修改一下，当然在target大于或者小于`input[tmp]`的时候low和high的指针变化是一样的，区别在于`target == input[tmp]`的时候，这个tmp是否是最小的index，因此我们需要在判断下`input[tmp -1]`的值是否依然和target相等，因为二分法的数组都是排序好的，如果`input[tmp -1]`和target不相等，那说明确实找到了最小的index，如果相等那么在`tmp`之前还存在着更小的index。对最大值来说反之亦然。
```java
int firstIndex(int[] input, int target) {
    int l = 0;
    int r = input.length - 1;
    while (l <= r) {
        int mid = (l + r) / 2;
        //key code
        if (input[mid] == target) {
            if (mid == 0) {
                return mid;
            } else if (input[mid - 1] != target) {
                return mid;
            } else {
                r = mid - 1;
            }
        } else if (target > input[mid]) {
            l = mid + 1;
        } else {
            r = mid - 1;
        }
    }
    return -1;
}

int lastIndex(int[] input, int target) {
    int l = 0;
    int r = input.length - 1;
    while (l <= r) {
        int mid = (l + r) / 2;
        //key code
        if (input[mid] == target) {
            if (mid == input.length - 1) {
                return mid;
            } else if (input[mid + 1] != target) {
                return mid;
            } else {
                l = mid + 1;
            }
        } else if (target > input[mid]) {
            l = mid + 1;
        } else {
            r = mid - 1;
        }
    }
    return -1;
}
```