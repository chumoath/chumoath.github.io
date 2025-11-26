# qemu

### 1、linux构建

```shell
# linux-5.10.240
# options: CONFIG_DRM CONFIG_DRM_VIRTIO_GPU

# build
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- O=build.arm64 defconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- O=build.arm64 menuconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- O=build.arm64 Image -j24
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- O=build.arm64 scripts_gdb
# make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- O=build.arm64 INSTALL_MOD_PATH=/root/sysroot modules_install
# make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- O=build.arm64 INSTALL_HDR_PATH=/root/sysroot headers_install

# compile_commands.json
cd build.arm64 && ../scripts/clang-tools/gen_compile_commands.py
```

### 2、qemu运行

```shell
# 一、host
# 1、host网络配置
ip tuntap add tap0 mode tap group 0
ip link set dev tap0 up
ip addr add 192.168.33.1/24 dev tap0
iptables -A POSTROUTING -t nat -j MASQUERADE -s 192.168.33.0/24
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -P FORWARD ACCEPT

# 2、guest运行
qemu-system-aarch64 -M virt,gic-version=3 \
-m 16G -cpu cortex-a72  -smp 4 -kernel Image \
-append "console=ttyAMA0 nokaslr root=/dev/vda rw" \
-device e1000e,netdev=tap0 -netdev tap,id=tap0,ifname=tap0,script=no,downscript=no \
-serial stdio -monitor none \
-drive format=raw,file=ubuntu22_arm64.img \
-device virtio-gpu-pci -display sdl \
-device qemu-xhci -device usb-mouse -device usb-kbd -device usb-tablet \
-s
# 鼠标漂移解决办法(只有该方法能解决)：
# -device virtio-mouse-pci -device virtio-keyboard-pci不能解决
-device usb-tablet

# 二、guest网络配置
# 1、临时配置
ip addr add 192.168.33.2/24 dev enp0s1
ip route add to 192.168.33.1 dev enp0s1
ip route add default via 192.168.33.1

# 2、持久化: 
# 1)配置静态/Manual IP (ip:192.168.33.2/24 gateway:192.168.33.1) 
# 2)Automatically connect
nmtui
```

### 3、vscode调试

- compile_commands.json

```shell
# Unknown argument: '-fno-allow-store-data-races'
# Unknown argument: '-fconserve-stack'
# unknown target ABI 'lp64'
sed -i 's@-fno-allow-store-data-races@@' compile_commands.json
sed -i 's@-fconserve-stack@@' compile_commands.json
sed -i 's@-mabi=lp64@@' compile_commands.json
sed -i 's@1UL@1@'build.arm64/scripts/gdb/linux/constants.py
```

- .vscode/launch.json

```json
// 先安装native debug插件
{
    "configurations": [
    {
        "type": "gdb",
        "request": "attach",
        "name": "linux_arm64",
        "executable": "${workspaceFolder}/build.arm64/vmlinux",
        "target": ":1234",
        "remote": true,
        "cwd": "${workspaceRoot}",
        "valuesFormatting": "parseText",
        "gdbpath": "/usr/bin/gdb-multiarch"
    }
    ]
}
```

- 技巧
  1. 先打断点，再连接gdbserver，否则可能断不住

### 4、ubuntu

```shell
# 一、准备根文件系统
mkdir ubuntu_arm64
# 安装基本包
debootstrap --arch=arm64 --variant=minbase --foreign jammy ubuntu_arm64 https://mirrors.ustc.edu.cn/ubuntu-ports

# 进入仿真环境
cp /usr/bin/qemu-aarch64-static ubuntu_arm64/usr/bin/
chroot ubuntu_arm64
/debootstrap/debootstrap --second-stage

# 配置镜像源
echo "deb https://mirrors.ustc.edu.cn/ubuntu-ports jammy main restricted universe multiverse" > /etc/apt/sources.list
echo "deb https://mirrors.ustc.edu.cn/ubuntu-ports jammy-updates main restricted universe multiverse" >> /etc/apt/sources.list
echo "deb https://mirrors.ustc.edu.cn/ubuntu-ports jammy-backports main restricted universe multiverse" >> /etc/apt/sources.list

# 安装软件包
apt update
apt install -y vim systemd iputils-ping network-manager
apt install -y pciutils usbutils openssh-server net-tools iproute2
apt install -y lightdm xfce4-session x11-apps libdrm-dev
apt install -y libatomic1 libkmod2 libgmp-dev libnuma-dev
apt install -y strace file bc make gcc pkg-config autoconf

# 设置systemd启动
ln -sf /lib/systemd/systemd /usr/sbin/init

sed -i '/BindsTo=*/d' /lib/systemd/system/serial-getty@.service
sed -i 's@^#\?PermitRootLogin.*@PermitRootLogin yes@' /etc/ssh/sshd_config

# nmcli device set enp0s3 managed yes 不生效, systemctl restart NetworkManager
# nmtui配置IP: 1)配置静态IP 2)Automatically connect
rm -f /usr/lib/NetworkManager/conf.d/10-globally-managed-devices.conf

# 配置lightdm默认session
echo "[Seat:*]" > /etc/lightdm/lightdm.conf.d/xfce.conf
# user-session=xfce 对应/usr/share/xsessions/xfce.desktop
# greeter-session=ukui-greeter对应/etc/lightdm/ukui-greeter.conf
echo "user-session=xfce" >> /etc/lightdm/lightdm.conf.d/xfce.conf

# 配置用户session，优先级最高；可以没有
echo "Desktop" > /root/.dmrc
echo "Session=xfce" >> /root/.dmrc

rm -f /root/.dmrc

(echo gxh; sleep 0.1; echo gxh) | passwd root
exit

# 二、打包镜像
dd if=/dev/zero of=ubuntu22_arm64.img bs=1G count=16
mke2fs -t ext4 -d ubuntu_arm64 ubuntu22_arm64.img
```

### 5、drm

- 获取Xorg使用的ioctl(DRM)

  ```shell
  apt install -y x11-apps libdrm-dev
  strace -ff -o task Xorg :0 -retro
  xclock -display :0
  
  grep DRM task* | cut -d' ' -f 2 | sort | uniq
  ```

- drm_ioctl

  ```shell
  DRM_IOCTL_DROP_MASTER
  DRM_IOCTL_ETNAVIV_GEM_INFO
  DRM_IOCTL_GET_CAP
  DRM_IOCTL_MODE_ADDFB
  DRM_IOCTL_MODE_CREATE_DUMB
  DRM_IOCTL_MODE_CURSOR
  DRM_IOCTL_MODE_CURSOR2
  DRM_IOCTL_MODE_DESTROY_DUMB
  DRM_IOCTL_MODE_DIRTYFB
  DRM_IOCTL_MODE_GETCONNECTOR
  DRM_IOCTL_MODE_GETCRTC
  DRM_IOCTL_MODE_GETENCODER
  DRM_IOCTL_MODE_GETPLANERESOURCES
  DRM_IOCTL_MODE_GETPROPBLOB
  DRM_IOCTL_MODE_GETPROPERTY
  DRM_IOCTL_MODE_GETRESOURCES
  DRM_IOCTL_MODE_LIST_LESSEES
  DRM_IOCTL_MODE_MAP_DUMB
  DRM_IOCTL_MODE_OBJ_GETPROPERTIES
  DRM_IOCTL_MODE_RMFB
  DRM_IOCTL_MODE_SETCRTC
  DRM_IOCTL_MODE_SETGAMMA
  DRM_IOCTL_MODE_SETPROPERTY
  DRM_IOCTL_SET_CLIENT_CAP
  DRM_IOCTL_SET_MASTER
  DRM_IOCTL_VERSION
  ```

  