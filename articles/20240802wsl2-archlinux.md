---
title: "WSLでArchLinuxを動かす"
emoji: "🦒"
type: "tech"
topics: [ "wsl", "archlinux" ]
published_at: 2024-08-02 18:00
published: true
---
## 環境
OS 名: Microsoft Windows 11 Pro
バージョン: 10.0.26120 ビルド 26120

```shell
sudo apt-get update
sudo apt-get upgrade
```
だけ実行したUbuntu24.04 LTS

## 導入
Ubuntuの中に入って、 https://ftp.tsukuba.wide.ad.jp/Linux/archlinux/iso/ から好きなArchLinuxを取ってきます。
```shell
wget https://ftp.tsukuba.wide.ad.jp/Linux/archlinux/iso/2024.08.01/archlinux-bootstrap-2024.08.01-x86_64.tar.zst
wget https://ftp.tsukuba.wide.ad.jp/Linux/archlinux/iso/2024.08.01/sha256sums.txt
```

Ubuntuにはzstdが初期インストールされていないため、別途インストールした後、tarコマンドで解凍します。

```shell
sudo apt install zstd
sudo tar --zstd -xvf archlinux-bootstrap-2024.08.01-x86_64.tar.zst
```

次にpacmanのmirrorlistを編集して、利用したいミラーサーバーのコメントアウトを外します。

私はArch導入後にreflectorコマンドでmirrorlistを更新するから適当でいいや、という安易な考えで、AustraliaとJapanのコメントアウトを全部外しました。
```shell
cd root.x86_64/
sudo vim etc/pacman.d/mirrorlist
```

tar.gzで圧縮して、
```shell
sudo tar czvf arch.tar.gz *

mkdir -p /mnt/c/wsltars
sudo mv arch.tar.gz /mnt/c/wsltars
```

cmd.exeかpowershellからインポートすると完了です。
```powershell
PS C:\> wsl --import ArchLinux C:\arch\ C:\wsltars\arch.tar.gz
```

`wsl --list`を実行することで導入されていることを確認できます。
```powershell
PS C:\> wsl --list
Linux 用 Windows サブシステム ディストリビューション:
ArchLinux
```

`wsl -d Name`でディストリビューションを選択して起動することができます。
```powershell
PS C:\> wsl -d ArchLinux
[root@HOSTNAME USERNAME ~]#
```

## どこかで間違えてよくわからなくなったら
```powershell
PS C:\> wsl --shutdown
PS C:\> wsl --unregister ArchLinux 
```

で削除することができます。
そして再度`wsl --import`を実行することで真っ新な環境に戻すことができます。

## おわりに
初期設定編も書くかも。書かないかも。


## 参考
[サクッとUbuntu (WSL)でzstd(ZStandard)を使う方法 #Linux - Qiita](https://qiita.com/tatsubey/items/7ec828d72659aeae8c34)
[Arch on WSLを構築する](https://zenn.dev/kyoh86/articles/4bf6513aabe517)
[WSL 2 で Arch Linux を使う - yukirii blog](https://blog.yukirii.dev/wsl2-arch-linux/)
