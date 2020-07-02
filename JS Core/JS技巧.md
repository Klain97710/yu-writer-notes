# JS技巧

### 忘记var的副作用

* 通过var创建的全局变量（任何函数之外的程序中创建）是不能被删除的
* 没有通过var创建的隐式全局变量（无论是否在函数中创建）是能被删除的

所以隐式全局变量并不是真正的全局变量，但它们是全局对象的属性。
属性是可以通过delete操作符删除的，而变量是不能的。

```javascript
var name = 'klain';
// 全局对象：{..., name:'klain', ...}
delete this.name // false

name = 'kevin';
// 全局对象：{..., name:'kevin', ...}
delete this.name // true
```

### 使用`Object.is()`进行比较操作

由于JS是弱语言，在一些情况下，`==`比较操作会出现隐式类型转换，导致出现意想不到的结果：

```javascript
0 == '' // true
null == undefined // true
[1] == true
```

因此JS提供了`===`全等操作符，但仍有特殊情况：

`NaN === NaN // false`

ES6提供了新的`Object.is()`方法：

```javascript
Object.is(0, '') // false
Object.is(null, undefined) // false
Object.is([1], true) // false
Object.is(NaN, NaN) // true
```

### 

## Array技巧

### 迭代一个空数组

```javascript
var arr = new Array(4);
// [,,,]
// 谷歌浏览器中是 [empty × 4]

// 一个松散的数组去循环调用一些转换是非常难的
arr.map((elem, index) => index);
// [,,,] 毫无作用

// 解决
var arr = Array.apply(null, new Array(4));
// [undefined, undefined, undefined, undefined]
arr.map((elem, index) => index);
// [0, 1, 2, 3]
// 更多方法参考GO ALL OUT/积跬步/1.5.1
```

### 数组去重

```javascript
var arr = [...new Set([1, 2, 3, 3])];
// [1, 2. 3]
```

