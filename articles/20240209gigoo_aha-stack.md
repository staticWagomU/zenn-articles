---
title: "AHAスタックでTODOアプリを作った"
emoji: "🦒"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["astro", "htmx", "alpinejs", "tailwindcss"]
published: true
publication_name: "gigooo_blog"
---
## はじめに
新年早々に[The AHA Stack](https://flaviocopes.com/the-aha-stack/)というタイトルのブログ記事の投稿を見かけました。

Astro + htmx + Alpine.jsの頭文字をとったスタックで、[ドキュメント](https://ahastack.dev/)のトップページにはこのように書かれています。
> Combine Astro, htmx and Alpine.js to create modern web applications sending HTML over the wire, replacing the SPA JS-heavy approach with a much simpler set of mental models and workflows.
> 
>翻訳:
>Astro、htmx、Alpine.js を組み合わせて、HTML をネットワーク経由で送信する最新の Web アプリケーションを作成し、SPA JS を多用したアプローチをはるかにシンプルなメンタル モデルとワークフローのセットに置き換えます。

自分の好きなフレームワークのAstroを採用したスタックであること、[JavaScript ライジングスター 2023](https://risingstars.js.org/2023/ja#section-framework)のフロントエンドフレームワーク部門で2位になっているhtmxが採用されている点に惹かれて触ってみることにしました。
ドキュメントのexampleではAstroとhtmxを使ってカウンターアプリを作っていて、Alpine.jsだけ仲間外れにされているのは可哀そうなので３つを使ってTODOアプリを作りました。

完成品はこちらです。
https://github.com/staticWagomU/AHA-TODO

## 作った感想
htmlを返すバックエンドが既に用意されていて、実装するものもシンプル（単発の個人開発など）であれば選択肢に入るのかなと感じました。
バックエンドも同じフレームワークを使うのであれば、例えばbunのほうが扱い易いだろうなと感じました。
https://zenn.dev/yusukebe/articles/e8ff26c8507799

リクエスト周りの処理は全部htmxにするぞ！！と意気込んで作ったものの最終的にはAlpine.jsも使うことになってしまい悔しい結果となりました。

## 各フレームワークについて
### Astro
ブログ等の静的ページを扱うことが得意なフレームワーク

ドキュメントはこちら
https://astro.build

### htmx
`<div hx-post="/hoge"></div>`のようにhtmlの属性を使ってAJAX通信等が扱える軽量なライブラリ

ドキュメントはこちら
https://htmx.org

scriptタグを追加するだけで使えるため雑に導入することができます。
### Alpine.js
シンプルで軽量でパワフルなフレームワーク
>Simple.
  Lightweight.
  Powerful as hell.

らしいです。  
htmxと同じくhtmlの属性を使うので軽微なインタラクションをjsを書かずに実現できます。

ドキュメントはこちら
https://alpinejs.dev

Alpine.jsもhtmxと同様にscriptタグを追加するだけで使えるフレームワークです。

## 採用したライブラリ

今回作成したAHA-TODOでは、Astro・htmx・Alpine.jsに加えて
- tailwind.css
- zod

を採用していますが、これらのフレームワークについての説明は割愛します。

## 実装した機能
今回下記の機能を実装しました。
TODOの追加
![create TODO item](/images/gigoo_aha-stack/create.gif)
TODOの削除
![delete TODO item](/images/gigoo_aha-stack/delete.gif)
TODOの更新
![change TODO title](/images/gigoo_aha-stack/change.gif)

バックエンド処理はAstroで実装しています。

AHAのHにあたるhtmxではリクエストの発火とレスポンスをdomに反映する部分。具体的には、
- 各種HTTPリクエストの発火
- レスポンスの反映

で使っています。

AHAの最後のAにあたるAlpine.jsでは、主にＵＩインタラクションを必要とする部分。具体的には、
- TODOが未入力のときにボタンを押せないように制御する
- TODO編集時にラベルをテキストボックスに切り替える

で使っています。


## 実装

### Astroを使ったバックエンド処理
Astroを使ったバックエンド処理の中核をなすのが、Astro3.4から追加されたPage Partials機能です。AHA stackのチュートリアルでも使われている機能で、`<!DOCTYPE html>`などの無駄なタグが含まずにhtmlの部品としてレンダリングされるので返ってきたhtmlをそのまま埋め込むhtmxと相性がよくPage Partials機能とライブラリを組み合わせる例としてhtmxについて触れられています。
https://docs.astro.build/en/basics/astro-pages/#using-with-a-library

バックエンドのソースは
- [`todos/index.astro`](https://github.com/staticWagomU/AHA-TODO/blob/main/src/pages/todos/index.astro)
- [`todos/[id].astro`](https://github.com/staticWagomU/AHA-TODO/blob/main/src/pages/todos/%5Bid%5D.astro)
 の２ファイルです
APIは下記設計で作成しました。

| 処理 | エンドポイント | メソッド |
| --- | --- | --- |
| 取得 | /todos | GET |
| 作成 | /todos | POST |
| チェックの更新 | /todos/{id} | PUT |
| タイトルの更新 | /todos/{id} | PATCH |
| 削除 | /todos/{id} | DELETE |
### htmxを使ったリクエスト処理
主に`hx-*`始まりの属性がhtmx由来の独自属性です。

この部分ではsubmit時にTODOを作成するエンドポイントへpostメソッドを使ってリクエストを発行しています。
https://github.com/staticWagomU/AHA-TODO/blob/main/src/pages/index.astro#L26-L50

### Alpine.jsを使ったＵＩインタラクション
主に`x-*`始まりの属性がAlpine.js由来の独自属性です。

TODO作成ボタンの制御は、この部分で実装しています。
`x-data`で定義した`input`が空のときはボタンを非活性化してスタイルを変化させるインタラクションを実現しています。
https://github.com/staticWagomU/AHA-TODO/blob/main/src/pages/index.astro#L32-L49

TODOの編集処理はこの部分で実装しています。理想としては`<template>`タグと`x-if`を使って表示を制御したかったのですが、Astroでは？`<template>`タグをうまく認識してくれず`x-show`を使って表示制御をしています。
https://github.com/staticWagomU/AHA-TODO/blob/main/src/pages/todos/index.astro#L39-L108

### Alpine.jsを使ったTODOの更新処理
悔しいことにタイトルの編集時のPATCHリクエストに関してはhtmxだけで実装することが困難でした。そのため、jsを書いてAlpine.jsからAJAX処理を行うことでTODOの更新処理を実装しました。
https://github.com/staticWagomU/AHA-TODO/blob/main/src/pages/todos/index.astro#L39-L108


## 課題
まるで問題なく動作しているような顔をしてアニメーションgifを貼っていますが、実際のところ２点ほどうまくいかなかった処理があります。
### 2回目以降のPOST処理実行時に表示崩れが発生する
![badpoint 1](/images/gigoo_aha-stack/badpoint1.gif)

２回目以降のPOST処理を実行したときに、最後のTODO以外で編集時に表示されるはずのDOM要素が見えてしまいます。初期ロード時のGETでは同様の現象が発生しないのでどのようにこの問題を回避すればよいのか悩んでいます。

### TODO作成処理の実行後にtextboxを空にする
![badpoint 1](/images/gigoo_aha-stack/create.gif)
こちらは`hx-post`を使ってPOSTリクエストを発火しているため、POST発火後にAlpine.jsの処理を挟むことができず入力文字列が残ってしまいます。この問題はPOSTメソッドをAlpine.jsから発火することで解決できます。しかし、そうなるとhtmxの存在意義が薄れてしまうのでうーん。。。となっています。

## おわりに
最終的にjsを書くことになってしまいとっても悔しいです。

## おわり２
今回は原典に従って「Astro + htmx + Alpine.js」という組み合わせを使ってTODOアプリを作成しました。しかし、このページを読めばわかるようにThe AHA StackはFlavioさん（The AHA Stackの考案者）がこれが良いと思い発表したものであって、気に入らないものがあれば組み合わせを替えて使えば良いというふうにも言っています。
https://ahastack.dev/faq/4/
https://ahastack.dev/comparisons/similar-things/
Astroが気に入らなければDjangoやbunを使うといったように入れ替え可能だということです。ですからこのAHA Stackはあくまで１つの例に過ぎないということです。ですから私は上であげた課題を解決するために「Astro + Alpine AJAX + Alpine.js」で同じTODOアプリを作ろうと思います。

## 参考
https://flaviocopes.com/the-aha-stack/
https://astro.build/
https://htmx.org/
https://alpinejs.dev/
