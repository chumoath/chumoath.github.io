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

### 四、x86异构创建ubuntu-aarch64镜像

1. 准备qemu-aarch64的binfmt环境

```shell
# 1) 安装qemu-aarch64-static和binfmt_misc(内核支持)
apt install -y binfmt-support qemu-user-static debootstrap
# 2) 使用qemu的脚本手动注册qemu-aarch64 binfmt, qemu/scripts/qemu-binfmt-conf.sh
# 注册所有架构，将配置文件生成到/etc/binfmt.d/qemu-aarch64.conf
# ./qemu-binfmt-conf.sh -s ALL -Q /usr/bin -F -static -c yes
# -s 指定CPU，-Q 指定 qemu-aarch64-static 路径，-F指定qemu-aarch64后面的后缀
# 必须使用 qemu-aach64-static，执行chroot没有x86的动态库
./qemu-binfmt-conf.sh -s aarch64 -Q /usr/bin -F -static -c yes
# 3) 配置文件生效
systemctl restart systemd-binfmt.service
```

2. 创建ubuntu镜像

```shell
# 1) 创建debootstrap目录结构并下载deb包
mkdir ubuntu-arm64
# 跨架构必须使用--foreign，用于第一阶段下载和解包(没有apt)，不chroot；Release下载失败多次重试即可
# 只打印deb: debootstrap --arch=arm64 --variant=minbase --print-debs jammy ubuntu-arm64 https://mirrors.ustc.edu.cn/ubuntu-ports
# 只下载deb: debootstrap --arch=arm64 --variant=minbase --download-only jammy ubuntu-arm64 https://mirrors.ustc.edu.cn/ubuntu-ports
debootstrap --arch=arm64 --variant=minbase --foreign jammy ubuntu-arm64 https://mirrors.ustc.edu.cn/ubuntu-ports
# chroot还是x86，但是bash使用qemu-aarch64-static；必须使用 static，因为 ubuntu-arm64没有x86的库
cp /usr/bin/qemu-aarch64-static ubuntu-arm64/usr/bin/
# 2) 进入chroot，相当于执行 chroot ubuntu-arm64 bash，在chroot后，所有exec文件相当于都被qemu-aarch64-static解释执行
chroot ubuntu-arm64
# 如果是局域网，则需要配置 https 的证书，否则不能访问https镜像源
cp company_root.crt /usr/local/share/ca-certificates/
update-ca-certificates
# 3) 第二阶段安装核心包(包括apt)
/debootstrap/debootstrap --second-stage
apt install -y vim systemd
# 4) # 配置启动
ln -sf /lib/systemd/systemd /usr/sbin/init
sed -i '/BindsTo=*/d' /lib/systemd/system/serial-getty@.service
passwd root
exit
# 5) 清理apt缓存
chroot ubuntu-arm64/ apt-get clean
```

3. 打包ubuntu镜像

```shell
# 1) 计算目录大小（以4K块为单位）
dir_size=$(du -s --block-size=4096 ubuntu-arm64 | cut -f1)
# 2) 增加20%的额外空间用于文件系统元数据
fs_blocks=$((dir_size + dir_size/5))
# 3) 创建镜像文件
dd if=/dev/zero of=ubuntu22.img bs=4K count=$fs_blocks
# 4) 使用mke2fs创建文件系统并填充内容
mke2fs -t ext4 -d ubuntu-arm64 ubuntu22.img
```

4. Qemu验证

```shell
ip tuntap add tap0 mode tap group 0
ip link set dev tap0 up
ip addr add 192.168.33.1/24 dev tap0
iptables -A POSTROUTING -t nat -j MASQUERADE -s 192.168.33.0/24
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -P FORWARD ACCEPT

qemu-system-aarch64 -M virt,gic-version=3 -nographic \
    -m 1024M -cpu cortex-a72 -smp 1 -kernel Image-qemuarm64.bin \
    -drive format=raw,file=ubuntu22.img \
    -append "console=ttyAMA0 root=/dev/vda rw nokaslr" \
    -net nic,netdev=tap0,model=virtio \
    -netdev tap,id=tap0,ifname=tap0,script=no,downscript=no  \
    -serial stdio -monitor none
```

[Image-qemuarm64.bin](https://downloads.yoctoproject.org/releases/yocto/yocto-5.1.4/machines/qemu/qemuarm64/Image-qemuarm64.bin)

5. deb包下载失败解决方法

```shell
# 使用 wget 下载镜像源的包，镜像源会有限制
# 解决方法：使用rsync下载
# 1、/usr/sbin/debootstrap 添加 set -x，执行过一遍，收集所有包的日志
# 获取一个文件中所有的https链接，去掉重复的
grep -o 'https://[^[:space:]]*' ubuntu-arm64/debootstrap/debootstrap.log | sort | uniq
grep -o 'https://[^[:space:]]*' ubuntu-arm64/debootstrap/debootstrap.log | sort | uniq | wc -l
# rsync可以用http/https的代理
export RSYNC_PROXY=192.168.32.1:10809
bash rsync.sh
# 2、全部改为使用 rsync下载，并将所有deb包放到 /root/ubuntu-arm64/var/cache/apt/archives/partial/
# 3、重新执行: debootstrap --arch=arm64 --variant=minbase --foreign jammy ubuntu-arm64 https://mirrors.ustc.edu.cn/ubuntu-ports
```

6. debootstrap流程

```shell
/usr/sbin/debootstrap

	--foreign
		WHAT_TO_DO=finddebs dldebs save_variables first_stage
	--second-stage
		WHAT_TO_DO="second_stage"
	--print-debs
		WHAT_TO_DO="finddebs printdebs kill_target"
	--download-only
		WHAT_TO_DO="finddebs dldebs"
	
    . "$DEBOOTSTRAP_DIR/functions"
	
	WHAT_TO_DO="finddebs dldebs save_variables first_stage second_stage"
	
	# 判断一个目标是否在 WHAT_TO_DO，在，返回0(true)，返回1(false)
	am_doing_phase
		for x in "$@"; do
			if echo " $WHAT_TO_DO " | grep -q " $x "; then return 0; fi
		done
		return 1
	
	# 标准输出被重定向到 debootstrap/debootstrap.log
	exec >>"$TARGET/debootstrap/debootstrap.log"
	exec 2>&1
	
	. "$SCRIPT"
	
	am_doing_phase finddebs     => download_indices work_out_debs all_debs="$required $base"
	am_doing_phase printdebs    => echo "$all_debs"
	am_doing_phase dldebs       => download "$all_debs" => /usr/share/debootstrap/functions: DOWNLOAD_DEBS => download_main/download_release => get => just_get => wgetprogress => wget
	am_doing_phase first_stage  => first_stage_install  
	am_doing_phase second_stage => second_stage_install => /usr/share/debootstrap/scripts/jammy:  => in_target /bin/true   先验证 true 二进制是否正确
														=> setup_dynamic_devices
	                                                    => setup_proc => /usr/share/debootstrap/functions: in_target => $CHROOT_CMD $@ => chroot $TARGET $@
	                                                    => x_core_install base-passwd => in_target dpkg --force-depends --install $(debfor "$@")
						                                => x_core_install base-files
                                                        => x_core_install dpkg
														=> x_core_install $LIBC

/usr/share/debootstrap/functions
/usr/share/debootstrap/scripts/jammy
```

7. rsync下载deb脚本

```shell
#!/bin/bash

mkdir /home/root
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/a/acl/libacl1_2.3.1-1_arm64.deb                                  /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/a/adduser/adduser_3.118ubuntu5_all.deb                           /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/a/apt/apt_2.4.5_arm64.deb                                        /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/a/apt/libapt-pkg6.0_2.4.5_arm64.deb                              /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/a/attr/libattr1_2.5.1-1build1_arm64.deb                          /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/a/audit/libaudit-common_3.0.7-1build1_all.deb                    /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/a/audit/libaudit1_3.0.7-1build1_arm64.deb                        /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/b/base-files/base-files_12ubuntu4_arm64.deb                      /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/b/base-passwd/base-passwd_3.5.52build1_arm64.deb                 /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/b/bash/bash_5.1-6ubuntu1_arm64.deb                               /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/b/bzip2/libbz2-1.0_1.0.8-5build1_arm64.deb                       /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/c/ca-certificates/ca-certificates_20211016_all.deb               /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/c/cdebconf/libdebconfclient0_0.261ubuntu1_arm64.deb              /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/c/coreutils/coreutils_8.32-4.1ubuntu1_arm64.deb                  /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/d/dash/dash_0.5.11+git20210903+057cd650a4ed-3build1_arm64.deb    /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/d/db5.3/libdb5.3_5.3.28+dfsg1-0.8ubuntu3_arm64.deb               /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/d/debconf/debconf_1.5.79ubuntu1_all.deb                          /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/d/debianutils/debianutils_5.5-1ubuntu2_arm64.deb                 /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/d/diffutils/diffutils_3.8-0ubuntu2_arm64.deb                     /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/d/dpkg/dpkg_1.21.1ubuntu2_arm64.deb                              /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/e/e2fsprogs/e2fsprogs_1.46.5-2ubuntu1_arm64.deb                  /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/e/e2fsprogs/libcom-err2_1.46.5-2ubuntu1_arm64.deb                /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/e/e2fsprogs/libext2fs2_1.46.5-2ubuntu1_arm64.deb                 /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/e/e2fsprogs/libss2_1.46.5-2ubuntu1_arm64.deb                     /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/e/e2fsprogs/logsave_1.46.5-2ubuntu1_arm64.deb                    /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/f/findutils/findutils_4.8.0-1ubuntu3_arm64.deb                   /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/g/gcc-12/gcc-12-base_12-20220319-1ubuntu1_arm64.deb              /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/g/gcc-12/libgcc-s1_12-20220319-1ubuntu1_arm64.deb                /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/g/gcc-12/libstdc++6_12-20220319-1ubuntu1_arm64.deb               /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/g/glibc/libc-bin_2.35-0ubuntu3_arm64.deb                         /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/g/glibc/libc6_2.35-0ubuntu3_arm64.deb                            /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/g/gmp/libgmp10_6.2.1+dfsg-3ubuntu1_arm64.deb                     /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/g/gnupg2/gpgv_2.2.27-3ubuntu2_arm64.deb                          /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/g/gnutls28/libgnutls30_3.7.3-4ubuntu1_arm64.deb                  /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/g/grep/grep_3.7-1build1_arm64.deb                                /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/g/gzip/gzip_1.10-4ubuntu4_arm64.deb                              /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/h/hostname/hostname_3.23ubuntu2_arm64.deb                        /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/i/init-system-helpers/init-system-helpers_1.62_all.deb           /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/k/keyutils/libkeyutils1_1.6.1-2ubuntu3_arm64.deb                 /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/k/krb5/libgssapi-krb5-2_1.19.2-2_arm64.deb                       /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/k/krb5/libk5crypto3_1.19.2-2_arm64.deb                           /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/k/krb5/libkrb5-3_1.19.2-2_arm64.deb                              /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/k/krb5/libkrb5support0_1.19.2-2_arm64.deb                        /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/l/lsb/lsb-base_11.1.0ubuntu4_all.deb                             /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/l/lz4/liblz4-1_1.9.3-2build2_arm64.deb                           /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/libc/libcap-ng/libcap-ng0_0.7.9-2.2build3_arm64.deb              /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/libc/libcap2/libcap2_2.44-1build3_arm64.deb                      /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/libf/libffi/libffi8_3.4.2-4_arm64.deb                            /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/libg/libgcrypt20/libgcrypt20_1.9.4-3ubuntu3_arm64.deb            /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/libg/libgpg-error/libgpg-error0_1.43-3_arm64.deb                 /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/libi/libidn2/libidn2-0_2.3.2-2build1_arm64.deb                   /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/libn/libnsl/libnsl2_1.3.0-2build2_arm64.deb                      /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/libs/libseccomp/libseccomp2_2.5.3-2ubuntu2_arm64.deb             /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/libs/libselinux/libselinux1_3.3-1build2_arm64.deb                /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/libs/libsemanage/libsemanage-common_3.3-1build2_all.deb          /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/libs/libsemanage/libsemanage2_3.3-1build2_arm64.deb              /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/libs/libsepol/libsepol2_3.3-1build1_arm64.deb                    /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/libt/libtasn1-6/libtasn1-6_4.18.0-4build1_arm64.deb              /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/libt/libtirpc/libtirpc-common_1.3.2-2build1_all.deb              /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/libt/libtirpc/libtirpc3_1.3.2-2build1_arm64.deb                  /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/libu/libunistring/libunistring2_1.0-1_arm64.deb                  /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/libx/libxcrypt/libcrypt1_4.4.27-1_arm64.deb                      /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/libz/libzstd/libzstd1_1.4.8+dfsg-3build1_arm64.deb               /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/m/mawk/mawk_1.3.4.20200120-3_arm64.deb                           /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/n/ncurses/libncurses6_6.3-2_arm64.deb                            /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/n/ncurses/libncursesw6_6.3-2_arm64.deb                           /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/n/ncurses/libtinfo6_6.3-2_arm64.deb                              /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/n/ncurses/ncurses-base_6.3-2_all.deb                             /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/n/ncurses/ncurses-bin_6.3-2_arm64.deb                            /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/n/nettle/libhogweed6_3.7.3-1build2_arm64.deb                     /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/n/nettle/libnettle8_3.7.3-1build2_arm64.deb                      /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/o/openssl/libssl3_3.0.2-0ubuntu1_arm64.deb                       /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/o/openssl/openssl_3.0.2-0ubuntu1_arm64.deb                       /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/p/p11-kit/libp11-kit0_0.24.0-6build1_arm64.deb                   /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/p/pam/libpam-modules-bin_1.4.0-11ubuntu2_arm64.deb               /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/p/pam/libpam-modules_1.4.0-11ubuntu2_arm64.deb                   /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/p/pam/libpam-runtime_1.4.0-11ubuntu2_all.deb                     /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/p/pam/libpam0g_1.4.0-11ubuntu2_arm64.deb                         /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/p/pcre2/libpcre2-8-0_10.39-3build1_arm64.deb                     /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/p/pcre3/libpcre3_8.39-13build5_arm64.deb                         /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/p/perl/perl-base_5.34.0-3ubuntu1_arm64.deb                       /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/p/procps/libprocps8_3.3.17-6ubuntu2_arm64.deb                    /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/p/procps/procps_3.3.17-6ubuntu2_arm64.deb                        /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/s/sed/sed_4.8-1ubuntu2_arm64.deb                                 /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/s/sensible-utils/sensible-utils_0.0.17_all.deb                   /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/s/shadow/login_4.8.1-2ubuntu2_arm64.deb                          /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/s/shadow/passwd_4.8.1-2ubuntu2_arm64.deb                         /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/s/systemd/libsystemd0_249.11-0ubuntu3_arm64.deb                  /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/s/systemd/libudev1_249.11-0ubuntu3_arm64.deb                     /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/s/sysvinit/sysvinit-utils_3.01-1ubuntu1_arm64.deb                /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/t/tar/tar_1.34+dfsg-1build3_arm64.deb                            /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/u/ubuntu-keyring/ubuntu-keyring_2021.03.26_all.deb               /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/u/usrmerge/usrmerge_25ubuntu2_all.deb                            /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/u/util-linux/bsdutils_2.37.2-4ubuntu3_arm64.deb                  /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/u/util-linux/libblkid1_2.37.2-4ubuntu3_arm64.deb                 /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/u/util-linux/libmount1_2.37.2-4ubuntu3_arm64.deb                 /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/u/util-linux/libsmartcols1_2.37.2-4ubuntu3_arm64.deb             /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/u/util-linux/libuuid1_2.37.2-4ubuntu3_arm64.deb                  /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/u/util-linux/mount_2.37.2-4ubuntu3_arm64.deb                     /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/u/util-linux/util-linux_2.37.2-4ubuntu3_arm64.deb                /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/x/xxhash/libxxhash0_0.8.1-1_arm64.deb                            /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/x/xz-utils/liblzma5_5.2.5-2ubuntu1_arm64.deb                     /home/root/
rsync rsync://mirrors.ustc.edu.cn/ubuntu-ports/pool/main/z/zlib/zlib1g_1.2.11.dfsg-2ubuntu9_arm64.deb                     /home/root/
```

