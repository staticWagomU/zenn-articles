---
title: "NixOSをミニマルインストールしてswayを動かす"
emoji: "🦒"
type: "tech"
topics: [ "nix", "nixos", "sway" ]
published_at: 2024-09-06 18:00
published: true
---

## はじめに

タイトルのとおりですが、[ここ](https://nixos.org/download/)からダウンロードしたNixOSのMinimal ISO Imageを使ってNixOSをインストールして、swayを動かすまでの手順を記した記事です。

USBのブートイメージの作成方法は他のディストリビューションと同様のため割愛しています。

## 手順


![](/images/20240906_nixos_and_sway/2024-09-05_08-21-39.png)

まずは`sudo -i`でrootユーザーにログインします。

![](/images/20240906_nixos_and_sway/2024-09-05_08-23-10.png)
 こちら[^1]の記事を参考にgdiskコマンドを使ってパーティションを分割します。
 
![](/images/20240906_nixos_and_sway/2024-09-06_11-12-49.png)

次にNixOSのマニュアル[^2]を参考に、Ext4パーティション、スワップパーティション、bootパーティションを作成します。

```shell
mkfs.fat -F 32 -n BOOT /dev/nvme0n1p1
mkfs.ext4 -L nixos /dev/nvme0n1p2
mkswap /dev/nvme0n1p3
```

![](/images/20240906_nixos_and_sway/2024-09-06_11-29-09.png)

最後に、`/mnt`へマウントとNixOSの設定ファイルを生成して[^3]下準備は完了です。

```shell
mount /dev/nvme0n1p2 /mnt
mkdir -p /mnt/boot
mount -o umask=077 /dev/nvme0n1p1 /mnt/boot
swapon /dev/nvme0n1p3

nixos-generate-config --root /mnt
```

![](/images/20240906_nixos_and_sway/2024-09-06_11-32-13.png)

ここから`/mnt/etc/nixos/configuration.nix`ファイルを修正していきます。ここが一番の鬼門なのですが、`vim`が入っておらず`nano`で編集する必要があります。マウスが使えないため矢印キーとスペースの連打で頑張って凌いでください。

```nix
# Edit this configuration file to define what should be installed on
# your system. Help is available in the configuration.nix(5) man page, on
# https://search.nixos.org/options and in the NixOS manual (`nixos-help`).

{ config, lib, pkgs, ... }:

{
  imports =
    [ # Include the results of the hardware scan.
      ./hardware-configuration.nix
    ];

  # Use the systemd-boot EFI boot loader.
  boot.loader.systemd-boot.enable = true;
  boot.loader.efi.canTouchEfiVariables = true;

  # Set your time zone.
  time.timeZone = "Asia/Tokyo";

  # Select internationalisation properties.
  i18n.defaultLocale = "ja_JP.UTF-8";
  console = {
    useXkbConfig = true; # use xkb.options in tty.
  };

  # Enable the X11 windowing system.
  services.xserver = {
    xkb = {
      layout = "jp"; # 日本語配列にする
      variant = "";
      options = "terminate:ctrl_alt_bksp";
    };
    enable = true;
    displayManager = {
      gdm.enable = true;
    };
  };

  # swayを自動起動させるための設定
  services.displayManager = {
    defaultSession = "sway";
  };
  # swayを有効化
  programs.sway = {
    enable = true;
    wrapperFeatures.gtk = true;
  };

  # List packages installed in system profile. To search, run:
  # $ nix search wget
  environment.systemPackages = with pkgs; [
    firefox
    gh
    git
    neovim
    vim # Do not forget to add an editor to edit configuration.nix! The Nano editor is also installed by default.
    wget
    # https://nixos.wiki/wiki/Sway 参照
    grim # screenshot functionality
    slurp # screenshot functionality
    wl-clipboard # wl-copy and wl-paste for copy/paste from stdin / stdout
    mako # notification system developed by swaywm maintainer
  ];

  # For more information, see `man configuration.nix` or https://nixos.org/manual/nixos/stable/options#opt-system.stateVersion .
  system.stateVersion = "24.05"; # Did you read the comment?

}
```


設定ファイルを修正し`nixos-install`を実行してください。ミスっているとエラーがでます。基本的にtypoだと思うので、頑張って修正してください。
最後にパスワードの実行を求められるので、rootのパスワードを設定して`reboot`で再起動してください。reboot前は日本語配列が適用されていないので、パスワードに記号を含める際には気を付けて入力してください。

再起動で画面が暗くなったタイミングでUSBメモリを抜いておきましょう。

![](/images/20240906_nixos_and_sway/2024-09-06_12-17-42.png)

![](/images/20240906_nixos_and_sway/2024-09-06_12-18-03.png)

GUIのログイン画面が表示されると成功です！！！

![](/images/20240906_nixos_and_sway/2024-09-06_12-18-20.png)

![](/images/20240906_nixos_and_sway/2024-09-06_12-18-41.png)


## おわりに

あとはhome-managerを入れるなり、flakeを入れるなり好きにしてください。
ユーザーも作ってないミニマルな設定なので、まずはユーザー作ってくださいね。

## 参考

[^1]:https://qiita.com/hkaomua/items/75bef1cb25c6d63306f4
[^2]:https://nixos.org/manual/nixos/stable/index.html#sec-installation-manual-partitioning-formatting
[^3]:https://nixos.org/manual/nixos/stable/index.html#sec-installation-manual-installing

https://nixos.wiki/wiki/Sway
