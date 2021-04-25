# JavaScript 一行流

# JavaScript: one-row-zen

## 2021 年 3月 27 日

## Mar 27 2021

---

这篇博客用于记录一些 JavaScript 的一行流语法。

This blog is to record some magic algorithm methods of JavaScipt.

### 生成二维数组

```js
let row = 3;
let col = 4;

let arr = new Array(row).fill(0).map(() => new Array(col));
```

- `new Array(row).fill(0)`：预先分配 row 个位置，并且在每个位置上赋值 0；
- `.map(() => new Array(col))`：每个位置都返回一个新的长度为 col 的 Array；

### 判断一个数组是否单调递增

```js
function isSorted(arr) {
  return arr.slice(1).every((item, i) => arr[i] <= item);
}
```

- `arr.slice(start, end)`：slice 是切片函数，表示截取数组的 start 位到 end 位，当 end 为空时默认截取到数组结束。
- `.every(function)`：对数组每一个元素都执行 funciton，如果都返回 true，则该表达式返回 true，否则为 false；
- `arr[i] <= item`：注意，item 是切片后新数组的元素，这里巧妙的利用了原数组 arr 与新数组的错位关系，将 arr 的第 i 个元素与（切片后实际上是）第 i+1 个元素进行了比较，从而判断一个数组是否递增。
- 若要判断一个数组是否搭建，只需输入`isSorted(arr.reverse())`

### 数组的前缀和

```js
function preSum(arr) {
  return arr.map((v, i) => arr.slice(0, i + 1).reduce((a, b) => a + b));
}
```

### 二维数组压平到一维

```js
const arr = [[1,2], [3,4], [5,6]];
arr.reduce((acc, cur) => acc.concat(cur), [])
```

- 这里涉及到`Array.prototype.reduce()`的妙用，通过提供一个初始空列表来实现不断的列表连接。

### 数组去重

```js
const arr = [1,2,2,3,3,3,3,3,4,5,6,6,7];
arr.reduce((acc, cur) => acc.includes(cur) ? acc : [...acc, cur], [])
```

- 同上，结合`reduce`和展开（spread）运算符`...`的用法。

关于`reduce`方法的许多妙用，[可以参考字节前端的这篇推送](https://mp.weixin.qq.com/s/6dnWIQUr8lbDGGbN1h6uZw)。

### 数组扁平化+去重+升序排列

```js
const arr = [ [1, 2, 2], [3, 4, 5, 5], [6, 7, 8, 9, [11, 12, [12, 13, [14] ] ] ], 10];
Array.from(new Set(arr.flat(Infinity))).sort((a,b) => a - b)
```

### 判断两个数组是否完全相等

```js
arr1.length === arr2.length && arr1.every((val, idx) => val === arr2[idx])
```

### 对数组的每个元素赋值

```javascript
dp.forEach((v, i) => dp[i] = [nums[i]]);
```

### 从二维数组返回长度最大的一个数组

```javascript
dp.reduce((a,b) => a.length > b.length ? a : b);
```

