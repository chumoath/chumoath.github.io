# qemu安装ubuntu14.04

## 一、基本安装

```shell
# 创建硬盘
qemu-img create -f qcow2 ubuntu.qcow2 50G

# 安装 - 安装不同版本的ubuntu，需要 -M help 找到相应的板子
qemu-system-i386 -M pc-i440fx-trusty -m 4G -smp 8 -accel kvm -hda ubuntu.qcow2 -cdrom ubuntu-14.04.6-server-i386.iso

# 配置
qemu-system-i386 -M pc-i440fx-trusty -m 4G -smp 8 -accel kvm -hda ubuntu.qcow2

# 运行
qemu-system-i386 -M pc-i440fx-trusty -m 4G -smp 8 -accel kvm -hda ubuntu.qcow2 -net nic,netdev=tap0,model=virtio -netdev tap,id=tap0,ifname=tap0,script=no,downscript=no -display none
```

## 二、网络配置

```shell
# host配置
ip tuntap add tap0 mode tap group 0
ip link set dev tap0 up
ip addr add 192.168.33.1/24 dev tap0
iptables -A POSTROUTING -t nat -j MASQUERADE -s 192.168.33.0/24
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -P FORWARD ACCEPT

# ubuntu配置
# 手动
ip addr add 192.168.33.2/24 dev eth0
ip route add to 192.168.33.1 dev eth0
ip route add default via 192.168.33.1

# 固化
/etc/network/interfaces
iface eth0 inet static
	address 192.168.33.2
	netmask 255.255.255.0
	gateway 192.168.33.1
	# dns-nameservers 192.168.54.44
```

## 三、nfs配置

```shell
# 挂载nfs
mkdir /root/nfs
apt install -y nfs-common
mount 192.168.33.1:/nfs /root/nfs

# 自动挂载nfs
/etc/fstab
192.168.33.1:/nfs /root/nfs nfs defaults,_netdev 0 0

# 测试 nfs 挂载
mount -a
```

## 四、misc

```shell
# ssh
PermitRootLogin yes

# /etdc/resolv.conf
配置为host的域名服务器即可

# 镜像源
/etc/apt/sources.list
sed -i 's/us.archive.ubuntu.com/mirrors.tuna.tsinghua.edu.cn/g' /etc/apt/sources.list
sed -i 's/security.ubuntu.com/mirrors.tuna.tsinghua.edu.cn/g' /etc/apt/sources.list

# 个人本地wsl需要配置代理才能访问, 原因未知

# 安装软件包
apt update
apt install -y nfs-common gcc git make libncurses5-dev
```

