---
date: 2018-04-02 15:32
status: public
title: Archlinux安装记录
---

# 1. 硬盘分区及格式化

## 1.1 分区

```bash
# 启动 parted 
parted

# 使用 msdos 分区表
mklabel msdos

# 创建 boot 分区
mkpart primary ext4 1M 100M
set 1 boot on

# 创建交换分区
mkpart primary linux-swap 100M 1.1G

# 其他分区，一般为 /
mkpart primary ext4 1.1G -1

# 退出
q
```

## 1.2 格式化

```bash
# 将 boot 分区格式化为 ext4 格式
mkfs.ext4 /dev/sda1

# 将 swap 分区进行格式化
mkswap /dev/sda2

# 将 / 分区格式化为 ext4 格式
mkfs.ext4 /dev/sda3
```

## 1.3 挂载各分区

```bash
# 挂载 / 到安装盘的 /mnt
mount /dev/sda3 /mnt

# 创建 boot 挂载目录，并挂载
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot

# 启用交换分区
swapon /dev/sda2
```

# 2. 更换源并安装基本系统

```bash
# 编辑 /etc/pacman.d/mirrorlist
vim /etc/pacman.d/mirrorlist

# 在合适位置加上阿里云的源
Server = http://mirrors.aliyun.com/archlinux/$repo/os/$arch

# 更新源
pacman -Syy

# 安装基本系统
pacstrap /mnt base base-devel

# 将当前挂载关系映射到安装的系统中
genfstab -U /mnt >> /mnt/etc/fstab
```

# 3. 配置 ArchLinux

## 3.1 引导配置

```bash
# 切换到新安装的系统
arch-chroot /mnt /bin/bash

# 安装 grub
pacman -S grub

# 安装 grub 到 sda 设备并生成配置文件
grub-install /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg

# 重启，无密码登陆 root 账户
```

## 3.2 语言环境配置

```bash
# 编辑 /etc/locale.gen 找到 en_US.UTF-8 zh_CN.UTF-8 放开注释
# 生成编码
locale-gen

# 配置当前语言环境
echo LANG=en_US.UTF-8 > /etc/locale.conf
```

## 3.3 配置时间

```bash
# 删除原有信息
rm /etc/localtime

# 链接上海的时间文件
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

## 3.4 配置主机名

```bash
echo lixifun > /etc/hostname
```

## 3.5 创建用户
```bash
# 创建普通用户生成默认用户目录加入 wheel 组使其具有 sudo 权限
useradd -m -G wheel -s /bin/bash lixifun

# 创建密码
passwd lixifun
```

## 3.6 为用户添加 sudo 权限

```bash
# 安装 sudo
pacman -S sudo

# 编辑 sudo 放开 %wheel ALL=(ALL) ALL 前的注释
visudo
```

## 3.7 配置网络

```bash
# 启动 dhcpcd
systemctl start dhcpcd

# 将 dhcpcd 加入开机启动服务中
systemctl enable dhcpcd
```