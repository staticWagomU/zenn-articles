---
title: "remarkとmarkdown-itでbudouxを扱うライブラリを書いたよ"
emoji: "🦒"
type: "tech"
topics: ["remark", "markdownit", "budoux"]
published: true
publication_name: "gigooo_blog"
published_at: 2024-12-13 09:00
---

## はじめに

みなさんは「budoux」というツールを知っていますか？
英語等では単語間にスペースが入るため、テキスト処理や改行位置の調整が比較的容易です。しかし、日本語では通常、文中で分かち書きを行わないため、自然な改行は意外と困難です。  
このような問題を解決するために、Googleが開発したのが「budoux」というライブラリです。
budouxは、日本語テキストに対して、適切な改行位置を自動的に判定し、そこにゼロ幅スペース（`&#8203;`）を挿入する軽量なライブラリです。これにより、ページレイアウトやデバイス幅に応じた自然な改行を実現することができます。特別なプラグインや追加要素を付与しなくとも、軽量な構成で自然な文字区切りを行える点は嬉しいポイントです。

手動で`<wbr />`タグを埋め込むことで改行位置を制御することもできますが、あまりにも現実的ではありません。そこでbudouxを用いることで、自動的かつ適切な改行ポイントを挿入でき、みんなハッピーというわけです。

私自身は、以下のようなユースケースでbudouxを使いたいと考え、`remark-budoux`と`markdown-it-budoux`というプラグインを作成しました。

- Astroで構築している自分のブログ
- Slidev（Vueを用いたスライド作成フレームワーク）

## remarkプラグインについて

https://www.npmjs.com/package/remark-budoux

簡潔に書くことができました。
下記のコードがその実装です。

https://github.com/staticWagomU/remark-budoux/blob/main/index.js

やっていることは非常にシンプルで、テキストノードに対してbudouxの`translateHTMLString`関数を適用することで、budouxが挿入したゼロ幅スペース入りのHTML文字列を生成します。最後にノードタイプを`html`に変更することで、出力結果がHTMLとして正しく解釈されるようにしています。

![](/images/20241213gigooo_budoux/img1.png)


## markdown-itプラグインについて

https://www.npmjs.com/package/markdown-it-budoux


重要なのはこの部分だけです。

https://github.com/staticWagomU/markdown-it-budoux/blob/main/src/index.ts#L37-L70


`paragraph_open`は、markdown-itがMarkdownをHTMLに変換する過程で使用する内部トークンの一種であり、おおよそのケースでは`<p>`タグへと変換されます。その`<p>`タグに対して、下記のようなスタイルを適用しています。これは`remark-budoux`で出力されたものと同じスタイルです。

```ts
const WORD_BREAK_STYLE = 'word-break:keep-all;overflow-wrap:anywhere;';
```

次に、budouxのparse関数を用いて分ち書きされた文字列の配列を取得します。次にゼロ幅スペースで結合することで、remark-budouxと似た挙動を実現しています。

理想としては`translateHTMLString`によるHTML文字列をそのまま解釈させたかったのですが、`markdown-it`の知識が足りず期待通りの動作をさせることができなかったため、このような力技による実装方法を取っています。



## プラグインの使いかた


### remark-budouxの使いかた

Astroでの使いかたを例に出します。

```shell
$ npm install remark-budoux
```

ライブラリをプロジェクトに追加した後、`Astro.config.mjs`に組み込みます。

```diff js:Astro.config.mjs
+import remarkBudoux from "remark-budoux";

 export default defineConfig({
+  markdown: {
+    remarkPlugins: [remarkBudoux],
+  },
 });
```


### markdown-it-budouxの使いかた

次にSlidevでの使いかたを例に出します。

```shell
$ npm install markdown-it-budoux
```

ライブラリをプロジェクトに追加し、`vite.config.js`に設定を追記します。

```diff js:vite.config.js
 import { defineConfig } from 'vite'
 import '@slidev/cli'
+import markdownItBudoux from 'markdown-it-budoux'

 export default defineConfig({
   slidev: {
 +   markdown: {
 +     markdownItSetup(md) {
 +       md.use(markdownItBudoux)
 +     },
 +   },
   },
 })
```



## おわりに

今回初めてnpmへpublishする経験をすることができました。うれしいね。

残タスクとしてはGithub Actionsが整備しきれていないので、そこを進めていきたいです。

## 参考リンク

https://developers-jp.googleblog.com/2023/09/budoux-adobe.html
