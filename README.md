[[简体中文]](https://github.com/jellyterra/bananapi-f3-archlinux/blob/master/README_zh.md)

# Banana Pi F3 ArchLinux
Arch Linux RISC-V image `rootfs.ext4` for Banana Pi F3 with SpacemiT X60/K1.

Latest post: [https://jellyterra.com/blog/202410250930](https://jellyterra.com/blog/202410250930)

### NOTE: I've packed the [ROM package](https://github.com/jellyterra/bpi-f3-archlinux/releases) with

- `/boot/bianbu.bmp` is the boot splash image of ArchLinux
- `/etc/resolve.conf` with `nameserver 1.1.1.1`
- `/etc/systemd/network/0-end.network` with DHCP configuration set up

# Make the image by yourself

> [!TIP]
>
> The image can be found in the ROM package I made.

[Felix Yan's wiki](https://wiki.felixc.at/linux_risc-v) about the ArchLinux RISC-V porting.

## Get ArchLinux

Download the the ArchLinux RISC-V porting rootfs [here](https://archriscv.felixc.at/images/).

## Make an Ext4 image

```shell
# Allocate image file.
dd if=/dev/zero of=rootfs.ext4 bs=1G count=14

# Map the file to device.
losetup /dev/loop0 rootfs.ext4

# Make Ext4 file system on the image.
mkfs.ext4 /dev/loop0

# Mount the mapped image.
mount /dev/loop0 /mnt

# Copy rootfs.
cp -rp .../archriscv/* /mnt/

# Unmount
umount /mnt
losetup -d /dev/loop0
```

## Replace boot splash image (optional)

```shell
# Mount
losetup /dev/loop0 .../bootfs.ext4
mount /dev/loop0 /mnt

# Replace
cp .../archlinux.bmp /mnt/bianbu.bmp

# Write back
umount /mnt
losetup -d /dev/loop0
```

## Modify the ROM package

Download the Bianbu distro ROM [here](https://archive.spacemit.com/image/k1/version/bianbu/).

It looks like:
```
.
├── bootfs.ext4
├── env.bin
├── factory
│   ├── bootinfo_emmc.bin
│   ├── bootinfo_sd.bin
│   ├── bootinfo_spinand.bin
│   ├── bootinfo_spinor.bin
│   └── FSBL.bin
├── fastboot.yaml
├── fw_dynamic.itb
├── genimage.cfg
├── partition_2M.json
├── partition_flash.json
├── partition_universal.json
├── rootfs.ext4
│     ^ The image to be replaced.
└── u-boot.itb
```

Replace the `rootfs.ext4` with the image you made.

# Flashing to eMMC

The SoC will load one program, for processing packets from the host about firmware flashing.

The flashing protocol is based on **fastboot**.
You may have heard of it on Android.

Requires `fastboot` CLI utility.

### Flash tool

Documentation about flashing [Bianbu](https://bianbu.spacemit.com/en/installation_and_upgrade).
There is a [bundle of scripts](https://archive.spacemit.com/image/k1/flash-all.zip) for flashing via **fastboot** CLI utility.

But `flashserver` supports compression, which makes the transmission faster than purely using **fastboot**.

> [!TIP]
>
> The flash tool can be found in the ROM package I made.

Download [flash tool from SpacemiT](https://developer.spacemit.com/documentation?token=B9JCwRM7RiBapHku6NfcPCstnqh).

Extract the AppImage with option `--appimage-extract`.

Copy `squashfs-root/resources/app/flashserver` to one of the `$PATH`.

## Connect to the board

Enter flash mode: press and hold `FDL`, press `RST` on the board.

Connect USB port on the board to your computer. It will show as a **DFU** device, and turn to `U-Boot USB download gadget` while flashing.
```shell
lsusb
```
```
Bus 001 Device 009: ID 361c:1001 U-Boot USB download gadget
```

### Allow unprivileged users to access the board on DFU mode (optional)

Make the users in the group `plugdev` able to access it.

```shell
nano /etc/udev/rules.d/99-spacemit.rules
```

```
SUBSYSTEMS=="usb", ATTR{idVendor}=="361c", ATTR{idProduct}=="1001", GROUP="plugdev", MODE:="0660"
```

## Flash

Run `flashserver` under the ROM directory.

```shell
flashserver
```

```
 _____  _  _                  _____  _              _                 
|_   _|(_)| |_   __ _  _ __  |  ___|| |  __ _  ___ | |__    ___  _ __ 
  | |  | || __| / _` || '_ \ | |_   | | / _` |/ __|| '_ \  / _ \| '__|
  | |  | || |_ | (_| || | | ||  _|  | || (_| |\__ \| | | ||  __/| |   
  |_|  |_| \__| \__,_||_| |_||_|    |_| \__,_||___/|_| |_| \___||_|   
                                                                      
__     __  _     ____   _____  _            _          
\ \   / / / |   |___ \ |___ / | |__    ___ | |_   __ _ 
 \ \ / /  | |     __) |  |_ \ | '_ \  / _ \| __| / _` |
  \ V /   | | _  / __/  ___) || |_) ||  __/| |_ | (_| |
   \_/    |_|(_)|_____||____/ |_.__/  \___| \__| \__,_|
                                                       


--- Available ports:
---  1: 1-7-2                'fastboot VID:PID=0x361c:0x1001 SER=None'
--- Enter port index or full name: 1
---》开始刷机
执行 fastboot getvar version-brom
Variable not implemented
刷机总进度: 2%:  ▓
刷机总进度: 4%:  ▓▓
刷机总进度: 6%:  ▓▓▓
刷机总进度: 9%:  ▓▓▓▓
刷机总进度: 11%:  ▓▓▓▓▓
执行 fastboot getvar mtd-size
NULL
刷机总进度: 13%:  ▓▓▓▓▓▓
执行 fastboot getvar blk-size
universal
刷机总进度: 15%:  ▓▓▓▓▓▓▓
开始匹配分区表: ['partition_{size0}.json', 'partition_{size1}.json']
匹配原始分区表: partition_universal.json
分区表匹配结果为: partition_universal.json
执行 fastboot flash gpt partition_universal.json
max-download-size:  268435456

parse gpt/mtd table okay
执行 fastboot flash bootinfo bootinfo_sd.bin
max-download-size:  268435456


执行 fastboot flash fsbl FSBL.bin
max-download-size:  268435456


执行 fastboot flash env env.bin
max-download-size:  268435456


执行 fastboot flash opensbi fw_dynamic.itb
max-download-size:  268435456


执行 fastboot flash uboot u-boot.itb
max-download-size:  268435456


执行 fastboot flash bootfs bootfs.ext4
max-download-size:  268435456
start split gzip file in 2024-10-22 04:17:52
使用gzip压缩等级为: 5
切片缓存的数量:  1
split gzip file over, begin download in 2024-10-22 04:17:54


刷机总进度: 17%:  ▓▓▓▓▓▓▓▓
download over in  2024-10-22 04:18:02
执行 fastboot flash rootfs rootfs.ext4
max-download-size:  268435456
start split gzip file in 2024-10-22 04:18:02
使用gzip压缩等级为: 5
切片缓存的数量:  1
split gzip file over, begin download in 2024-10-22 04:18:04

切片缓存的数量:  1
切片缓存的数量:  2
切片缓存的数量:  3

刷机总进度: 18%:  ▓▓▓▓▓▓▓▓▓
download over in  2024-10-22 04:18:11
split gzip file over, begin download in 2024-10-22 04:18:11
切片缓存的数量:  3

... ... ...

刷机总进度: 97%:  ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓
download over in  2024-10-22 04:24:03
split gzip file over, begin download in 2024-10-22 04:24:03


刷机总进度: 99%:  ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓
download over in  2024-10-22 04:24:12
split gzip file over, begin download in 2024-10-22 04:24:12


download over in  2024-10-22 04:24:23
---》刷机成功!
<-------------flash over ---------------->
```

# Enjoy!

Reset BPi-F3.
ArchLinux will boot!

By [Felix Yan](https://archriscv.felixc.at/), the password of `root` is `archriscv`.

## Start services

### Network

```shell
systemctl enable --now systemd-networkd
```

### Time Sync

```shell
systemctl enable --now systemd-timesyncd
```

> [!NOTE]
>
> SSL connections only work with correct time.

## Install packages

```shell
pacman -Syu
pacman -S nano openssh
```

### Choose the best mirror
```shell
nano /etc/pacman.d/mirrorlist
```

```
##
## Arch Linux RISC-V repository mirrorlist
##

## Worldwide
Server = https://riscv.mirror.pkgbuild.com/repo/$repo

## Canada
#Server = https://archriscv.felixc.at/repo/$repo

## China
#Server = https://mirror.iscas.ac.cn/archriscv/repo/$repo
#Server = https://mirrors.sustech.edu.cn/archriscv/repo/$repo
#Server = https://mirror.nju.edu.cn/archriscv/repo/$repo
#Server = https://mirrors.wsyu.edu.cn/archriscv/repo/$repo

## Finland
#Server = https://mirrors.felixc.at/archriscv/repo/$repo
```

## Desktop Environment

### KDE

```shell
pacman -S icu plasma-desktop plasma-nm plasma-pa plasma-workspace plasma-workspace-wallpapers xdg-desktop-portal-kde dolphin kate konsole okular spectacle
```

```shell
dbus-run-session startplasma-wayland
```

### Xfce4

```shell
pacman -S icu xfce4 xorg
```

```shell
dbus-run-session startxfce4
```

### GNOME
Unavailable because `libmutter` was missing.

# FAQ

## About the graphics hardware acceleration

The SoC integrated [Imagination BXE-2-32](https://www.imaginationtech.com/product/img-bxe-2-32/) **without opensource driver yet**.

The Chinese capital *Canyon Bridge Capital* has been the current parent company of *Imagination* since 2017.

The alternative is the software implementation `llvmpipe`.

## About the media hardware codec

Spacemit has designed [their own media API](https://bianbu-linux.spacemit.com/development_guide/media/) about their codec.
They called it *Multi Processing Platform (MPP)*.

They supported this media API in *FFmpeg* and *Gstreamer* on their distro *Bianbu*.
