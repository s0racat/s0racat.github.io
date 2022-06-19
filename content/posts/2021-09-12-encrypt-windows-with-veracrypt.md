+++
title = "Windows 10 homeでCドライブを暗号化"
date = "2021-09-12"
author = "soracqt"
description = "画像多数注意"
tags = ["security", "encrypt", "veracrypt", "windows"]
+++

Windows 10 proはbitlockerを使ってCドライブを暗号化できるのですが、Windows 10 homeはできません

なのでVeraCryptを使ってCドライブを暗号化する方法を書きます。bitlockerを使いたくない人にも良いかもしれません

# インストール

なんらかの方法でVeraCryptを使える状態にします

__Portable版ではsystem disk暗号化機能が使えません__

# 構成

ここをクリック

![](/img/2021-09-12-encrypt-windows-with-veracrypt/screenshot_001.jpg)

![](/img/2021-09-12-encrypt-windows-with-veracrypt/screenshot_002.jpg)

![](/img/2021-09-12-encrypt-windows-with-veracrypt/screenshot_003.jpg)

## シングルブートかマルチブートか

![](/img/2021-09-12-encrypt-windows-with-veracrypt/screenshot_004.jpg)

## 初心者にはmulti-boot configurationはよくないよ

続けますか？

![](/img/2021-09-12-encrypt-windows-with-veracrypt/screenshot_005.jpg)

## 暗号化オプション

![](/img/2021-09-12-encrypt-windows-with-veracrypt/screenshot_006.jpg)

## パスワード

![](/img/2021-09-12-encrypt-windows-with-veracrypt/screenshot_007.jpg)

## 短かいパスワード警告

20字以上のパスワードをおすすめするよ

それでも続けますか？

![](/img/2021-09-12-encrypt-windows-with-veracrypt/screenshot_008.jpg)

## ランダムデータ収集

バーがフルになるまでやった

![](/img/2021-09-12-encrypt-windows-with-veracrypt/screenshot_009.jpg)

![](/img/2021-09-12-encrypt-windows-with-veracrypt/screenshot_010.jpg)

## レスキューディスク

Skip Rescue Disk verificationにチェックすると物理メディアを作成しなくて済む

![](/img/2021-09-12-encrypt-windows-with-veracrypt/screenshot_011.jpg)

![](/img/2021-09-12-encrypt-windows-with-veracrypt/screenshot_012.jpg)

## 前にレスキューディスクを作成していたとしても前のレスキューディスクは使えないよ

![](/img/2021-09-12-encrypt-windows-with-veracrypt/screenshot_013.jpg)

## 何回データ上書きするか

なんとなく3passにした

1回くらいはするべきでは、と思う

![](/img/2021-09-12-encrypt-windows-with-veracrypt/screenshot_014.jpg)

![](/img/2021-09-12-encrypt-windows-with-veracrypt/screenshot_015.jpg)

## 正常にアンロックできるかテスト

![](/img/2021-09-12-encrypt-windows-with-veracrypt/screenshot_016.jpg)

![](/img/2021-09-12-encrypt-windows-with-veracrypt/screenshot_017.jpg)

![](/img/2021-09-12-encrypt-windows-with-veracrypt/screenshot_018.jpg)

![](/img/2021-09-12-encrypt-windows-with-veracrypt/screenshot_019.jpg)

![](/img/2021-09-12-encrypt-windows-with-veracrypt/screenshot_020.jpg)

## 暗号化中...

![](/img/2021-09-12-encrypt-windows-with-veracrypt/screenshot_021.jpg)

## 終了

![](/img/2021-09-12-encrypt-windows-with-veracrypt/screenshot_022.jpg)
