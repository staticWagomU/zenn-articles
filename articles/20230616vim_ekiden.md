---
title: "vim.defer_fn でコマンドを直列実行する"
emoji: "🦒"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["neovim", "lua"]
published: true
published_at: 2023-06-16 00:00
publication_name: "vim_jp"
---

:::message
この記事は Vim 駅伝 2023/06/16(月) の記事です。

前回は kawarimidoll さんの [Vim scriptで1回しか使えないコマンド・関数を定義する](https://zenn.dev/kawarimidoll/articles/22856ed2627056) です。 次回は 2023/06/19(月) に公開される予定です。
:::

## はじめに

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
vim-jpでそのことを呟くと、`vim.defer_fn()`で実現できますよと[ryoppippiさん](https://github.com/ryoppippi)に教えていただきました。ありがとうございます！

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

※ 2025年5月27日追記
```help
open({dir}, {opts}, {cb})                                               *oil.open*
    Open oil browser for a directory

    Parameters:
      {dir}  `nil|string` When nil, open the parent of the current buffer, or
             the cwd if current buffer is not a file
      {opts} `nil|oil.OpenOpts`
          {preview} `nil|oil.OpenPreviewOpts` When present, open the preview
                    window after opening oil
              {vertical}   `nil|boolean` Open the buffer in a vertical split
              {horizontal} `nil|boolean` Open the buffer in a horizontal split
              {split}      `nil|"aboveleft"|"belowright"|"topleft"|"botright"` S
                           plit modifier
      {cb}   `nil|fun()` Called after the oil buffer is ready
```

現在のoil.nvimではopen関数の第3引数にcallback関数を渡すことができるため、下記のように書きかえることでdefer_fnを使わずとも同様の動作をさせることができるようになっています。

```diff lua
 vim.keymap.set("n", "<leader>e",
 function ()
-  vim.cmd"Oil ."
-  vim.defer_fn(function()
-    vim.cmd.FuzzyMotion()
-  end, 150)
+  require("oil").open(".", nil, vim.cmd.FuzzyMotion)
 end,
 { noremap=true, silent=true })
```

※ 2025年5月27日追記 おわり

## おわりに

`vim.schedule()`で直列実行がうまくいかない場合は`vim.defer_fn()`を使ってみてね。
