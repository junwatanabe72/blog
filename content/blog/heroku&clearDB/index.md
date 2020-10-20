---
title: herokuでclearDBを使用
date: "2020-10-20"
description: "herokuでclearDBを使用"
tags: ["heroku", "Typescript", "sequelize", "clearDB"]
---

フロント React、バック express でアプリを作ってます。
20 年 10 月までに完了させたかったのですが、
思うように進まず、10 月中完了と目標を修正しました笑

なんとか、動かせるまでにはなったので、demo をデプロイ。
フロントは、netlify。
バックは、heroku。
express を heroku にデプロイした際に、
ハマったところを備忘録として、残したいと思います。

ハマり点
heroku の clearDB 内の日本語が文字化けする。
(日本語が"?"に変換されてしまう。)
mysql では、my.cnf 等々で、エンコードを設定しますが、
heroku では、それをいじることが出来ません。（よね？）
db を確認したら、「latin1」となってました。
「latin1」=>「utf8mb4」とするために、
いろいろググって、試行錯誤しました。。。

環境変数を修正する。

https://stackoverflow.com/questions/59244361/how-to-set-utf8-encoding-for-cleardb-mysql-on-heroku

https://xyk.hatenablog.com/entry/2015/01/14/143508

解決できず。。。

他に dump ファイルの各 table に直接書き込んで修正を試みましたが、
一旦は、成功するものの、時間が立つと「latin1」に戻ってしまいました。

結論として、express の sequelize 側の default 設定を修正することで
解決しました。

migrations フォルダに charset を変更する migration ファイルを追加する

https://blog.inomar.me/posts/20191204_sequelize

heroku,clearDB では、検索にヒットしなかったので、
いろんな角度から検索することが大切だと感じました。。。

```js
"use strict"

module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.sequelize.query(
      `ALTER DATABASE ${queryInterface.sequelize.config.database} CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;`
    )
  },

  down: (queryInterface, Sequelize) => {},
}
```
