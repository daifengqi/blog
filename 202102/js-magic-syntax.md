# JavaScript 一行流

# JavaScript: one-row-zen

## 2021 年 2 月 28 日

## Feb 28 2021

---

这篇博客用于记录一些 JavaScript 的一行流语法。（带上#号的方法慎用，性能较低）。

This blog is to record some magic algorithm methods of JavaScipt.

**生成二维数组**

```js
let row = 3;
let col = 4;

let arr = new Array(row).fill(0).map(() => new Array(col));
```

- `new Array(row).fill(0)`：预先分配 row 个位置，并且在每个位置上赋值 0；
- `.map(() => new Array(col))`：每个位置都返回一个新的长度为 col 的 Array；

**判断一个数组是否单调递增**

```js
function isSorted(arr) {
  return arr.slice(1).every((item, i) => arr[i] <= item);
}
```

- `arr.slice(start, end)`：slice 是切片函数，表示截取数组的 start 位到 end 位，当 end 为空时默认截取到数组结束。
- `.every(function)`：对数组每一个元素都执行 funciton，如果都返回 true，则该表达式返回 true，否则为 false；
- `arr[i] <= item`：注意，item 是切片后新数组的元素，这里巧妙的利用了原数组 arr 与新数组的错位关系，将 arr 的第 i 个元素与（切片后实际上是）第 i+1 个元素进行了比较，从而判断一个数组是否递增。
- 若要判断一个数组是否搭建，只需输入`isSorted(arr.reverse())`

**数组的前缀和#**

```js
function preSum(arr) {
  return arr.map((v, i) => arr.slice(0, i + 1).reduce((a, b) => a + b));
}
```
