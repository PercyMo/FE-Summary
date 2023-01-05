## DFS和BFS

### 一. 概述
![DFS&BFS](http://img.vanilla.ink/me/webproject/FE-Summary/Algorithm/Algorithm/DFS%26BFS.png?x-oss-process=image/resize,w_500)

1. **广度优先**
    它的特点是越接近根节点越早遍历，所以广度优先搜索一般使用队列实现。
2. **深度优先**
    与 BFS 不同，更早访问的节点可能不是更靠近根节点的节点，一般使用栈实现。

### 二. 二叉树操作
1. [从上到下打印二叉树](./BinaryTreePrint.md)
2. [二叉树的中序遍历](./BinaryTreeInorderTraversal.md)
3. [二叉树的最大深度](./BinaryTreeBasicOperation.md)
4. [路径总和](./PathSum.md)

### 三. 员工重要性
#### 题目
[leetcode - 690. 员工的重要性](https://leetcode.cn/problems/employee-importance/solution/yuan-gong-de-zhong-yao-xing-by-leetcode-h6xre/)  

这里的原题目描述不清晰，应该特别强调是员工的**所有关联下属**，下属的下属...

#### 代码
方法一：深度优先搜索
```js
const GetImportance = function(employees, id) {
  const map = new Map();
  for (let employee of employees) {
    map.set(employee.id, employee);
  }

  const dfs = function(id) {
    const item = map.get(id);
    let total = item.importance;
    const subordinates = item.subordinates;
    for (let subId of subordinates) {
      total += dfs(subId);
    }
    return total;
  };
  return dfs(id);
};
```
方法二：广度优先搜索
```js
const GetImportance = function(employees, id) {
  const map = new Map();
  for (let employee of employees) {
    map.set(employee.id, employee);
  }
  let total = 0;
  const queue = [];
  queue.push(id);
  while (queue.length) {
    const id = queue.shift();
    const employee = map.get(id);
    total += employee.importance;
    const subordinates = employee.subordinates;
    for (let subId of subordinates) {
      queue.push(subId);
    }
  }
  return total;
};
```


### 四. 岛屿数量
#### 题目
[leetcode - 200. 岛屿数量](https://leetcode.cn/problems/number-of-islands/solution/dao-yu-shu-liang-by-leetcode/)  
```js
输入：grid = [
  ["1","1","0","0","0"],
  ["1","1","0","0","0"],
  ["0","0","1","0","0"],
  ["0","0","0","1","1"]
]
输出：3
```

#### 代码
方法一：深度优先搜索
```js
const infect = function(grid, i, j) {
  const len = grid[0].length;
  if (i < 0 || j < 0 || i >= grid.length || j >= len || grid[i][j] !== '1') {
    return;
  }
  grid[i][j] = '0';
  infect(grid, i - 1, j);
  infect(grid, i + 1, j);
  infect(grid, i, j - 1);
  infect(grid, i, j + 1);
};

const numIslands = function(grid) {
  if (!grid || grid.length < 1) {
    return 0;
  }
  const len = grid[0].length;
  let islandsNum = 0;
  for (let i = 0; i < grid.length; i++) {
    for (let j = 0; j < len; j++) {
      if (grid[i][j] === '1') {
        islandsNum++;
        infect(grid, i, j);
      }
    }
  }
  return islandsNum;
};
```
方法二：广度优先搜索
```js
const numIslands = function(grid) {
  if (!grid || grid.length < 1) {
    return 0;
  }
  const len = grid[0].length;
  let islandsNum = 0;
  for (let i = 0; i < grid.length; i++) {
    for (let j = 0; j < len; j++) {
      if (grid[i][j] === '1') {
        islandsNum++;
        const queue = [];
        queue.push([i,j]);
        grid[i][j] = '0';
        while (queue.length) {
          // 这里每一次往队列push时都置成0的原因是防止队列消耗不及时造成bug
          // 早期的写法while循环一次，只感染一个岛屿，导致往队列中重复添加岛屿，本身就会造成重复循环浪费资源，个别情况下还会死循环
          const task = queue.shift();
          const y = task[0];
          const x = task[1];

          if (grid[y - 1] !== undefined && grid[y - 1][x] === '1') {
            queue.push([y - 1, x]);
            grid[y - 1][x] = '0';
          }
          if (grid[y + 1] !== undefined && grid[y + 1][x] === '1') {
            queue.push([y + 1, x]);
            grid[y + 1][x] = '0';
          }
          if (grid[y][x - 1] === '1') {
            queue.push([y, x - 1]);
            grid[y][x - 1] = '0';
          }
          if (grid[y][x + 1] === '1') {
            queue.push([y, x + 1]);
            grid[y][x + 1] = '0';
          }
        }
      }
    }
  }
  return islandsNum;
};
```

### 五. 课程表

### 六. 单词接龙
