---
layout: post
title:  Quick Sort

tags: [Algorithm, Quick Sort, Sort]
---


# 快速排序划分的另一种实现形式

快速排序的划分方法(partation)就是选取一个元素pivot，把小于pivot的元素放在数组的左边，大于pivot的元素放在数组的右边，pivot放在中间，然后递归处理pivot左右两边的数组，最终使得整个数组达到有序的状态。其中partation方法在实现的时候，会自然的想到以数组的最左侧作为小于pivot数组的基，右侧作为大于pivot的数据的基，然后，左侧的游标向右移动，右侧的游标向左移动，直到两个游标相遇，说明原数组按照pivot划分为左右两个不相交的数组。并且把pivot放在左右游标相遇的位置。代码大致如下：

```Java
int partation(int[] array, int begin, int end) {
    int pivot = array[begin];
    int i = begin + 1, j = end;
    while (i <= j) {
        while (i <= j && array[i] <= pivot) i++;
        while (i <= j && pivot < array[j]) j--;
        if (i < j) {
            swap(array, i, j);
            i++;
            i--;
        }
    }
    if (j <= i) {
        swap(array, begin, j);
    }
    return j;
}
```

其中while循环中又包含了两个while循环。这两个while循环是为了分别从左向右遍历及从右向左遍历。默认最左边是小数组的起点，最右边是大数组的起点，中间相遇的点是两个数组的边界。

下面这种划分while循环中没有while循环。只有一个判断，代码看起来更简洁。

```Java
int partation(int[] array, int begin, int end) {
    int pivot = array[end];
    int i = begin;
    for (int j = begin; j <= end - 1; j++) {
        if (array[j] < pivot) {
            swap(array, i, j);
            i += 1;
        }
    }
    swap(array, i, end);
    return i;
}
```

这种划分的思路是默认把整个数组看成是大于pivot的数组，然后在从左到右遍历整个数组的过程中，如果有一个元素小于pivot，则将该元素置换到小于pivot的数组中，在这个过程中，小于pivot的数组的边界不断右移，数组不断扩大；同时大于pivot的数组的边界不断左移，数组不断缩小。在这个过程中，两个数组的边界在不断的变化，最终找到pivot元素恰好把左右两个数组划分开的位置。


基于快速排序，很容易实现快速选择算法。快速选择算法用于在一个无序数组中找到第K大或者第K小的元素。其基本思路就是根据pivot不断地划分区间，如果K命中pivot，则直接返回pivot，终止程序。如果没有命中pivot，则于pivot比较大小后选择大于pivot的区间或者小于pivot的区间，这种，原问题就被缩小到该区间内，继续执行上述操作。快速选择的(平均, 摊返)时间复杂度是O(n)，分析如下：第一次划分，需要扫描n个元素，第二次需要扫描n/2个元素，第三次需要扫描n/4个元素..., 最后一次只需要扫描1个元素。所以有T(n) = n  + n / 2  + n /4 + ... + 1 = n * (1 + 1 / 2 + 1 / 4  + ...) + 1 = n * 2 + 1。所以时间复杂度是O(n)。 如果数组一开始就是有序的，那么再用快速选择寻找数组中的第K个元素，此时的时间复杂度如同快速排序一样，退化为O(n * 2)。但是，这种极端的情况发生的概率很小，而且，如果数组在有序的情况下，也不需要快速选择算法寻找第K个元素，直接返回 array[k - 1]就可以了。或者是在数组基本有序的情况下，使用插入排序使得数组有序，然后直接返回 array[ k - 1]。
