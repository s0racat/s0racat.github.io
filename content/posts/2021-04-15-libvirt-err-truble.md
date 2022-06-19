+++
title = "libvirtdのエラー解決方法メモ"
date = "2021-04-15"
author = "soracqt"
tags = ["libvirt", "network", "memo", "linux"]
+++

_Internal error:Failed to initialize a valid firewall backend_

というエラーで仮想NATが起動しませんでした

前までは`ebtables`を入れることで治りましたが

最近公式リポジトリから`ebtables`が消滅したので`iptables+nft`を入れてください

```bash
sudo pacman -S --needed iptables-nft dnsmasq
```

libvirtdを再起動してください

```bash
sudo systemctl restart libvirtd
```

https://superuser.com/questions/1063240/libvirt-failed-to-initialize-a-valid-firewall-backend
