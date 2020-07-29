---
title: Hello World
date: "2020-07-29"
description: "Hello World"
tags: ["GatsbyJS","Javascript"]
---

GatubyJSを使ってブログを書き始めようと思います。



とりあえず、忘れないようにjavascriptのreduceメソッド
備忘録として。。


```js
const arr = [0, 1, 2]; 
const initial = {};

const result = arr.reduce((acc,value, index) => {
  acc[`number${index}`] = value;
  return acc;
}, initial);
console.log(result)
//resultの値は、{number0: 0,number1: 1,number2: 2}
// reduceの第一引数はコールバック関数
// reduceの第二引数は、reduce一発目のacc
// reduceの二発目は一発目の戻り値(上の例では、acc==={number0: 0})
// reduceの二発目は一発目の戻り値(上の例では、acc==={number0: 0,number1: 1})
// reduceの二発目は一発目の戻り値(上の例では、acc==={number0: 0,number1: 1,number2: 2})
```


