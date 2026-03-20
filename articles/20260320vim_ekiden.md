---
title: "AIエージェントのお供にNeovimはいかが？ ― コードビューアーとしてのNeovim
emoji: "🦒"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["neovim", "vim"]
published: true
published_at: 2026-03-20
publication_name: "vim_jp"

---

:::message
この記事は[Vim駅伝](https://vim-jp.org/ekiden/)2026年3月20日(金)の記事です。

前回の記事は [Sina-TehraniFard](https://github.com/Sina-TehraniFard)さんの「[本番環境が完全停止！緊急対応から再発防止策まで](https://scriptlab.jp/production-502-disk-full-incident-response/)」という記事でした。

次回の記事は 3月23日(月) に投稿される予定です。
:::

---

## はじめに

早起きしすぎて生活リズムがおかしくなっている輪ごむです。

最近はGhosttyなどのターミナルでClaude CodeやCodex CLI、OpenCodeといったAIエージェントを走らせている方も増えてきたのではないでしょうか。

ターミナルで作業していると

- AIにコード生成・編集してもらっている最中に「どのファイルが変わったんだ？」「差分を確認したいんだけど」と思う瞬間
- ターミナルとエディタを行き来するのが面倒じゃね？

といった経験があるかもしれません。

これらのAIエージェントツールでは、プロンプト入力中に`Ctrl+G`を押すと`$EDITOR`に設定したエディタが開きます。そこでプロンプトを書くだけでなく、AIが編集中のコードを横目で確認できたら便利ですよね。

この記事では、テキストエディタではなく、コードビューアーとしてのNeovimを最小構成で構築します。VSCodeに近い操作感になるように設定するので、Vim怖い勢の方も安心してください。


:::message
この記事のゴール

Neovimでファイルを開いて、ハイライト付きで読めて、ファイルを切り替えられて、gitの差分が見られる状態になること。
:::


## セットアップ

### $EDITORをnvimに設定する

シェルの設定ファイルに以下を追記してください。

```bash
# ~/.zshrc や ~/.bashrc の場合
export EDITOR=nvim
```

```fish
# ~/.config/fish/config.fish の場合
set -gx EDITOR nvim
```

反映するには `source ~/.zshrc` するか、ターミナルを再起動してください。Claude Codeなどを再起動すると、`Ctrl+G`でNeovimが開くようになります。

分からなかったらAI エージェントに聞いてください

### Neovim nightlyをインストール

この設定ではNeovim nightly（>= 0.12.0）が必要です。Neovim 0.12で追加された`vim.pack`という組み込みパッケージマネージャーを使っているためです。

brew経由（macOS）：

```bash
brew install neovim --HEAD
```

mise経由：

```bash
mise use -g neovim@nightly
```

winget経由（Windows）：

```bash
winget install --id Neovim.Neovim.Nightly
```

分からなかったら、AI エージェントに「Neovimのnightlyを入れたい」と聞いてください


### Denoをインストール

この設定では、私の好みでvim-fallやvim-ginなどのdenoを依存関係として必要とするプラグインを導入します。

brew経由（macOS）：
```bash
brew install deno
```

mise経由：
```bash
mise use -g deno@latest
```

winget経由（Windows）：

```
winget install --id DenoLand.Deno
```



詳しくは[こちら](https://docs.deno.com/runtime/getting_started/installation/)を参照してください

https://docs.deno.com/runtime/getting_started/installation/


分からなかったら、(略

::: message
代替プラグインを使う場合はDenoなしでもOKなので、スキップしても大丈夫です。
:::

## 覚えてほしい最低限のキー操作

キーマッピングを追加して、VSCodeに似た操作になるようにしています



| 操作 | キー | 説明 |
|------|------|------|
| Neovimを閉じる | `Ctrl+Q` | 保存せず強制終了。もう閉じられなくて困らない！ |
| 保存 | `Ctrl+S` | おなじみの保存 |
| 入力モードに入る | `i` | テキストが打てるようになる |
| 入力モードを抜ける | `Esc` | ノーマルモードに戻る |
| ファイルを閉じる | `Ctrl+W` | 開いているタブを閉じる |
| ファイルを開く | `Ctrl+P` | ファジーファインダーでファイルを検索 |
| サイドバー表示 | `Ctrl+Shift+E` | ファイルエクスプローラーを開く |
| サイドバートグル | `Ctrl+B` | ファイルエクスプローラーの表示/非表示 |

VSCodeとほとんど同じキーバインドにしているので、乗り換えコストはかなり低いはずです。

:::message
上記のキーバインドのいくつかはこの設定ファイルで追加したカスタムキーバインドです。素のNeovimには存在しないので、次のセクションの設定コードを適用してから使ってください。
:::

## 初期設定コード

以下のコードを `~/.config/nvim/init.lua` にVSCodeなどを使って貼り付けてください。初回起動時にプラグインが自動でインストールされます。


設定の中身をざっくり説明すると：

- `Ctrl+Q`で閉じる、`Ctrl+S`で保存、`Ctrl+W`でタブを閉じる等、VSCode風のキーバインド
- OSのコピペと統合されるので、Neovimでコピーした内容がそのまま他のアプリにペーストできる
- Neovim 0.12の組み込みパッケージマネージャーでプラグインをインストール
- コードに色がつく（シンタックスハイライト）
- コメントアウトした追加キーマッピング例も含めてあるので、自分好みにカスタマイズできます

こんな画面になるので、「Y」あるいは「A」を押してください

![image](https://obsidian-image.wagomu.me/f4eaf40cf13c242f2fc8e3f7a88d1118.png)


それらが終わるとこんな感じの表示がされるので、無心でエンターキーを連打してください

![image](https://obsidian-image.wagomu.me/6aba21649f0e4901845a96f5f7046634.png)

こうなるとOKです

![image](https://obsidian-image.wagomu.me/51d773a1c7980f2bd68bc12e0385beb5.png)



```lua
-- ---------------------------------------------------------------------------
-- 基本設定
-- ---------------------------------------------------------------------------

-- クリップボードをOSと共有（コピペが自然にできる）
vim.opt.clipboard = "unnamedplus"

-- 行番号を表示
vim.opt.number = true

-- 24bitカラーを有効化（bufferline等で必要）
vim.opt.termguicolors = true

-- マウス操作を有効化（タブのクリック切り替え等）
vim.opt.mouse = "a"

-- 検索時に大文字小文字を区別しない（大文字が含まれる場合は区別する）
vim.opt.ignorecase = true
vim.opt.smartcase = true

-- インデント設定
vim.opt.expandtab = true
vim.opt.shiftwidth = 2
vim.opt.tabstop = 2

-- サインカラム（gitsignsの表示用）を常に表示
vim.opt.signcolumn = "yes"

-- ---------------------------------------------------------------------------
-- キーマッピング（VSCode風）
-- ---------------------------------------------------------------------------
-- Ctrl+Q: Neovimを確実に閉じる（保存しない）
-- もう「Vimを閉じられない」で困ることはありません！
vim.keymap.set("n", "<C-q>", "<Cmd>qa!<CR>", { desc = "Neovimを閉じる（強制）" })
vim.keymap.set("i", "<C-q>", function()
  -- 入力モード中にCtrl+Qを押した場合は注意メッセージを表示
  vim.api.nvim_echo({
    { "Escキーを押してからCtrl+Qしてください", "WarningMsg" },
  }, true, {})
end, { desc = "閉じ方のヒントを表示" })

-- Ctrl+S: 保存
vim.keymap.set("n", "<C-s>", "<Cmd>w<CR>", { desc = "保存" })

-- Ctrl+W: 現在のバッファ（タブ）を閉じる
-- VSCodeのCtrl+Wと同じ感覚で使えます
vim.keymap.set("n", "<C-w>", "<Cmd>bd<CR>", { desc = "現在のバッファを閉じる" })

-- ---------------------------------------------------------------------------
-- denops.vim（プラグインフレームワーク）
-- ---------------------------------------------------------------------------
-- vim-fall, vim-gin, nvim-aibo が依存しています。
vim.pack.add({
  -- === プラグインフレームワーク: denops.vim ===
  "https://github.com/vim-denops/denops.vim",
})

-- ---------------------------------------------------------------------------
-- Treesitter（コードハイライト）
-- ---------------------------------------------------------------------------
-- ファイルを開いたとき、コードに色がつくようになります。
vim.pack.add({
  -- === コードハイライト: treesitter ===
  { src = "https://github.com/nvim-treesitter/nvim-treesitter", version = "main" },
})

local ts_parsers = {
  "bash",
  "css",
  "dockerfile",
  "fish",
  "git_config",
  "git_rebase",
  "gitcommit",
  "gitignore",
  "go",
  "html",
  "javascript",
  "json",
  "lua",
  "make",
  "markdown",
  "markdown_inline",
  "python",
  "rust",
  "sql",
  "toml",
  "tsx",
  "typescript",
  "vim",
  "vimdoc",
  "yaml",
  "zig",
}

local ts = require "nvim-treesitter"
ts.install(ts_parsers)

local filetypes = {}
for _, lang in ipairs(ts.get_available(2)) do
  for _, filetype in ipairs(vim.treesitter.language.get_filetypes(lang)) do
    table.insert(filetypes, filetype)
  end
end

vim.api.nvim_create_autocmd("FileType", {
  pattern = filetypes,
  callback = function(args)
    -- size check
    local max_filesize = 512 * 1024 -- 512[KB]
    local ok, stats = pcall(vim.loop.fs_stat, vim.api.nvim_buf_get_name(args.buf))
    if ok and stats and stats.size > max_filesize then
      return
    end

    vim.treesitter.start()
    vim.bo.indentexpr = "v:lua.require'nvim-treesitter'.indentexpr()"
  end,
})

-- ---------------------------------------------------------------------------
-- gitsigns.nvim（Git差分表示）
-- ---------------------------------------------------------------------------
-- ファイルの変更箇所の左側にカラーマークが表示されます。
vim.pack.add({
  -- === Git差分表示: gitsigns.nvim ===
  "https://github.com/lewis6991/gitsigns.nvim",
})

require("gitsigns").setup({
  signs = {
    add          = { text = "┃" },
    change       = { text = "┃" },
    delete       = { text = "_" },
    topdelete    = { text = "‾" },
    changedelete = { text = "~" },
    untracked    = { text = "┆" },
  },
  -- 現在行のblame情報を表示したい場合はtrueに
  current_line_blame = false,
})

-- ---------------------------------------------------------------------------
-- bufferline.nvim（タブライン）
-- ---------------------------------------------------------------------------
-- 複数ファイルを開くと画面上部にファイル名がタブ表示されます。
-- クリックでも切り替えられます。
vim.pack.add({
  -- === タブライン: bufferline.nvim ===
  "https://github.com/akinsho/bufferline.nvim",
})

require("bufferline").setup({
  options = {
    -- 閉じるボタンをクリックでバッファを閉じられる
    close_command = "bdelete! %d",
    -- マウスの右クリックで閉じる
    right_mouse_command = "bdelete! %d",
    -- タブの見た目
    separator_style = "thin",
    -- 数字の表示
    numbers = "ordinal",
  },
})

-- === 代替: barbar.nvim（アニメーション付きのタブバー） ===
-- vim.pack.add({
--   "https://github.com/romgrk/barbar.nvim",
-- })

-- === 代替: mini.tabline（最小構成のタブライン） ===
-- vim.pack.add({
--   "https://github.com/echasnovski/mini.tabline",
-- })
-- require("mini.tabline").setup({})

-- ---------------------------------------------------------------------------
-- vim-fall（ファイルピッカー）
-- ---------------------------------------------------------------------------
-- VSCode の Ctrl+P と同じ感覚でファイルを開けます。
-- Ctrl+P → ファイル名を入力 → Enter で開く
vim.pack.add({
  -- === ファイルピッカー: vim-fall ===
  -- Deno (denops.vim) が必要です。
  "https://github.com/lambdalisue/vim-fall",
})

vim.keymap.set("n", "<C-p>", "<Cmd>Fall file<CR>", { desc = "ファイルピッカー (vim-fall)" })

-- === 代替: telescope.nvim（最も有名で情報が多い。困ったときに検索しやすい） ===
-- vim.pack.add({
--   "https://github.com/nvim-lua/plenary.nvim",
--   "https://github.com/nvim-telescope/telescope.nvim",
-- })
-- require("telescope").setup({})
-- local builtin = require("telescope.builtin")
-- vim.keymap.set("n", "<C-p>", builtin.find_files, { desc = "ファイルピッカー (telescope)" })

-- === 代替: mini.pick（最小構成で依存が少ない。Denoも不要） ===
-- vim.pack.add({
--   "https://github.com/echasnovski/mini.pick",
-- })
-- require("mini.pick").setup({})
-- vim.keymap.set("n", "<C-p>", "<Cmd>Pick files<CR>", { desc = "ファイルピッカー (mini.pick)" })

-- ---------------------------------------------------------------------------
-- vim-fern（ファイルエクスプローラー）
-- ---------------------------------------------------------------------------
-- VSCode の Ctrl+Shift+E / Ctrl+B と同じ感覚でサイドバーを操作できます。
vim.pack.add({
  -- === ファイルエクスプローラー: vim-fern ===
  "https://github.com/lambdalisue/vim-fern",
})

-- Ctrl+Shift+E: ファイルエクスプローラーを開く
vim.keymap.set("n", "<C-S-e>", "<Cmd>Fern . -reveal=% -drawer -width=30<CR>", {
  desc = "ファイルエクスプローラーを開く (vim-fern)",
})

-- Ctrl+B: ファイルエクスプローラーをトグル
vim.keymap.set("n", "<C-b>", "<Cmd>Fern . -reveal=% -drawer -width=30 -toggle<CR>", {
  desc = "ファイルエクスプローラーをトグル (vim-fern)",
})

-- === 代替: neo-tree.nvim（Lua製のモダンなファイルエクスプローラー） ===
-- vim.pack.add({
--   "https://github.com/nvim-lua/plenary.nvim",
--   "https://github.com/MunifTanjim/nui.nvim",
--   "https://github.com/nvim-neo-tree/neo-tree.nvim",
-- })
-- require("neo-tree").setup({})
-- vim.keymap.set("n", "<C-S-e>", "<Cmd>Neotree toggle<CR>", { desc = "ファイルエクスプローラー (neo-tree)" })
-- vim.keymap.set("n", "<C-b>", "<Cmd>Neotree toggle<CR>", { desc = "ファイルエクスプローラートグル (neo-tree)" })

-- === 代替: oil.nvim（バッファ自体がファイルエクスプローラーになるユニークなUI） ===
-- vim.pack.add({
--   "https://github.com/stevearc/oil.nvim",
-- })
-- require("oil").setup({})
-- vim.keymap.set("n", "<C-S-e>", "<Cmd>Oil<CR>", { desc = "ファイルエクスプローラー (oil.nvim)" })

-- ---------------------------------------------------------------------------
-- vim-gin（Git操作）
-- ---------------------------------------------------------------------------
-- ターミナルでの git コマンドと同じ感覚で使えます。
--
-- 使い方:
--   :Gin status           → git status
--   :Gin commit           → git commit
--   :Gin log              → git log
--   :GinStatus            → 対話的なステータス画面
--     ファイル上で << を押すとステージング
--     ファイル上で >> を押すとリセット
--   :GinLog               → コミット履歴
--   :GinDiff              → 差分表示
vim.pack.add({
  -- === Git操作: vim-gin ===
  -- Deno (denops.vim) が必要です。
  "https://github.com/lambdalisue/vim-gin",
})

-- === 代替: vim-fugitive（Tim Pope作の老舗Gitプラグイン。Deno不要） ===
-- vim.pack.add({
--   "https://github.com/tpope/vim-fugitive",
-- })
-- 使い方:
--   :Git status   → git status
--   :Git commit   → git commit
--   :Git log      → git log

-- === 代替: neogit（EmacsのMagitにインスパイアされたGitプラグイン） ===
-- vim.pack.add({
--   "https://github.com/nvim-lua/plenary.nvim",
--   "https://github.com/NeogitOrg/neogit",
-- })
-- require("neogit").setup({})

-- ---------------------------------------------------------------------------
-- nvim-aibo（AIエージェント操作）
-- ---------------------------------------------------------------------------
-- Neovim上でバッファにプロンプトを書いて、AIエージェントに送信できます。
-- ターミナルを分割してAIエージェントを横に並べて作業するイメージです。
vim.pack.add({
  -- === AIエージェント操作: nvim-aibo ===
  "https://github.com/lambdalisue/nvim-aibo",
})

require('aibo').setup()

-- ---------------------------------------------------------------------------
-- マークダウンプレビュー
-- ---------------------------------------------------------------------------
vim.pack.add({
  "https://github.com/MeanderingProgrammer/render-markdown.nvim",
})

-- ---------------------------------------------------------------------------
-- カラースキーム
-- ---------------------------------------------------------------------------
vim.opt.background="dark"
vim.cmd.colorscheme("catppuccin")
```

## プラグイン紹介

### 設計方針

今回のプラグイン構成では、[lambdalisue](https://github.com/lambdalisue)さんのプラグインをメインで採用しています。
便利で安定しているが、メジャーからは少し離れた選定です。

ありすえさんのプラグインは「入れるだけで使える」という思想で作られているので、初心者の方にもぴったりだと思います。

各プラグインには代替プラグインも紹介するので、好みに合わせて選んでみてください。

### コードハイライト：treesitter

:::details この部分
```lua
-- ---------------------------------------------------------------------------
-- Treesitter（コードハイライト）
-- ---------------------------------------------------------------------------
-- ファイルを開いたとき、コードに色がつくようになります。
vim.pack.add({
  -- === コードハイライト: treesitter ===
  { src = "https://github.com/nvim-treesitter/nvim-treesitter", version = "main" },
})

local ts_parsers = {
  "bash",
  "css",
  "dockerfile",
  "fish",
  "git_config",
  "git_rebase",
  "gitcommit",
  "gitignore",
  "go",
  "html",
  "javascript",
  "json",
  "lua",
  "make",
  "markdown",
  "markdown_inline",
  "python",
  "rust",
  "sql",
  "toml",
  "tsx",
  "typescript",
  "vim",
  "vimdoc",
  "yaml",
  "zig",
}

local ts = require "nvim-treesitter"
ts.install(ts_parsers)

local filetypes = {}
for _, lang in ipairs(ts.get_available(2)) do
  for _, filetype in ipairs(vim.treesitter.language.get_filetypes(lang)) do
    table.insert(filetypes, filetype)
  end
end

vim.api.nvim_create_autocmd("FileType", {
  pattern = filetypes,
  callback = function(args)
    -- size check
    local max_filesize = 512 * 1024 -- 512[KB]
    local ok, stats = pcall(vim.loop.fs_stat, vim.api.nvim_buf_get_name(args.buf))
    if ok and stats and stats.size > max_filesize then
      return
    end

    vim.treesitter.start()
    vim.bo.indentexpr = "v:lua.require'nvim-treesitter'.indentexpr()"
  end,
})

```
:::

Treesitterは、コードを構文解析して正確にハイライトしてくれる仕組みです。設定コードに含まれているので、初回起動時に自動で有効になります。


対応言語が非常に多く、正規表現ベースの従来のハイライトよりもずっと正確です。初期設定ではよく使われる言語のパーサーを指定していますが、`:TSInstall all<CR>` で全言語を一括インストールすることもできます。

### ファイルピッカー：vim-fall

![image](https://obsidian-image.wagomu.me/acc1e3d4fae4744336c320a3d27aa903.png)

:::details この部分
```lua
-- ---------------------------------------------------------------------------
-- vim-fall（ファイルピッカー）
-- ---------------------------------------------------------------------------
-- VSCode の Ctrl+P と同じ感覚でファイルを開けます。
-- Ctrl+P → ファイル名を入力 → Enter で開く
vim.pack.add({
  -- === ファイルピッカー: vim-fall ===
  -- Deno (denops.vim) が必要です。
  "https://github.com/lambdalisue/vim-fall",
})

vim.keymap.set("n", "<C-p>", "<Cmd>Fall file<CR>", { desc = "ファイルピッカー (vim-fall)" })
```
:::


VSCodeの`Ctrl+P`と同じ感覚でファイルを開けるプラグインです。

**操作方法：**
1. `Ctrl+P`を押す
2. ファイル名の一部を入力（ファジー検索）
3. `Enter`で開く

これだけです。Deno（denops.vim）が必要ですが、動作が軽快でおすすめです。

**代替プラグイン：**

| プラグイン | 特徴 |
|------------|------|
| [telescope.nvim](https://github.com/nvim-telescope/telescope.nvim) | 最も有名で情報が多い。困ったときに検索しやすい |
| [mini.pick](https://github.com/echasnovski/mini.pick) | 最小構成で依存が少ない。Denoも不要 |

### ファイルエクスプローラー：vim-fern

![image](https://obsidian-image.wagomu.me/76a72826005bc84484287d890ee7d6a1.png)


:::details この部分
```lua
-- ---------------------------------------------------------------------------
-- vim-fern（ファイルエクスプローラー）
-- ---------------------------------------------------------------------------
-- VSCode の Ctrl+Shift+E / Ctrl+B と同じ感覚でサイドバーを操作できます。
vim.pack.add({
  -- === ファイルエクスプローラー: vim-fern ===
  "https://github.com/lambdalisue/vim-fern",
})

-- Ctrl+Shift+E: ファイルエクスプローラーを開く
vim.keymap.set("n", "<C-S-e>", "<Cmd>Fern . -reveal=% -drawer -width=30<CR>", {
  desc = "ファイルエクスプローラーを開く (vim-fern)",
})

-- Ctrl+B: ファイルエクスプローラーをトグル
vim.keymap.set("n", "<C-b>", "<Cmd>Fern . -reveal=% -drawer -width=30 -toggle<CR>", {
  desc = "ファイルエクスプローラーをトグル (vim-fern)",
})
```
:::

VSCodeのサイドバーにあるファイルツリーのイメージです。

**操作方法：**
- `Ctrl+Shift+E`：ファイルエクスプローラーを開く
- `Ctrl+B`：トグル（表示/非表示を切り替え）
- 矢印キーの上下で移動、`Enter`でディレクトリ展開 or ファイルを開く

**代替プラグイン：**

| プラグイン | 特徴 |
|------------|------|
| [neo-tree.nvim](https://github.com/nvim-neo-tree/neo-tree.nvim) | Lua製のモダンなファイルエクスプローラー |
| [oil.nvim](https://github.com/stevearc/oil.nvim) | バッファ自体がファイルエクスプローラーになるユニークなUI |

### タブライン：bufferline.nvim

![image](https://obsidian-image.wagomu.me/c6c095d1a43f9901a2d4eea2a53aeac0.png)

:::details この部分
```lua
-- ---------------------------------------------------------------------------
-- bufferline.nvim（タブライン）
-- ---------------------------------------------------------------------------
-- 複数ファイルを開くと画面上部にファイル名がタブ表示されます。
-- クリックでも切り替えられます。
vim.pack.add({
  -- === タブライン: bufferline.nvim ===
  "https://github.com/akinsho/bufferline.nvim",
})

require("bufferline").setup({
  options = {
    -- 閉じるボタンをクリックでバッファを閉じられる
    close_command = "bdelete! %d",
    -- マウスの右クリックで閉じる
    right_mouse_command = "bdelete! %d",
    -- タブの見た目
    separator_style = "thin",
    -- 数字の表示
    numbers = "ordinal",
  },
})

```
:::

複数ファイルを開くと画面上部にファイル名がタブとして表示されます。クリックでも切り替えられるので、VSCodeのタブバーと同じ感覚で使えます。

**代替プラグイン：**

| プラグイン | 特徴 |
|------------|------|
| [barbar.nvim](https://github.com/romgrk/barbar.nvim) | アニメーション付きのタブバー |
| [mini.tabline](https://github.com/echasnovski/mini.tabline) | 最小構成のタブライン |

### Git操作：vim-gin

![image](https://obsidian-image.wagomu.me/3732f5c1d3950082adec10a241b06f69.png)

![image](https://obsidian-image.wagomu.me/1a1ca57b5ec301e1400b4073d7449d55.png)

![image](https://obsidian-image.wagomu.me/821b1fd291c25880aa9b66bf8e160786.png)

:::details この部分
```lua
-- ---------------------------------------------------------------------------
-- vim-gin（Git操作）
-- ---------------------------------------------------------------------------
-- ターミナルでの git コマンドと同じ感覚で使えます。
--
-- 使い方:
--   :Gin status           → git status
--   :Gin commit           → git commit
--   :Gin log              → git log
--   :GinStatus            → 対話的なステータス画面
--     ファイル上で << を押すとステージング
--     ファイル上で >> を押すとリセット
--   :GinLog               → コミット履歴
--   :GinDiff              → 差分表示
vim.pack.add({
  -- === Git操作: vim-gin ===
  -- Deno (denops.vim) が必要です。
  "https://github.com/lambdalisue/vim-gin",
})
```
:::


ターミナルでのgitコマンドと同じ感覚で使えるのがvim-ginの良いところです。

**使い方：**
- ノーマルモードで `:Gin` + 普段のgitサブコマンドで`Enter`
- `:GinStatus` でステータス確認 → ファイル上で `<<` するとステージング、`>>` するとリセット
  - これが特に便利です！！
- `:GinLog` でコミット履歴を確認

ノーマルモードで`:Gin`と入力後に`Ctrl+D`を押してみてください、沢山の候補が表示されます。

**代替プラグイン：**

| プラグイン | 特徴 |
|------------|------|
| [vim-fugitive](https://github.com/tpope/vim-fugitive) | Tim Pope作の老舗Gitプラグイン。Deno不要 |
| [neogit](https://github.com/NeogitOrg/neogit) | EmacsのMagitにインスパイアされたGitプラグイン |

### Git差分表示：gitsigns.nvim

![image](https://obsidian-image.wagomu.me/9be273b9dddda72f2b16e4bae5ba3617.png)

![image](https://obsidian-image.wagomu.me/0fc20e38ea73e4686e0bfb58fe0c4622.png)

:::details ここの部分
```lua
-- ---------------------------------------------------------------------------
-- gitsigns.nvim（Git差分表示）
-- ---------------------------------------------------------------------------
-- ファイルの変更箇所の左側にカラーマークが表示されます。
vim.pack.add({
  -- === Git差分表示: gitsigns.nvim ===
  "https://github.com/lewis6991/gitsigns.nvim",
})

require("gitsigns").setup({
  signs = {
    add          = { text = "┃" },
    change       = { text = "┃" },
    delete       = { text = "_" },
    topdelete    = { text = "‾" },
    changedelete = { text = "~" },
    untracked    = { text = "┆" },
  },
  -- 現在行のblame情報を表示したい場合はtrueに
  current_line_blame = false,
})

```
:::

ファイルを開くと変更箇所の左側にカラーマークが表示されます。変更した行、追加した行、削除した行がひと目で分かります。

変更箇所の差分プレビューやblameの確認もできるので、AIエージェントがどこを変更したかの確認にも使えます。

### AIエージェント操作：nvim-aibo

![image](https://obsidian-image.wagomu.me/63d1bb803e72d80fae2ef9f84faee72e.png)

[lambdalisue/nvim-aibo](https://github.com/lambdalisue/nvim-aibo)は、Neovimのバッファを使ってClaude CodeやCodex CLIに指示を出せるプラグインです。

`:Aibo claude`、`:Aibo codex`などを実行するとスクリーンショットのようになります。Neovimの中にいながら、すぐにNeovim内でClaude Codeなどを立ちあげられる、便利プラグインです。

## おまけ

### マークダウンプレビュー


 [MeanderingProgrammer/render-markdown.nvim](https://github.com/MeanderingProgrammer/render-markdown.nvim)

ブログやドキュメントを書く方には、マークダウンプレビュー系のプラグインもおすすめです。

render-markdown.nvim入れる前後のこの記事の見た目比較です。

![image](https://obsidian-image.wagomu.me/77e0d92a040e10aaa2934d466e46e6db.png)

![image](https://obsidian-image.wagomu.me/b505e46aea47768fa1a9ad16918be25d.png)

ヘディングやコードブロックが文字色だけではなく、背景色や分かりやすい記号が表示されて見やすくなります。



## 代替プラグインの設定例

メインで紹介したプラグインの代わりに使えるプラグインの設定例です。使いたいものがあれば、`vim.pack.add`にプラグインを追加し、対応するキーマッピングのコメントを外してください。


## おわりに

AIがコードを書く今では、「書く道具」だったNeovimが「読む・確認する道具」としての価値が増したと感じています。

この設定はNeovimのスタートを促すものです。もっとこうしたい！！！というのがあれば、vim-jp Slackに参加するか、AIエージェントに聞いてみてください。概ね実現してくれるはずです！！

まずは`init.lua`をコピーして、`Ctrl+P`でファイルを開くところから試してみてください！！！！
