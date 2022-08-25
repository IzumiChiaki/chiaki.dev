+++
title = "排序算法实现"
date = "2020-08-03T20:53:19+08:00"
tags = ["java", "algorithm", "sort"]
slug = "sorting-algorithms"
dropCap = false
+++

##### 分类

|类别|算法|
| ---- | ---- |
|插入排序类|插入排序、希尔排序|
|选择排序类|选择排序、堆排序|
|交换排序类|冒泡排序、快速排序|
|归并排序类|归并排序|

##### 复杂度

|算法|平均情况|最好情况|最坏情况|辅助空间|稳定性|复杂性|
| ---- | ---- | ---- | ---- | ---- | ---- | ---- |
|冒泡排序|O(n^2)|O(n)|O(n^2)|O(1)|稳定|简单|
|选择排序|O(n^2)|O(n^2)|O(n^2)|O(1)|稳定|简单|
|插入排序|O(n^2)|O(n)|O(n^2)|O(1)|稳定|简单|
|希尔排序|O(nlogn)~O(n^2)|O(n^1.3)|O(n^2)|O(1)|不稳定|较复杂|
|堆排序|O(nlogn)|O(nlogn)|O(nlogn)|O(1)|不稳定|较复杂|
|归并排序|O(nlogn)|O(nlogn)|O(nlogn)|O(n)|稳定|较复杂|
|快速排序|O(nlogn)|O(nlogn)|O(n^2)|O(logn)~O(n)|不稳定|较复杂|

##### 算法实现：排序算法类Sort

```java
package com.chiaki.demo;

/**
 * 排序算法类
 * @author Chiaki
 */
public class Sort {
    /**
     * 工具函数：打印数组
     * @param arr 原数组
     */
    public static void printArr(int[] arr) {
        for (int i = 0; i < arr.length; i++) {
            System.out.print(arr[i] + " ");
        }
        System.out.println();
    }

    /**
     * 工具函数：交换数组两个下标对应的值
     * @param arr 原数组
     * @param a 下标a
     * @param b 下标b
     */
    public static void swap(int[] arr, int a, int b) {
        int temp = arr[a];
        arr[a] = arr[b];
        arr[b] = temp;
    }

    /**
     * 冒泡排序
     * @param arr 原数组
     */
    public static void bubbleSort01(int[] arr) {
        // i从第1个元素开始
        for (int i = 0; i < arr.length - 1; i++) {
            // j从倒数第2个元素开始直到与i相遇
            for (int j = arr.length - 2; j >= i; j--) {
                // 比较倒数第1个元素与倒数第2个元素
                if (arr[j] > arr[j + 1]) {
                    // 满足条件进行交换
                    swap(arr, j, j + 1);
                }
            }
        }
    }

    /**
     * 改进冒泡排序：设置标志减少比较次数
     * @param arr 原数组
     */
    public static void bubbleSort02(int[] arr) {
        boolean flag = true;
        // i从第1个元素开始 flag为假时结束循环
        for (int i = 0; (i < arr.length) && flag; i++) {
            // flag初始化为假 若后续flag不改变为真则结束循环
            flag = false;
            // j从倒数第1个元素开始直到与i相遇
            for (int j = arr.length - 1; j > i; j--) {
                // 比较倒数第1个元素和倒数第2个元素
                if (arr[j - 1] > arr[j]) {
                    // 满足条件进行交换
                    swap(arr, j, j - 1);
                    // 发生交换则更新flag为真
                    flag = true;
                }
            }
        }
    }

    /**
     * 选择排序
     * @param arr 原数组
     */
    public static void selectSort(int[] arr) {
        for (int i = 0; i < arr.length - 1; i++) {
            // 将当前元素指针设置为最小值指针min
            int min = i;
            // j从i的下一个元素开始
            for (int j = i + 1; j < arr.length; j++) {
                // 若arr[j]小于arr[min]则更新min
                if (arr[min] > arr[j]) {
                    min = j;
                }
            }
            // 如果min发生了更新就将数组元素值进行交换
            if (i != min) {
                swap(arr, i, min);
            }
        }
    }

    /**
     * 插入排序
     * @param arr 原数组
     */
    public static void insertSort(int[] arr) {
        // i从第1个元素开始
        for (int i = 1; i < arr.length; i++) {
            // 将当前arr[i]保存在临时变量temp
            int temp = arr[i];
            // 设置与temp进行同组比较的索引j
            int j = i - 1;
            // 同组比较 若temp较小则进行后移操作
            while (j >= 0 && arr[j] > temp) {
                // 后移1位
                arr[j + 1] = arr[j];
                j--;
            }
            // 将temp插入到后移操作后的元素前面
            arr[j + 1] = temp;
        }
    }

    /**
     * 希尔排序
     * @param arr 原数组
     */
    public static void shellSort(int[] arr) {
        // 希尔变量
        int delta = arr.length / 2;
        // 满足delta>0进行循环
        while (delta > 0) {
            for (int i = delta; i < arr.length; i++) {
                // 将当前arr[i]保存在临时变量temp
                int temp = arr[i];
                // 设置与temp进行同组比较的索引j
                int j = i - delta;
                // 同组比较 若temp较小则进行后移操作
                while (j >= 0 && arr[j] > temp) {
                    // 后移delta位
                    arr[j + delta] = arr[j];
                    j -= delta;
                }
                // 将temp插入到后移操作后的元素前面
                arr[j + delta] = temp;
            }
            // 更新希尔增量
            delta /= 2;
        }
    }

    /**
     * 基于数组实现的大顶堆
     * @param arr 数组
     * @param size 堆大小
     * @param index 堆索引
     */
    private static void maxHeap(int[] arr, int size, int index) {
        // 根据完全二叉树的性质得到左孩子和右孩子的索引
        int left = 2 * index + 1;
        int right = 2 * index + 2;
        // 将当前索引index设置为最大值索引maxindex
        int maxIndex = index;
        // 若左孩子结点值比最大值索引结点值大则更新maxindex
        if (left < size && arr[left] > arr[maxIndex]) {
            maxIndex = left;
        }
        // 若右孩子结点值比最大值索引结点值大则更新maxindex
        if (right < size && arr[right] > arr[maxIndex]) {
            maxIndex = right;
        }
        // 若maxIndex发生了更新则进行交换操作
        if (maxIndex != index) {
            swap(arr, maxIndex, index);
            // 递归
            maxHeap(arr, size, maxIndex);
        }
    }

    /**
     * 基于大顶堆的堆排序
     * @param arr 原数组
     */
    public static void heapSort(int[] arr) {
        // 找到最后一个非叶子结点即最后一个叶子结点的父结点的索引作为开始
        int parent = arr.length / 2 - 1;
        // 从parent开始针对原数组构建大顶堆
        for (int i = parent; i >= 0; i--) {
            maxHeap(arr, arr.length, i);
            // 打印大顶堆的调整过程
            //printArr(arr);
        }
        // 基于大顶堆的排序
        for (int i = arr.length - 1; i > 0; i--) {
            // 将最后一个元素与根结点值进行交换
            swap(arr, 0, i);
            // 交换完成后重新调整大顶堆
            maxHeap(arr, i, 0);
        }
    }

    /**
     * 2路归并排序算法
     * @param arr 原数组
     * @param start 左边界
     * @param end 右边界
     */
    private static void merge(int[] arr, int start, int end) {
        // 满足条件进行递归
        if (start < end) {
            int mid = (end + start) / 2;
            // 对左子数组进行归并排序
            merge(arr, start, mid);
            // 对右子数组进行归并排序
            merge(arr, mid + 1, end);
            // 合并
            int[] temp = new int[end - start + 1];
            int i = start, j = mid + 1, k = 0;
            while (i <= mid && j <= end) {
                temp[k++] = arr[i] < arr[j] ? arr[i++] : arr[j++];
            }
            while (i <= mid) {
                temp[k++] = arr[i++];
            }
            while (j <= end) {
                temp[k++] = arr[j++];
            }
            System.arraycopy(temp, 0, arr, start, end - start + 1);
        }
    }

    /**
     * 调用2路归并排序算法
     * @param arr 原数组
     */
    public static void mergeSort(int[] arr) {
        merge(arr, 0, arr.length - 1);
    }

    /**
     * 快速排序
     * @param arr 原数组
     * @param start 左指针
     * @param end 右指针
     */
    private static void sort02(int[] arr, int start, int end) {
        // 满足条件进行递归
        if (start < end) {
            // 定义哨兵i和j指向数组头尾
            int i = start, j = end;
            // 哨兵不相遇则执行循环
            while (i < j) {
                // 一定是尾部哨兵j先行动
                // 若arr[j]>中轴值则进行循环 否则跳出循环
                while (i < j && arr[j] > arr[start]) {
                    j--;
                }
                // 尾部哨兵j停止行动后头部哨兵i行动
                // 若arr[j]<=中轴值则进行循环 否则跳出循环
                while (i < j && arr[i] <= arr[start]) {
                    i++;
                }
                // 哨兵行动完成后对i和j所在的值进行交换
                if (i < j) {
                    swap(arr, i, j);
                }
            }
            // 将中轴值与i处的值交换
            // 保证中轴左边都比它小右边都比它大
            swap(arr, start, i);
            // 递归对中轴左边的子数组进行快排
            sort02(arr, start, i - 1);
            // 递归对中轴右边的子数组进行快排
            sort02(arr, i + 1, end);
        }

    }

    /**
     * 调用快速排序算法
     * @param arr 原数组
     */
    public static void quickSort(int[] arr) {
        sort02(arr, 0, arr.length - 1);
    }
}
```

##### 算法测试：排序测试类SortTest

```java
package com.chiaki.demo;

import org.junit.After;
import org.junit.Before;
import org.junit.Test;

import static com.chiaki.demo.Sort.*;

/**
 * 排序测试类
 * @author Chiaki
 */
public class SortTest {
    private int[] arr;

    @Before
    public void init() {
        arr = new int[]{49, 38, 65, 97, 76, 13, 27, 50, 88};
        System.out.print("Before: ");
        printArr(arr);
    }

    @After
    public void show() {
        System.out.print("After : ");
        printArr(arr);
    }

    /**
     * 冒泡排序01测试结果
     * Before: 49 38 65 97 76 13 27 50 88
     * After : 13 27 38 49 50 65 76 88 97
     */
    @Test
    public void bubbleSortTest01() {
        bubbleSort01(arr);
    }

    /**
     * 冒泡排序02测试结果
     * Before: 49 38 65 97 76 13 27 50 88
     * After : 13 27 38 49 50 65 76 88 97
     */
    @Test
    public void bubbleSortTest02() {
        bubbleSort02(arr);
    }

    /**
     * 选择排序测试结果
     * Before: 49 38 65 97 76 13 27 50 88
     * After : 13 27 38 49 50 65 76 88 97
     */
    @Test
    public void selectSortTest() {
        selectSort(arr);
    }

    /**
     * 插入排序测试结果
     * Before: 49 38 65 97 76 13 27 50 88
     * After : 13 27 38 49 50 65 76 88 97
     */
    @Test
    public void insertSortTest() {
        insertSort(arr);
    }

    /**
     * 希尔排序测试结果
     * Before: 49 38 65 97 76 13 27 50 88
     * After : 13 27 38 49 50 65 76 88 97
     */
    @Test
    public void shellSortTest() {
        shellSort(arr);
    }

    /**
     * 堆排序测试结果
     * Before: 49 38 65 97 76 13 27 50 88
     * After : 13 27 38 49 50 65 76 88 97
     */
    @Test
    public void heapSortTest() {
        heapSort(arr);
    }

    /**
     * 归并排序测试结果
     * Before: 49 38 65 97 76 13 27 50 88
     * After : 13 27 38 49 50 65 76 88 97
     */
    @Test
    public void mergeSortTest() {
        mergeSort(arr);
    }

    /**
     * 快速排序测试结果
     * Before: 49 38 65 97 76 13 27 50 88
     * After : 13 27 38 49 50 65 76 88 97
     */
    @Test
    public void quickSortTest() {
        quickSort(arr);
    }
}
```

#### 结果显示上述算法均通过测试。