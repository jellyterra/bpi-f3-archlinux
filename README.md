[[简体中文]](https://github.com/jellyterra/bananapi-f3-archlinux/blob/master/README_zh.md)

# Banana Pi F3 ArchLinux
Arch Linux RISC-V image `rootfs.ext4` for Banana Pi F3 with SpacemiT X60/K1.

### What have it done?

- Copy drivers
    - /lib/dri
        - pvr_dri.so
        - spacemit_dri.so -> pvr_dri.so
    - /lib/firmware
    - /lib/modules
- Network configuration
    - /etc/systemd/network/default.network

[Felix Yan's wiki about archriscv](https://wiki.felixc.at/linux_risc-v)

# Flash to BPi-F3

## Prepare

### Dependencies

Install `fastboot`, `zstd`.

```sh
$ dnf install    fastboot zstd # For Fedora, CentOS, RHEL etc.
$ pacman -S android-tools zstd # For ArchLinux
```

### Flash tool

Download [flash tool from SpacemiT](https://download.banana-pi.dev/d/ca025d76afd448aabc63/files/?p=%2FTools%2Fimage_download_tools%2Ftitantools_for_linux-latest.zip).

Unzip and run the AppImage with option `--appimage-extract`.

Copy `squashfs-root/resources/app/flashserver` to one of the `$PATH`.

```sh
$ unzip titantools_for_linux-latest.zip
$ chmod +x titantools_for_linux-1.0.23beta.AppImage
$ ./titantools_for_linux-1.0.23beta.AppImage --appimage-extract
$ cp -p ./squashfs-root/resources/app/flashserver ~/.local/bin/
```

### ROM

Download [ROM from Banana Pi](https://drive.google.com/drive/folders/1LQoioz6N5YQpSOxY47OmetnPX4yggtT0): `bianbu-23.10-nas-k1-v1.0rc1-release-20240429192450.zip`

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
├── rootfs.ext4 <-- To be replaced.
└── u-boot.itb
```

## Connect to the board

Enter flash mode: press and hold `FDL`, press `RST` on the board.

Connect USB port on the board to your computer. It will show as a **DFU** device, and turn to `U-Boot USB download gadget` while flashing.
```
$ lsusb
Bus 001 Device 009: ID 361c:1001 U-Boot USB download gadget
```

## Flash

Download the ArchLinux rootfs image can be found in [releases](https://github.com/jellyterra/bananapi-f3-archlinux/releases).

```sh
$ zstd -d archriscv-2024-03-30.img.zst    # Decompress
$ cp archriscv-2024-03-30.img rootfs.ext4 # Replace rootfs.ext4
```

Run `flashserver` under the ROM directory.

```
$ flashserver
 _____  _  _                  _____  _              _                 
|_   _|(_)| |_   __ _  _ __  |  ___|| |  __ _  ___ | |__    ___  _ __ 
  | |  | || __| / _` || '_ \ | |_   | | / _` |/ __|| '_ \  / _ \| '__|
  | |  | || |_ | (_| || | | ||  _|  | || (_| |\__ \| | | ||  __/| |   
  |_|  |_| \__| \__,_||_| |_||_|    |_| \__,_||___/|_| |_| \___||_|   
                                                                      
__     __  _     _  _____  _            _          
\ \   / / / |   / ||___  || |__    ___ | |_   __ _ 
 \ \ / /  | |   | |   / / | '_ \  / _ \| __| / _` |
  \ V /   | | _ | |  / /  | |_) ||  __/| |_ | (_| |
   \_/    |_|(_)|_| /_/   |_.__/  \___| \__| \__,_|
                                                   

--- Available ports:
---  1: 1-1-3                'fastboot VID:PID=0x361c:0x1001 SER=dfu-device'
--- Enter port index or full name: 1
---》开始刷机
执行 fastboot getvar version-brom
1.0
刷机总进度: 2%:  ▓
执行 fastboot stage factory/FSBL.bin
刷机总进度: 4%:  ▓▓
002afa0
执行 fastboot continue
刷机总进度: 6%:  ▓▓▓
执行 fastboot stage u-boot.itb
reconnect fastboot device...
刷机总进度: 8%:  ▓▓▓▓
刷机总进度: 9%:  ▓▓▓▓
执行 fastboot continue
刷机总进度: 11%:  ▓▓▓▓▓
执行 fastboot getvar mtd-size
reconnect fastboot device...
NULL
刷机总进度: 13%:  ▓▓▓▓▓▓
执行 fastboot getvar blk-size
universal
刷机总进度: 15%:  ▓▓▓▓▓▓▓
开始匹配分区表: ['partition_{size0}.json', 'partition_{size1}.json']
匹配原始分区表: partition_universal.json
分区表匹配结果为: partition_universal.json
执行 fastboot flash gpt partition_universal.json
parse gpt/mtd table okay
执行 fastboot flash bootinfo bootinfo_sd.bin
执行 fastboot flash fsbl FSBL.bin
执行 fastboot flash env env.bin
执行 fastboot flash opensbi fw_dynamic.itb
执行 fastboot flash uboot u-boot.itb
执行 fastboot flash bootfs bootfs.ext4
生成sparse格式文件, split>>127
start split sparse file in 2024-06-03 15:51:57
bytesIo: Wrote 65535 blocks in 10 sparse chunks (12% compression)
split sparse file over, begin download in 2024-06-03 15:51:58

... ... ...

刷机总进度: 99%:  ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓
download over in  2024-06-03 15:57:42
bytesIo: Wrote 6517 blocks in 2 sparse chunks (100% compression)
split sparse file over, begin download in 2024-06-03 15:57:42
download over in  2024-06-03 15:57:43
---》刷机成功!
<-------------刷机结束---------------->
```
## Enjoy!

Reset BPi-F3.
ArchLinux will boot!

By [Felix Yan](https://archriscv.felixc.at/), the password of `root` is `archriscv`.

### Start services

Network
```sh
$ systemctl enable --now systemd-networkd
```

Time Sync
```sh
$ systemctl enable --now systemd-timesyncd
```

> [!NOTE]
> SSL connections only work with the correct time.

### Install packages

```sh
$ pacman -Syu
$ pacman -S nano openssh openssl
```

### Modify mirror list
```sh
$ nano /etc/pacman.d/mirrorlist
```

```
##
## Arch Linux RISC-V repository mirrorlist
##

## Worldwide
#Server = https://riscv.mirror.pkgbuild.com/repo/$repo

## Canada
#Server = https://archriscv.felixc.at/repo/$repo

## China
Server = https://mirror.iscas.ac.cn/archriscv/repo/$repo
#Server = https://mirrors.sustech.edu.cn/archriscv/repo/$repo
#Server = https://mirror.nju.edu.cn/archriscv/repo/$repo
#Server = https://mirrors.wsyu.edu.cn/archriscv/repo/$repo

## Finland
#Server = https://mirrors.felixc.at/archriscv/repo/$repo
```

## Desktop Environment

### KDE

```sh
$ pacman -S icu plasma-desktop plasma-nm plasma-pa plasma-workspace plasma-workspace-wallpapers xdg-desktop-portal-kde dolphin kate konsole okular spectacle
```

```
$ dbus-run-session startplasma-wayland
```

### Xfce4

```sh
$ pacman -S icu xfce4 xorg
```

```sh
$ dbus-run-session startxfce4
```

### GNOME
Unavailable because `libmutter` is missing.
