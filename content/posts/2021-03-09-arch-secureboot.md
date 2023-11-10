+++
title = "archlinuxでセキュアブート(簡易版)"
date = "2021-03-09"
author = "soracat"
tags = ["archlinux", "secureboot", "bootloader", "efi"]
+++


# grubの場合

bootパーティションが/bootにマウントされていてefiパーティションが/boot/efiにマウントされている場合の例です

```bash
yay -S shim-signed
```

```bash
sudo cp /usr/share/shim-signed/shimx64.efi /boot/efi/EFI/arch/BOOTX64.efi
sudo cp /usr/share/shim-signed/mmx64.efi /boot/efi/EFI/arch/
```

```bash
sudo efibootmgr --verbose --disk /dev/nvme0n1 --part 1 --create --label "Shim" --loader "\EFI\arch\BOOTX64.efi"
```

あとはBOOTX64.efiを起動してgrubx64.efiのhashを登録する

# systemd-bootの場合

/bootをefiパーティションとして使っている場合,主にsystemd-boot

```bash
yay -S preloader-signed
```

```bash
sudo cp /usr/share/preloader-signed/{PreLoader,HashTool}.efi /boot/EFI/systemd
sudo mv /boot/EFI/systemd/systemd-bootx64.efi /boot/EFI/systemd/loader.efi
```

```bash
cd ~
sudo efibootmgr --verbose --disk /dev/nvme0n1 --part 1 --create --label "PreLoader" --loader "\EFI\systemd\PreLoader.efi"
```

どちらもカーネルとefiバイナリのhashを登録しないと起動しません

ここでハマりました

https://wiki.archlinux.jp/index.php/%E3%82%BB%E3%82%AD%E3%83%A5%E3%82%A2%E3%83%96%E3%83%BC%E3%83%88
