# debian12_fs

## 一、debian12文件系统构建

1. 构建基础文件系统

```shell
# amd64
apt install -y debootstrap
mkdir -p debian-rootfs
debootstrap --arch=amd64 bookworm debian-rootfs http://deb.debian.org/debian
# 切换到待制作目录，自定义文件系统
chroot debian-rootfs /bin/bash

# arm64
# mkdir -p debian-arm64
# debootstrap --arch=arm64 bookworm debian-arm64 http://deb.debian.org/debian
# chroot debian-arm64/ /bin/bash
```

2. 自定义配置

```shell
mount -t devtmpfs none /dev
mount -t devpts none /dev/pts
mount -t proc none /proc
mount -t sysfs none /sys
mount -t tmpfs none /dev/shm

# libpam-dev lightdm用户认证
# network-manager 网口自动配置
# bash-completion ssh/bash 等自动补全
# locales 语言环境
# accountsservice lightdm验证需要dbus的用户列表
# auditd 审计，需内核支持
# gnome-session 没有 session，会导致lightdm认证完成后启动失败；不创建 /home/gxh/.xsession，通过 /home/gxh/.xsession-errors 可知
# /usr/share/xsessions/lightdm-xsession.desktop
# systemd: 基础系统已经安装
apt install -y vim openssh-server gcc python3-pip net-tools locales pciutils
apt install -y network-manager bash-completion libpam-dev accountsservice auditd
apt install -y xserver-xorg lightdm gnome-session gnome-terminal nautilus
apt install -y sudo

# 文件系统挂载
/etc/fstab
sysfs
proc
devpts
...

# 串口
systemctl enable serial-getty\@ttyAMA0.service
# systemctl start serial-getty\@ttyAMA0.service

# 虚拟终端
systemctl enable getty\@tty1.service
# systemctl start getty\@tty1.service

# shell - dash改为bash
ln -sf /usr/bin/bash /bin/sh
ln -sf /usr/bin/bash /usr/bin/sh
# dpkg-reconfigure dash

# ssh
systemctl enable ssh
# systemctl start ssh
# 允许root用户登录
sed -i 's@^#\?PermitRootLogin.*@PermitRootLogin yes@' /etc/ssh/sshd_config
# 连接
ssh root@localhost -p 2222

# 图形界面
# xorg用户态驱动，桌面 gnome lightdm gdm gnome-session
systemctl enable lightdm

# 网络 - 启动 systemd-networkd，否则网口不会自动link；启动也不会自动link；必须使用 network-manager
systemctl enable systemd-networkd

# 添加用户，方便登录
useradd gxh
passwd gxh
passwd root
chown -R gxh:gxh /home/gxh
usermod -a -G lightdm gxh
# /home/gxh/.Xauthority .dmrc 等会自动创建

# 添加用户到sudoers
echo "gxh ALL=(ALL:ALL) ALL" >> /etc/sudoers
# 免密sudo
echo "gxh ALL=(ALL:ALL) NOPASSWD:ALL" >> /etc/sudoers
# sudo组免密
%sudo ALL=(ALL:ALL) NOPASSWD:ALL
```

3. 退出chroot环境

```shell
exit
umount debian-rootfs/dev/pts
umount debian-rootfs/dev/shm
umount debian-rootfs/dev
umount debian-rootfs/proc
umount debian-rootfs/sys
```

4. 打包为ext4文件系统的img

```shell
# 1) 计算目录大小（以4K块为单位）
dir_size=$(du -s --block-size=4096 debian-rootfs | cut -f1)
# 2) 增加20%的额外空间用于文件系统元数据
fs_blocks=$((dir_size + dir_size/5))
# 3) 创建镜像文件
dd if=/dev/zero of=debian12.img bs=4K count=$fs_blocks
# 4) 使用mke2fs创建文件系统并填充内容
mke2fs -t ext4 -d debian-rootfs debian12.img
```

5. qemu验证

```shell
# linux-5.10.240
make defconfig
# VIRTIO VIRTIO_BLK DRM_VIRTIO_GPU VIRTIO_NET CONFIG_DRM_BOCHS(1234:1111的pci驱动，没有也可以) e1000 (网卡pci驱动)
make menuconfig
make bzImage -j$(nproc)

# 可以直接使用yocto下载的bzImage，因为没有用到 virtio_blk 驱动，设备节点不是 vda
# 必须指定 -m，否则图形界面会因为内存不足闪退
qemu-system-x86_64 -enable-kvm -m 10G -smp 4 -kernel bzImage-qemux86-64.bin -append "console=ttyS0 root=/dev/sda rw" \
-hda debian12.img -display gtk -net user,hostfwd=tcp::2222-:22 -net nic -serial stdio

# 串口和虚拟终端同时显示启动日志
qemu-system-x86_64 -enable-kvm -m 10G -smp 4 -kernel bzImage-qemux86-64.bin -append "console=ttyS0 console=tty1 root=/dev/sda rw" \
-hda debian12.img -display gtk -net user,hostfwd=tcp::2222-:22 -net nic -serial stdio

# -display gtk
# -display sdl
# -hda debian12.img
# -drive format=raw,file=debian12.img
# -accel kvm
# -vga std   Device 1234:1111 (rev 02) Subsystem: Red Hat, Inc. Device 1100
# -kernel bzImage-qemux86-64.bin

# 查看lightdm报错, /var/log/Xorg.0.log /var/log/lightdm/lightdm.log
journalctl
journalctl | cat -
systemctl status lightdm
```

- [qemux86_64 bzImage](https://downloads.yoctoproject.org/releases/yocto/yocto-5.1.4/machines/qemu/qemux86-64/bzImage-qemux86-64.bin)

## 二、文件系统镜像创建技巧

1. 使用mkfs.ext4从目录创建 img 文件

```shell
# 1) 计算目录大小（以4K块为单位）
dir_size=$(du -s --block-size=4096 /path/to/your/directory | cut -f1)
# 2) 增加20%的额外空间用于文件系统元数据
fs_blocks=$((dir_size + dir_size/5))
# 3) 创建镜像文件
dd if=/dev/zero of=your_image.img bs=4K count=$fs_blocks
# 4) 使用mke2fs创建文件系统并填充内容
mke2fs -t ext4 -d /path/to/your/directory your_image.img
```

2. 一个文件系统不使用整个分区

```shell
# /usr/sbin/mkfs.ext4 -> mke2fs
# 1、使用dd直接将img文件烧录到分区
# 确保 test.img 小于 /dev/sdd2分区大小，只用分区的一部分。也可以在 mkfs.ext4时指定大小。
# test.img 100M  sdd2 200M
dd if=/dev/zero of=test.img bs=1M count=100M
mke2fs -t ext4 -d /path/to/your/directory test.img
dd if=test.img of=/dev/sdd2

# 2、使用mkfs.ext4 指定文件系统大小，通过 file 可查看 UUID；不指定大小默认整个分区/文件作为文件系统
mkfs.ext4 -U 0964d8b2-281c-4c82-94d7-699999999940 your_image.img 100M
```

3. 文件系统分区的扩容和缩小

```shell
# 文件系统不使用整个分区，方便缩小镜像文件大小，可用resize2fs修改文件系统大小
# 1、resize2fs不指定大小，使用整个分区作为文件系统
resize2fs /dev/mmcblk0p6
# 2、resize2fs -M，缩小文件系统大小刚好容纳文件和元数据
resize2fs -M /dev/mmcblk0p6
```

4. other

```shell
# 创建一个稀疏文件
truncate -s 100M your_image.img

# 检查镜像的完整性
e2fsck -f your_image.img
# 调整文件系统大小最小化镜像
resize2fs -M your_image.img
```

### 三、ubuntu

1. ubuntu镜像构建

```shell
mkdir ubuntu-rootfs
debootstrap --arch=amd64 jammy ubuntu-rootfs https://mirrors.ustc.edu.cn/ubuntu
chroot ubuntu-rootfs
apt install -y vim
passwd root
exit

# 1) 计算目录大小（以4K块为单位）
dir_size=$(du -s --block-size=4096 ubuntu-rootfs | cut -f1)
# 2) 增加20%的额外空间用于文件系统元数据
fs_blocks=$((dir_size + dir_size/5))
# 3) 创建镜像文件
dd if=/dev/zero of=ubuntu22.img bs=4K count=$fs_blocks
# 4) 使用mke2fs创建文件系统并填充内容
mke2fs -t ext4 -d ubuntu-rootfs ubuntu22.img
```

2. Qemu验证

```shell
qemu-system-x86_64 -enable-kvm -m 10G -smp 4 -kernel bzImage-qemux86-64.bin -append "console=ttyS0 root=/dev/sda rw" \
-hda ubuntu22.img -display gtk -net user,hostfwd=tcp::2222-:22 -net nic -serial stdio
```

