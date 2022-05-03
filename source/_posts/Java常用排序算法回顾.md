---
title: Java常用排序算法回顾
layout: post
date: 2022/01/06 19:26:50
tags : Java

---
计算机中最基本排序算法，毕业后基本就没再碰过，虽然知道使用场景及优缺点，但是具体实现细节已经记不清了，趁着这次换工作的机会重新手撕一遍，权当是裨补阙漏。
<br>
### 简单排序之选择排序、冒泡排序、插入排序、希尔排序。选择 < 冒泡 < 插入 < 希尔

```java
// 选择排序，时间复杂度 O(n^2)，空间复杂度 O(1)，不稳定排序。
// 选出最小的换到最前，最没用的排序。
// 优化：选出最小和最大同时交换，时间减少一半，基本不用，不稳定，而且不比插入快。
public static int[] selectionSort(int[] arr) {
    for (int i = 0; i < arr.length - 1; i++) {
        int tempPos = i;
        for (int j = i + 1; j < arr.length; j++) {
            if (arr[j] < arr[tempPos]) {
                tempPos = j;
            }
        }
        if (tempPos != i) {
            swap(arr, i, tempPos);
        }
    }
    return arr;
}


// 冒泡排序，时间复杂度 O(n^2)，空间复杂度 O(1)，稳定排序。
// 当前元素逐渐上浮到应该在的位置，两两比较两两交换。
// 优化：最好情况能做到 O(n)，实际上基本不用，太慢。
public static int[] bubbleSort(int[] arr) {
    for (int i = 0; i < arr.length; i++) {
        for (int j = i; j < arr.length; j++) {
            if (arr[i] > arr[j]) {
                swap(arr, i, j);
            }
        }
    }
    return arr;
}


// 插入排序，时间复杂度 O(n^2),空间复杂度 O(1)，稳定排序。
// 当前元素往前插，比冒泡排序效率高，主要是体现在比较和交换次数上，比选择排序快一点。
// 样本小且基本有序的情况的时候效率最高。
public static int[] insertionSort(int[] arr) {
    for (int i = 1; i < arr.length; i++) {
        for (int j = i; j > 0; j--) {
            if (arr[j] < arr[j - 1]) {
                swap(arr, j, j - 1);
            }
        }
    }
    return arr;
}


// 希尔排序，时间复杂度 O(n^1.3)，空间复杂度 O(1)，不稳定，时间复杂度在 O(n^2) ~ O(n^(7/6)) 之间，适合中型数据。
// 对插入排序交换次数过多进行进行优化，对 gap 进行局部插入排序，最终递减到 gap=1 的插入排序，用来减少交换次数。
// 对于 gap 参数，Shell 最早采用二分的方式，后来发现不是最优解，诞生了 Knuth 序列，其他序列还有如质数序列等。
public static int[] shellSort(int[] arr) {
    // Knuth 序列
    int h = 1;
    while (h <= arr.length / 3) {
        h = h * 3 + 1;
    }

    for (int gap = h; gap > 0; gap = (gap - 1) / 3) {
        // 局部的插入排序
        for (int i = gap; i < arr.length; i++) {
            for (int j = i; j > gap - 1; j -= gap) {
                if (arr[j] < arr[j - gap]) {
                    swap(arr, j, j - gap);
                }
            }
        }
    }
    return arr;
}


private static void print(int[] arr) {
    System.out.println();
    for (int j : arr) {
        System.out.print(j + " ");
    }
    System.out.println();
}

private static int[] swap(int[] arr, int i, int j) {
    int temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
    return arr;
}
```
<br>
### 常用排序之快速排序、归并排序、堆排序（略）

```java
// 快速排序，时间复杂度 O(nlogn)，空间复杂度 O(n)，不稳定，最坏情况下降级到 O(n^2)。
// 是 Java 中对数组默认的排序是方式，不过内部也会根据实际情况调整，最终可能会使用归并排序（TIM Sort），双插入排序，单轴排序和双轴排序。
public static int[] quickSort(int[] arr) {
    quickSort(arr, 0, arr.length - 1);
    return arr;
}

private static void quickSort(int[] data, int left, int right) {
    if (data == null || left >= right) return;
    int i = left, j = right;
    int pivotKey = data[left];
    while (i < j) {
        while (i < j && data[j] >= pivotKey) j--;
        if (i < j) data[i++] = data[j];
        while (i < j && data[i] <= pivotKey) i++;
        if (i < j) data[j--] = data[i];
    }
    data[i] = pivotKey;
    quickSort(data, left, i - 1);
    quickSort(data, i + 1, right);
}


// 归并排序，时间复杂度 O(nlogn)，空间复杂度 O(n)，稳定，时间复杂度不管好坏都是 O(nlogn)。
// Java 和 Python 中的对象排序都是使用的归并排序，因为对象排序需要稳定的排序算法。
// TIM Sort 是改进的归并排序，内部还用到了二分插入排序，如果数据小于最小分组数时就使用二分插入排序。
public static int[] mergeSort(int[] arr) {
    mergeSort(arr, 0, arr.length - 1);
    return arr;
}

private static int[] mergeSort(int[] arr, int left, int right) {
    if (left == right) {
        return arr;
    }
    int mid = left + (right - left) / 2;
    mergeSort(arr, left, mid);
    mergeSort(arr, mid + 1, right);
    merge(arr, left, mid + 1, right);
    return arr;
}

private static void merge(int[] arr, int left, int right, int rightBound) {
    int mid = right - 1;
    int[] temp = new int[rightBound - left + 1];

    int i = left;
    int j = right;
    int k = 0;
    while (i <= mid && j <= rightBound) {
        temp[k++] = arr[i] <= arr[j] ? arr[i++] : arr[j++];
    }
    while (i <= mid) {
        temp[k++] = arr[i++];
    }
    while (j <= rightBound) {
        temp[k++] = arr[j++];
    }
    for (int m = 0; m < temp.length; m++) {
        arr[left + m] = temp[m];
    }
}
```
<br>
### 特殊场景排序之基数排序、计数排序、桶排序（略）
```java
// 基数排序，时间复杂度 O(n*k)，空间复杂度 O(n+k)，n 是原数组长度，k 是 count 数组长度。
// 非比较排序，多关键字排序，是桶思想的一种，有低位优先和高位优先两种方式，高位优先就是分治思想。
public static int[] radixSort(int[] arr) {
    int[] result = new int[arr.length];
    int[] count = new int[10];

    for (int i = 0; i < 3; i++) {
        int division = (int) Math.pow(10, i);
        for (int j = 0; j < arr.length; j++) {
            int num = arr[j] / division % 10;
            count[num]++;
        }

        // 转换为累加序列，才能保证该算法是稳定排序
        for (int k = 1; k < count.length; k++) {
            count[k] = count[k] + count[k - 1];
        }

        for (int m = arr.length - 1; m >= 0; m--) {
            int num = arr[m] / division % 10;
            result[--count[num]] = arr[m];
        }
        System.arraycopy(result, 0, arr, 0, arr.length);
        Arrays.fill(count, 0);
    }
    return result;
}

// 计数排序，时间复杂度 O(n+k)，空间复杂度 O(n+k)，内部使用累加序列来保证稳定。n 是原数组长度，k 是 count 数组长度。
// 计数排序是非比较排序，适用于特定的问题，比如数据量大但样本简单且集中，比如年龄，分数等。也是桶思想的一种。
public static int[] countSort(int[] arr) {
    int[] result = new int[arr.length];
    int[] count = new int[10];

    for (int i = 0; i < arr.length; i++) {
        count[arr[i]]++;
    }
    // 转换为累加序列，才能保证该算法是稳定排序
    for (int i = 1; i < count.length; i++) {
        count[i] = count[i] + count[i - 1];
    }

    for (int i = arr.length - 1; i >= 0; i--) {
        result[--count[arr[i]]] = arr[i];
    }
    return result;
}
```
<br>