---
title: "Windowsで作ったWPFアプリをCludflare R2とD1でバージョン管理する"
emoji: "🦒"
type: "tech"
topics: ["wpf", "windows", "r2", "d1"]
published: true
publication_name: "gigooo_blog"
published_at: 2024-05-20 09:00
---
## はじめに

WPFアプリをバージョンを管理しようと思いつき、Cludflare R2へアップロードしてD1へバージョンとリリース日をインサートするGitHub Actionsのワークフローを作ったよという記事です。

WPFアプリ内でwin32apiを使用しているためWindowsでbuildする必要があったのですが、その過程でいくつかポイントがあったので記事としてまとめようと思います。

runs-onにwindows-latestを指定するとshellがpwshになるということを知らず、ハマってしまいました。


最終状態のコードは下記レポジトリで公開しています。

https://github.com/staticWagomU/WpfAndCloudflare

## ワークフロー
ワークフローはこちらです。
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
        run: dotnet publish .\WpfAndCloudflare\WpfAndCloudflare.csproj -c Release -o .\dist
        
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
        run: wrangler r2 object put "wpf-and-cloudflare/${{ steps.get_tag_name.outputs.TAG_NAME }}/WpfAndCloudflare.exe" --file=dist/WpfAndCloudflare.exe

      - name: Insert to Cloudflare D1
        env:
          CLOUDFLARE_ACCOUNT_ID: ${{ secrets.CF_ACCOUNT_ID }}
          CLOUDFLARE_API_TOKEN: ${{ secrets.D1_API_TOKEN }}
        run: wrangler d1 execute DB --remote --command "INSERT INTO Release(Version, Release_date) VALUES('${{ steps.get_tag_name.outputs.TAG_NAME }}', '${{ steps.get_datetime.outputs.DATETIME }}')"
```

## ポイント

### pwshでtag名を取得する

windows-latestで動かすとshellがpwshになっています。
そのため、tag名はbash的な書きかたではなく、このように取得します。
`GitHub Actions tag名 取得`などで検索をするとbashで書かれてあるものしか出てこず、変にハマってしまいました。
先頭に、`refs/tags/`という文字列が入るため、置換を挟んであげましょう。
```yml
      - name: Get tag name
        shell: pwsh
        run: $tagName = $env:GITHUB_REF -replace 'refs/tags/',''
```
今回はoutputとして使うため、下記のようにしています。

```yml
      - name: Get tag name
        id: get_tag_name
        run: echo "TAG_NAME=$($Env:GITHUB_REF -replace 'refs/tags/','')" | Out-File -FilePath $Env:GITHUB_OUTPUT -Encoding utf8 -Append
```

### WPFのビルド
WPFアプリをビルドする際にも一工夫が必要です。
なにも気にせず、`dotnet publish`コマンドを実行するとexeのほかに大量のdllが作成されてしまいます。これではR2へアップロードする際にzipへ圧縮する等の手間がかかります。
単一exeにするために、`*.csproj`へ下記を追加しましょう。
```diff xml
 <Project Sdk="Microsoft.NET.Sdk">
     <PropertyGroup>
         <OutputType>WinExe</OutputType>
         <TargetFramework>net8.0-windows</TargetFramework>
         <Nullable>enable</Nullable>
         <ImplicitUsings>enable</ImplicitUsings>
         <UseWPF>true</UseWPF>
+        <PublishSingleFile>true</PublishSingleFile>
+        <SelfContained>true</SelfContained>
+        <DebugSymbols>False</DebugSymbols>
+        <DebugType>None</DebugType>
         <AssemblyVersion>1.0.0.0</AssemblyVersion>
         <FileVersion>1.0.0.0</FileVersion>
     </PropertyGroup>
 </Project>
```

### R2のAPIの作成
R2のAPIを作成する際に一点注意すべき点があります。それは、権限を「管理者読み取りと書き込み」にするということです。
![](/images/gigoo_wpf_to_cf/img1.png)
本来であれば、「オブジェクト読み取りと書き込み」にして、対象のバケットを指定したいところですが、それでは`wrangler r2 object put`コマンドが失敗してしまうため、権限は **「管理者読み取りと書き込み」** にしましょう。

この問題については、コミュニティーでも取りあげられていますが、未だ対応されていないようです。
https://community.cloudflare.com/t/wrangler-r2-usage-fails-when-using-non-admin-tokens/600481

### 何故Cloudflare公式のactionを使わなかった

GitHub Actionsからwranglerを使うためには、公式の出しているActionsも存在します。
https://github.com/cloudflare/wrangler-action
これを使わなかった理由としては、pwshを使ってコマンドに変数を組み込んでいる実例が見つからなかったことと、手っ取り早く動かしたかったという理由によってです。

## おまけ

以下、おまけです。
せっかくなのでR2とD1へ登録した内容を表示・ダウンロードできるところまでも書いておこうとおもいます。
今回はhonoとdrrizle ormを使用しました。
レポジトリはこちらになります。

セットアップにはこちらの記事を参考にしました。

https://zenn.dev/collabostyle/articles/31d2276f247e60

最終的なコードはこちらで公開しています。

https://github.com/staticWagomU/WpfAndCloudflare

Releaseテーブルのスキーマはこのようにしています。
```ts
// src/schema/release/ts

import { sqliteTable, text } from "drizzle-orm/sqlite-core";

export const release = sqliteTable("Release", {
  version: text("Version").primaryKey().notNull(),
  releaseDate: text("Release_date").notNull(),
});

```

バージョンの一覧とダウンロード処理はこのようにしています。
```tsx
// src/index.tsx
import { renderer } from "./renderer";
import type { FC } from "hono/jsx";
import { Hono } from "hono";
import { drizzle } from "drizzle-orm/d1";
import { release } from "./schema/release";
import { desc, eq } from "drizzle-orm";

type Bindings = {
  DB: D1Database;
  MY_BUCKET: R2Bucket;
};

type Version = {
  version: string;
  releaseDate: string;
  releaseNote: string | null;
};

const app = new Hono<{ Bindings: Bindings }>();

app.use(renderer);

const Versions: FC<{ versions: Version[] }> = ({ versions }) => (
  <div>
    <h1>WpfAndCloudflare Release Page</h1>
    <table style="border-collapse:collapse;" border={1}>
      <tr style="">
        <th>Version</th>
        <th>Release Date</th>
        <th>Download Link</th>
      </tr>
      {versions.map((version) => (
        <tr key={version.version}>
          <td> {version.version} </td>
          <td>{(new Date(version.releaseDate)).toDateString()}</td>
          <td>
            <a href={`/api/versions/${version.version}`}>ダウンロード</a>
          </td>
        </tr>
      ))}
    </table>
  </div>
);
app.get("/", async (c) => {
  const db = drizzle(c.env.DB);
  const result = await db.select().from(release).orderBy(desc(release.releaseDate),desc(release.version)).all();
  return c.html(<Versions versions={result} />);
});

app.get("/api/versions/:version", async (c) => {
  try {
    const version = c.req.param("version");
    const db = drizzle(c.env.DB);
    const result = await db
      .select()
      .from(release)
      .where(eq(release.version, version))
      .limit(1)
      .all();
    if (result === undefined || result.length !== 1) {
      return c.notFound();
    }

    const bucket = c.env.MY_BUCKET;
    const buecketObject = await bucket.get(`${version}/WpfAndCloudflare.exe`);
    if (buecketObject === null) {
      return c.notFound();
    }

    c.header("Content-Type", "application/octet-stream");
    c.header("etag", `"${buecketObject.etag}"`);
    c.header("Content-Disposition", "attachment; filename=WpfAndCloudflare.exe");
    c.status(200);

    return c.body(buecketObject.body);
  } catch (e) {
    return c.notFound();
  }
});

export default app;
```
D1へレコードが存在する且つ、R2へオブジェクトが存在するときのみダウンロードが可能になるつくりとなっています。

最終的な見た目はこのようになります。

![](/images/gigoo_wpf_to_cf/img2.png)


## おわりに

windows-latestを指定した際のdefault shellがpwshになるということに気が付かず、tag名を取得するステップで結構ハマってしましました。
