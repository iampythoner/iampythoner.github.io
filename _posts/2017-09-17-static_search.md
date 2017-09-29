---
layout: cnblog_post
title:  "查找-静态查找"
date:   2017-09-17 06:34:39
categories: 数据结构
---
查找分为两种类型<br/>
静态查找：只是查看元素是否存在和查看元素的各种属性；<br/>
动态查找：在查找的过程中会有删除元素、插入元素的操作。

这两种类型又有多种算法：<br/>
静态查找：<a href="#anchor1_0">顺序查找</a>、<a href="#anchor2_0">折半查找</a>、静态树表查找、索引顺序表查找<br/>
动态查找: <a href="/2017/09/bst/" target="_blank">二叉排序树</a>、<a href="/2017/09/bbt_avl/" target="_blank">平衡二叉树</a>、红黑树、B树和B+树、键树<br/>
<a href="/2017/09/hash_table/" target="_blank">哈希表</a>：可作为一种动态查找的方式

本篇主要介绍静态查找：
<h4 id="anchor1_0">顺序查找</h4>
```cpp
// 返回参数:所在的索引， 若没有找到则返回-1
int SequentialSearch(int array[], int arr_length, int key) {
    int i = arr_length - 1;
    for (; array[i] != key && i >= 0; i--);
    return i; // i < 0则没有找到
}

int main() {
    #define ARRAY_SIZE (3)
    int array[] = {10, 20, 30};
    int index = SequentialSearch(array, ARRAY_SIZE, 90);
    if (index == -1) {
        printf("没有找到\n");
    } else {
        printf("找到了，位置是：%zd\n", index);
    }
}
```
有一种做法是将数据序列的0号位作为空位，将它作为哨兵的辅助空间，而序列的长度是除了辅助位的剩下的元素的个数。使用辅助位之后，可以不用在每次对比之后做整个序列是否遍历完毕的判断(如上面的i >= 0 的判断)，若返回的值为0，就意味着遍历完成也没有找到。虽然这样多占用了一个数据的空间，但是省去了多次判断，在当数据总个数非常多的时候，使用哨兵效率提升非常明显。

```cpp
int SequentialSearch_Sentinel(int array[], int arr_length, int key) {
    array[0] = key;
    int i = arr_length;
    for (; array[i] != key; i--);
    return i;
}

int main() {
    #define ARRAY_SIZE (3)
    int array[] = {NULL, 10, 20, 30};
    int index = SequentialSearch_Sentinel(array, ARRAY_SIZE, 90);
    if (index == 0) {
        printf("没有找到\n");
    } else {
        printf("找到了，位置是：%zd\n", index);
    }
}
```
<h4 id="anchor2_0">折半查找</h4>

```cpp
// 折半查找 只适合于 有序数组
int BinarySearch(int array[], int arr_length, int key) {
    int low = 0, high = arr_length - 1;
    while (low <= high) {
        int mid = (low + high) / 2;
        if ( key == array[mid] ) return mid;
        if ( key < array[mid]) high = mid - 1;
        else low = mid + 1;
    }
    return -1;
}
```








