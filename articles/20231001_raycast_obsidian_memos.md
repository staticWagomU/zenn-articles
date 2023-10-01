---
title: "RaycastでObsidianのデイリーノートのハードルを更に下げる"
emoji: "🍡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["obsidian", "raycast"]
published: true
---

![](/images/20231001_raycast_obsidian/1.gif)
## はじめに

最近Raycastを使い始めた輪ごむです。
開発機として使用しているMacは基本的にターミナルとブラウザ、あとはObsidianしか開かないため、標準搭載されているSpotlightだけも不満なく作業は完結していました。そんな私がRaycastを導入した理由はただひとつ、自作のスクリプトを動かすことができるという点に惹かれたからです。  
私は日記や作業・勉強メモなどのテキストをすべてObsidianに集約させています。特に、日記を毎日書くためにデイリーノートへタイムスタンプ付きで呟くことができるObsidian Memosというコミュニティプラグインを愛用しています。そこにつぶやいた内容を見ならがら記憶を呼び戻して日記を書いています。しかし、作業中に毎度毎度Obsidianを開き、その時思ったことを呟くのは少し手間だなと感じていました。ところが、Raycastのスクリプトコマンドを使えばシームレスに呟くことができる。ツイ廃に近づくことができるのではないかと思い試した結果うまくいき、呟くハードルが大きく下がったためこの知見を共有したいと思います。

## 必要なもの
- Raycast
- Obsidian
	- Advanced URI(コミュニティプラグイン)
	- Obsidian Memos(コミュニティプラグイン)

### [Raycast](https://www.raycast.com/)
> Raycast is a blazingly fast, totally extendable launcher. It lets you complete tasks, calculate, share common links, and much more.

Deepl先生訳
> Raycastは非常に高速で、完全に拡張可能なランチャーです。タスクの完了、計算、共通リンクの共有など、様々なことができます。

ざっくりいうと,Spotlightより便利なランチャーです。  
インストーラーをダウンロードするか、Home brew経由でインストールすることができます。
```shell
brew install --cask raycast
```

### [Obsidian](https://obsidian.md/)
ローカルで動作する便利なメモアプリ。  
私はシンプルな機能であること、ローカルで動作するのでオフラインでも使えること、モバイルでも十分扱えること。この３点が気に入りここ半年ほど、使用しています。
iOS、Android、Windows、Mac、Linuxで動作するマルチプラットフォームな端末で使用できるため、使っているスマホがAndroidでPCがWindowsとMacと偶にLinuxの私にとってはどの環境でも使える点も魅力的です。
### コミュニティプラグイン
`設定 > コミュニティプラグイン`の制限モードを有効化し、コミュニティプラグイン右側の`閲覧`から下記のプラグインを検索し、インストールと有効化をしてください。

#### [Advanced URI](https://github.com/Vinzent03/obsidian-advanced-uri)
URIを使って外部からObsidianを操作することができるプラグイン。

#### [Obsidian Memos](https://github.com/Quorafind/Obsidian-Memos)
OSSの[Memos](https://github.com/usememos/memos)と[flomo](https://flomoapp.com/)をベースとして作られたプラグイン。
Twitter(現X)のような見た目をしており、呟いた内容がデイリーノートへリスト、あるいはタスク形式で書き込まれていくプラグインです。メモを取るハードルを大幅に下げてくれるのでObsidianを始めたての人にもおすすめできるコミュニティープラグインです。  
`- HH:MM つぶやいた内容`  
`- [ ] HH:MM つぶやいた内容`  
という形式でデイリーノートへ追記されます。

## やりかた
RaycastのScript Commandsという機能を使って、デイリーノートへリスト形式で追記するコマンドを追加します。

### Script Command
BashやApple Script,Ruby,Python,Node.jsを使ってスクリプトを書き、それを実行することができます。
詳細はドキュメントの[Script Commands](https://manual.raycast.com/script-commands)をみてください。
また、Script Commandの作り方は[ランチャーツールRaycastの使い方と設定 | DevelopersIO](https://dev.classmethod.jp/articles/eetann-used-raycast/#toc-9)を参照してください。
私はとくに理由はありませんが、Bashを選択しました。


### Script
```bash
#!/bin/bash

# Required parameters:
# @raycast.schemaVersion 1
# @raycast.title Append Memos
# @raycast.mode compact

# Optional parameters:
# @raycast.icon 🗒️
# @raycast.argument1 { "type": "text", "placeholder": "Take memos" }

current_time=$(date +"%H:%M")
memo=$(echo "$1" | sed 's/ /%20/g' )
open --background "obsidian://advanced-uri?vault=MyLife&daily=true&mode=append&data=-%20$current_time%20$memo" # MyLifeの部分を自身のvault名に書き換えてください。
```

#### 雰囲気解説

コメント部分は割愛して、なにをしているのかざっくりと説明します。
`- H:M つぶやいた内容`というフォーマットにして、URI形式でopenコマンドを実行しています。
つぶやいた内容に空白が含まれていると空白以降がObsidianに追記されないため、`%20`に変換をかけて実行しています。

## 実行
Raycastを開いて`Append Memos`と入力し、Tabを押下することでとなりにあるテキストボックスへフォーカスが移動します。あとはXのようにつぶやいてエンターキーを押すだけで、デイリーノートへ入力した内容が転記されていることを確認できるはずです。
![](/images/20231001_raycast_obsidian/3.gif)
さらに手間を省きたい人はHotkeyを設定するといいでしょう。自分はMemoの頭文字をとって`Command + m`で一発で起動するようにしています。
![](/images/20231001_raycast_obsidian/2.png)

## おわりに
快適なObsidianライフをお過ごしください。

windowsでも同じようなことを実現できないか模索中。

