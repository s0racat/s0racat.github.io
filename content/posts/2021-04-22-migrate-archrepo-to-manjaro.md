+++
title = "archlinuxのリポジトリををmanjaroのリポジトリに切り替える"
date = "2021-04-22"
author = "soracqt"
tags = ["manjaro", "archlinux", "pacman"]
+++

# mirrorlistをバックアップ

```bash
mkdir ~/tmp
cd ~/tmp
cp /etc/pacman.d/mirrorlist .
```

# キャッシュを削除

```bash
yes|sudo pacman -Scc
```

# 必要パッケージを適当なミラーから落としてインストール

```bash
wget http://ftp.riken.jp/Linux/manjaro/stable/community/x86_64/python-npyscreen-4.10.5-5-any.pkg.tar.zst
wget http://ftp.riken.jp/Linux/manjaro/stable/core/x86_64/pacman-mirrors-4.21.0-1-any.pkg.tar.zst
wget http://ftp.riken.jp/Linux/manjaro/stable/core/x86_64/manjaro-keyring-20201216-1-any.pkg.tar.zst
sudo pacman -U *.pkg.tar.zst
```

バージョンが更新されると404になるので適当にurl置き換えてください

# manjaroのミラーリストを生成

```bash
sudo pacman-mirrors --country Japan && sudo pacman -Syyu
```

# パッケージのダウングレード

```bash
sudo pacman -Qnq|sudo pacman -S - --needed
```

適当にインストール済みのパッケージ(linux,linux-lts)をmanjaroリポジトリに変えてください

# 参考リンク

https://blog.fascode.net/2021/01/20/archlinux-move-to-manjaro/

ありがとうございました
