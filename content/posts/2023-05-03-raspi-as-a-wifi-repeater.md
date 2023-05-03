+++
title = "RaspberryPi 3b+をWiFiリピーターとして動かす(イーサネット、WiFiドングル不要)"
date = "2023-05-03"
author = "soracat"
tags = ["raspberrypi", "linux","access point"]
+++

# 環境

ArchLinux ARM aarch64

systemd-networkd + dnsmasq + hostapd + ufw + (dnscrypt-proxy) 

```bash
+--------+         +- RPi -------------------+
| router |     (((-+ 192.168.xxx.xxx(dhcp)   |              +-Laptop-------------+
+--------+   wlan0 |     WLAN AP             |              | WLAN Client        |
                   |    192.168.50.1(static) +-)))      (((-+ 192.168.50.x(dhcp) |
                   +-------------------------+ uap0         |                    |
                                                            +--------------------+
```

# インストール

```bash
sudo pacman -S dnsmasq hostapd dnscrypt-proxy ufw --needed
```

dnscrypt-proxyはオプション。

# uap0インターフェースを作成

```bash
echo 'ACTION=="add", SUBSYSTEM=="ieee80211", KERNEL=="phy0", \
    RUN+="/sbin/iw phy %k interface add uap0 type __ap"' | sudo tee /etc/udev/rules.d/90-uap0.rules
```

再起動するとuap0インターフェースが作成される

# uap0インターフェースのIPアドレスを固定

```bash
echo '[Match]
Name=uap0

[Network]
Address=192.168.50.1/24' | sudo tee /etc/systemd/network/80-wifi-ap.network
```

# hostapdの設定

```bash
sudo cp /etc/hostapd/hostapd.conf /etc/hostapd/hostapd.conf.bak

echo 'country_code=JP
interface=uap0
ssid=raspiap
hw_mode=g
channel=7
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=passphrase
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP' | sudo tee /etc/hostapd/hostapd.conf
```

設定項目はあまり理解してませんが`wpa_passphrase`項目を変えると接続時に求められるパスワードを変更できます。

# dnscrypt-proxyの設定(オプション)

```bash
sudo cp /etc/dnscrypt-proxy/dnscrypt-proxy.toml /etc/dnscrypt-proxy/dnscrypt-proxy.toml.bak
```

`/etc/dnscrypt-proxy/dnscrypt-proxy.toml`

```diff
+listen_addresses = ['127.0.0.1:50000']
```

listen_addressのポートを50000番に変えてます。多分53ポートはバッティングすると思うので

# dnsmasqの設定

dnsmasqはdhcp serverとlocal dns cache serverとして動いてくれます。

```bash
sudo cp /etc/dnsmasq.conf /etc/dnsmasq.conf.bak

echo 'interface=lo,uap0
bind-interfaces
domain-needed
bogus-priv
dhcp-range=192.168.50.2,192.168.50.20,255.255.255.0,12h
server=127.0.0.1#50000
no-resolv
listen-address=192.168.50.1
proxy-dnssec
cache-size=1000' | sudo tee /etc/dnsmasq.conf
```

`server`オプションは上流DNSキャッシュサーバーを指定できます。

この例ではさっきセットアップしたdnscrypt-proxyのアドレスとポートを指定しています。

# IPマスカレードを有効化

```bash
echo 'net.ipv4.ip_forward=1' | sudo tee /etc/sysctl.d/routed-ap.conf
```

## ufw

iptablesを直接弄るのはやりたくないのでufw経由でiptablesを設定します。

```bash
sudo cp /etc/default/ufw /etc/default/ufw.bak
```

`/etc/default/ufw`

```diff
+DEFAULT_FORWARD_POLICY=“ACCEPT"
```

```bash
sudo cp /etc/ufw/before.rules /etc/ufw/before.rules.bak
```

`/etc/ufw/before.rules`

```bash
*nat
-F
-A POSTROUTING -s 192.168.50.0/24 ! -d 192.168.50.0/24 -j MASQUERADE
COMMIT

...
# Don't delete these required lines, otherwise there will be errors
*filter
...
```

*filterの前に書きます。

source ipが192.168.50.0/24でdestination ipが192.168.50.0/24でないパケットをIPマスカレードするルールです

# systemdで起動時有効化

```bash
sudo systemctl enable ufw dnscrypt-proxy dnsmasq hostapd --now
```

# ufw許可

```bash
sudo ufw allow dns
sudo ufw allow 67/udp
```

# 再起動

```bash
sudo systemctl reboot
```

# 参考元

https://www.raspberrypi.com/documentation/computers/configuration.html#setting-up-a-routed-wireless-access-point

https://docs.raspap.com/ap-sta/

https://www.mikan-tech.net/entry/raspi-ap-sta-router

https://qiita.com/hoto17296/items/3aa5863ba9ef13400283