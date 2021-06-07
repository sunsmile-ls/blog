# 归并排序

## 算法思想

归并排序的核心思想还是蛮简单的。如果要排序一个数组，我们先把数组从中间分成前后两部分，然后对前后两部分分别排序，再将排好序的两部分合并在一起，这样整个数组就都有序了.

## 算法图解

![](./images/merge.jpg)

## 算法实现

```javascript
const mergeArr = (left, right) => {
    let temp = []
    let leftIndex = 0
    let rightIndex = 0
    // 判断2个数组中元素大小，依次插入数组
    while (left.length > leftIndex && right.length > rightIndex) {
        if (left[leftIndex] <= right[rightIndex]) {
            temp.push(left[leftIndex])
            leftIndex++
        } else {
            temp.push(right[rightIndex])
            rightIndex++
        }
    }
    // 合并 多余数组
    return temp.concat(left.slice(leftIndex)).concat(right.slice(rightIndex))
}

const mergeSort = (arr) => {
    // 当任意数组分解到只有一个时返回。
    if (arr.length <= 1) return arr
    const middle = Math.floor(arr.length / 2) // 找到中间值
    const left = arr.slice(0, middle) // 分割数组
    const right = arr.slice(middle)
    // 递归 分解 合并
    return mergeArr(mergeSort(left), mergeSort(right))
}
```

## 算法分析

1. 归并排序不是原地排序算法，空间复杂度为**O(n)**
2. 归并排序是稳定的排序算法。
3. 归并排序的时间复杂度: **O(nlogn)**