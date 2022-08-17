+++
title = "Linux環境でのパフォーマンス向上と省電力化、発熱防止"
date = "2022-08-17"
author = "soracqt"
tags = ["power-management", "linux"]
+++

# - [Thermald](https://github.com/intel/thermal_daemon)

Intel製のやつ

CPUの周波数はコントロールしてくれない

正直効果はあまり...な気がする

デフォルトでは、利用可能なCPUデジタル温度センサーを使用してCPU温度を監視し、ハードウェアが積極的な修正アクションを取る前に、CPU温度を制御下に維持します。thermal sysfsに皮膚温度センサーがある場合、皮膚温度を45℃以下に維持しようとします。

引用: [archwiki](https://wiki.archlinux.org/title/CPU_frequency_scaling#thermald)より

# - [cpupower](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/tools/power/cpupower)

論理的にCPU周波数コントロールができる

これは効果がわかりやすいと思う

例えば`/etc/default/cpupower`に

```bash
min_freq="800MHz"
max_freq="1.8GHz"
```

と書いてcpupowerのデーモンを有効化しておくと最小周波数を800MHz、最大周波数を1.8GHzに固定してくれる

# - [TLP](https://github.com/linrunner/TLP)

言わずと知れた省電力化デーモン

ラップトップの場合、。バッテリーが充電状態かしていないかで良い感じ切り替えをやってくれる

[powertop](https://github.com/fenrus75/powertop)でどの項目を省電力化したのかが見れる

# - [ananicy-cpp](https://gitlab.com/ananicy-cpp/ananicy-cpp)

元の[Ananicy](https://github.com/Nefelim4ag/Ananicy)をC++で書き直した物

プロセスの優先度をいじってるらしいです。

I just wanted a tool for auto set programs nice in my system, i.e.:

    Why do I get lag, while compiling kernel and playing games?
    Why does dropbox client eat all my IO?
    Why does torrent/dc client make my laptop run slower?

引用: [README.md](https://github.com/Nefelim4ag/Ananicy/blob/master/README.md)

マルチタスク中にメインのタスクが使い物にならなかったので作ったらしいです。

**CPUの発熱が少なくなります。マジで**

AURから入れるには`ananicy-cpp`と`ananicy-rules-git`を入れるのみです。
