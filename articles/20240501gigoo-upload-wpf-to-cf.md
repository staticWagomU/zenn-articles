---
title: "Windowsで作ったWPFのexeをCludflareR2とD1でバージョン管理する"
emoji: "😤"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["wpf", "windows", "r2", "d1"]
published: false
publication_name: "gigooo_blog"
---
## はじめに

WPFでつくったソフトをバージョンを管理する必要がありました。しかし、インストーラーを作成するレベルの規模ではないため、Github acrionsを使ってCludflare R2へアップロードするのはどうかという話になったため、実践した内容となります。
最終的に、バイナリをR2へアップロードして、バージョンとリリース日をD1へinsertするワークフローになりました。

win32apiを使用していたため、windowsでbuildする必要があり、そのためのワークフローを組み上げていく過程でいくつかハマりポイントがあったのでブログとしてまとめようと思います。

余談ですが、そもそも情報が少ないことや、初めてGithub actionsを触ったこともあり結構大変でした。


最終状態のコードは下記レポジトリで公開しています。
TODO: レポジトリのURL貼る

## ワークフロー
```yml
name: Build and Upload on Tag

on:
  push:
    tags:
      - '*.*.*.*'

jobs:
  build-and-upload:
    runs-on: windows-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        
      - name: Get current datetime
        id: get_datetime
        run: echo "DATETIME=$(Get-Date -Format "yyyy-MM-ddThh:mm:ss")" | Out-File -FilePath $Env:GITHUB_OUTPUT -Encoding utf8 -Append
        
      - name: Get tag name
        id: get_tag_name
        run: echo "TAG_NAME=$($Env:GITHUB_REF -replace 'refs/tags/','')" | Out-File -FilePath $Env:GITHUB_OUTPUT -Encoding utf8 -Append
        
      - name: Setup .NET
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: '8.0.x'
          
      - name: Build and publish
        run: dotnet publish .\SampleProject\SampleProject.csproj -c Release -o .\dist
        
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
        
      - name: Install pnpm
        uses: pnpm/action-setup@v3
        with:
          version: 8
          run_install: false
          
      - name: Install Wrangler
        run: pnpm add wrangler -g
          
      - name: Upload to Cloudflare R2
        env:
          CLOUDFLARE_ACCOUNT_ID: ${{ secrets.CF_ACCOUNT_ID }}
          CLOUDFLARE_API_TOKEN: ${{ secrets.R2_API_TOKEN }}
        run: wrangler r2 object put "sample-project/${{ steps.get_tag_name.outputs.TAG_NAME }}/SampleProject.exe" --file=dist/SampleProject.exe

      - name: Insert to Cloudflare D1
        env:
          CLOUDFLARE_ACCOUNT_ID: ${{ secrets.CF_ACCOUNT_ID }}
          CLOUDFLARE_API_TOKEN: ${{ secrets.D1_API_TOKEN }}
        run: wrangler d1 execute DB --remote --command "INSERT INTO Release(Version, Release_date) VALUES('${{ steps.get_tag_name.outputs.TAG_NAME }}', '${{ steps.get_datetime.outputs.DATETIME }}')"
```


1. WPFをbuildして実行ファイルを作成

- WPFのexeをR2へアップロード
- tagとinsert日時をd1へinsert
- honoを使って画面へ表示とダウンロード

ブログの構成と概要は以下のようになります。

タイトル：Cloudflare R2とD1を活用したWPFアプリのリリース管理

1. はじめに
   - WPFアプリのexeをバージョン管理する動機について説明
   - インストーラー作成の手間を避けるためにexe直接のバージョン管理を選択

2. 実装の流れ
   - GitHubでのtagをトリガーにした一連の処理の概要
     1. win32apiを使用しているWPFソフトウェアのビルド
     2. ビルドしたexeのCloudflare R2へのアップロード
     3. tag名のCloudflare D1へのinsert
     4. honoを使ったリリースページの作成

3. GitHub ActionsでWindowsを使う際の注意点
   - Windowsを使った事例が少ないため、このブログを投稿する動機について
   - PowerShellを使う際のtag名の取得方法
     - `$Env:GITHUB_REF`をreplaceする必要性について

4. Cloudflare Wranglerコマンド実行時のエラー対処法
   - `cloudflare/wrangler-action`の代わりに`pnpm install -g wrangler`を使う
   - R2のAPIの権限を「管理者読み取りと書き込み」に設定する必要性

5. まとめ
   - WPFアプリのexe直接バージョン管理の利点
   - GitHub ActionsとCloudflareの組み合わせによるスムーズなリリース管理
   - Windows環境でのGitHub Actionsの使い方と注意点

このブログでは、WPFアプリのexe直接バージョン管理の実践例を紹介し、GitHub ActionsとCloudflare R2、D1を組み合わせたリリース管理の方法を解説します。また、Windows環境でのGitHub Actionsの使い方と注意点についても触れ、同様の課題に直面している開発者に有益な情報を提供します。
