+++
title = "Flatpakアプリケーションでのフォント変更方法"
date = "2021-03-17"
author = "soracat"
tags = ["flatpak", "linux", "fontconfig"]
+++

[fontconfig](https://wiki.archlinux.org/index.php/Font_configuration)の設定ファイルをコピーするだけです

```bash
~/.var/app/${Application ID}/config/fontconfig/ # ~/.var/app/us.zoom.Zoom/config/fontconfig/
```

Application IDは`flatpak list`で確認できます

私はnoto-fonts-cjkの設定ファイルをコピーしてみました

Symlinkは元のファイルにアクセス権が無いと反映されないのでコピーしないといけない

```bash
mkdir ~/.var/app/us.zoom.Zoom/config/fontconfig/
cd ~/.var/app/us.zoom.Zoom/config/fontconfig/
cp ../../../../../.config/fontconfig/fonts.conf .
```

archlinux,font: noto-fonts-cjkの場合は`/usr/share/fontconfig/conf.avail/70-noto-cjk.conf`にあります
