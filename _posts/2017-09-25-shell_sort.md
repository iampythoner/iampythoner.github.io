---
layout: cnblog_post
title:  "希尔排序"
date:   2017-09-25 06:34:39
categories: 数据结构
---

希尔排序(Shell Sort)又称“缩小增量排序”(Diminishing Increment Sort)，它是一种属插入类排序的算法，但在时间效率上又有较大的改进。<br/>
对于直接插入排序，我们知道当序列本身有序的时候，效率非常高，n个元素的序列只需进行n次比较，没有一次移动的过程就完成了排序，其时间复杂度为O(n).<br/>
另外，由于直接插入排序算法简单，在n的值比较小的时候，效率也是非常高的。<br/>
希尔排序就是从上面两点出发对直接插入排序进行改进。<br/>
它的基本思想是：<br/>
①先将整个待排记录序列分割成为若干子序列分别进行直接插入排序，（减小n）<br/>
②待整个序列中的记录“基本有序”时，再对全体记录进行一次直接插入排序。（对有序序列排序）<br/>

如下图中所示：<br/>
首先将该序列中的记录分成5个子序列{a[0], a[5]}, {a[1], a[6]}, ..., {a[4], a[9]},分别对每个子序列进行直接插入排序，得到一趟排序结果；<br/>
然后进行第二趟希尔排序，即分别对这三个子序列{a[0], a[3], a[6], a[9]}, {a[1], a[4], a[7]} 和{a[2], a[5], a[8]}进行直接插入排序，得到第二趟排序结果；<br/>
最后对整个序列进行一趟直接插入排序。至此排序完成。<br/>
<img src="http://7vim0m.com1.z0.glb.clouddn.com/dssort_shell.png" width="500" alt="shell_sort"/>
对照直接插入排序写出一趟希尔排序的算法：

```cpp
void OnceShellSort(int array[], int length, int delta) {
    for (int i = delta; i < length; i++) {
        int support = array[i];
        int j = i - delta;
        while (array[j] >= support && j >= 0) {
            array[j + delta] = array[j]
            j -= delta;
        }
        if (j == i - delta) continue;
        array[j + delta] = support;
    }
}

void ShellSort(int array[], int arr_len, int delta_arr[], int delta_arr_len) {
    for (int i = 0; i < delta_arr_len; i++) {
        int delta = delta_arr[i];
        if (delta >= arr_len) continue;
        OnceShellSort(array, arr_len, );
    }
}

// for test
int main(int argc, const char * argv[]) {
    
    #define ARRAY_SIZE (6)
    int array[] = {50, 10, 3, 8, 9, 11};
    
    int delta_array[] = {9, 5, 3, 2, 1};
    const int delta_arr_len = 5;

    ShellSort(array, ARRAY_SIZE, delta_array, delta_arr_len);

    // print result
    for (int i = 0; i < ARRAY_SIZE; i++) {
        printf("%d  ", array[i]);
    }
    return 0;
}

```

从上述排序过程可见，希尔排序的一个特点是：**子序列的构成不是简单地“逐段分割”，而是将相隔某个“增量”的记录组成一个子序列。**如上面的排序过程，第一趟排序时的增量为5，第二趟排序的增量为3，由于在前两趟的插入排序中记录是和同一子序列中的前一个记录进行比较，因此**记录的比较不是一步一步地往前挪动，而是跳跃式地往前移**，从而使得在进行最后一趟增量为1的插入排序时，序列已基本有序，只要对记录进行少量的比较和移动即完成排序，因此希尔排序的时间复杂度较直接插入排序低。<br/><br/>

希尔排序的分析是一个复杂问题，因为它的时间是所取“增量”序列的函数，这涉及一些数学上尚未解决的难题。因此，到目前为止尚未有人求得一种最好的增量序列，但大量的研究已得出一些局部的结论。<br/>
如有人指出，当增量序列为delta_arr[k] = 2^(d_arr_len - k + 1) - 1 时，希尔排序的时间复杂度为O(n^(3/2))（其中 1<=k<=d_arr_len<=floor(log2(n + 1))）；<br/>
还有人在大量实验基础上推出：当n在某个特定范围内，希尔排序所需的比较和移动次数约为n^1.3，当n→∞时，可减少到n*(log2(n))^2。<br/>
增量序列可以有各种取法，但需注意：**应使增强序列中的值没有除1之外的公因子，并且最后一个增量值必须等于1**。<br/>
常用的增量序列：<br/>
① ...9, 5, 3, 2, 1<br/>
delta_arr[k] = 2^(d_arr_len - k) + 1<br/>
其中0 <= k <= d_arr_len <= floor(log2(n-1))<br/>
② ...40, 13, 4, 1<br/>
delta_arr[k] = 1/2 * (3^(d_arr_len - k) - 1)<br/>
其中0 <= k <= d_arr_len <= floor(log3(2n+1))<br/>