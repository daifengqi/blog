# 图与搜索：从图的拷贝开始说起吧

# Graph & Search

## 2021 年 4 月 27 日

## Apr 27 2021

---

### 引言

图和搜索紧密相连。理解图论的首要基础是理解深度优先搜索（DFS）和宽度优先搜索（BFS）及其背后的数据结构和算法思想。我们从[克隆图](https://leetcode-cn.com/problems/clone-graph/)开始说起吧。

### 图拷贝：深搜

```javascript
var cloneGraph = function(node) {
    const hash = new Map();
    
    function dfs(node) {
        if (node === null) {
            return null;
        }
        if (hash.has(node.val)) {
            return hash.get(node.val);
        }

        const cloneNode = new Node(node.val, []);
        hash.set(node.val, cloneNode);
        for (const n of node.neighbors) {
            cloneNode.neighbors.push(dfs(n));
        }
        return cloneNode;
    }

    return dfs(node);
};
```

注意，JavaScript的Map和Object对象都是只能以字符串作为“键”，这里默认“键”值就是点的索引号，所以只能以点的值作为键，从旧图的点映射到拷贝后新图的点。

另一种更直观的写法是1.先通过遍历获得所有点的引用，储存到一个数组里，2.然后挨个取出点，创建旧点到新点到映射；3.再挨个取出新点，创造边，最后返回给定点映射到新点的引用。

下面我们来看看BFS的写法。

### 图拷贝：宽搜

```javascript
var cloneGraph = function(node) {
    if (node === null) {
        return node;
    }
    const que = [];
    que.push(node);
    const hash = new Map();
    hash.set(node.val, new Node(node.val, []));

    while(que.length > 0) {
        const curr = que.shift();
        for (const nei of curr.neighbors) {
            if(!hash.has(nei.val)) {
                hash.set(nei.val, new Node(nei.val, []));
                que.push(nei);
            }
            const cloneNode = hash.get(curr.val);
            cloneNode.neighbors.push(hash.get(nei.val));
        }
    }

    return hash.get(node.val);
};
```

宽搜利用队列作为数据结构，确保入列的节点都是没有访问过的（`!hash.has(node.val)`）每拿出一个节点，都遍历这个节点的邻接节点，如果邻接节点没有访问过，那么加入队列，并且设置为已经访问。

注意，虽然树是一类特殊的图，但是它们之间有个重要的区别：树的遍历一般是从根节点开始，到叶子结点结束，中途是不能重复地访问同一个结点的，这是因为并没有从子节点到父节点的路径，所以我们不需要记录已访问过的节点；而在图的搜索中，不论是dfs还是bfs，都需要一个集合来储存已访问过的节点，从而避免重复访问。使用哈希表来判断节点有没有被访问过几乎是避免重复访问的唯一且通用的方法。

### 最短路问题

通过BFS + 一个记录距离的哈希表可以快速简单地完成最短路算法，有兴趣的话可以试一试。<u>最短路问题一定要用宽搜</u>，为什么？因为需要逐步记录每一个点到起始点的距离，这就需要由近到远的遍历。

最短路涉及到BFS的层序遍历，即每次取`size=queue.length`，然后对`size`的规模遍历，这样做的好处是，控制遍历时的节点一定是上一轮遍历加入时的节点，在每轮遍历之间可以进行某些处理。

### 拓扑排序

宽度优先搜索（队列）的另一个应用是拓扑排序`（topSort）`，用在有依赖关系的「有向图」中。这个图必须是「无环」的，否则会造成类似死锁的情况。拓扑排序的思想是不断地找到入度为0度点，加入队列，然后更新其他点的入度，不断循环。<u>拓扑排序的本质还是搜索，不论是深搜还是宽搜</u>。

### 排列组合：回溯型搜索

图和搜索问题有三种：

1. 典型的树（考点：树的遍历）
2. 典型的图（考点：深搜、宽搜、并查集）
3. 排列组合问题（考点：递归与回溯）

组合问题的复杂度是`O(2^n)`，排列问题的复杂度是`O(n!)`，都是非常高的，所以当要使用这些方法时，要确保数据量是比较小的。当确定要用时，就不要再考虑复杂度的问题了；当你用这些方法时，就已经将复杂度弃之脑后了。

组合问题还有一种「状态压缩」的做法，即利用二进制位代表对应位置选或不选。一个例子是：

```javascript
const n = nums.length;

for (let i=0; i<(1<<n); ++i) {
  for (let j=0; j<n; ++j) {
    if ( i & (1 << j) != 0) {
      subset.push(nums[j]);
    }
  }
  res.push(subset);
}
```

话说回来，我们来看看遍历型回溯的经典模板：

```javascript
const n = nums.length;
function helper(res, path, nums) {
  if (path.length === n) {
    res.push(path.slice());
    return;
  }
  
  for (let i=0; i<n; ++i) {
    path.push(nums[i]);
    helper(res, path, nums);
    path.pop();
  }
}
```

有时候回溯也需要一个`visited`数组或集合来去重。

```javascript
for (let i=0; i<n; ++i) {
  if (visited[i] === 1) {
    continue;
  }
  if (i > 0 && nums[i] === nums[i-1] && visited[i-1] === 0) {
    continue;
  }
  
  visited[i] = 1;
  path.push(nums[i]);
  helper(path);
  path.pop();
  visited[i] = 0;
}
```

回溯和深度优先搜索有一个很好的练习题：[N皇后](https://leetcode-cn.com/problems/n-queens/)，这个问题比较复杂，需要写多块函数，本质也是回溯的思想。

一般来说，「<u>return all possible solutions</u>」的问题，都需要考虑回溯，也就是当碰到一个解时，把<u>一个解</u>放入到题解组里，然后回退，继续回溯接下来的解。

### 关于回溯的练习题

- [全排列](https://leetcode-cn.com/problems/permutations/)
- [组合总和](https://leetcode-cn.com/problems/combination-sum/?utm_source=LCUS&utm_medium=ip_redirect&utm_campaign=transfer2china)，[~2](https://leetcode-cn.com/problems/combination-sum-ii/description/)，[~3](https://leetcode-cn.com/problems/combination-sum-iii/)，[~4](https://leetcode-cn.com/problems/combination-sum-iv/)

这里我写了两篇题解，第一篇是[组合问题1](https://leetcode-cn.com/problems/combination-sum/solution/javascript-dai-you-suo-yin-zhi-de-hui-su-5mtp/)，使用带索引值`i`的回溯，即每次递归的子函数都传入索引值`i`或者`i+1`；另一篇题解是[组合问题2](https://leetcode-cn.com/problems/combination-sum-ii/solution/javascript-bu-dai-suo-yin-zhi-zhi-jie-bi-5c3p/)，这个写法不带索引，而是在每次递归里都遍历整个数组。

对比两个写法，有什么区别吗？在大多数情况下，<u>带索引的写法更好</u>，因为代码量更少，逻辑清晰，而且可以避免重复。

在[组合总和3](https://leetcode-cn.com/problems/combination-sum-iii/solution/javascript-hui-su-he-er-jin-zhi-fa-by-ce-ruju/)这个问题的题解中，我使用了「回溯」和「二进制」两种方法。回溯法是解决此类问题的通用方法，而单纯地在数组的某个位置上选（1）或不选（0）的问题也可以用二进制，即「状态压缩」的方法解决。注意对比两个方法的复杂度，和两种完全不同的思考问题的方式。

[组合问题4](https://leetcode-cn.com/problems/combination-sum-iv/)属于完全背包-顺序无关-组合问题，只用套[这篇文章](https://cescdf.com/blog/202104/dynamic-programming-and-knapsack-problem)里讲过的模板就好。

### 关于图论的练习题

- [单词接龙](https://leetcode-cn.com/problems/word-ladder/)，[~2](https://leetcode-cn.com/problems/word-ladder-ii/)

一些问题第一眼无法看出是图论问题，最典型的例子就是这两个【word ladder】问题。类似问题首先需要考虑「建图」，即从给定的输入转换成一张图（用邻接表或矩阵表示），之后转换成图论的经典问题，比如单词接龙就是经典的最短路问题，

### 并查集

关于图论，还有一个非常好用的数据结构也建议掌握，那就是「<u>并查集</u>」，以下给出并查集的一个JS代码：

```javascript
class Union {
    constructor(n) {
        this.arr = new Array(++n);
        while(n--) {
            this.arr[n-1] = n-1;
        }
    }

    find(x) {
        return x === this.arr[x]? x : this.find(this.arr[x]);
    }

    conn(x, y) {
        let ux = this.find(x);
        let uy = this.find(y);
        if(ux ^ uy) {
            this.arr[ux] = uy;
        }
    }

    isConn(x ,y) {
        return this.find(x) === this.find(y);
    }
}
```

并查集可以快速、轻易、高效地解决连通性问题，一旦写出了并查集的数据结构，代码正文往往可以很简洁地写出。

有一些题目用到了并查集的拓展，比如[【打砖块】](https://leetcode-cn.com/problems/bricks-falling-when-hit/)需要维护连通分支的个数，打砖块这道题还体现了一个常用处理二维数组或矩阵的技巧：规模为`(h,w)`矩阵可以<u>转换为一维</u>数组，初始化长度为`(h * w + 1)`，原来的索引为`(i,j)`的点可以表示为`(i * w + j)`。

还有一些并查集需要维护「有向性」，找到例题之后会继续更新。