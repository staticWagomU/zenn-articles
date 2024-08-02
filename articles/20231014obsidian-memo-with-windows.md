---
title: "Windowsでもデイリーノートのハードルを下げる"
emoji: "🦒"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["obsidian", "windows"]
published: true
---

## はじめに

前回の[RaycastでObsidianのデイリーノートのハードルを更に下げる](https://zenn.dev/wagomu/articles/20231001_raycast_obsidian_memos)の続きです。
最後にwindowsでも同じことが実現できないかと悩んでいましたが、自分が満足できるそれっぽいものができたので共有したいと思います。

## 完成品
![](/images/20231014obsidian-memo-with-windows/animation.gif)

## 必要なもの
- Obsidian
	- Advanced URI(コミュニティプラグイン)
	- Obsidian Memos(コミュニティプラグイン)
- AutoHotkey
- PowerShell
- 前に出さないZ

### コミュニティプラグイン
`設定 > コミュニティプラグイン`の制限モードを有効化し、コミュニティプラグイン右側の`閲覧`から`Advanced URI`と`Obsidian Memos`のプラグインを検索し、インストールと有効化をしてください。

Obsidianとコミュニティプラグインについての説明は割愛します。知りたい方は[前回の記事](https://zenn.dev/wagomu/articles/20231001_raycast_obsidian_memos)を参照ください。

### [AutoHotkey](https://www.autohotkey.com)

> Powerful. Easy to learn.
> The ultimate automation scripting language for Windows.

Deepl先生訳
>パワフル。学びやすい。  
>Windowsのための究極の自動化スクリプト言語。

今回はショートカットキーを定義して、PowerShellを呼び出すために使用します。
### PowerShell

Windowsに搭載されているコマンドシェル

### [前に出さないZ](https://www.vector.co.jp/soft/winnt/util/se517075.html)

特定のウィンドウを最背面に固定するためのソフトです。
:::message alert
Advanced URI経由でObsidianを起動する際に、バックグラウンド実行ができずに毎度Obsidianが最前面に来てしまう問題を解決するために、前に出さないＺを導入します。もし、最前面に来ても問題ないという方がいましたらこちらは不要です。
導入された方も、Obsidianを最背面に固定すると登録解除しないかぎり前面にもってこれなくなるので、Memos以外の用途の場合は登録解除することをおすすめします。
:::


## やりかた
PowerAutomateでも実現できないかと模索しましたが、うまくいかなかったため、PowerShellのスクリプトをAutoHotkeyから起動するという方針をとります。


### Scripts
#### Powershell
```powershell
# アセンブリの読み込み
[void][System.Reflection.Assembly]::Load("Microsoft.VisualBasic, Version=8.0.0.0, Culture=Neutral, PublicKeyToken=b03f5f7f11d50a3a")

# inputboxの表示
$input = [Microsoft.VisualBasic.Interaction]::InputBox("take a memo", "Take Obsidian Memos")
$memo = $input -replace ' ', '%20'
$now = Get-Date
$current_time = $now.ToString("HH:mm")

# MyVaultの部分を自身のvault名に書き換えてください。
Start-Process "obsidian://advanced-uri?vault=MyVault&daily=true&mode=append&data=-%20$current_time%20$memo"
```
##### 雰囲気解説(AutoHotkey)
InputBoxの引数は、
メッセージ：テキストボックスの上のメッセージ
タイトル：ウィンドウのタイトル
です。
```powershell
[Microsoft.VisualBasic.Interaction]::InputBox("メッセージ", "タイトル")
```

ターミナルでの実行方法は、[Getting Started | Obsidian Advanced URI](https://vinzent03.github.io/obsidian-advanced-uri/getting_started#terminal)を参考に、`Start-Process`コマンドを使います。

#### AutoHotkey
![](/images/20231014obsidian-memo-with-windows/1.png)
AutoHotkeyを起動し、左上の`New script`を選択してください。
![](/images/20231014obsidian-memo-with-windows/2.png)
好きな場所にスクリプトを作成して、下記のソースを貼り付けてください。
このとき、`C:\to\your\ps1\path\takeAmemos.ps1`は先ほど作成したpowershellスクリプトのパスになります。

```autohotkey
!m::
{
  Run "powershell -WindowStyle Hidden -NoProfile -ExecutionPolicy Unrestricted C:\to\your\ps1\path\takeAmemos.ps1"
}
```

##### 雰囲気解説(AutoHotkey)
１行目の`!m`は`!`が`Alt`キーのことを指しており、`Alt` + `m`でRun以下を実行します。今回はPowerShellのスクリプトを起動するようにしています。  
`-WindowStyle Hidden`をつけることで、実行時にPowerShellのウィンドウを閉じた状態で実行することができます。しかし、何故か不定期でウィンドウが開いたまま起動する問題があり正直謎です。

## 実行
作成した`.ahk`ファイルをダブルクリックで実行して、前に出さないZとObsidianを起動してください。インジケーターから前に出さないZを右クリックし、`ウィンドウ登録(W)`からObsidianを選択してくだい。
これで準備は完了です。
あとは、`Alt` + `m`でpowershellスクリプトを起動しインプットボックスにテキストを入力するとObsidianに追記されます。

もし、PCを立ちあげる度に`.ahk`ファイルと前に出さないZを起動するのが面倒な方がいれば、`win`+`r`で`shell:startup`を実行し、スタートアップフォルダに`.ahk`ファイルのショートカットと前に出さないZのショートカットを作成してください。これでPCを立ち上げる度に自動で起動してくれます。

## 認知している問題
- PowerShellのウィンドウが開いたままのことが多々ある。原因は分からないが、私はそこまで気にならないので無視。
- Inputboxが空のままでも書き込まれてしまう。こちらも自分は気にならないので無視。


## おわりに
では、Windowsでも快適なObsidianライフをお過ごしください。

## 参考
[PowerShellをはじめよう　～PowerShell入門～: PowerShellでInputBoxを使用する](https://letspowershell.blogspot.com/2015/06/powershellinputbox.html)
