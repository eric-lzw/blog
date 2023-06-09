---
title: JS进阶之数组扁平化的六种实现方式
date: 2018-09-22 13:12:26
type: "js-base"
tag: js
description:
keywords:
top_img:
mathjax:
katex:
aside:
---

## 数组扁平化的六种实现方式

给定数组：

```javascript
let arr = [1, [2, [3, [4, 5, [6, 7]]]], 8];
let str = JSON.stringify(arr);
```

实现其扁平化：

```javascript
[1, 2, 3, 4, 5, 6, 7, 8]
```



### 1. flat()

```javascript
var arr_flat = arr.flat(Infinity) // 参数：深度，默认为1
```



### 2. 正则+split()

```javascript
var arr_flat = str.replace(/(\[|\])/g, '').split(',')
```



### 3. 正则+JSON.parse()

```javascript
str = str.replace(/(\[|\])/g, '')
str = `[${str}]`
var arr_flat = JSON.parse(str)
```



### 4. for循环递归判断

```javascript
function MyFlat (arr) {
  let result = [];
  let fn = function (arr) {
    for (var i = 0; i < arr.length; i++) {
      if (Array.isArray(arr[i])) {
        fn(arr[i])
      } else {
        result.push(arr[i])
      }
    }
  }
  fn(arr)
  return result;
}
var arr_flat = MyFlat(arr)
```



### 5. reduce

```javascript
function reduceFlat (arr) {
  let fn = function (arr) {
    return arr.reduce((pre, cur) => {
      return pre.concat(Array.isArray(cur) ? fn(cur) : cur)
    }, [])
  }
  return fn(arr)
}
var arr_flat = reduceFlat(arr)
```



### 6. 扩展运算符+some()

```javascript
function MyFlat (arr) {
  while(arr.some(Array.isArray)) {
    arr = [].concat(...arr)
  }
  return arr
}
var arr_flat = MyFlat(arr)
```

