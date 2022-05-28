---
title: 'Archlinux安装'
date: 2022-05-25T12:31:48+08:00
lastmod: 2022-05-25T12:31:48+08:00
draft: false
# cover: /img/cover.jpg
categories:
  - 杂记
tags:
  - 技术
---

为开发环境定制优化的 Archlinux，与 Win10 组成双系统

<!--more-->

## 物理机必须条件

- BIOS 禁用 Secure Boot
- BIOS 设置硬盘模式 AHCI
- BIOS 设置启动模式 UEFI
- 硬盘分区表含有 ESP 分区(EFI)
- 硬盘删除分区，保持“空闲”状态

## 检查互联网是否连接

- 确保系统已经启用了网络接口

  ```sh
  ip link
  ```

- 用 ping 检查网络连接

  ```sh
  ping www.baidu.com
  ```

## 更新系统时间

- 确保系统时间是准确的

  ```sh
  timedatectl set-ntp true
  ```

- 检查服务状态

  ```sh
  timedatectl status
  ```

## 硬盘分区

- 找到需要装系统的分区

  ```sh
  lsblk
  ```

- 创建分区

  ```sh
  cfdisk /dev/nvme0n1
  ```

  1. 选择分区
  2. 选择 New，回车
  3. 输入系统所需容量，回车
  4. 选择 Write，输入 yes，回车

- 检查分区

  ```sh
  lsblk
  ```

- 格式化分区

  ```sh
  mkfs.ext4 /dev/nvme0n1p1
  ```

- 挂载分区

  ```sh
  mount /dev/sda1 /mnt
  ```

- 找到 ESP 分区(EFI)

  ```sh
  lsblk
  ```

- 创建 boot 目录

  ```sh
  mkdir /mnt/boot
  ```

- 挂载 ESP 分区(EFI)
  ```sh
  mount /dev/sda1 /mnt/boot
  ```

## 安装基本系统

- 安装必需的软件包

  nano 方便编辑配置文件

  ```sh
  pacstrap /mnt base linux linux-firmware nano
  ```

- 生成 fstab 文件

  ```sh
  genfstab -U /mnt >> /mnt/etc/fstab
  ```

- 检查生成的 fstab 文件

  ```sh
  cat /mnt/etc/fstab
  ```

- 切换进入新安装的系统

  ```sh
  arch-chroot /mnt
  ```

## 基础配置

- 设置时区

  ```sh
  ln -sf /usr/share/zoneinfo/Asia/Chongqing /etc/localtime
  ```

- 同步硬件时钟

  ```sh
  hwclock --systohc
  ```

- 本地化

  ```sh
  nano /etc/locale.gen
  ```

  1. 启用英文：Ctrl+W，输入 #en_US，回车，找到 UTF-8 那行，删掉前面的#
  2. 启用中文：Ctrl+W，输入 #zh_CN，回车，找到 UTF-8 那行，删掉前面的#
  3. 保存：Ctrl+X，输入 Y，回车

- 生成 locale

  ```sh
  locale-gen
  ```

- 创建并写入/etc/locale.conf 文件

  ```sh
  nano /etc/locale.conf
  ```

  填入内容，注意这里只能填这个

  ```
  LANG=en_US.UTF-8
  ```

  保存：Ctrl+X，输入 Y，回车

- 创建并写入 hostname

  ```sh
  nano /etc/hostname
  ```

  填入内容

  ```
  soarch
  ```

  保存：Ctrl+X，输入 Y，回车

- 修改 hosts，间隔空白使用 Tab

  ```sh
  nano /etc/hosts
  ```

  填入内容

  ```
  127.0.0.1     localhost
  ::1           localhost
  127.0.1.1     soarch.localdomain      soarch
  ```

  保存：Ctrl+X，输入 Y，回车

- 检查 hosts

  ```sh
  cat /etc/hosts
  ```

- 设置 Root 密码

  ```sh
  passwd
  ```

- 安装引导程序

  ```sh
  pacman -S grub efibootmgr networkmanager network-manager-applet dialog os-prober mtools dosfstools ntfs-3g base-devel linux-headers reflector git sudo wget
  ```

- 安装 WiFi 工具(可选)

  ```sh
  pacman -S wireless_tools wpa_supplicant
  ```

- 安装 CPU 微码文件

  intel

  ```sh
  pacman -S intel-ucode
  ```

  amd

  ```sh
  pacman -S amd-ucode
  ```

- 多系统引导

  ```sh
  nano /etc/default/grub
  ```

  1. 启用：Ctrl+W，输入 #GRUB，回车，找到 GRUB_DISABLE_OS_PROBER 那行，删掉前面的#
  2. 保存：Ctrl+X，输入 Y，回车

- 安装 grub

  ```sh
  grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Arch
  ```

- 生成 grub.cfg

  ```sh
  grub-mkconfig -o /boot/grub/grub.cfg
  ```

  提示 Found 和 done， 则正确

- 退出

  ```sh
  exit
  ```

- 卸载分区

  ```sh
  umount /mnt/boot
  umount /mnt
  ```

- 重启

  ```sh
  reboot
  ```

  启动时拔出 u 盘

---

## 图形配置

### 开机服务

- 立即启动网络服务，并设置开机启动

  ```sh
  systemctl enable --now NetworkManager
  ```

### 用户和用户组

- 创建普通用户

  ```sh
  useradd -m -G wheel soarch
  ```

- 为该用户创建密码

  ```sh
  passwd soarch
  ```

- 启用 sudo

  授权 nano 使用 visudo，必须使用 visudo 编辑该文件防止出错

  ```sh
  EDITOR=nano visudo
  ```

  1. 启用 sudo：Ctrl+W，输入 #%wheel，回车，找到 ALL=(ALL) ALL 那行，删掉前面的#
  2. 保存：Ctrl+X，输入 Y，回车

### 包管理

- 按下载速度生成镜像源

  ```sh
  reflector -c China -a 6 --sort rate --save /etc/pacman.d/mirrorlist
  ```

- 修改 reflector.conf

  ```sh
  nano /etc/xdg/reflector/reflector.conf
  ```

  找到对应行，修改以下内容

  ```
  --country China
  --sort rate
  ```

  保存：Ctrl+X，输入 Y，回车

- reflector 设置开机启动

  ```sh
  systemctl enable reflector
  ```

- 修改 pacman.conf

  ```sh
  nano /etc/pacman.conf
  ```

  新增 AUR 仓库：在末尾处，填入内容

  ```
  [archlinuxcn]
  Server = https://repo.archlinuxcn.org/$arch
  ```

  1. 启用 multilib 仓库：Ctrl+W，输入 #[multilib，回车，找到 multilib 那行和下一行，删掉前面的#
  2. 启用多线程下载：Ctrl+W，输入 #Xfer，回车，找到 wget 那行，删掉前面的#
  3. 保存：Ctrl+X，输入 Y，回车

- 同步镜像源

  ```sh
  pacman -Syu && pacman -S archlinuxcn-keyring
  ```

- 安装 AUR 助手

  ```sh
  pacman -S yay
  ```

### 安装图形界面依赖

- 安装图形服务器

  ```sh
  pacman -S xorg
  ```

- 安装显示管理器

  ```sh
  pacman -S lightdm
  ```

- 安装桌面环境

  ```sh
  pacman -S xfce4 xfce4-goodies
  ```

### 配置 NVIDIA 显卡

- 安装驱动

  我的显卡为 gtx760，其他 nvidia 驱动，参考[NVIDIA](<https://wiki.archlinux.org/title/NVIDIA_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)>)

  ```sh
  su soarch
  yay -S nvidia-470xx-dkms
  exit
  ```

- 检查驱动安装正常

  ```sh
  nvidia-smi
  ```

- 自动配置

  ```sh
  nvidia-xconfig
  ```

### 配置 lightdm

- 安装 xrandr

  ```sh
  pacman -S xorg-xrandr
  ```

- 编辑 lightdm.conf

  - 多显卡仅使用 nvidia 显卡，显示管理器需增加启动脚本 display-setup-script
  - 启动太快导致显卡驱动未加载，需设置 logind-check-graphical=false

  ```sh
  nano /etc/lightdm/lightdm.conf
  ```

  1. 启用：Ctrl+W，输入 #logind，回车，找到 graphical 那行，删掉前面的#
  2. 启用：Ctrl+W，输入 #display-setup，找到 script 那行，删掉前面的#
  3. 修改：display-setup-script=/etc/lightdm/display_setup.sh
  4. 保存：Ctrl+X，输入 Y，回车

- 创建并写入 display_setup.sh

  ```sh
  #!/bin/bash
  xrandr --setprovideroutputsource modesetting NVIDIA-0
  xrandr --auto
  ```

- display_setup.sh 追加可执行权限

  ```sh
  chmod +x display_setup.sh
  ```

---

## 桌面配置

### 本地化

- 安装字体

  - noto-fonts-cjk 中文字体
  - ttf-dejavu  编程字体

  ```sh
  pacman -S noto-fonts-cjk ttf-dejavu 
  ```

- 设置桌面环境显示中文

  ```sh
  nano ~/.xprofile
  ```

  填入内容

  ```
  export LANG=zh_CN.UTF-8
  export LANGUAGE=zh_CN:en_US
  ```

- 设置显示管理器中文

### 控制台优化

- 配置 ls 命令

  ```sh
  sudo nano ~/.bashrc
  ```

  追加内容

  ```
  alias ll="ls -l"
  alias la="ls -a"
  alias lla="ls -al"
  ```

  立即生效

  ```sh
  source ~/.bashrc
  ```

- 刷新图标缓存，以防首次进入桌面图标显示异常

  ```sh
  gtk-update-icon-cache --force /usr/share/icons/hicolor
  ```

- 显示管理器中文
- 调整 Windows 为第一引导顺序
- 开机时打开 Num Lock
- 生成用户目录
- 配置自动调节 CPU 频率
- 配置声音
- 配置 ssh
- 远程显示分辨率无法填满
- 双显示器配置
- 安装中文输入法
- 配置中文输入法
- 安装常用软件
- 建立文件索引和搜索
- 固态硬盘配置
-

- 重启

  ```sh
  reboot
  ```
