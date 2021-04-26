# 图论基础：从图的拷贝开始

# Graph Theory Introduction

## 2021 年 4 月 27 日

## Apr 27 2021

---

### 引言

图和搜索紧密相连。

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
```

注意，JavaScript的Map和Object对象都是只能以字符串作为“键”，这里默认“键”值就是点的索引号，所以可以直接以点的值作为键，从旧图的点映射到拷贝后新图的点。

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

