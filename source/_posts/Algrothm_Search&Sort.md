---
title: Search & Sort
date: 2019/03/05
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
## 快速排序
快速排序主要的思想是分而治之，先选取一个基准数，第一次排序的时候将比基准数小的全部移到左边，而比基准数大的全部移到右边。这样就分成了两个数组，然后递归排序即可。

时间复杂度：
- Worst: O(n^2)
- Best: O(nLogn)

下面这种以最后一个值作为基准值。假设原数组为`10, 7, 8, 9, 1, 5`

第一次排序：指针为low - 1也就是-1，当找到比5小的元素时，也就是1，就将指针右移一位，并且与1交换。

当之后再也找不到比5小的元素的时候就将指针+1的索引和基准值交换

![](/assets/Algorith_SearchAndSort/quick_sort.jpg)
```java
public int partition(int input[], int low, int high) {
    int pivot = input[high];
    int i = low - 1;
    for (int j = low; j < high; j++) {
        if (input[j] < pivot) {
            i++;

            int tmp = input[i];
            input[i] = input[j];
            input[j] = tmp;
        }
    }
    int tmp = input[i + 1];
    input[i + 1] = input[high];
    input[high] = tmp;
    return i + 1;
}

public void quickSort(int input[], int low, int high) {
    if (low < high) {
        int pi = partition(input, low, high);

        quickSort(input, low, pi - 1);
        quickSort(input, pi + 1, high);
    }
}
```

快速排序还有另一种是有两个指针的，如果以第一个元素作为基准值，那么就先从右边找比基准值小的，找到了之后从左边开始找比基准值大的，然后交换元素，当两个指针相遇之后将基准值归为。

而如果以最后一个元素作为基准值，那么就从左边找比基准值大的，找到了之后从右边找比基准值小的，然后交换元素，当两个指针相遇之后将基准值归为。

![](/assets/Algorith_SearchAndSort/quick_sort_two_pointer.jpg)
```java
public void sort(int input[], int low, int high) {
    if (low > high) return;
    int start = low;
    int end = high;
    int pivot = input[low];
    while (start < end) {
        while (start < end && input[end] >= pivot) {
            end--;
        }
        while (start < end && input[start] <= pivot) {
            start++;
        }
        if (start < end) {
            int tmp = input[start];
            input[start] = input[end];
            input[end] = tmp;
        }
    }
    input[low] = input[start];
    input[start] = pivot;

    sort(input, start + 1, high);
    sort(input, low, start - 1);
}
```

## 冒泡排序
冒泡排序的基本思想就是两两比较，小的往左，大的往右。

时间复杂度:
- Best: n
- Worst: n^2
- Average: n^2

代码中的内层循环减去i的原因是每一轮排序结束之后都会把最大的移到最右边。`swapped`的作用是如果已经排序ok了则不需要再循环。
```
public void bubbleSort(int array[]) {
    for (int i = 0; i < array.length - 1; i++) {
        boolean swapped = false;
        for (int j = 0; j < array.length - i - 1; j++) {
            if (array[j] > array[j + 1]) {
                int tmp = array[j];
                array[j] = array[j + 1];
                array[j + 1] = tmp;
                swapped = true;
            }
        }
        if (!swapped) {
            break;
        }
    }
}
```

## 选择排序
选择排序的主要思路是从数组中找出最小的一个数放在新的数组第一个，对剩下的数组作同样处理。

时间复杂度:
- Best: n^2
- Worst: n^2
- Average: n^2

```java
public void selectionSort(int array[]) {
    for (int i = 0; i < array.length; i++) {
        int min_index = i;
        for (int j = i; j < array.length; j++) {
        	if (array[j] < array[min_index]) {
            	min_index = j;
        	}
    	}
    	int tmp = array[min_index];
    	array[min_index] = array[i];
    	array[i] = tmp;
    }
}
```
## 合并排序
合并排序是先将数组拆分然后挨个合并。

时间复杂度:
- Best: n*Log(n)
- Worst: n*Log(n)
- Average: n*Log(n)

![](/assets/Algorith_SearchAndSort/merge_sort.jpg)
```java
 public void merge(int arr[], int l, int m, int r) {
    int tmp[] = new int[r - l + 1];
    int i = l;
    int j = m + 1;
    int k = 0;

    while (i <= m && j <= r) {
        if (arr[i] <= arr[j]) {
            tmp[k] = arr[i];
            i++;
        } else {
            tmp[k] = arr[j];
            j++;
        }
        k++;
    }

    while (i <= m) {
        tmp[k] = arr[i];
        i++;
        k++;
    }

    while (j <= r) {
        tmp[k] = arr[j];
        j++;
        k++;
    }
        
    for (i = l; i <= r; i++) {
        arr[i] = tmp[i - l];
    }
}

public void sort(int arr[], int l, int r) {
    if (l < r) {
        int m = (l + r) / 2;

        sort(arr, l, m);
        sort(arr, m + 1, r);
        merge(arr, l, m, r);
    }
}
```

## 插入排序
选择排序的思想是假设第0个元素为一张扑克牌，之后将数组中剩余的元素挨个交到你手中并放入牌堆中。

比如有个数组为`12,11,13,5,6` ，12是牌堆原有的，从索引1开始拿到了11，将其插入12之前，继续拿到索引2 - 13，放在11、12之后，之后都这样处理。

时间复杂度：
- Best: O(n^2)
- Worst: O(n)
```java
public void selectionSort(int array[]) {
    for (int i = 0; i < array.length; i++) {
        int min_index = i;
        for (int j = i; j < array.length; j++) {
            if (array[j] < array[min_index]) {
                min_index = j;
            }
        }
        int tmp = array[min_index];
        array[min_index] = array[i];
        array[i] = tmp;
    }
}
```

