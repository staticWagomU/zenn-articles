---
title: "vimとdenoでZennの執筆環境を作る"
emoji: "🦒"
type: "tech"
topics: ["vim", "deno", "textlint"]
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

### 動作イメージ
![](/images/vim_deno_zenn/gif1.gif)
## 下準備
### denoのインストール
npmが安定版として対応したバージョンは2.18です。それ以降のバージョンが導入できる方法であればなんでもよいため、公式サイトからお好きな方法で導入してください。

### zenn-cliのインストールと初期化
次にZennの記事を管理するディレクトリで下記コマンドを実行し、セットアップを行います。

```shell
$ deno run -A npm:zenn-cli@latest init
```

### 必要なパッケージのインストール
textlintのインストールを行います
textlintは単体では動作せず、校正ルールというものが必要になります。これらは別パッケージとして提供されているためお好きなものを導入してください。私は今回初めてtextlintを使うため、[ゴリラさん](https://zenn.dev/skanehira/articles/2020-11-16-vim-writing-articles)の記事を参考に下記3つのパッケージを導入しようと思います。
 - textlint-rule-preset-jtf-style 
 - textlint-rule-preset-ja-technical-writing
 - textlint-rule-prh

```shell
# インストールコマンド
$ deno cache --node-modules-dir npm:textlint@latest npm:textlint-rule-prh@latest npm:textlint-rule-preset-jtf-style@latest npm:textlint-rule-preset-ja-technical-writing@latest
```
[--node-modules-dir](https://deno.land/manual@v1.28.0/node/npm_specifiers#--node-modules-dir-flag)フラグを使うことでコマンドを実行したディレクトリへnpm installをしたときと同じ形式でnodemodulesフォルダが作成されます。

## 設定

### プラグインの導入

vimのプラグインは下記3つをお好みのプラグインマネージャーからインストールしてください。
| プラグイン | 説明 |
| ---------- |--------|
|[prabirshrestha/vim-lsp](https://github.com/prabirshrestha/vim-lsp)|lspの本体 |
|[mattn/vim-lsp-settings](https://github.com/mattn/vim-lsp-settings)| lspの設定を楽にしてくれる|
|[tyru/open-browser.vim](https://github.com/tyru/open-browser.vim)| vimからブラウザを起動できる|

### lspの導入
textlintのエラーを動的に反映させるために、efm-langserverというlspを使用します。
導入するには、vimにて`:LspInstallServer efm-langserver`と入力してインストールしてください。
また、もう少し詳細な説明が気になる人はゴリラさんの記事を参照してください。

インストール後.vimrcに下記をコードを貼りつけてください。
```vim
" markdownでefm-langserverを有効にします
let g:lsp_settings = {
			\ 'efm-langserver': {
			\   'disabled': 0,
			\   'allowlist': ['markdown'],
			\  }
			\ }

" ホバーした時にエラー内容が表示されるよ
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
        " bufferでlspが有効だったら、関数を呼びだしてキーマッッピングを提供するよ
	au User lsp_buffer_enabled call s:on_lsp_buffer_enabled()
augroup END
```

次に`efm-langserver`の設定ファイルを`~/.config/efm-langserver/`へ`config.yaml`という名前で作成して下記の内容を貼りつけてください。

```config.yaml
version: 2
tools:
  markdown-textlint: &markdown-textlint
    lint-command: 'deno run -A --node-modules-dir npm:textlint@latest --format unix --stdin ${INPUT}'
    lint-ignore-exit-code: true
    lint-stdin: true
    lint-formats:
      - '%f:%l:%c: %m [%trror/%r]'
    root-markers:
      - .textlintrc
languages:
  markdown:
    - <<: *markdown-textlint
```

### textlintの設定
textlintの設定ファイルは作成したフォルダ直下に`.textlintrc`という名前で作成して下記の内容を貼りつけてください。
```.textlintrc
{
  "filters": {},
  "rules": {
    "preset-ja-technical-writing": {
      "ja-no-weak-phrase": false,
      "ja-no-mixed-period": false,
      "no-exclamation-question-mark": false
    },
    "preset-jtf-style": true,
    "prh": {
      "rulePaths": [
        "node_modules/prh/prh-rules/media/WEB+DB_PRESS.yml",
        "node_modules/prh/prh-rules/media/techbooster.yml"
      ]
    }
  }
}
```
## コマンドの作成
::: message alert
現在、windows環境下においてdenoからは`preview`コマンドを実行できません。
:::

ターミナルに戻ることなくvim内で記事の執筆を完結させるために、
 - 記事の作成
 - プレビュー+ブラウザの起動

を行うためのコマンドを作成します。

### 記事の作成
```vim
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

command! -nargs=1 ZennCreate call <sid>zenn_create_article(<f-args>)
```
zenn-cliの`new:article`コマンドでは幾つかのフラグが存在しますが、今回作成したコマンドではslugとtypeを指定できるようにしています。

### プレビュー+ブラウザの起動
```vim
function! s:zenn_preview() abort
	execute "bo term deno run -A npm:zenn-cli@latest preview"
	execute "resize -100"
	execute "normal! \<c-w>\<c-w>"
	execute "sleep"
	execute "OpenBrowser localhost:8000"
endfunction

command! ZennPreview call <sid>zenn_preview()
```
zenn-cliからプレビューコマンドをたたくためにはターミナルを開く必要があります。しかし、ターミナルのせいで画面が狭くなってしまうことをさけたかったので、垂直分割でターミナルを開き`resize -100`で目一杯ウィンドウを小さくしたあとに元いたウィンドウへ戻るようにしています。

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

## さいごに
もしzenn-cliのプレビューを起動したときに、アップグレードしてねと言われたときは`deno cache --reload npm:zenn-cli@latest`を実行してください。

## 参考
https://zenn.dev/skanehira/articles/2020-11-16-vim-writing-articles
https://deno.land/manual@v1.29.1/getting_started/installation
https://deno.land/manual@v1.28.0/node/npm_specifiers#--node-modules-dir-flag
https://deno.land/manual@v1.28.2/basics/modules/reloading_modules
