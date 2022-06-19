+++
title = "Flatpakアプリケーションでのフォント変更方法"
date = "2021-03-17"
author = "soracqt"
tags = ["flatpak", "linux", "fontconfig"]
+++

fontconfig(https://wiki.archlinux.org/index.php/Font_configuration)の設定ファイルをコピーするだけです

```bash
~/.var/app/${Application ID}/config/fontconfig/ # ~/.var/app/us.zoom.Zoom/config/fontconfig/
```

Application IDは`flatpak list`で確認できます

私はnoto-fonts-cjkの設定ファイルをコピーしてみました

Symlinkするべきかも

archlinux,font: noto-fonts-cjkの場合は`/usr/share/fontconfig/conf.avail/70-noto-cjk.conf`にあります
