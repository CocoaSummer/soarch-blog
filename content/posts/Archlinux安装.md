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

1. BIOS 禁用 Secure Boot
2. BIOS 设置硬盘模式 AHCI
3. BIOS 启动模式 UEFI
4. 硬盘分区表含有 ESP 分区(EFI)
5. 硬盘删除分区，保持“空闲”状态

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

- 安装必需的软件包，nano 方便编辑配置文件

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

  1. 找到目标行：Ctrl+W，输入 #en_US，回车，找到 UTF-8
  2. 取消注释：删掉前面的#
  3. 找到目标行：Ctrl+W，输入 #zh_CN，回车，找到 UTF-8
  4. 取消注释：删掉前面的#
  5. 保存：Ctrl+X，输入 Y，回车

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
  pacman -S grub efibootmgr networkmanager network-manager-applet dialog os-prober mtools dosfstools ntfs-3g base-devel linux-headers reflector git sudo
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

  1. 找到目标行：Ctrl+W，输入 #GRUB，回车，找到 GRUB_DISABLE_OS_PROBER=false
  2. 取消注释：删掉前面的#
  3. 保存：Ctrl+X，输入 Y，回车

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
