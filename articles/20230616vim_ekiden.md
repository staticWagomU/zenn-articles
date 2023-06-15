---
title: "vim.defer_fn でコマンドを直列実行する"
emoji: "💬"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["neovim", "lua"]
published: true
published_at: 2023-06-16 09:00
---

:::message
この記事は Vim 駅伝 2023/06/16(月) の記事です。

前回は kawarimidoll さんの [Vim scriptで1回しか使えないコマンド・関数を定義する](https://zenn.dev/kawarimidoll/articles/22856ed2627056) です。 次回は 2023/06/19(月) に公開される予定です。
:::

## 動機

2023年6月現在、私はファイラーに[oil.nvim](https://github.com/stevearc/oil.nvim)を、バッファ内のあいまい検索に[fuzzy-motion.vim](https://github.com/yuki-yano/fuzzy-motion.vim)を使っています。
そこで、ファイラーを開いた直後にあいまい検索ができたら便利じゃないのか？という疑問を抱き、このようなキーマップを定義しました。
```lua
vim.keymap.set("n", "<leader>e",
function ()
  vim.cmd"Oil ."
  vim.schedule(function()
    vim.cmd.FuzzyMotion()
  end)
end,
{ noremap=true, silent=true })

```
![](/images/20230616_vim_ekiden/media1.gif)
しかし、結果はファイラーが起動する前にあいまい検索が起動してしまい使いものになりませんでした。
vim-jpでそのことを呟くと、`vim.defer_fn()`で実現できますよと[ryopippiさん](https://github.com/ryoppippi)に教えていただきました。ありがとうございます！

ドキュメントを確認したところ、第2引数へ時間を設定できるため起動タイミングをずらすことができるということでした。
そのため、`vim.defer_fn()`を使って書き換えたところ、


```lua
vim.keymap.set("n", "<leader>e",
function ()
  vim.cmd"Oil ."
  vim.defer_fn(function()
    vim.cmd.FuzzyMotion()
  end, 150)
end,
{ noremap=true, silent=true })
```

![](/images/20230616_vim_ekiden/media2.gif)
思ったとおりに動作しました。

## 結論

`vim.schedule()`で直列実行がうまくいかない場合は`vim.defer_fn()`を使ってみてね。
