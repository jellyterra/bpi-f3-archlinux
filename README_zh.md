# Banana Pi F3 ArchLinux
适用于带 SpacemiT X60/K1 的 Banana Pi F3 的 Arch Linux RISC-V 映像 `rootfs.ext4`。

### 这个映像做了什么？

- 复制驱动
    - /lib/dri
        - pvr_dri.so
        - spacemit_dri.so -> pvr_dri.so
    - /lib/firmware
    - /lib/modules
- 网络配置
    - /etc/systemd/network/default.network

[Felix Yan 关于 archriscv 的 wiki](https://wiki.felixc.at/linux_risc-v)

# 烧录到 BPi-F3

## 准备

### 依赖

Install `fastboot`, `zstd`.

```sh
$ dnf install    fastboot zstd # For Fedora, CentOS, RHEL etc.
$ pacman -S android-tools zstd # For ArchLinux
```

### 烧录工具

下载 [进迭时空](https://download.banana-pi.dev/d/ca025d76afd448aabc63/files/?p=%2FTools%2Fimage_download_tools%2Ftitantools_for_linux-latest.zip) 的烧录工具.

解压并带选项 `--appimage-extract` 运行 AppImage。

复制 `squashfs-root/resources/app/flashserver` 到 `$PATH` 的任意一项下。

```sh
$ unzip titantools_for_linux-latest.zip
$ chmod +x titantools_for_linux-1.0.23beta.AppImage
$ ./titantools_for_linux-1.0.23beta.AppImage --appimage-extract
$ cp -p ./squashfs-root/resources/app/flashserver ~/.local/bin/
```

### ROM

下载 [ROM from Banana Pi](https://drive.google.com/drive/folders/1LQoioz6N5YQpSOxY47OmetnPX4yggtT0): `bianbu-23.10-nas-k1-v1.0rc1-release-20240429192450.zip`

应该看起来像这样:
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
├── rootfs.ext4 <-- 将被替换
└── u-boot.itb
```

## 连接到开发板

进入烧录模式：在板上按住 `FDL` 并按下 `RST`。

将开发板连接到你的主机。开发板会以 **DFU** 设备出现并在烧录时显示为 `U-Boot USB download gadget`。
```
$ lsusb
Bus 001 Device 009: ID 361c:1001 U-Boot USB download gadget
```

## 烧录

下载 ArchLinux rootfs 映像。请找： [releases](https://github.com/jellyterra/bananapi-f3-archlinux/releases).

```sh
$ zstd -d archriscv-2024-03-30.img.zst    # Decompress
$ cp archriscv-2024-03-30.img rootfs.ext4 # Replace rootfs.ext4
```

在 ROM 所在的目录下运行 `flashserver`。

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
## 大功告成！

复位 BPi-F3，ArchLinux 会随后启动！

据 [Felix Yan](https://archriscv.felixc.at/)，`root` 的密码为 `archriscv`。

### 启动服务

网络
```sh
$ systemctl enable --now systemd-networkd
```

时间同步
```sh
$ systemctl enable --now systemd-timesyncd
```

> [!NOTE]
> SSL 连接仅在正确的时间下有效。

### 安装软件包

```sh
$ pacman -Syu
$ pacman -S nano openssh openssl
```

### 更改镜像列表
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

## 桌面环境

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
最新的构建中缺失 `libmutter` 所以尚不可用。
