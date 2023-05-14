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
driver=nl80211

ssid=raspiap
# Whether to Broadcast ssid, 0 = off,1 = stealth ssid
ignore_broadcast_ssid=0
hw_mode=a
channel=36
ieee80211ac=1
# MAC Address access control, 0 = off
macaddr_acl=0
# WMM(Wi-Fi Multimedia)
wmm_enabled=1

# https://en.wikipedia.org/wiki/IEEE_802.11d-2001
ieee80211d=1
# https://en.wikipedia.org/wiki/IEEE_802.11h-2003
ieee80211h=1
# https://qiita.com/JhonnyBravo/items/5df2d9b2fcb142b6a67c
local_pwr_constraint=3
spectrum_mgmt_required=1

# Use WPA
auth_algs=1
wpa_key_mgmt=WPA-PSK
# Use WPA2
wpa=2
# Use CCMP
wpa_pairwise=CCMP
# passphrase
wpa_passphrase=passphrase' | sudo tee /etc/hostapd/hostapd.conf
```

設定項目はあまり理解してませんが`wpa_passphrase`項目を変えると接続時に求められるパスワードを変更できます。

# dnscrypt-proxyの設定(オプション)

```bash
sudo cp /etc/dnscrypt-proxy/dnscrypt-proxy.toml /etc/dnscrypt-proxy/dnscrypt-proxy.toml.bak
```

`/etc/dnscrypt-proxy/dnscrypt-proxy.toml`

```diff
...
+listen_addresses = ['127.0.0.1:50000']
...
+cache = false
...
```

listen_addressのポートを50000番に変えてます。多分53ポートはバッティングすると思うので

# dnsmasqの設定

dnsmasqはdhcp serverとlocal dns cache serverとして動いてくれます。

```bash
sudo cp /etc/dnsmasq.conf /etc/dnsmasq.conf.bak

echo "# listen interface
interface=uap0
# On systems which support it, dnsmasq binds the wildcard address,
# even when it is listening on only some interfaces. It then discards
# requests that it shouldn't reply to. This has the advantage of
# working even when interfaces come and go and change address. If you
# want dnsmasq to really bind only the interfaces it is listening on,
# uncomment this option. About the only time you may need this is when
# running another nameserver on the same machine.
bind-interfaces
# dhcp range 2~20, 12hrs to update dhcp lease
dhcp-range=192.168.50.2,192.168.50.20,255.255.255.0,12h

# Never forward plain names (without a dot or domain part)
domain-needed
# Never forward addresses in the non-routed address spaces.
bogus-priv
# upstream dns server
server=127.0.0.1#50000
# don't read /etc/resolv.conf
no-resolv
# listen for dns server on 192.168.50.1
listen-address=192.168.50.1
# dns cache size
cache-size=1000

conf-dir=/etc/dnsmasq.d/,*.conf" | sudo tee /etc/dnsmasq.conf
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
...
+DEFAULT_FORWARD_POLICY=“ACCEPT"
...
```

```bash
sudo cp /etc/ufw/before.rules /etc/ufw/before.rules.bak
```

`/etc/ufw/before.rules`

```bash
...
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

https://www.itmedia.co.jp/news/spv/2008/14/news042.html

https://github.com/imp/dnsmasq/blob/master/dnsmasq.conf.example
