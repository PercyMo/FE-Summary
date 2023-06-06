## 查找

### 一. 二分查找
二分查找的条件必须是有序数组
```js
const binarySearch = function(target, array, start, end) {
  if (start > end) {
    return -1;
  }
  const mid = Math.floor((start + end) / 2);
  if (array[mid] === target) {
    return mid;
  } else if (array[mid] < target) {
    return binarySearch(target, array, mid + 1, end);
  } else if (array[mid] > target) {
    return binarySearch(target, array, start, mid - 1);
  }
};
```

### 二. 二维数组查找
#### 题目
在一个二维数组中（每个一维数组的长度相同），每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

#### 基本思路
二维数组是有序的，比如下面的数据：
```js
const arr = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9]
];
```
可以直接利用左下角数字开始查找：  
大于：比较上移  
小于：比较右移  

#### 代码
```js
const find = function(target, array) {
  let i = array.length - 1; // y坐标
  let j = 0; // x坐标
  return compare(target, array, i, j);
};

const compare = function(target, array, i, j) {
  if (array[i] === undefined || array[i][j] === undefined) {
    return false;
  }
  const temp = array[i][j];
  if (temp === target) {
    return true;
  } else if (temp < target) {
    return compare(target, array, i, j + 1);
  } else if (temp > target) {
    return compare(target, array, i - 1, j);
  }
};
```

### 三. 旋转数组中的最小数字
#### 题目
把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。 输入一个非减排序的数组的一个旋转，输出旋转数组的最小元素。 例如数组`[3,4,5,1,2]`为`[1,2,3,4,5]`的一个旋转，该数组的最小值为1。

#### 基本思路
旋转数组其实是由两个有序数组拼接而成的，因此我们可以使用二分法，只需要找到拼接点即可。  
1. `array[mid] > array[right]`:  
    出现这种情况的 array 类似 `[3,4,5,6,0,1,2]`，此时最小数字一定在 mid 的右边。`left = mid + 1`
2. `array[mid] == array[right]`:  
    出现这种情况的 array 类似 `[1,0,1,1,1]`或者 `[1,1,1,0,1]`，此时最小数字不好判断在 mid 左边还是右边，这时只好一个一个试。`right = right - 1`

3. `array[mid] < array[right]`:  
    出现这种情况的 array 类似 `[2,2,3,4,5,6,6]`，此时最小数字一定就是 `array[mid]` 或者在 mid 的左边。因为右边必然都是递增的。`right = mid`

#### 代码
```js
const minNumberInRotateArray = function(array) {
  const len = array.length;
  if (len === 0) return 0;
  let left = 0;
  let right = len - 1;
  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    if (array[mid] > array[right]) {
      left = mid + 1;
    } else if (array[mid] === array[right]) {
      right--;
    } else {
      right = mid;
    }
  }
  return array[left];
};
```

### 四. 排序数组中查找数字
#### 题目
统计一个数字在排序数组中出现的次数。  
`getNumberOfK([0,1,2,3,3,3,3,3,3,3,3,3,3,4,5,6], 3); // 10`

#### 基本思路
1. 直接遍历数组，判断前后的值是否相同，找到元素开始位置和结束位置，时间复杂度 `O(n)`
2. 使用二分查找找到目标值，在向前向后遍历，找到所有的数，比上面略优，时间复杂度也是 `O(n)`
3. 使用二分查找分别找到第一个目标值出现的位置和最后一个位置，时间复杂度 `O(logn)`

#### 代码
```js
const getNumberOfK = function(array, target) {
  if (array && array.length > 0 && target !== null) {
    const len = array.length;
    const firstIndex = getFirstK(array, target, 0, len - 1);
    const lastIndex = getLastK(array, target, 0, len - 1);
    if (firstIndex !== -1 && lastIndex !== -1) {
      return lastIndex - firstIndex + 1;
    }
  }
  return 0;
};

const getFirstK = function(array, target, left, right) {
  if (left > right) {
    return -1;
  }
  const mid = Math.floor((left + right) / 2);
  if (array[mid] === target) {
    if (array[mid - 1] !== target) {
      return mid;
    } else {
      return getFirstK(array, target, left, mid - 1);
    }
  } else if (array[mid] < target) {
    return getFirstK(array, target, mid + 1, right);
  } else if (array[mid] > target) {
    return getFirstK(array, target, left, mid - 1);
  }
};

const getLastK = function(array, target, left, right) {
  if (left > right) {
    return -1;
  }
  const mid = Math.floor((left + right) / 2);
  if (array[mid] === target) {
    if (array[mid + 1] !== target) {
      return mid;
    } else {
      return getLastK(array, target, mid + 1, right);
    }
  } else if (array[mid] < target) {
    return getLastK(array, target, mid + 1, right);
  } else if (array[mid] > target) {
    return getLastK(array, target, left, mid - 1);
  }
};
```

### 五. x 的平方根
#### 题目
给你一个非负整数 x ，计算并返回 x 的 算术平方根。  
由于返回类型是整数，结果只保留整数部分，小数部分将被舍去。  
[leetcode - 69. x 的平方根](https://leetcode.cn/problems/sqrtx/)

#### 代码
```js
const mySqrt = function(x) {
  let l = 0;
  let r = x;
  let ans = -1;
  while (l <= r) {
    const mid = Math.floor((l + r) / 2);
    if (mid * mid === x) {
      return mid;
    } else if (mid * mid < x) {
      ans = mid;
      l = mid + 1;
    } else {
      r = mid - 1;
    }
  }
  return ans;
};
```

### 六. 猜数字大小
#### 题目
每轮游戏，我都会从 1 到 n 随机选择一个数字。请你猜选出的是哪个数字。  
如果你猜错了，我会告诉你，你猜测的数字比我选出的数字是大了还是小了。  
[leetcode - 374. 猜数字大小](https://leetcode.cn/problems/guess-number-higher-or-lower/)  

#### 代码
```js
const guessNumber = function(n) {
  let l = 1;
  let r = n;
  while (l <= r) {
    const mid = Math.floor((l + r) / 2);
    if (guess(mid) === 0) {
      return mid;
    } else if (guess(mid) === 1) {
      l = mid + 1;
    } else {
      r = mid - 1;
    }
  }
  return l;
};
```