---
title: 'Archlinux安装'
date: 2022-05-25T12:31:48+08:00
lastmod: 2022-06-03T12:31:48+08:00
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

## 检查互联网

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

### 调节 CPU 频率

可选

- 安装工具

  ```sh
  pacman -S thermald i7z cpupower
  ```

- 开机启动 cpupower

  ```sh
  systemctl enable cpupower.service
  ```

- 查看所有可用的模块

  ```sh
  ls /usr/lib/modules/$(uname -r)/kernel/drivers/cpufreq/
  ```

  (未验证)加载合适的模块。一旦合适的 cpufreq 驱动模块被加载成功

- 查询到 CPU 的信息

  ```sh
  cpupower frequency-info
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

### NVIDIA 显卡

- 安装驱动

  我的显卡为 gtx760，其他 nvidia 驱动，参考[NVIDIA](<https://wiki.archlinux.org/title/NVIDIA_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)>)

  ```sh
  su soarch
  yay -S nvidia-470xx-dkms nvidia-settings
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

### 视频硬件加速

- 安装硬件加速依赖

  ```sh
  pacman -S libva-vdpau-driver libva-utils vdpauinfo
  ```

- 验证 VA-API

  ```sh
  vainfo
  ```

- 验证 VDPAU

  ```sh
  vdpauinfo
  ```

### 双显卡

仅使用 NVIDIA 显卡

- 安装屏幕配置工具 xrandr

  ```sh
  pacman -S xorg-xrandr
  ```

- 创建并写入/etc/X11/xorg.conf.d/10-nvidia-drm-outputclass.conf

  ```sh
  nano /etc/X11/xorg.conf.d/10-nvidia-drm-outputclass.conf
  ```

  填入内容

  ```
  Section "OutputClass"
    Identifier "aspeed"
    MatchDriver "ast"
    Driver "modesetting"
  EndSection

  Section "OutputClass"
    Identifier "nvidia"
    MatchDriver "nvidia-drm"
    Driver "nvidia"
    Option "AllowEmptyInitialConfiguration"
    Option "PrimaryGPU" "yes"
    ModulePath "/usr/lib/nvidia/xorg"
    ModulePath "/usr/lib/xorg/modules"
  EndSection
  ```

  保存：Ctrl+X，输入 Y，回车

- 创建并写入 /etc/lightdm/display_setup.sh

  ```sh
  nano /etc/lightdm/display_setup.sh
  ```

  填入内容

  ```sh
  #!/bin/bash
  xrandr --setprovideroutputsource modesetting NVIDIA-0
  xrandr --auto
  ```

  保存：Ctrl+X，输入 Y，回车

- display_setup.sh 追加可执行权限

  ```sh
  chmod +x /etc/lightdm/display_setup.sh
  ```

### 配置显示管理器

- 安装数字键盘激活工具 numlockx

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

- 设置开机启动

  ```sh
  systemctl enable lightdm
  ```

- 设置中文

  ```sh
  nano /usr/lib/systemd/system/lightdm.service
  ```

  在[Service]下一行，追加以下内容

  ```
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

- 安装 rime 输入法

  ```sh
  yay -S fcitx5-rime
  ```

- 注销当前用户，重新登录即可生效

### 导入中文词库

- 安装中文词库和中文输入方案

  ```sh
  yay -S fcitx5-pinyin-zhwiki-rime fcitx5-pinyin-moegirl-rime rime-cloverpinyin
  ```

- 导入自定义词库

  创建并写入 default.custom.yaml

  ```sh
  nano ~/.local/share/fcitx5/rime/default.custom.yaml
  ```

  填入内容

  ```yaml
  patch:
    'menu/page_size': 8
  schema_list:
    - schema: clover
  ```

  保存：Ctrl+X，输入 Y，回车

  创建并写入 clover.custom.yaml

  ```sh
  nano ~/.local/share/fcitx5/rime/clover.custom.yaml
  ```

  填入内容

  ```yaml
  patch:
    'translator/dictionary': pinyin_extended
  ```

  保存：Ctrl+X，输入 Y，回车

  创建并写入 pinyin_extended.dict.yaml

  ```sh
  nano ~/.local/share/fcitx5/rime/pinyin_extended.dict.yaml
  ```

  填入内容

  ```yaml
  ---
  name: pinyin_extended
  version: '1.0'
  sort: by_weight
  use_preset_vocabulary: false
  import_tables:
    - clover
    - clover.base
    - clover.phrase
    - sogou_new_words
    - moegirl
    - zhwiki
  ```

  保存：Ctrl+X，输入 Y，回车

- 鼠标右键单击桌面的 rime 输入法图标，选择“重新部署”即可生效

### 同步盘

- 安装 dropbox

  ```sh
  yay -S dropbox thunar-dropbox
  ```

- 阻止 dropbox 自动更新

  ```sh
  rm -rf ~/.dropbox-dist
  install -dm0 ~/.dropbox-dist
  ```

### 远程控制

- 安装远程工具

  ```sh
  yay -S anydesk-bin
  yay -S todesk-bin
  ```

- 设置开机启动

  ```sh
  sudo systemctl enable anydesk
  sudo systemctl enable todeskd
  ```

### 护眼模式

- 安装软件

  ```sh
  yay -S redshift
  ```

- 创建并写入 redshift 配置文件

  ```sh
  nano ~/.config/redshift.conf
  ```

  填入内容

  ```
  [redshift]
  temp-day=6500
  temp-night=3500
  transition=1
  location-provider=manual

  [manual]
  lat=29.5689
  lon=106.5577
  ```

  保存：Ctrl+X，输入 Y，回车

- 查看当前经纬度的昼夜状态

  ```sh
  timeout 1 redshift -v |head
  ```

- 设置开机启动

  ```sh
  cp /usr/share/applications/redshift-gtk.desktop ~/.config/autostart/
  ```

### 显示农历

- 安装农历

  ```sh
  yay -S lunar-calendar
  ```

- 编辑.xprofile

  ```sh
  nano ~/.xprofile
  ```

  追加内容

  ```
  export GTK3_MODULES=lunar-calendar-module
  export GTK_MODULES=lunar-calendar-module
  ```

  保存：Ctrl+X，输入 Y，回车

  注销后重新登录即可生效

### 安装常用软件

```sh
# 生产力
yay -S cherrytree # 笔记工具
yay -S visual-studio-code-bin # vscode高级编辑器
yay -S wps-office wps-office-mui-zh-cn ttf-wps-fonts # WPS 办公
yay -S wechat-devtools # 微信开发工具
# 多媒体(开发所需)
yay -S alsa-utils alsa-plugins pulseaudio pulseaudio-alsa pavucontrol # 声卡组件
yay -S gnome-sound-recorder # 录音机
yay -S obs-studio sndio luajit v4l2loopback-dkms # 虚拟摄像头
yay -S cheese # 摄像头预览工具
# 浏览器
yay -S firefox firefox-i18n-zh-cn # 火狐浏览器
yay -S google-chrome # 谷歌浏览器
# 命令行工具
yay -S hugo # markdown生成博客
yay -S lshw # 硬件查询
yay -S mlocate # 建立文件索引
yay -S unzip # 解压zip
yay -S zip # 压缩zip
yay -S rar # 解压缩rar
# 腾讯IM
yay -S linuxqq # 腾讯QQ
yay -S wechat-uos scrot # 腾讯微信
# 其他软件
yay -S multiload-ng-common xfce4-multiload-ng-plugin # 任务栏硬件监控工具
yay -S xunlei-bin # 迅雷下载
yay -S file-roller # 归档管理器
yay -S variety # 自动下载与切换壁纸工具
yay -S archlinux-wallpaper # archlinux壁纸
yay -S thunderbird thunderbird-i18n-zh-cn birdtray # 邮箱客户端
yay -S cifs-utils gvfs-smb # samba客户端
yay -S filezilla # ftp/ftps/sftp客户端
yay -S galculator # 计算器
```

## 定制系统配置

使用鼠标操作，根据个人习惯定制

- 配置自动下载并切换壁纸(variety)
- 设置电源超时关闭显示器
- 设置显示器超时显示屏保
- 配置邮件通知(birdtray)
- 配置任务栏

  - 任务栏置底
  - 调整面板项目顺序
  - 网络监控(netload)
  - 硬件监控(multiload-ng)
  - 显示桌面
  - 修改时间格式
  - 删除多余工作区，仅保留一个
  - 窗口仅显示图标
  - 修改应用菜单(whisker)
  - 修改应用菜单图标
  - 设置快捷键 Win 唤醒应用菜单
  - 修改输入法切换快捷键为左 Shift

## 双系统时间问题

让 windows 使用 UTC 时间

- 关闭自动更新时间和时区

- 打开注册表

  ```
  HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation\
  ```

- 新建十六进制 DWORD 值：RealTimeIsUniversal，并设置值为 1
- 重启即可生效

## 开发环境配置

- 安装开发所需工具

  ```sh
  yay -S github-cli # github授权工具
  yay -S git-lfs # git LFS 命令工具
  yay -S docker # docker容器
  ```

### Node

- 安装 nvm 命令

  ```sh
  yay -S nvm # node版本切换管理工具
  ```

- 配置 nvm 命令

  ```sh
  nano ~/.bashrc
  ```

  追加内容

  ```
  export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
  [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
  ```

  保存：Ctrl+X，输入 Y，回车

  立即生效

  ```sh
  source ~/.bashrc
  ```

- 安装 node 并使用

  ```sh
  nvm install 16
  nvm use 16
  ```

- 检查当前使用版本

  ```sh
  node -v
  ```

### Electron

- 安装 electron-builder 打包所需依赖

  ```sh
  yay -S libxcrypt-compat
  ```

### Python

- 安装 pip

  ```sh
  yay -S python-pip
  ```

- 安装 formater 和 linter

  ```sh
  sudo pip install black flake8
  ```
