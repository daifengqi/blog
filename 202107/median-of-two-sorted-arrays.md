# 寻找两个有序数组的中位数

# Median Of Two Sorted Arrays

## 2021 年 7 月 5 日

## Jun 5 2021

### 前言

这个问题已经不是第一次看到了，`O(log(m+n))`方法还是不会写，甚至写个`O(m+n)`的解法都磕磕绊绊，搞得我很烦。这次决定写个博客彻底探究一下这个问题，希望以后遇到这个题能够快速AC。

> 这道题是二分查找的（变体）天花板之一，请深刻理解。

### 题目描述

给定两个大小分别为 `m` 和 `n` 的正序（从小到大）数组 `nums1` 和 `nums2`。请你找出并返回这两个正序数组的 **中位数** 。

### 问题转换

先把这道题**“升格”**为*<u>寻找两个有序数组中的第k小的数</u>*。（注意，最小的数是k=1的时候）。

这样做有两个好处，

1. 这个问题变成了一个二分查找问题，满足二段性，即一边的元素都小于（或等于）第k小的数，另一边的元素都大于第k小的数。
2. 可以分别处理`m+n`的值是奇数偶数的情况，不然就会很繁琐...

假如“找到第k小的数”这个函数的名称是`getKthElement`，我们就可以这样做，（假设`len = m + n`）

- 奇数长度：找到两个数组中第`(len + 1) / 2`个小的数；
- 偶数长度：第`len / 2`个数和`(len + 1) / 2`个数的平均数；

代码如下，

```js
function findMedianSortedArrays(nums1, nums2) {
  const len = nums1.length + nums2.length;
  if (len % 2 == 1) {
    return getKthElement(nums1, nums2, (len + 1) / 2);
  } else {
    const x = getKthElement(nums1, nums2, len / 2);
    const y = getKthElement(nums1, nums2, len / 2 + 1);
    return (x + y) / 2;
  }
}
```

### 二分查找

一般的二分查找就是指一个一维序列，满足二段性，一边的元素符合某个条件而另一边不符合，于是通过一个`check`函数，不断检查中间位置（mid）的元素是否符合条件，然后调整左右边界（left、right），最后缩小到目标值。

对这道题来说，

1. **二分查找的左边界是left（初始为0），右边界是left + k/2，分别比较两个数组在第k/2数的大小，较小的数的数组左边的所有数都<u>一定不是第k小的数</u>，而是更小的数，所以可以排出掉。**
2. 每次排除掉k/2个数，剩下查询的目标就是`k - 排除的数的个数 = 新的k`，而排除的数的个数为：`mid-left+1`。

注意，这个问题和一般的二分查找不同的是，每次查询后都调整左右边界，而且调整方式非常奇葩，**较小的左边界移动到中点+1，共同的右边界k减去左边界移动的距离（也就是排除元素的个数）**，这样不断调整k的值，直到`k=1`或者`某个数组的左边界已经到尾部`的时候，就从此时两个分别指向nums1、nums2数组的指针中，返回指向元素中较小的那一个。

代码如下，

```javascript
function getKthElement(nums1, nums2, k) {
  const m = nums1.length;
  const n = nums2.length;
  let left1 = 0;
  let left2 = 0;

  while (true) {
    // 返回条件
    if (left1 == m) {
      return nums2[left2 + k - 1];
    }
    if (left2 == n) {
      return nums1[left1 + k - 1];
    }
    if (k == 1) {
      return Math.min(nums1[left1], nums2[left2]);
    }
    // 调整边界
    const mid1 = Math.min(left1 + (k >> 1) - 1, m - 1);
    const mid2 = Math.min(left2 + (k >> 1) - 1, n - 1);
    // 注意，先调整k
    if (nums1[mid1] <= nums2[mid2]) {
      k -= (mid1 + 1) - left1;
      left1 = mid1 + 1;
    } else {
      k -= (mid2 + 1) - left2;
      left2 = mid2 + 1;
    }
  }
}
```

此外，还有一些线性复杂度的做法。

### 一次遍历，额外空间

我们再来看看`O(n)`的解法，虽然在时间复杂上不如二分，但是也有很多可以学习的地方。

先来看需要**额外空间**的，我们开辟一个新数组`merged`，然后用三个指针`i, j, k`分别指向三个数组，返回合并后的数组。

```javascript
const mergeTwoArray = (nums1, nums2) => {
  const len1 = nums1.length;
  const len2 = nums2.length;
  const merged = new Array(len1 + len2);

  let i = 0;
  let j = 0;
  let k = 0;
  while (i < len1 && j < len2) {
    merged[k++] = nums1[i] < nums2[j] ? nums1[i++] : nums2[j++];
  }
  while (i < len1) {
    merged[k++] = nums1[i++];
  }
  while (j < len2) {
    merged[k++] = nums2[j++];
  }

  return merged;
};
```

最后，根据数组的奇偶性，返回中位数。

```js
const findMedianSortedArrays = (nums1, nums2) => {
  const merged = mergeTwoArray(nums1, nums2);
  const len = merged.length;
  return len % 2 == 0
    ? (merged[len / 2] + merged[len / 2 - 1]) / 2
    : merged[(len - 1) / 2];
};
```

### 一次遍历，不开空间

其实，我们不需要将两个数组真的合并，我们只需要找到中位数在哪里就可以了。

下面看这段代码，

```js
const findMedianSortedArrays = (nums1, nums2) => {
  const m = nums1.length;
  const n = nums2.length;
  const len = m + n;
  let left = -1;
  let right = -1;
  let i = 0;
  let j = 0;

  for (let k = 0; k <= len / 2; ++k) {
    left = right;
    if (i < m && (j >= n || nums1[i] < nums2[j])) {
      right = nums1[i++];
    } else {
      right = nums2[j++];
    }
  }

  if (len & 1) {
    return right;
  } else {
    return (left + right) / 2.0;
  }
};
```

这段代码博大精深，巧妙之处暂时难以说清楚，只是简单说两个地方，

- 每次比较开始时，做赋值`left=right`，记录上一个数，这样偶数的情况就可以做平均值。
- `if (i < m && (j >= n || nums1[i] < nums2[j])) {...}`这个判断，很有意思。

