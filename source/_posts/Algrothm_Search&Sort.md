---
title: Search & Sort
date: 2019/03/05
categories: Algorithm
---

## 二分法
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