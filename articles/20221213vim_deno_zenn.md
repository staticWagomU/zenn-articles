---
title: "vimとdenoでZennの執筆環境を作る"
emoji: "👏"
type: "tech"
topics: ["vim", "deno"]
published: true
published_at: 2022-12-20 09:00
publication_name: "vim_jp"
---

:::message
この記事は[Vim Advent Calendar 2022](https://qiita.com/advent-calendar/2022/vim)の20日目の記事です。
:::

:::message
当記事は今回作成した環境で書いています。
:::

## 動機
アドベントカレンダーに参加をしたものの何を書こうか悩んでいたときvim-jpのSlackにこのような発言がありました。
![](/images/vim_deno_zenn/img1.png)

denoの1.28がリリースされたことにより`--unstable`フラグを使わずにnpmパッケージが扱えるようになりました。それによりZennへ記事を投稿するためにNode.jsを入れなくてもよいのではないかと考えこの記事を書こうと思いました。

Zennの記事を執筆するにあたり、
 - zenn-cli
 - textlint

これらが動作すれば快適に記事を執筆できるため、この記事では
 - denoからtextlintを動作させる
 - lspとtextlintの連携

の2つを目標として進めていきます。

私の使用している環境は以下のとおりです。
```shell
$ uname -a
Darwin makkubukkukun 21.3.0 Darwin Kernel Version 21.3.0: Wed Jan  5 21:37:58 PST 2022; root:xnu-8019.80.24~20/RELEASE_ARM64_T8101 arm64

$ vim --version
VIM - Vi IMproved 9.0 (2022 Jun 28, compiled Dec 06 2022 09:11:39)
macOS version - arm64
Included patches: 1-1018
Compiled by Homebrew

$ deno --version
deno 1.28.3 (release, aarch64-apple-darwin)
v8 10.9.194.5
typescript 4.8.3
```
## 下準備
### denoのインストール
npmが安定版として対応したバージョンは2.18です。それ以降のバージョンが導入できる方法であればなんでもよいため、公式サイトからお好きな方法で導入してください。

### zenn-cliのインストールと初期化
次にZennの記事を管理するディレクトリで下記コマンドを実行し、セットアップを行います。

```shell
$ deno run -A npm:zenn-cli@latest init
```

### 必要なパッケージのインストール
そして、textlintのインストールを行います
textlintは単体では動作せず、校正ルールというものが必要になります。これらは別パッケージとして提供されているためお好きなものを導入してください。私は今回初めてtextlintを使うため、[ゴリラさん](https://zenn.dev/skanehira/articles/2020-11-16-vim-writing-articles)の記事を参考に下記3つのパッケージを導入しようと思います。
 - textlint-rule-preset-jtf-style 
 - textlint-rule-preset-ja-technical-writing
 - textlint-rule-prh

```shell
# インストールコマンド
$ deno cache --node-modules-dir npm:textlint npm:textlint-rule-prh@latest npm:textlint-rule-preset-jtf-style@latest npm:textlint-rule-preset-ja-technical-writing@latest
```
[--node-modules-dir](https://deno.land/manual@v1.28.0/node/npm_specifiers#--node-modules-dir-flag)フラグを使うことでコマンドを実行したディレクトリへnpm installをしたときと同じ形式でnodemodulesフォルダが作成されます。

## 設定

### プラグインの導入

プラグインは下記3つをお好みのプラグインマネージャーからインストールしてください。
| プラグイン | 説明 |
| ---------- |--------|
|[prabirshrestha/vim-lsp](https://github.com/prabirshrestha/vim-lsp)|lspの本体 |
|[mattn/vim-lsp-settings](https://github.com/mattn/vim-lsp-settings)| lspの設定を楽にしてくれる|
|[tyru/open-browser.vim](https://github.com/tyru/open-browser.vim)| vimからブラウザを起動できる|

### lspの導入

### textlintの設定

## コマンドの作成
:::message alert
現在、windows環境下においてdenoからは`preview`コマンドを実行できません。
しかし、windows(wsl)環境において動作することは確認しています。
:::

## まとめ

## vimrc全体

```vim
" plugins ================================
call plug#begin()

    " lsp本体
    Plug 'prabirshrestha/vim-lsp'
    " lspの設定を簡単にしてくれる
    Plug 'mattn/vim-lsp-settings'
    " vimからブラウザを起動できる
    Plug 'tyru/open-browser.vim'

call plug#end()

" lsp settings ================================
let g:lsp_settings = {
			\ 'efm-langserver': {
			\   'disabled': 0,
			\   'allowlist': ['markdown'],
			\  }
			\ }
let g:lsp_diagnostics_float_cursor = 1
let g:lsp_diagnostics_enabled = 1
function! s:on_lsp_buffer_enabled() abort
	setlocal completeopt=menu
	setlocal omnifunc=lsp#complete
	if exists('+tagfunc') | setlocal tagfunc=lsp#tagfunc | endif
	nmap <buffer> gd <plug>(lsp-definition)
	nmap <buffer> gs <plug>(lsp-document-symbol-search)
	nmap <buffer> gS <plug>(lsp-workspace-symbol-search)
	nmap <buffer> gr <plug>(lsp-references)
	nmap <buffer> gi <plug>(lsp-implementation)
	nmap <buffer> gt <plug>(lsp-type-definition)
	nmap <buffer> <leader>rn <plug>(lsp-rename)
	nmap <buffer> [g <plug>(lsp-previous-diagnostic)
	nmap <buffer> ]g <plug>(lsp-next-diagnostic)
	nmap <buffer> K <plug>(lsp-hover)
endfunction

augroup lsp_install
	au!
	au User lsp_buffer_enabled call s:on_lsp_buffer_enabled()
augroup END

" user command ================================

function! s:zenn_create_article(article_name) abort
	let aname = a:article_name
	" slugは12文字以上、50文字以下
	" 先頭に日付(yyyymmdd)を加えるため、実質4文字以上、42文字以下
	if strlen(a:article_name) > 42
		let aname = a:article_name[0:42]
	endif
	if strlen(a:article_name) < 4
		let aname = a:article_name .. "___"
	endif
	echo "1:tech 2:idea"
	let a = getchar()
	let type = "tech"
	if a == 50
		let type = "idea"
	endif
	let date = strftime("%Y%m%d")
	let slug = date .. aname
	call system("deno run -A npm:zenn-cli@latest new:article --slug " .. slug .. " --type " .. type )
	execute "edit articles/" .. slug .. ".md"
endfunction

function! s:zenn_preview() abort
	execute "bo term deno run -A npm:zenn-cli@latest preview"
	execute "resize -100"
	execute "normal! \<c-w>\<c-w>"
	execute "sleep"
	execute "OpenBrowser localhost:8000"
endfunction

command! -nargs=1 ZennCreate call <sid>zenn_create_article(<f-args>)
command! ZennPreview call <sid>zenn_preview()
```

## 参考
https://zenn.dev/skanehira/articles/2020-11-16-vim-writing-articles


