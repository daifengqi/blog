# 承前启后：中级数据结构

# Mid-level Data Structure

## 2021 年 4 月 25 日

## Apr 25 2021

---

### 引言

中级数据结构指的是单调栈、单调队列、优先队列等<u>在多数语言的标准库里内置或可直接用数组表示且要求某些特征</u>的数据结构；与之相比，初级数据结构指链表、栈、队列、二叉树等数据结构入门知识包括的内容；高级数据结构指并查集、线段树、树状数组等竞赛要求的数据结构。此外，还有终极数据结构：红黑树、跳跃表等...

> 如果面试官让你手写红黑树，那恐怕是在劝退了，建议直接起身说拜拜。
>
> ——[沃兹基·硕徳](https://cescdf.com)

### 单调栈

单调栈的本质是栈，但是从栈底到栈顶的元素数值大小，或元素为索引指向的数值大小符合（严格或非严格）单调性。

```javascript
// 单调递增栈，后面元素的比较用“>”还是“>=”要依据具体情况
while(stk.length > 0 && heights[stk.top()] >= heights[i]) {
  stk.pop();
}
```

[点击这里见题解：【柱状图中最大的矩形】](https://leetcode-cn.com/problems/largest-rectangle-in-histogram/solution/javascript-dan-diao-zhan-by-cescdf-pzyo/)

### 单调队列

单调队列既具有栈的性质，也具有队列的性质，所以它必须是一个双端队列。

1. `deque`内仅包含窗口内的元素 ⇒ 每轮窗口滑动移除了元素 `nums[i - 1]`，需将 `deque`内的对应元素一起删除。
2. `deque` 内的元素非严格递减 ⇒ 每轮窗口滑动添加了元素 `nums[j + 1]`，需将`deque`内所有 `< nums[j + 1]`的元素删除，因为要维护最大值。

```javascript
// 单调队列：已知窗口长度k，当碰到（之前添加过的）该元素时，从队列首部取出
if (deque[0] === nums[i-k]) {
  deque.shift();
}
while (deque.length > 0 && deque.last() < nums[i]) {
  deque.pop();
}
```

[点击这里见题解：【滑动窗口的最大值】](https://leetcode-cn.com/problems/hua-dong-chuang-kou-de-zui-da-zhi-lcof/solution/javascript-dan-diao-zhan-dan-diao-dui-li-3g3h/)

### 优先队列

在<u>ECMAScript</u>标准中并不具备优先队列的实现，这里贴一个手动实现的最小堆，它用数组表示一个二叉树，树的左子节点索引为`2i+1`，右子节点索引为`2i+2`，所以通过子节点得到父节点索引的方式为`(i-1 >> 1)`。

```javascript
// 最小堆
class MinHeap {
    constructor() {
        this.heap = [];
    }

    getParentIndex(i) {
        return (i - 1) >> 1;
    }

    getLeftIndex(i) {
        return i * 2 + 1;
    }

    getRightIndex(i) {
        return i * 2 + 2;
    }

    shiftUp(index) {
        if(index === 0) { return; }
        const parentIndex = this.getParentIndex(index);
        if(this.heap[parentIndex] > this.heap[index]){
            this.swap(parentIndex, index);
            this.shiftUp(parentIndex);
        }  
    }

    swap(i1, i2) {
        const temp = this.heap[i1];
        this.heap[i1]= this.heap[i2];
        this.heap[i2] = temp;
    }

    insert(value) {
        this.heap.push(value);
        this.shiftUp(this.heap.length - 1);
    }

    pop() {
        this.heap[0] = this.heap.pop();
        this.shiftDown(0);
        return this.heap[0];
    }

    shiftDown(index) {
        const leftIndex = this.getLeftIndex(index);
        const rightIndex = this.getRightIndex(index);
        if (this.heap[leftIndex] < this.heap[index]) {
            this.swap(leftIndex, index);
            this.shiftDown(leftIndex);
        }
        if (this.heap[rightIndex] < this.heap[index]){
            this.swap(rightIndex, index);
            this.shiftDown(rightIndex);
        }
    }

    peek() {
        return this.heap[0];
    }

    size() {
        return this.heap.length;
    }
}
```

### 结尾语

其他数据结构，比如[最小栈](https://leetcode-cn.com/problems/min-stack/)，也属于一种“中级”的数据结构，它们通常基于某种基础结构，然后加入了某些特定的方法，包装成一个新类。新方法往往有时间复杂度的限制。

练习题与题解：

- [LRU Cache](https://leetcode-cn.com/problems/lru-cache/)
- [四种方法解决TopK问题](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/solution/javascriptsi-chong-fang-shi-jie-topkwen-ti-by-user/)