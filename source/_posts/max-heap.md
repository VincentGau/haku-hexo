---
title: 最大/最小堆
date: 2018-10-19 14:39:03
tags:
- algorithm
---

# 堆的特性
- 堆是一棵完全二叉树（除了最底层，其余各层节点数都达到最大，最底层的节点都在左边）；
- 堆的左右子树仍然是堆；
- 堆中每个节点的值都不大于/不小于其子节点的值；

由于堆是完全二叉树，所以对于序号`i`的节点（根节点序号为0）, 其左子节点序号为`2i + 1`，右子节点序号为`2i + 2`， 父节点序号为`(i-1)/2`；

# 堆的操作
# 调整节点
找到节点和其子节点中最大的那个，将它与该节点交换，如果节点本身就是最大则不发生交换；一次调整之后，子树不一定仍满足最大堆特性，因此需要继续调整，使整棵树保持最大堆特性。
```python
//调整最大堆，将值大的节点上移
private static void heapifyMax(int[] array, int index, int size)
{
    int left = 2 * index + 1;
    int right = 2 * index + 2;
    int max = index;

    if(left < size && array[left] > array[max])
    {
        max = left;
    }

    if(right < size && array[right] > array[max])
    {
        max = right;
    }

    // 有调整，将大的节点上移
    if(max != index)
    {
        swap(ref array[index], ref array[max]);
        heapifyMax(array, max, size);
    }
}

private static void swap(ref int a, ref int b)
{
    int temp = a;
    a = b;
    b = temp;
}
```
## 构造最大堆
从最后一个叶子节点的父节点开始调整节点，直到前面所有节点都调整完毕，最大堆即构造完成；
```python
private static void buildMaxHeap(int[] array)
{
    // 从最后一个节点的父节点开始，向上调用调整最大堆
    for(int i = array.Length /2 -1; i >= 0; i--)
    {
        heapifyMax(array, i, array.Length);
    }
}
```

# 堆排原理
对于最大堆，堆顶元素的值最大，取出堆顶元素后调整堆中余下节点，使继续保持最大堆的特性，再次取出堆顶元素，持续此过程直到最后一个元素。
```python
// 初始化最大堆，堆顶元素最大，将堆顶元素和最后一个叶子结点交换，重新调整二叉树保持最大堆特
//性；下次操作时排除尾部已排序的元素（通过heapifyMax 方法的最后一个参数决定）
private static void heapSort(int[] array)
{
    buildMaxHeap(array);
    for(int i = array.Length - 1; i > 0; i--)
    {
        swap(ref array[0], ref array[i]);
        heapifyMax(array, 0, i);
    }
}

```

# TOP K
从大量数据中找出前K 大的数。最直接的方法对所有数据进行排序后取前K 个。我们的目的是找到这K个数，既不需要管其他的数是何顺序，也不要求这K个数有序，数据量大时对所有数据进行排序性价比太低。一种思路是维护一个大小为K 的最小堆，首先将任意K 个数放入堆中，余下的每个元素依次与堆顶元素进行比较，如果它比堆顶元素大，则删除堆顶元素，并将新的元素放入堆中，重新调整堆；
```python
// 找到最大的K个元素
private static void topK(int[] array, int K, int[] kMax)
{
    for(int i = 0; i < K; i++)
    {
        kMax[i] = array[i];
    }
    buildMinHeap(kMax);
    for (int j = K; j < array.Length; j++)
    {
        if(array[j] > kMax[0])
        {
            swap(ref kMax[0], ref array[j]);
            heapifyMin(kMax, 0, K);
        }
    }
}
```