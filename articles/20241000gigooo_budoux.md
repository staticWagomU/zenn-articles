---
title: "remarkとmarkdown-itでbudouxを扱う"
emoji: "🦒"
type: "tech"
topics: ["remark", "markdownit", "budoux"]
published: false
publication_name: "gigooo_blog"
---
何を伝えたいか
- remarkとmarkdown-itでbudouxを使えるようになった
- remarkはastroで使える
- markdown-itはslidevで使える
- いい感じに日本語が改行されると嬉しい


## はじめに

budouxというものを知っていますか？
英語であればスペースが単語の区切りとして機能しますが、日本語では単語ごとにスペースを空ける文章の書きかた(分かち書き)をしません。
その分かち書きを実現するためにGoogleが開発したライブラリがbudouxです。
軽量なため扱いやすいのは嬉しいポイントですね。


手動で`<wbr />`タグを埋め込むことで自然な改行を実現することができますが、ブログ等の長文でそれをするのはあまりにも無謀です。しかしbudouxを使うこと楽をすることができます。

自分は下記のユースケースでbudouxを使いたいです。

- Astroで作っている自分のブログ
- SlidevというVueでスライドを作成するためのフレームワーク


そして、remark-budouxとmarkdown-it-budouxを作りました。

:::message
調べてみたところ小学生低学年の国語教科書では分かち書きされているようです。
:::


4. markdown-itプラグインの作成
   - プラグインの基本構造
   - budouxを組み込む方法
   - 実装上の注意点

5. remarkプラグインの作成
   - プラグインの基本構造
   - budouxを組み込む方法
   - 実装上の注意点

6. プラグインの使用方法
   - インストール方法
   - 設定方法
   - 使用例

8. まとめ
   - プラグイン作成の振り返り
   - 今後の展望や改善点

## 参考リンク

https://developers-jp.googleblog.com/2023/09/budoux-adobe.html
