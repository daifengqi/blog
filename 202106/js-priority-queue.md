# JavaScript实现优先队列和有序集合

# JavaScript Priority Queue And Sorted Set

## 2021 年 6 月 27 日

## Jun 27 2021



### 前言

优先队列和有序集合都是很常用的数据结构，然后JS没有自带的实现，所以这里手写一份，并解读一下实现过程。

> redis使用跳跃表+字典实现[有序集合](http://redisbook.com/preview/object/sorted_set.html)，可以参考一下。
>
> —— [《redis设计与实现》](http://redisbook.com/)

另外一说，redis设计与实现真是很fashion的数据结构教材，有空记得看一下。

### 基本结构

优先队列的实现通常使用二叉堆，二叉堆是一个二叉树，这里用数组来实现，`i`的左右子节点分别是`2*i+1`和`2*i+2`，我们的类如下，

```javascript
class Heap {
  // ...
}
```

我们要有一个辅助函数`swap`，它可以原地交换数组中两个元素的位置，没有返回值

```js
swap(i, j) {
  let data = this.data;
  [data[i], data[j]] = [data[j], data[i]];
}
```

现在，我们首先要理解一个函数`heapify(index)`，这个函数的作用是让`index`这个位置的元素到它该到的位置去，可以理解为“下沉”。

```js
// 下沉函数
heapify(index) {
  let target = index;
  let left = index * 2 + 1; // 左子节点
  let right = index * 2 + 2; // 右子节点
  if (left < this.data.length &&
     this.compare(this.data[left], this.data[target])
     ) {
    target = left;
  }
  if (right < this.data.length &&
     this.compare(this.data[right], this.data[target])
     ) {
    target = right;
  }
  if (target !== index) {
    this.swap(target, index);
    // 递归下沉
    this.heapify(target);
  }
}
```

构造函数中会用到`heapify`，先接收一个数组`data`作为参数，和一个比较函数`compare`，

```js
constructor(data, compare) {
  this.data = data;
  this.compare = compare;
  
  // data.length >> 1的作用是，从非叶子节点开始处理，叶子节点不用管，因为已经在最底层
  // 反向处理，确保最后根部的节点是最“强”的
  for (let i = (data.length >> 1) - 1; i>=0; --i) {
    this.heapify(i);
  }
}
```

从优先队列中加入一个数，我们要用到`push`方法，遵循以下步骤，

1. 先把元素添加到数组末尾
2. 不断找到元素的父节点，若父节点更弱，则上升子节点

```js
push(item) {
  // 加到数组末尾
  this.data.push(item);
  // 取到数组末尾的索引
  let index = this.data.length - 1;
  let father = ((index + 1) >> 1) - 1;
  // 当有父节点并且父节点的值更“弱”，就不断把index的值上升
  // father >= 0：等于零表示还有根节点，也可以
  while (father >= 0) {
    if (this.compare(this.data[index], this.data[father])) {
      this.swap(index, father);
      index = father;
      father = ((index + 1) >> 1) - 1;
    } else {
      break;
    }
  }
}
```

移除一个元素很简单，从根节点（即数组第一个元素）移除，然后重新安排第0个元素的位置，

```js
pop() {
  // 已经空了，直接返回undefined
  if (this.empty()) {
    return undefined;
  }
  
  const ret = this.data.shift();
  this.heapify(0);
  return ret;
}

// 顺便实现另外两个方法
empty() {
  return this.data.length === 0;
}
top() {
  return this.data[0];
}
```

到此，优先队列的实现也就全部结束了。

> 注意compare函数，更强则返回true，更弱则返回false，这里和JS的比较函数设计不一样。
>
> 比如，你希望最大的数在根节点（大顶堆），那么应该有`compare(a,b) => a > b`;

### 有序集合

有序集合的设计比较trick，通常的做法是，使用`Set`储存元素，在拿出元素时，使用`Array.from`转成数组，然后对整个数组排序，这在`Set`规模比较小的时候适用。

参考下面的题，

- [设计电影租借系统](https://leetcode-cn.com/problems/design-movie-rental-system/)、[题解](https://leetcode-cn.com/problems/design-movie-rental-system/solution/javascript-an-zhao-guan-fang-ti-jie-shi-au4nu/)

