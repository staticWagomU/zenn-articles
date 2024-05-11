---
title: "Windowsで作ったWPFアプリをCludflare R2とD1でバージョン管理する"
emoji: "🦒"
type: "tech"
topics: ["wpf", "windows", "r2", "d1"]
published: false
publication_name: "gigooo_blog"
---
## はじめに

WPFアプリをバージョンを管理しようと思いつき、Cludflare R2へアップロードしてD1へバージョンとリリース日をinsertするGitHub Actionsのワークフローを作ったよという記事です。

WPFアプリ内でwin32apiを使用しているためWindowsでbuildする必要があったのですが、その過程でいくつかポイントがあったので記事としてまとめようと思います。


最終状態のコードは下記レポジトリで公開しています。
TODO: レポジトリのURL貼る

## ワークフロー
最終的なワークフローはこちらです。
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

## ポイント

### pwshを使ってtag名を取得する

```yml
      - name: Get tag name
        id: get_tag_name
        run: echo "TAG_NAME=$($Env:GITHUB_REF -replace 'refs/tags/','')" | Out-File -FilePath $Env:GITHUB_OUTPUT -Encoding utf8 -Append
```

尚、outputとして使わないのであればこれだけで十分です。
```yml
      - name: Get tag name
        shell: pwsh
        run: |
          $tagName = $env:GITHUB_REF -replace 'refs/tags/',''
```


### WPFのビルド

https://wagomu.me/blog/2024-04-07-wpf-single-binary/


### R2のAPIの作成
R2のAPIを作成する際に一点注意すべき点があります

この問題については、コミュニティーでも取りあげられていますが、未だ対応されていないようです。
https://community.cloudflare.com/t/wrangler-r2-usage-fails-when-using-non-admin-tokens/600481

### 何故Cloudflare公式のactionを使わなかった
GitHub Actionsからwranglerを使うためには、公式の出しているActionsも存在します。
https://github.com/cloudflare/wrangler-action
これを使わなかった理由としては、手っ取り早く動かしたいという理由がありました。

## おまけ
以下、おまけです。

R2とD1を使って管理しているデータをフロント側でどのように表示・ダウンロードさせているかについて書いていきます。

フロントエンドとAPIはhonoを使っています。
また、ormとしてdrizzleを採用しました。

## さいごに

このブログでは、WPFアプリのexe直接バージョン管理の実践例を紹介し、GitHub ActionsとCloudflare R2、D1を組み合わせたリリース管理の方法を解説します。また、Windows環境でのGitHub Actionsの使い方と注意点についても触れ、同様の課題に直面している開発者に有益な情報を提供します。
