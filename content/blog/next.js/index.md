---
title: next.js
date: "2020-11-26"
description: "Reactアプリをnext.js化"
tags: ["next.js", "Typescript", "react", "vercel"]
---

フロント React、バック express でアプリを作りました。
ある程度形にはなったんですが、問題が発覚。React は、SPA。
URL を直接打ち込んで、ページ遷移ができません。
どうしても、実装したい機能なので、
作成した React アプリを next.js 化することにしました。

そもそも、なぜ当初から next.js で実装しなかったか。
その理由は私の知識不足です。
私の当初の印象は、next.js は「SSR できる React」でした。
せっかく SPA なのに、
わざわざ rails 等と同じようにサーバーに問合せするなんでどうなの？
と思っていたのです。
そのような先入観から、next.js をスルー。
しっぺ返しを食らいました。

next.js とは client 側のサーバーを使用し、
ページをプリレンダリングすることで、
高速にページを表示することを可能にした React を使ったフレームワークです。

詳しくは、下記ページが非常に参考になります。
https://tadtadya.com/summary-of-the-web-site-display-process-flow/

プリレンダリングによって、表示が高速になります。
動的なコンテンツも、SSG,SSR の２パターンがあり、
SSG は build 時にデータを取得して、静的なページを作成するので
非常に高速です。
SSR は、毎リクエストごとに server へデータを問合せるため、
SSG には及びませんが、動的なコンテンツ以外の事前準備は
バッチリしているので、非常に軽快な動きをしてくれます。

React と比較して、下記の学習が必要となります。
他にも多々あると思いますが、勉強したら、都度ブラッシュアップします。

- app.tsx,\_document.tsx などの特殊なページファイル。
- /src/pages/以下ファイルでのルーティング。
- getServerSideProps,getStaticProps など独自の関数
