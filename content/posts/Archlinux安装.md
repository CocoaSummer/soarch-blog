---
title: 'Archlinux安装'
date: 2022-05-25T12:31:48+08:00
lastmod: 2022-05-25T12:31:48+08:00
draft: false
cover: https://oss.soarch.top/archlinux.jpg
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

- 安装 linux 内核和固件，以及纯文本编辑器

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

## 基本系统配置

### 同步时间

- 设置时区

  ```sh
  ln -sf /usr/share/zoneinfo/Asia/Chongqing /etc/localtime
  ```

- 同步硬件时钟

  ```sh
  hwclock --systohc
  ```

### 本地化

- 编辑 locale.gen 文件

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

### 设置主机名

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

### 其他配置

- 设置 Root 密码

  ```sh
  passwd
  ```

- 安装生成引导的工具

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

## 进桌面前配置

重启后，使用 root 用户登录

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

### 软件包管理

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

### 安装图形依赖

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

### 配置显卡

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

- 生成配置

  ```sh
  nvidia-xconfig
  ```

### 配置 lightdm

- 安装 xrandr

  ```sh
  pacman -S xorg-xrandr
  ```

- 安装 numlockx

  ```sh
  pacman -S numlockx
  ```

- 编辑 lightdm.conf

  ```sh
  nano /etc/lightdm/lightdm.conf
  ```

  1. 启动太快导致显卡驱动未加载，需设置 logind-check-graphical=false

     - Ctrl+W，输入 #logind，回车，找到 graphical 那行，删掉前面的#

  2. 多显卡仅使用 nvidia 显卡，显示管理器需增加启动脚本

     - Ctrl+W，输入 #display-setup，找到 script 那行，删掉前面的#
     - 修改：display-setup-script=/etc/lightdm/display_setup.sh

  3. 登录界面开启 Num Lock

     - Ctrl+W，输入 #greeter-setup，找到 script 那行，删掉前面的#
     - 修改：greeter-setup-script=/usr/bin/numlockx on

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

- 设置开机启动

  ```sh
  systemctl enable lightdm
  ```

- 设置中文

  ```sh
  nano /usr/lib/systemd/system/lightdm.service
  ```

  追加以下内容

  ```
  [Service]
  Environment=LANG=zh_CN.UTF-8
  ```

  保存：Ctrl+X，输入 Y，回车

### 其他配置

- 刷新图标缓存，首次进入桌面图标可能显示异常

  ```sh
  gtk-update-icon-cache --force /usr/share/icons/hicolor
  ```

- 禁用蜂鸣器

  ```sh
  rmmod pcspkr # 卸载蜂鸣器模块
  echo "blacklist pcspkr" > /etc/modprobe.d/nobeep.conf # 加入黑名单，阻止udev加载，重启后生效
  ```

- 安装字体

  - noto-fonts-cjk 中日韩字体
  - ttf-dejavu  编程字体
  - adobe-source-han-sans-cn-fonts 中文字体
  - adobe-source-han-serif-cn-fonts 中文字体

  ```sh
  pacman -S noto-fonts-cjk ttf-dejavu adobe-source-han-sans-cn-fonts adobe-source-han-serif-cn-fonts
  ```

- 启用回收站

  ```sh
  pacman -S gvfs
  ```

- 登录普通用户，并进入用户目录

  ```sh
  su soarch
  cd ~
  ```

- 生成用户目录，在$HOME 路径下执行

  ```sh
  LC_ALL=C xdg-user-dirs-update --force # 强制创建英语目录
  ```

- 创建并写入.xprofile

  ```sh
  nano ~/.xprofile
  ```

  填入内容

  ```
  export LANG=zh_CN.UTF-8
  export LANGUAGE=zh_CN:en_US
  ```

  保存：Ctrl+X，输入 Y，回车

- 重启，进入桌面

  ```sh
  reboot
  ```

---

## 进桌面后配置

### 开启 Num Lock

1. 通过 设置 > 键盘 中的 启动时恢复数字按键状态 进行勾选，生成配置文件
2. 在~/.config/xfce4/xfconf/xfce-perchannel-xml/keyboards.xml 中确保以下值设定为 true

   ```sh
   nano ~/.config/xfce4/xfconf/xfce-perchannel-xml/keyboards.xml
   ```

   对应修改以下内容

   ```xml
   <property name="Numlock" type="bool" value="true"/>
   <property name="RestoreNumlock" type="bool" value="true"/>
   ```

   保存：Ctrl+X，输入 Y，回车

   下一次进入桌面时生效

### 修改默认引导

- 编辑 grub 文件

  ```sh
  sudo nano /etc/default/grub
  ```

  由于 Windows 索引为 2，修改以下内容

  ```
  GRUB_DEFAULT=2
  ```

  保存：Ctrl+X，输入 Y，回车

- 重新生成引导列表

  ```sh
  sudo grub-mkconfig -o /boot/grub/grub.cfg
  ```

  重启后生效

### 中文输入法

- 安装输入法引擎

  ```sh
  yay -S fcitx5-im
  ```

- 配置输入法引擎

  ```sh
  sudo nano /etc/environment
  ```

  填入内容

  ```
  GTK_IM_MODULE=fcitx
  QT_IM_MODULE=fcitx
  XMODIFIERS=@im=fcitx
  INPUT_METHOD=fcitx
  SDL_IM_MODULE=fcitx
  GLFW_IM_MODULE=ibus
  ```

  保存：Ctrl+X，输入 Y，回车

- 安装小狼毫输入法

  ```sh
  yay -S fcitx5-rime
  ```

- 安装常用中文词库

  ```sh
  yay -S fcitx5-pinyin-zhwiki fcitx5-pinyin-sougou fcitx5-pinyin-zhwiki-rime fcitx5-pinyin-moegirl-rime
  ```

- 注销当前用户，重新登录即可生效

### 控制台优化

- ls 命令设置别名

  ```sh
  sudo nano ~/.bashrc
  ```

  追加内容

  ```
  alias ll="ls -l"
  alias la="ls -a"
  alias lla="ls -al"
  ```

  保存：Ctrl+X，输入 Y，回车

  立即生效

  ```sh
  source ~/.bashrc
  ```

## 安装常用软件

- 安装软件

  ```sh
  yay -S firefox firefox-i18n-zh-cn # 火狐浏览器
  yay -S google-chrome # 谷歌浏览器
  yay -S leafpad # 纯文本编辑器
  yay -S mlocate # 建立文件索引工具
  yay -S github-cli # github授权工具
  yay -S vlc # vlc视频播放器
  yay -S visual-studio-code-bin # vscode高级编辑器
  yay -S unzip # 解压zip工具
  yay -S cherrytree # 笔记工具
  yay -S anydesk-bin # 远程工具
  yay -S todesk-bin # 远程工具
  yay -S wps-office wps-office-mui-zh-cn ttf-wps-fonts # WPS 办公工具
  yay -S dropbox thunar-dropbox # 同步盘
  yay -S multiload-ng-common xfce4-multiload-ng-plugin # 任务栏硬件监控工具
  yay -S hugo # markdown生成博客工具
  ```

- 设置开机启动

  ```sh
  sudo systemctl enable anydesk
  sudo systemctl enable todeskd
  ```

- 阻止 dropbox 自动更新
  ```sh
  rm -rf ~/.dropbox-dist
  install -dm0 ~/.dropbox-dist
  ```

<!--
- 配置任务栏
- 配置工作区
- 配置扬声器
- 配置麦克风
- 配置摄像头
- 装qq
- 装微信
- 改主题
- 自动切换主题
-->

<!-- - 调节 CPU 频率

安装工具

```sh
pacman -S thermald i7z cpupower
```

开机启动 cpupower

```sh
systemctl enable cpupower.service
```

查看所有可用的模块

```sh
ls /usr/lib/modules/$(uname -r)/kernel/drivers/cpufreq/
```

加载合适的模块。一旦合适的 cpufreq 驱动模块被加载成功，就可以通过以下命令查询到 CPU 的信息

```sh
cpupower frequency-info
``` -->
