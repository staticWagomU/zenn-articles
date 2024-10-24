---
title: "個人的にいいと思うVim_Neovimの始め方"
emoji: "🦒"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["neovim", "vim"]
published: true
published_at: 2024-10-25 00:00
publication_name: "vim_jp"
---
:::message
この​記事は[Vim駅伝](https://vim-jp.org/ekiden/)2024年10月25日(金)の​記事です。

前回の​記事は​ [hokorobi](https://github.com/hokorobi)さんの​「[osyo-manga/vim-operator-stay-cursor に PR した](https://zenn.dev/hokorobi/articles/27234d7ede254e)」と​いう​記事でした。

次回の​記事は​ 10月28日(月) に​投稿される​予定です。
:::

---

## はじめに

自分のVimの始め方が新たに初心者がVimを始める人にも良いのではないかと感じたので書き下してみます。

※ 以下Vim/NeovimはVimと表記します。

## まずはメモ帳として使おう！

はじめからVimに全ベットするのは難易度が高いのでまずはメモ帳として使うことをお勧めします。

そのためには、
- インサートモードへの入り方 `:h i`
- ノーマルモードへの入り方 `:h i_esc`
- ファイルの作成・保存・終了方法 `:h e`, `:h w`, `:h q`

これを覚えればメモは取れるので、メモを取るついでにカーソル操作をマスターしましょう

## 気になった操作方法を覚えてみる！

メモを取っていると、、
- 範囲選択 `:h v`
- コピーペースト `:h y`, `:h p`
- 行削除 `:h d`, `:h dd`

等の操作をしたくなると思います！
追加で
- オペレーター `:h 04.1
- モーション `:h 04.1`
- テキストオブジェクト `:h 04.8`

を覚えるときっとVimにはまり、メインエディタをVimに変えたいなーと思うはずです！！！！

### プラグインマネージャーを入れてみる！

ここまでたどり着いた人は、そろそろプラグインを入れたくなっている頃合いだと思います。
プラグインを入れるためには、プラグインマネージャーを導入するのが一般です。
ここで、はじめに導入するプラグインマネージャーとしては以下のプラグインマネージャーがお勧めです。
- [junegunn/vim-plug](https://github.com/junegunn/vim-plug) (Vim/Neovim)
- [tani/vim-jetpack](https://github.com/tani/vim-jetpack) (Vim/Neovim)

Neovimでluaを使いたい人はこちらもおすすめです。
- [folke/lazy.nvim](https://github.com/folke/lazy.nvim) (Neovim限定)

導入方法についてはREADMEをgoogle 翻訳にかけながら頑張ってみましょう！！

## プラグインを入れてみる

NeoVim使いの人は
https://github.com/yutkat/my-neovim-pluginlist
https://dotfyle.com/

これらのサイトでプラグインを探すのがおすすめです。

Vim使いの人は
https://vimawesome.com/
https://github.com/akrawchyk/awesome-vim

あたりがいいんですかね？Web検索で`Vim プラグイン　おすすめ`などで検索してみてください。

### プラグインの設定方法がわからないときは？

まずは、プラグインのヘルプをよく読んでみましょう。英語で分からない人はGoogle翻訳へコピペしましょう。大体理解できます。
あとは、GitHubで調べてみましょう。意外と沢山の設定例が出てくるので大抵の問題は解決できるはずです。

### 設定の楽しさを覚え始めたあなたへ

楽しくなってきた人には、Shougoさんが作っている「dark powered」なプラグインをおすすめします。
具体的には
- プラグインマネージャーの[Shougo/dpp.vim](https://github.com/Shougo/dpp.vim) 
- ファジーファインダープラグインの[Shougo/ddu.vim](https://github.com/Shougo/ddu.vim)
	- 少し語弊のある説明なので、詳しく知りたい人は作者の書いた[こちらの](https://zenn.dev/shougo/articles/ddu-vim-beta)記事を参照してみてください。
- 補完プラグインの[Shougo/ddc.vim](https://github.com/Shougo/ddc.vim)

等です。
設定なしではなにも動作しないというハードモードプラグインですが、その分柔軟な設定ができる良いプラグインなので気になった方はぜひ使ってみてください。

他人の設定ファイルを見ることで色々な発見があるのでとってもおすすめです。
`dotfiles`というレポジトリで管理していることが多いので、検索してみるといいでしょう。
ちなみに私のNeovimの設定ファイルはこちらです。
https://github.com/staticWagomU/dotvim

## おわりに

Vimは楽しいぞ！！！！やろう！！！！！！！


## おしらせ

2024年11月23日にはVimのカンファレンスである`VimConf 2024`が開催されます。
チケットは[こちら](https://vimconf-2024-ticket.peatix.com/)で販売しています。

まだチケットは残っているそうなので、購入するなら今のうちです！！！！！

---

毎週月曜日（第５週目を除く）には`エンジニアの楽園 vim-jpラジオ`というラジオが配信されています。有名エンジニアもゲストに呼んでいますので、気になった方はぜひ聞いてみてください！
https://audee.jp/program/show/300008578

ラジオのお便りはvim-jp Slackにて受け付けています。
https://vim-jp.org/docs/chat.html

SUZURIでグッズ販売もしています。
https://suzuri.jp/vim-jp-radio
