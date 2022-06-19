+++
title = "Intel GVT-g"
date = "2021-05-14"
author = "soracqt"
tags = ["intel", "linux", "qemu", "pci-passthrough"]
+++

# 前提条件

linux >= 4.16

qemu >= 2.12

# 設定

`/etc/modprobe.d/i915.conf`

```bash
options i915 ... enable_gvt=1 enable_guc=0 ...
```

`/etc/modules-load.d/intel_gvt-g.conf`

```bash
kvmgt
vfio-iommu-type1
vfio-mdev
```

`/etc/default/grub`

```bash
GRUB_CMDLINE_LINUX="... intel_iommu=on ..."
```

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

```bash
sudo reboot
```

```bash
lspci -Dnn|grep VGA
```

```bash
GVT_PCI=0000\:00\:02.0
ls /sys/bus/pci/devices/$GVT_PCI/mdev_supported_types/
```

```bash
GVT_GUID=$(uuidgen)
GVT_PCI=0000\:00\:02.0
GVT_TYPE=i915-GVTg_V5_4
echo "$GVT_GUID"|sudo tee "/sys/bus/pci/devices/$GVT_PCI/mdev_supported_types/$GVT_TYPE/create"
```

```bash
qemu-system-x86_64 \
  -device vfio-pci,sysfsdev=/sys/bus/mdev/devices/$GVT_GUID
```

# 参考

https://kagasu.hatenablog.com/entry/2021/01/05/201126

https://wiki.archlinux.org/title/Intel_GVT-g

