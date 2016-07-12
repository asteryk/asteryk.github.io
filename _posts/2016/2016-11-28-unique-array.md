---
layout: post
title: JS数组去重
categories:
- web
tags:
- javascript
---

## JS数组去重

---

> 不是乱写垃圾文章拼凑数量，而是实际业务中确实有很大的数组处理尤其是后端LIST数据类型的处理，要比对数组元素要去重，这里分享一下最近思考的方法

### 利用ES6的Map和Set来去重

Set的数据类型自带去重的效果，Map可以用has()方法判断key存不存在（这两个都不支持比对数组里的对象的，这里用转成字符串的方法就可以有效解决数组里有复杂数据类型没法一次去重的问题）

`````````````````````````````````
// ES6
function unique (arr) {
  const seen = new Map();
  return arr.filter((a) => {
       // 利用Map的校验自己有没有key的能力，每次构造一个value为1的key值为字符串的键值对
       return !seen.has(JSON.stringify(a)) && seen.set(JSON.stringify(a), 1);
  })
}
// or
function unique(arr) {
    let seen = new Array();
    // 每一项字符串化
    arr.forEach((a) => seen.push(JSON.stringify(a)));
    // 利用set自动去重
    seen = Array.from(new Set(seen));
    // 每一项反字符串化
    return seen.map((a) => JSON.parse(a));
}
`````````````````````````````````

---

### 利用indexOf来判断去重

`````````````````````````````````
function unique(array){
    var n = [];//临时数组
    for(var i = 0;i < array.length; i++){
        if(n.indexOf(JSON.stringify(array[i])) == -1) n.push(JSON.stringify(array[i]));
    }
    for (var j = 0;j < n.length; j++) {
         n[j] = JSON.parse(n[j]);
    }
    return n;
}
```````````````````````````````````

---

PS:以上方法均支持数组里有复杂数据对象的，比如这样的
```````````````
var a = [
    { a: 1 },
    { b: 2 },
    { a: 3 },
    { c: 2 }
];
```````````````
indexOf可以换成JQuery的inArray方法，不做去重做数组内容比对也是很容易的

