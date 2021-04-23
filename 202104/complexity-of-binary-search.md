# 二分骗局：二分搜索的复杂性

# Complexity of Binary Search

## 2021 年 4 月 22 日

## Apr 22 2021

---

> 「二分」的本质是二段性，并非单调性。只要一段满足某个性质，另外一段不满足某个性质，就可以用「二分」。
>
> ——[宫水三叶](https://leetcode-cn.com/problems/search-in-rotated-sorted-array-ii/solution/gong-shui-san-xie-xiang-jie-wei-he-yuan-xtam4/)

### 引言

二分搜索通常是各类算法书所写的第一种算法，也是很多人入门时学的第一个算法，它的表面看起来十分简单，然而真正写二分搜索时常常会有各种各样的问题；许多困难问题的某个中间步骤也需要深刻理解并写出二分搜索以提高效率，降低时间复杂度，这篇博客的目的是捋一捋二分搜索在实际写的过程中的各种问题、以及几下各类模板。

### 二分搜索的出口

非递归写法的二分搜索从一个`while`循环开始，不论`while`里面关于左右端点的比较判断写法是什么，是否加一，是小于还是小于等于，总归逃不出三种出口方式：

1. 出循环时`left`等于`right`，即两者同值；
2. 出循环时`left+1`等于`right`，即两者并邻，`left`在左边一格；
3. 出循环时`left`等于`right+1`，即两者并邻，`left`在右边一格；

### 避免死循环的写法

[九章](https://www.lintcode.com/)推荐过一个模板：

```c++
while (left + 1 < right) {
  mid = left + (right - left >> 1);
  if (nums[mid] == target) {
    return mid;
  } else if (nums[mid] < target) {
    left = mid;
  } else {
    right = mid;
  }
}

if (nums[left] == target) {
  return left;
} else if (nums[right] == target) {
  return right;
} else {
  return -1;
}
```

这里要强调的点是，

1. `while (left + 1 < right)`
2. `left=mid`, `right=mid`

这两种写法搭配，可以保证循环不会出错，且一定可以跳出循环，并且满足`left`在`right`左边一格的条件；在跳出循环之后，可以分别检查`left`和`right`两个端点，最终得到结果。

这个写法虽然繁琐，但是胜在不出错，适用性强，对于新手来说，这是一种推荐的写法。这种写法在解决一些较复杂问题时也很好用，推荐一个leetcode例题[【在排序数组中查找元素的第一个和最后一个位置】](https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/solution/javascript-bi-mian-si-xun-huan-de-er-fen-hp7a/)，通过做题和看题解可以加深理解。

### 利用辅助指针记录位置

当目标值`target`不存在于`nums`数组中，且`num`无重复元素时，二分搜索还有一个技巧是，利用一个`pos`指针记录位置，模板如下：

```c++
pos = 0;
while (left <= right) {
  mid = left + (right - left >> 1);
  if (nums[mid] < target) {
    pos = mid;
    left = mid + 1;
  } else {
    right = mid - 1;
  }
}
```

如此，`pos`指向`nums`中最后一个小于`target`的元素，而`pos+1`指向第一个大于`target`的元素。见leetcode[【马戏团人塔】](https://leetcode-cn.com/problems/circus-tower-lcci/solution/javascriptshu-zu-er-fen-by-cescdf-h9w6/)。

在设定`pos`的初始值和转换时，技巧为：若`while`循环内部的判断块一直执行`else`内的语句，跳出循环的期望目标值应该是`pos`的初始值，应该初始值没有改变过。

1. 若要搜索的是某个最小满足条件的值，则应该令初始值`pos=0`；见[【最小体力消耗路径】](https://leetcode-cn.com/problems/path-with-minimum-effort/solution/javascript-chou-chi-er-fen-sou-suo-he-lu-oapf/)。
2. 若是搜索不到时希望返回最右端的位置，则令`pos=n`或`pos=right`；见[【搜索插入位置】](https://leetcode-cn.com/problems/search-insert-position/solution/javascript-fu-zhu-zhi-zhen-by-cescdf-p3jj/)。

其实两者是可以转换的，关键在于`pos`的初始值满足一直执行条件控制块的`else`语句后的期望结果。

### 避免无法进入循环的错误

有时候二分搜索会因为无法进入`while`循环而出错，这通常是因为循环条件的写法。要避免错误，只有写法是

```c++
while (left <= right)
```

时可以保证数组长度为1时也能进入循环，其他写法都需要做额外的边界（特殊）情况判断并处理。

### 二分骗局：总结

二分搜索是看起来简单的一类算法，忽视它往往会造成在写二分搜索时出现各种各样的瑕疵导致代码不能通过。我们不能陷入这种“表面似乎很简单”的骗局，而应该深入本质去理解它。

关于【搜索插入位置】这道题，还可以参考[这里](https://leetcode.wang/leetCode-35-Search-Insert-Position.html)，这篇也仔细阐述了二分查找的某些性质，比如中点偏左性，开闭性，右端点懒更新性等等。

其他练习：

1. [和至少为K的最短子数组](https://leetcode-cn.com/problems/shortest-subarray-with-sum-at-least-k/)