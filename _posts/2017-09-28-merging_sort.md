---
layout: cnblog_post
title:  "归并排序"
date:   2017-09-28 06:34:39
categories: 数据结构
---

归并排序(Merging Sort)是一类不同的排序方法。“归并”的含义是将两个或两个以上的有序表组合成一个新的有序表。它的实现方法无论是对于顺序存储结构还是链表存储结构都非常容易，假如两个有序表的长度分别为m和n，都可以在O(m+n)的时间量级上实现。利用归并的思想容易实现排序：<br/>
可将n个元素的序列看成是n个子序列，则每个子序列的长度为1，然后两两归并，得到ceil(n/2)个长度为2的子序列(如果n为奇数则有一个只有一个元素的子序列)；<br/>
再两两归并...，<br/>
如此重复，直至得到一个长度为n的有序序列。这种排序方法称为2-路归并排序。如下图所示：
<img src="http://qiniu.storage.mikezh.com/dsmerging_sort.png" width="500" alt="2路归并排序"/>

2-路归并中的核心操作是将两个有序列合并为一个有序序列，它的算法如下所示：

```cpp
void Merge(int source_array[], int result_array[], int low_index, int sep_index, int high_index) {
    // 将有序的source_array[low..sep]和有序的source_array[sep+1...high]归并为有序的result_array[low...high]
    int left_pointer = low_index, right_pointer = sep_index + 1;
    int result_index = low_index;
    while (left_pointer <= sep_index && right_pointer <= high_index) {
        if (source_array[left_pointer] <= source_array[right_pointer]) {
            result_array[result_index++] = source_array[left_pointer++];
        } else {
            result_array[result_index++] = source_array[right_pointer++];
        }
    }
    while (left_pointer <= sep_index) { // source_array[sep+1...high]遍历完毕
        result_array[result_index++] = source_array[left_pointer++];
    }
    while (right_pointer <= high_index) { // source_array[low...sep]遍历完
        result_array[result_index++] = source_array[right_pointer++];
    }
}

```
一趟归并排序的操作是：
假如这一趟操作时，每个子序列有h个元素，则调用ceil(n/2h)次merge方法将序列进行两两归并，得到前后相邻的长度为2h的有序端，有序端的个数变为这一趟之前的一半。
因此**整个归并排序需要进行ceil(log2(n))趟**。可见，实现归并排序需要和待排记录等数量级的辅助空间，它的时间复杂度为O(nlog(n))。

```cpp
void MergeSort(int source_array[], int *&result_array, int length) {
    
    for (int count_per_g = 1, group_num = length; count_per_g <= length; count_per_g *= 2) {
        int tmep_group_num = group_num;
        
        int *temp_result = (int *)malloc(sizeof(int) * length);
        memcpy(temp_result, source_array, sizeof(int) * length);
        for (int i = 0; i < group_num && (i+2)*count_per_g - 1 < length; i+=2) {
            Merge(source_array, temp_result, i * count_per_g, (i+1) *count_per_g - 1, (i+2)*count_per_g - 1);
            tmep_group_num --;
        }
        source_array = temp_result;
        
        group_num = tmep_group_num;
    }
    result_array = source_array;
}
```

```cpp
int main(int argc, const char * argv[]) {
    #define ARRAY_SIZE (6)
    int array[ARRAY_SIZE] = {10, 5, 6, 3, 43, 37};
    int *result;
    
    MergeSort(array, result, ARRAY_SIZE);
    
    for (int i = 0; i < ARRAY_SIZE; i++) {
        printf("%d ", result[i]);
    }
    return 0;
}
```

```txt
3 5 6 10 37 43
```

若使用递归的形式，则算法如下。要注意的是，递归形式的算法在形式上较简洁，但实用性很差。

```cpp
void MergeSort(int source_array[], int result_array[], int low, int high) {
    if (low == high) { // 递归结束条件，只有一个元素
        result_array[low] = source_array[low];
        return;
    }

    int sep = (low + high) / 2;
    int *temp_result = (int *)malloc(sizeof(int) * (high - low + 1));
    MergeSort(source_array, temp_result, low, sep);
    MergeSort(source_array, temp_result, sep + 1, high);
    Merge(temp_result, result_array, low, sep, high);
}

int main(int argc, const char * argv[]) {
    #define ARRAY_SIZE (6)
    int array[ARRAY_SIZE] = {10, 5, 6, 3, 43, 37};
    int result[ARRAY_SIZE] = {0};
    MergeSort(array, result, 0, ARRAY_SIZE - 1);
    for (int i = 0; i < ARRAY_SIZE; i++) {
        printf("%zd ", result[i]);
    }
    return 0;
}
```

```txt
3 5 6 10 37 43
```

与快速排序和堆排序相比，归并排序的最大特点是：它是一种稳定的排序方法。但在一般情况下，很少利用2-路归并排序法进行内部排序。