+++
title = "archlinuxでセキュアブート with grub"
date = "2022-12-12"
author = "soracqt"
tags = ["archlinux", "secureboot", "bootloader", "efi"]
+++

[2021-03-09-arch-secureboot](/posts/2021-03-09-arch-secureboot/)

こっちが古くなってたので

## インストール

```bash
yay -S shim-signed mokutil sbsigntools
```

## 鍵作成

```bash
mkdir -p /var/lib/shim-signed
cd !$
openssl req -newkey rsa:4096 -noenc -keyout MOK.key -new -x509 -sha256 -days 3650 -subj "/CN=my Machine Owner Key/" -out MOK.crt
openssl x509 -outform DER -in MOK.crt -out MOK.cer # MokManagerにenrollするのにDERフォーマットな証明書が必要
```

## grubのstandalone版を生成

```bash
grub-mkconfig -o /tmp/grub.cfg
grub-mkstandalone -O x86_64-efi --sbat=/usr/share/grub/sbat.csv -o /boot/efi/EFI/arch/grubx64.efi "/boot/grub/grub.cfg=/tmp/grub.cfg"
```

## さっきの鍵で起動したいファイルに署名

```bash
sbsign --key MOK.key --cert MOK.crt --output /boot/vmlinuz-linux /boot/vmlinuz-linux
sbsign --key MOK.key --cert MOK.crt --output /boot/efi/EFI/arch/grubx64.efi /boot/efi/EFI/arch/grubx64.efi
```

## shimとMokManagerのefiバイナリをコピー && NVRAMにエントリ作成

```bash
cp /usr/share/shim-signed/shimx64.efi /boot/efi/EFI/arch/BOOTx64.EFI
cp /usr/share/shim-signed/mmx64.efi /boot/efi/EFI/arch/
cp /var/lib/shim-signed/MOK.cer /boot/efi/EFI/arch/
efibootmgr --unicode --disk /dev/nvme0n1 --part 1 --create --label "Shim" --loader "\EFI\arch\BOOTx64.EFI"
```

それぞれのUEFIの設定を変更するなどしてShimを起動 > Enroll key from disk > Mok.cerを選択 > Continue > Yes > Continue Boot

## MokManagerにenrollした.cerを削除

```bash
mokutil --delete /var/lib/shim-signed/MOK.cer
```

## アップグレードされたカーネルに署名を自動化

https://wiki.archlinux.org/title/Secure_Boot#shim_with_key

## 参考

https://wiki.archlinux.org/title/Secure_Boot#shim
