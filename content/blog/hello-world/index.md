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

// reduceの第一引数はコールバック関数 =>
// (acc,value, index) => {acc[`number${index}`] = value;return acc;}
// reduceの第二引数は、reduce一発目のacc => {}
// reduce二回目のaccの値は一発目の戻り値(上の例では、acc==={number0: 0})
// reduceの三回目のaccの値は二発目の戻り値(上の例では、acc==={number0: 0,number1: 1})
// 結果の、resultの値は、{number0: 0,number1: 1,number2: 2}
```


