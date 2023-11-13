---
title: "Neovimのプラグインってどうやっていれるの？"
emoji: "✋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["neovim", "lua"]
published: true
publication_name: "vim_jp"
---
:::
この記事は[Vim駅伝](https://vim-jp.org/ekiden/)2023年11月13日(金)の記事です。

前回の記事は kawarimidoll さんの「[耳元にVimを](https://gist.github.com/kawarimidoll/32f50e51cffee826ae13d5e455a3c9b6)」という記事でした。

次回の記事は 11月15日(水) に投稿される予定です。
:::

---



## はじめに
NeovimはVSCodeみたいなUIにできるぞ！！という話を聞き、「俺もNeovim使いになるか...」と使い始めたところ、プラグインの導入方法がよく分からず諦めた。なんて経験はありませんか？  
この記事はそのような人向けに書いた記事です。この記事を読んだことで「Neovim面白そうやん」となってくだされば幸いです。

Neovimを始め方をざっくりと分類すると、
1. ディストリビューションを利用する
2. 1から設定をする
の２つがあると思います。

前者の「ディストリビューションを利用する」についですが、Neovimには、すでにしっかりとカスタマイズされた設定ファイルをディストリビューションとして配布しているものがあります。
具体的には、
- [SpaceVim](https://github.com/SpaceVim/SpaceVim)
- [NvChad](https://github.com/NvChad/NvChad)
- [LunarVim](https://github.com/LunarVim/LunarVim)
- [AstroNvim](https://github.com/AstroNvim/AstroNvim)
- [LazyVim](https://github.com/LazyVim/LazyVim)
ざっとこれらのものが挙げられます。
これらを使うと初めからVSCodeと似たようなリッチなUIが用意されており、直ぐに便利な状態から使い始められるというメリットがあります。しかし、やりたい操作がどのキーマッピングに振られているのか、自分の操作しているこの機能はどのプラグインが提供している機能なのか分からない等の問題があります。また、それに伴って自分の望む設定を加えることが非常に困難になってしまうという大きなデメリットを抱えています。Neovimやプラグインの設定を自分で書くことが簡単でないことは事実です。しかし、それこそがVimの醍醐味だと自分は感じており、初めからディストリビューションを使用するのはVimの楽しみの一つを奪うようなものだと思っています。

ですから、この記事では後者の「１から設定をする」を選択して進めていきます。
これにより、自分の意志でプラグインを追加し、自分の望む設定を付け加えることができます。

### この記事のゴール
- プラグインを導入できる
- プラグインの設定ができる
- プラグインを探せる

この３つのゴールを目指してやっていきましょう。

## はじめに結論
記事が長くなってしまったので、結論を最初に書いておきます。
### プラグインを導入
- githubにあるプラグインは`ユーザー名/レポジトリ名`をプラグイン追加するための関数の引数に渡す（関数はプラグインマネージャーによって異なる点には注意）
- lua製のプラグインは`.setup()`が必要

### プラグインの設定
- docを読む
- READMEを読む
- githubで検索する

### プラグインを探せる
- [yutkat/my-neovim-pluginlist](https://github.com/yutkat/my-neovim-pluginlist)を見る
- [This Week in Neovim | Neovim news](https://dotfyle.com/this-week-in-neovim)を見る
- vim-jpに入る

## この記事で導入するプラグインたちの紹介

VSCodeと似た機能を提供する以下のプラグインを選定しました。
王道プラグインを選んだので、設定方法が分からないときに他の人の設定を探しやすいと思います。
### プラグインマネージャー
[lazy.nvim](https://github.com/folke/lazy.nvim)
- lazy.nvim自体の導入が簡単
- プラグインの管理がしやすい
- リッチなUI
### ファイラー
[nvim-tree.lua](https://github.com/nvim-tree/nvim-tree.lua)
VSCodeに似たtree形式のファイラープラグイン

### シンタックスハイライト
[nvim-treesitter](https://github.com/nvim-treesitter/nvim-treesitter)
Neovimはシンタックスハイライト機能を持っています。しかし、正規表現によって実装されているため、複雑な構文のハイライトを行うのが苦手です。nvim-treesitterはtreesitterというパーサ生成ツールを使ってシンタックスハイライト等の機能を提供します。treesitterの拡張プラグインを導入することで、シンタックスハイライト以外の機能を使うこともできます。

### LSP・自動補完
[coc.nvim](https://github.com/neoclide/coc.nvim#example-lua-configuration)を使います。
coc.nvimの特徴は、LSPの機能を提供するだけでなく、自動補完機能を提供することです。coc.nvimを使わない場合は、LSPの機能を提供するプラグインと、自動補完機能を提供するプラグインを別々に導入する必要があります。しかし、coc.nvimを使うことで、LSPの機能と自動補完機能を一つのプラグインで導入することができます。

### ファイル検索
[telescope.nvim](https://github.com/nvim-telescope/telescope.nvim)
現時点(2023年11月)のNeovimトピック内では一番star数の多いプラグインとなります。主にはファイルのFuzzyFind（あいまい検索）をするためのプラグインで、VSCodeのクイックオープンやコマンドパレットと似た機能を提供しています。

レポジトリには
>Community driven builtin [pickers](https://github.com/nvim-telescope/telescope.nvim#pickers), [sorters](https://github.com/nvim-telescope/telescope.nvim#sorters) and [previewers](https://github.com/nvim-telescope/telescope.nvim#previewers).

と書かれてあるように、telescope.nvimユーザーが作った拡張プラグインが多数存在します。

### ステータスライン
[lualine.nvim](https://github.com/nvim-lualine/lualine.nvim)VSCodeの下側に色々表示されてるやつ↓のNeovim版
![[Pasted image 20231113153250.png]]

## プラグインを導入・設定していく

### 初期のファイル構成


### プラグインマネージャー(lazy.nvim)
プラグインマネージャーによってはこれを実行してね！とcurlコマンドが書かれています。中には何してるわからなくて怖いと思う人がいるかもしれません。実態としてはプラグインをNeovimがプラグインとして読み込める場所に配置しているだけなので怖がる必要はありません。

lazy.nvimはcurlコマンドではなく、git cloneコマンドを使って、ファイルを配置します。そのコードが、[ここ](https://github.com/folke/lazy.nvim#-installation)に書かれてあります。
```lua
vim.loader.enable()
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({
    "git",
    "clone",
    "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable", -- latest stable release
    lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)
```

このコードを`init.lua`に貼り付けてください。

lua製のプラグインの多くは入れるだけではそのプラグインを使うことができず、setupを自分で記述する必要があります。
 ```diff
 vim.loader.enable()
 local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
 if not vim.loop.fs_stat(lazypath) then
   vim.fn.system({
     "git",
     "clone",
     "--filter=blob:none",
     "https://github.com/folke/lazy.nvim.git",
     "--branch=stable", -- latest stable release
     lazypath,
   })
 end
 vim.opt.rtp:prepend(lazypath)

+require("lazy").setup("plugins")
```
lazyのsetup関数に対して"plugins"という引数を渡しています。これは、
[folke/lazy.nvim　#📂 Structuring Your Plugins](https://github.com/folke/lazy.nvim#-structuring-your-plugins)に説明があります。
> Any lua file in `~/.config/nvim/lua/plugins/*.lua` will be automatically merged in the main plugin spec


deepl先生訳
> `~/.config/nvim/lua/plugins/*.lua` にある lua ファイルは、メインのプラグイン仕様に自動的にマージされます。

`~/.config/nvim/lua/plugins.lua`や`~/.config/nvim/lua/plugins/init.lua`を配置する方法もありますが、今回は`plugins/*`にプラグイン事にluaファイルを配置する方法を取りたいと思います。
そうする理由としては、
- プラグイン名 == ファイル名にすれば探すのが楽
- ファイルを消すことでプラグインを消すことができる
という点があげられます。
大量のプラグインとその設定が一つのファイルに記述されていると非常に見通しが悪くなってしまいます。ですから、分割して配置しようということです。

git cloneしてきたlazy.nvimを、この行でruntimepathに追加しています。これでNeovimがlazy.nvimを認識してくれるようになります。
```lua
 vim.opt.rtp:prepend(lazypath)
```

それでは、lazy.nvimにlazy.nvim自体を管理してもらいましょう。そうすることで、自分自身ののバージョンアップに追従することができます。

プラグインの記述法はgithubに置かれているプラグインの場合は、`ユーザー名/レポジトリ名`を渡してあげます。まれにgithub以外のホスティングサービスで提供されている場合がありますが、そのようなときはフルパスを指定してください。
```lua
-- lua/plugins/lazy.lua
return {
  "folke/lazy.nvim"
}
```
lazy.nvimのsetup関数は`init.lua`に記述しているため、これでプラグインマネージャーの導入は完了です。
 
Neovimを再起動して`:Lazy`を実行したときに、下記のような画面が表示されればlazy.nvimの導入は成功です。
![[Pasted image 20231113164009.png]]
### 設定を変更してみる
`:h lazy.nvim-lazy.nvim-installation`を見ると設定は、`setup()`の第二引数のoptsにデフォルトから変更したい設定を渡すようです。
```lua
require("lazy").setup(plugins, opts)
```

`:h lazy.nvim-lazy.nvim-configuration`を参考に自分はこのように設定を変更してみました。

```diff
-require("lazy").setup("plugins")
+require("lazy").setup("plugins",
+{
+	ui = {
+		icons = {
+			cmd = "⌘",
+			config = "🛠",
+			event = "📅",
+			ft = "📂",
+			init = "⚙",
+			keys = "🗝",
+			plugin = "🔌",
+			runtime = "💻",
+			require = "🌙",
+			source = "📄",
+			start = "🚀",
+			task = "📌",
+			lazy = "💤 ",
+		},
+	},
+	checker = {
+		enabled = true, -- プラグインのアップデートを自動的にチェック
+	},
+	diff = {
+		cmd = "delta",
+	},
+	rtp = {
+		disabled_plugins = {
+			"gzip",
+			"matchit",
+			"matchparen",
+			"netrwPlugin",
+			"tarPlugin",
+			"tohtml",
+			"tutor",
+			"zipPlugin",
+		},
+	},
+})
```

### ファイラー(nvim-tree)
```lua
-- lua/plugins/nvim-tree.lua
return {
  "nvim-tree/nvim-tree.lua"
}
```
ここまで読めば、これでnvim-tree.luaが入ることが分かるかと思います。
では、setupはどこに書けばいいのでしょうか？

`:h lazy.nvim-lazy.nvim-plugin-spec`を見てみましょう
>  config         
>  fun(LazyPlugin, opts:table) or true                             
> config is executed when the plugin loads. The
>                                                                                 default implementation will automatically run
>                                                                                 require(MAIN).setup(opts). Lazy uses several
>                                                                                 heuristics to determine the plugin’s MAIN module
>                                                                                 automatically based on the plugin’s name. See also
>                                                                                 opts. To use the default implementation without opts
>                                                                                 set config to true.

意訳ですが、configを渡したいならopsに渡してあげてください。何も渡さなくていいならconfigをtrueにしてください。ということです。
では、nvim-tree.luaのconfigをtrueにしてみましょう。
```diff
 -- lua/plugins/nvim-tree.lua
 return {
   "nvim-tree/nvim-tree.lua"
+  config = true,
}
```

ファイルを保存したときに、右下になにか通知がでると思います。これはlazy.nvimがファイルの変更を検知してくれているからです。
通知が出た後で、`:Lazy install`を実行してください。すると、nvim-tree.luaがインストールされます。
![Alt text](/images/20231113_vim_ekiden/image2.png)

Neovimを再起動して、`:NvimTreeToggle`を実行してみましょう。すると、左側にファイラーが表示されると思います。
![Alt text](/images/20231113_vim_ekiden/image.png)

せっかくなので、lazy.nvimの機能を使って`:NvimTreeToggle`をキーマッピングしてみましょう。
```diff
 -- lua/plugins/nvim-tree.lua
 return {
   "nvim-tree/nvim-tree.lua"
   config = true,
   keys = {
+    {mode = "n", "<C-n>", "<cmd>NvimTreeToggle<CR>", desc = "NvimTreeをトグルする"},
   }
 }
```
Neovimを再起動して、`<C-n>`を押してみましょう。すると、先ほどと同じように左側にファイラーが表示されると思います。

### シンタックスハイライト
```lua
-- lua/plugins/nvim-treesitter.lua
return {
    "nvim-treesitter/nvim-treesitter",
    opts = {},
}
```
次にoptsを使ってみましょう。
`:h nvim-treesitter-quickstart`を確認してみると、
```lua
  require'nvim-treesitter.configs'.setup {
    -- A directory to install the parsers into.
    -- If this is excluded or nil parsers are installed
    -- to either the package dir, or the "site" dir.
    -- If a custom path is used (not nil) it must be added to the runtimepath.
    parser_install_dir = "/some/path/to/store/parsers",

    -- A list of parser names, or "all"
    ensure_installed = { "c", "lua", "rust" },

    -- Install parsers synchronously (only applied to `ensure_installed`)
    sync_install = false,
...
```
このような記述があると思います。
今回はensure_installedとsync_installを設定してみます。

```diff
 -- lua/plugins/nvim-treesitter.lua
 return {
     "nvim-treesitter/nvim-treesitter",
-    opts = {},
+    opts = {
+        ensure_installed = { "lua", "typescript" },
+        sync_install = true,
+    }, 
 }
```
この調子でじゃんじゃん行きましょう。

### LSP・自動補完(coc.nvim)

**申し訳ございません。時間と集中力が尽きてしまい最後まで書ききれませんでした。今週中には完成版を更新します。**

### ファイル検索(telescope)

### ステータスライン(lualine)


## プラグインの探し方
- [GitHub - yutkat/my-neovim-pluginlist: My personal list of Neovim plugins](https://github.com/yutkat/my-neovim-pluginlist)
- vim-jpに参加する
- [This Week in Neovim | Neovim news](https://dotfyle.com/this-week-in-neovim)
- Reddit


## 設定の仕方が分からない時
1. doc/READMEを読む
2. githubで検索する
3. vim-jpで聞いてみる

## おわりに
後半にかけて雑になっている感は否めないので、後日修正します。
