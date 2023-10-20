---
title: "一歩ずつ始めるddu.vim"
emoji: "📑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["neovim"]
published: true
---

## はじめに
ddu.vim触ってみたいけど分からないよーって人に向けに書いています。
タイトルに「一歩ずつ」とあるように、ミニマルな状態から少しずつコードを足していくという方針で、最終的には自分でヘルプを読みながら設定できるようになるための補助輪としてこの記事を活用してください。
自分はNeovim+luaを使っているので、この記事に出てくるコードはすべてluaを使います。Vimscriptを使っている人は適宜読み替えてください。

### 最終的な見た目
![](/images/20231020step-by-step-ddu/media1.gif)

### 環境
```shell
$ nvim --version
NVIM v0.10.0-dev-09a17f9
Build type: RelWithDebInfo
LuaJIT 2.1.1695653777
Run "nvim -V1 -v" for more info
```

| | |
|-|-|
|OS| Windows 11 pro|
|Plugin Manager| Lazy.nvim|

## ddu.vimとは
ddu.vimは`vim.ui.select()`を高度にしたものです。
表示されている項目群から選択した項目について何らかのアクションを行うものをプラグイン化したものと思っていただけば大体大丈夫です。
作者がShougoさんということもあり、プラグインを導入しただけでは何も動きません。そうです、一から設定する必要があります。Telescope等の入れるだけで動くプラグインに慣れている人にとっては少しハードルが高く感じてしまうかもしれませんが、その分愛着が湧きます。

ddu.vimにはこれらの要素が存在します。
- Source
- Kind
- UI
- Filter
### Source
リストを生成する。
例）
- ファイル
- カラースキーム
- 開いてるバッファ
- `$ git status`の結果
等...
### Kind
Sourceに対して実行するアクションを定義する。
例)
- ファイル操作（開く、削除、作成...)
- カラースキームの適用
- バッファを開く
- `$ git add`等の外部コマンド実行
等...

### UI
Sourceを表示する。

### Filter
ユーザーからの入力を受け取り、UIを変化させる。
例）
- 絞り込み
- 並び順を変更
- ファイルアイコンの表示
等...

一口にFilterといっても実際はmatcher、sorter、converterというものが存在します。上記の例で取り上げると
- matcherは「絞り込み」を行うためのFilter
- sorterは「並び順を変更」するためのFIlter
- converterは「ファイルアイコンの表示」等、UIを変換するためのFilter
となっています。

## STEP1.まずは動かしてみる
### 1-1. 何も意味のないコード
```lua:diff
{
  "Shougo/ddu.vim",
  dependencies = {
    "vim-denops/denops.vim",
    ------------------------------
    -- | filter                   |
    ------------------------------
    "Shougo/ddu-filter-matcher_substring",
    ------------------------------
    -- | source                   |
    ------------------------------
    "Shougo/ddu-source-file_rec", -- file_recursiveの略です。
    ------------------------------
    -- | ui                       |
    ------------------------------
    "Shougo/ddu-ui-ff", -- fuzzy_finder(あいまい検索)
  },
  config = function()
    vim.fn["ddu#custom#patch_local"]("file_recursive", {
        ui = "ff",
        sources = {
          {
            name = { "file_rec" },
          },
        },
      })
  end,
}
```
上記のコードを貼り付けて`:call ddu#start(#{name:"file_recursive"})`でdduを実行してください。
![](/images/20231020step-by-step-ddu/1.png)
そうするとバッファが分割されてオレンジで囲んだ部分にファイルパスがずらっと表示されます。
先ほどの説明を読んで何が起きているのかピンと来ている人がいるかもしれません。このコードはUIに**file_rec**の内容を表示させており、この設定に**file_recursive**という名前を定義しています。しかし、Sourceを表示させただけなので何もすることができません。
次に、このUIを操作できるようにしましょう。

### 1-2. UIにアクションを設定する
ddu.vimではUIに対してアクションを設定をしないと何も動きません。
![](/images/20231020step-by-step-ddu/1.png)
このオレンジで囲まれたバッファのファイルタイプは**ddu-ff**です。
:::message
ファイルタイプは`:set ft?`で確認できます。
:::
ですから、ファイルタイプ**ddu-ff**に対して、アクションを設定していけばいいのです。
アクションを設定するには、`ddu#ui#ff#do_action()`を使います。
その他**ddu-ui-ff**で使える関数は`:h ddu-ui-ff-functions`を参照してください。

`q`に**ddu-ff**から脱出するキーマップと、`<CR>`に自分のいる行に対してアクションを実行するキーマップを設定します。
```diff
    vim.fn["ddu#custom#patch_local"]("file_recursive", {
      ui = "ff",
      sources = {
        {
          name = { "file_rec" },
        },
      },
    })
+
+   vim.api.nvim_create_autocmd("FileType",{
+       pattern = "ddu-ff",
+       callback = function()
+         local opts = { noremap = true, silent = true, buffer = true }
+         vim.keymap.set({ "n" }, "q", [[<Cmd>call ddu#ui#ff#do_action("quit")<CR>]], opts)
+         vim.keymap.set({ "n" }, "<Cr>", [[<Cmd>call ddu#ui#ff#do_action("itemAction")<CR>]], opts)
+       end,
+     }
+   )

```
その他のアクションは`:h ddu-ui-ff-actions`を参照してください。

`<CR>`にアクションを実行するキーマップを割り当てましたが、肝心のアクションがSourceに対して設定されていないので、Kindを追加しましょう。
```diff
    vim.fn["ddu#custom#patch_local"]("file_recursive", {
      ui = "ff",
      sources = {
        {
          name = { "file_rec" },
        },
      },
+     kindOptions = {
+       file = {
+         defaultAction = "open",
+       }
+     }
    })

...
```
その他のKindの設定については`:h ddu-kind-file`を参照してください。

これで、デフォルトのItemAcrionであるopenを使ってファイルが開けるようになります。

### 1-3. Filterを使う
ファイル一覧からファイルを開くことはできましたが、ファイルが大量にあると開きたいファイルに直ぐにアクセスできず不便です。
Filterの設定と、`i`でFilterWindowを開くマッピングを追加します。
```diff
    vim.fn["ddu#custom#patch_local"]("file_recursive", {
      ui = "ff",
      sources = {
        {
          name = { "file_rec" },
+         options = {
+           matchers = {
+             "matcher_substring",
+           },
+         },
        },
      },
      kindOptions = {
        file = {
          defaultAction = "open",
        }
      }
    })


   vim.api.nvim_create_autocmd("FileType",{
       pattern = "ddu-ff",
       callback = function()
         local opts = { noremap = true, silent = true, buffer = true }
         vim.keymap.set({ "n" }, "q", [[<Cmd>call ddu#ui#ff#do_action("quit")<CR>]], opts)
         vim.keymap.set({ "n" }, "<Cr>", [[<Cmd>call ddu#ui#ff#do_action("itemAction")<CR>]], opts)
+        vim.keymap.set({ "n" }, "i", [[<Cmd>call ddu#ui#ff#do_action("openFilterWindow")<CR>]], opts)
       end,
     })

...
```
今回filterには**matcher_substring**を使います。名前の通り、ユーザーの入力に部分一致するSourceへ絞り込むためのmatcherです。

Step1の設定をすべて追加すると、このようになっています。

![](/images/20231020step-by-step-ddu/media2.gif)
## Step2 カスタマイズ
Step1では紹介しなかった機能を使ってさらにカスタマイズしていきましょう。
### 2-1. Sourceをカスタマイズ
Step1の最終状態ではnode_modulesや.git以下のファイルも表示されます。これらは実際の開発時に使用するには少し不便です。また、なによりファイル数が増えると実行に時間がかかってしまいます。ですから**file_rec**の設定を変更して、特定のディレクトリを除外して表示されないようにしましょう。
```diff
...
      sources = {
        {
          name = { "file_rec" },
          options = {
            matchers = {
              "matcher_substring",
            },
          },
+          params = {
+            ignoredDirectories = { "node_modules", ".git", ".vscode" },
+          },
        },
      },
...
```
これで無駄なファイルが表示されなくなりました。
その他の設定を追加したい場合`:h ddu-source-file_rec-params`を参照してください。

### 2-3. Filterをカスタマイズ
FilterからUIに戻るときに`<Esc>`押して、`:close<CR>`とするのは大変なのでFilterに対して`<CR>`でUIに戻るキーマッピングを設定してみましょう。
```diff
 ...
 

   vim.api.nvim_create_autocmd("FileType",{
       pattern = "ddu-ff",
       callback = function()
         local opts = { noremap = true, silent = true, buffer = true }
         vim.keymap.set({ "n" }, "q", [[<Cmd>call ddu#ui#ff#do_action("quit")<CR>]], opts)
         vim.keymap.set({ "n" }, "<Cr>", [[<Cmd>call ddu#ui#ff#do_action("itemAction")<CR>]], opts)
          vim.keymap.set({ "n" }, "i", [[<Cmd>call ddu#ui#ff#do_action("openFilterWindow")<CR>]], opts)
       end,
     })
+
+   vim.api.nvim_create_autocmd("FileType",{
+       pattern = "ddu-ff-filter",
+       callback = function()
+         local opts = { noremap = true, silent = true, buffer = true }
+         vim.keymap.set({ "n", "i" }, "<CR>", [[<Esc><Cmd>close<CR>]], opts)
+       end,
+     })

```

ファイルパスの文字列がただ表示されるだけでは味気ないのでファイルアイコンを表示できるようにします。プラグインのdependenciesに`uga-rosa/ddu-filter-converter_devicon`を追加してください。
そして、下記のコードを追加してください。ついでに、大文字小文字関係なく検索できるようにもしましょう。
```diff
        sources = {
          {
            name = { "file_rec" },
            options = {
              matchers = {
                "matcher_substring",
              },
+             converters = {
+               "converter_devicon",
+             },
+             ignoreCase = true,
            },
            params = {
              ignoredDirectories = { "node_modules", ".git", ".vscode" },
            },
          },
        },
        kindOptions = {
          file = {
            defaultAction = "open",
          }
        }
      })

```
### 2-3. UIをカスタマイズ
見た目を変更してみす。
それにあわせて`P`にプレビューの表示を切り替えるキーマッピングを追加します。
```diff
    vim.fn["ddu#custom#patch_local"]("file_recursive", {
      ui = "ff",
```diff
    vim.fn["ddu#custom#patch_local"]("file_recursive", {
        ui = "ff",
+       uiParams = {
+         ff = {
+           filterFloatingPosition = "bottom",
+           filterSplitDirection = "floating",
+           floatingBorder = "rounded",
+           previewFloating = true,
+           previewFloatingBorder = "rounded",
+           previewFloatingTitle = "Preview",
+           previewSplit = "horizontal",
+           prompt = "> ",
+           split = "floating",
+           startFilter = true,
+         }
+       },
        sources = {
        {
...

    vim.api.nvim_create_autocmd("FileType",{
        pattern = "ddu-ff",
        callback = function()
          local opts = { noremap = true, silent = true, buffer = true }
          vim.keymap.set({ "n" }, "q", [[<Cmd>call ddu#ui#ff#do_action("quit")<CR>]], opts)
          vim.keymap.set({ "n" }, "<Cr>", [[<Cmd>call ddu#ui#ff#do_action("itemAction")<CR>]], opts)
          vim.keymap.set({ "n" }, "i", [[<Cmd>call ddu#ui#ff#do_action("openFilterWindow")<CR>]], opts)
+          vim.keymap.set({ "n" }, "P", [[<Cmd>call ddu#ui#ff#do_action("togglePreview")<CR>]], opts)
        end,
      }
    )

...
```
UIをfloatingにして、ボーダーをつけたりとおしゃれな見た目にしています。

### 2-4. 別のSourceも追加してみよう
せっかくなので、別のSourceも追加してみましょう。この記事では`4513ECHO/ddu-source-colorscheme`を使おうと思います。プラグインのdependenciesに`4513ECHO/ddu-source-colorscheme`を追加して、下記のコードを追加します。
```diff
...

+   vim.fn["ddu#custom#patch_local"]("colorscheme", {
+       ui = "ff",
+       sources = {
+         {
+           name = { "colorscheme" },
+           options = {
+             matchers = {
+               "matcher_substring",
+             },
+             ignoreCase = true,
+           },
+         },
+       },
+       kindOptions = {
+         colorscheme = {
+           defaultAction = "set",
+         }
+       }
+     })

...
```

Step2の設定をすべて追加すると、最終的にこのようになります。

![](/images/20231020step-by-step-ddu/media3.gif)
## Step3 リファクタリング
### 3-1. patch_globalを使ってまとめる - その１
Step2で新しく追加したSourceとStep1から作り上げたSourceではUIが違っていました。もし同じ見た目にしたい場合はStep1のコードをコピペすれば解決します。しかし、さらにいくつもSourceを追加した場合に何度もコピペするのは大変です。スマートじゃないので、共通した設定をまとめましょう。
そのために`ddu#custom#patch_local`関数ではなく、`ddu#custom#patch_global`関数を使って共通部分を置き換えていきます。
```diff
+   vim.fn["ddu#custom#patch_global"]({
+     ui = "ff",
+     uiParams = {
+       ff = {
+         filterFloatingPosition = "bottom",
+         filterSplitDirection = "floating",
+         floatingBorder = "rounded",
+         previewFloating = true,
+         previewFloatingBorder = "rounded",
+         previewFloatingTitle = "Preview",
+         previewSplit = "horizontal",
+         prompt = "> ",
+         split = "floating",
+         startFilter = true,
+       }
+     },
+   })

    vim.fn["ddu#custom#patch_local"]("file_recursive", {
-       ui = "ff",
-       uiParams = {
-         ff = {
-           filterFloatingPosition = "bottom",
-           filterSplitDirection = "floating",
-           floatingBorder = "rounded",
-           previewFloating = true,
-           previewFloatingBorder = "rounded",
-           previewFloatingTitle = "Preview",
-           previewSplit = "horizontal",
-           prompt = "> ",
-           split = "floating",
-           startFilter = true,
-         }
-       },
        sources = {
          {
            name = { "file_rec" },
            options = {
              matchers = {
                "matcher_substring",
              },
              converters = {
                "converter_devicon",
              },
              ignoreCase = true,
            },
            params = {
              ignoredDirectories = { "node_modules", ".git", ".vscode" },
            },
          },
        },
        kindOptions = {
          file = {
            defaultAction = "open",
          }
        }
      })

    vim.fn["ddu#custom#patch_local"]("colorscheme", {
-       ui = "ff",
        sources = {
          {
            name = { "colorscheme" },
            options = {
              matchers = {
                "matcher_substring",
              },
              ignoreCase = true,
            },
          },
        },
        kindOptions = {
          colorscheme = {
            defaultAction = "set",
          }
        }
      })

...
```
これで**colorscheme**と**file_recursive**のUIが統一されます。
### 3-2. patch_globalを使ってまとめる - その2
まだまとめられるコードがあります。**file_recursive**と**colorscheme**のmatcherは**mathcer_substring**が使われており、`ignoreCase = true`が設定されています。これらの設定は新しくどのソースでも共通されていて構わない設定項目だと思います。ですから、これらもまとめましょう。
全体で共通化したい設定は`_`に書きます。
```diff
    vim.fn["ddu#custom#patch_global"]({
        ui = "ff",
        uiParams = {
          ff = {
            filterFloatingPosition = "bottom",
            filterSplitDirection = "floating",
            floatingBorder = "rounded",
            previewFloating = true,
            previewFloatingBorder = "rounded",
            previewFloatingTitle = "Preview",
            previewSplit = "horizontal",
            prompt = "> ",
            split = "floating",
            startFilter = true,
          }
        },
+       sourceOptions = {
+         _ = {
+           matchers = {
+             "matcher_substring",
+           },
+           ignoreCase = true,
+         },
+       },
      })

    vim.fn["ddu#custom#patch_local"]("file_recursive", {
        sources = {
          {
            name = { "file_rec" },
            options = {
-             matchers = {
-               "matcher_substring",
-             },
              converters = {
                "converter_devicon",
              },
-             ignoreCase = true,
            },
            params = {
              ignoredDirectories = { "node_modules", ".git", ".vscode" },
            },
          },
        },
        kindOptions = {
          file = {
            defaultAction = "open",
          }
        }
      })

    vim.fn["ddu#custom#patch_local"]("colorscheme", {
        sources = {
          {
            name = { "colorscheme" },
-           options = {
-             matchers = {
-               "matcher_substring",
-             },
-             ignoreCase = true,
-           },
          },
        },
        kindOptions = {
          colorscheme = {
            defaultAction = "set",
          }
        }
      })

...
```

## 最終的なソースコード
```lua
return {
  "Shougo/ddu.vim",
  dependencies = {
    "vim-denops/denops.vim",
    ------------------------------
    -- | filter                   |
    ------------------------------
    "Shougo/ddu-filter-matcher_substring",
    "uga-rosa/ddu-filter-converter_devicon",
    ------------------------------
    -- | kind                     |
    ------------------------------
    "Shougo/ddu-kind-file",
    ------------------------------
    -- | source                   |
    ------------------------------
    "4513ECHO/ddu-source-colorscheme",
    "Shougo/ddu-source-file_rec",
    ------------------------------
    -- | ui                       |
    ------------------------------
    "Shougo/ddu-ui-ff",
  },
  config = function()
    vim.fn["ddu#custom#patch_global"]({
        ui = "ff",
        uiParams = {
          ff = {
            filterFloatingPosition = "bottom",
            filterSplitDirection = "floating",
            floatingBorder = "rounded",
            previewFloating = true,
            previewFloatingBorder = "rounded",
            previewFloatingTitle = "Preview",
            previewSplit = "horizontal",
            prompt = "> ",
            split = "floating",
            startFilter = true,
          }
        },
        sourceOptions = {
          _ = {
            matchers = {
              "matcher_substring",
            },
            ignoreCase = true,
          },
        },
      })
    vim.fn["ddu#custom#patch_local"]("file_recursive", {
        sources = {
          {
            name = { "file_rec" },
            options = {
              converters = {
                "converter_devicon",
              },
            },
            params = {
              ignoredDirectories = { "node_modules", ".git", "dist", ".vscode" },
            },
          },
        },
        kindOptions = {
          file = {
            defaultAction = "open",
          }
        }
      })
    vim.fn["ddu#custom#patch_local"]("colorscheme", {
        sources = {
          {
            name = { "colorscheme" },
          },
        },
        kindOptions = {
          colorscheme = {
            defaultAction = "set",
          }
        }
      })

    vim.api.nvim_create_autocmd("FileType",{
        pattern = "ddu-ff",
        callback = function()
          local opts = { noremap = true, silent = true, buffer = true }
          vim.keymap.set({ "n" }, "q", [[<Cmd>call ddu#ui#ff#do_action("quit")<CR>]], opts)
          vim.keymap.set({ "n" }, "<Cr>", [[<Cmd>call ddu#ui#ff#do_action("itemAction")<CR>]], opts)
          vim.keymap.set({ "n" }, "i", [[<Cmd>call ddu#ui#ff#do_action("openFilterWindow")<CR>]], opts)
          vim.keymap.set({ "n" }, "P", [[<Cmd>call ddu#ui#ff#do_action("togglePreview")<CR>]], opts)
        end,
      }
      )

    vim.api.nvim_create_autocmd("FileType",{
        pattern = "ddu-ff-filter",
        callback = function()
          local opts = { noremap = true, silent = true, buffer = true }
          vim.keymap.set({ "n", "i" }, "<CR>", [[<Esc><Cmd>close<CR>]], opts)
        end,
      }
      )
  end,
}

```

ここから先は、いままでコマンドラインモードで打っていた関数呼び出しをキーマッピングに割りあてたり、他の人の設定をまねしたりと、好きに設定してください。
## おわりに
この記事はdduを始めてみたい人の手助けとなれば、と思い書き始めました。ただ、書いている中で思ったことは、「ドキュメントを読めばそこまで難しくない」ということです。結構丁寧にドキュメントが書かれているので、ぜひご自身でドキュメントをよんでみてください。英語で書かれてあるので難しいと感じる方もいるかもしれませんが、中学校レベルの英語の私でもなんとなく読めるのできっと皆さんも読めると思います。今ではdeepl翻訳等の素晴らしい翻訳サービスもあるのでそれらを活用して、気になる部分を少しずつ読むことをお勧めします。
この記事を読んでddu.vimを使いたいと思った方は、[ddu.vimの基本設定概観](https://zenn.dev/vim_jp/articles/c0d75d1f3c7f33)もおすすめなのでぜひ読んでみてください。

皆さんのこれからのddu.vim生活を応援しています！！！

`:q!`
