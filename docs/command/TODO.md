# TODO

- dmsetup

- losetup

- lvm

- exportfs

- showmount

- autoconf

- pushd、popd

- history

- ld-linux

- strace

- ltrace

- pstack

- ethtool

- man history -> 快捷命令

- man bash -> 快捷键

- man ld-linux.so

- 指定动态链接器

- 查看默认链接脚本: ld --verbose

- 修改main函数入口点

- 分析可执行文件链接脚本和动态库链接脚本

- -no-pie   -fcommon -PIC

- kernel debug

- user mode linux

- sphinx

- pelican

- vnc xrdp配置  Xorg、Xvnc、x11vnc、无显卡分辨率配置

- xrandr

- loginctl

- yum provides vncpasswd 查看哪个包有该文件

- firewall-cmd

- vmware pc添加串口，使用虚拟串口，用mobaxterm的serial连接

- boost构建

- xming使用

- qemu命令

- slirp

- libslirp-dev

- openbmc/phosphor-logging, Structured Logging

- i2ctool

- objdump

- objcopy

- readelf

- strings

- nm

- patchelf

- dos2unix

- find

- grep

- constructor

- \_\_attribute\_\_

- plymouth

- `hdparm --yes-i-know-what-i-am-doing --please-destroy-my-drive --fwdownload ME619HXELDF3TE_ISP_TEX38L.bin /dev/sda`

- bash内建命令

- Xorg同屏配置

- kernel shipped

- dd源代码

- `find . | cpio -H newc -o | xz -zc9 > ../initrd_boot.cpio.xz`

- `find . | cpio -H newc -o | gzip -c9 > ../initrd_boot.cpio.gz`

- `find . | cpio -H newc -o | gzip -c > ../initrd_boot.cpio.gz`

- `gzip -dc initrd_boot.cpio.gz | cpio -idmv`

- `gzip -dc initrd_boot.cpio.gz > initrd.cpio`

- `gunzip -k initrd_boot.cpio.gz`

- `isoinfo -d -i xxx.iso`

- `mkisofs -R -J -T -r -l -d -joliet-long -allow-multidot -allow-leading-dots -no-bak -V EulerOS-V2.0SP8-aarch64 -o ./Euler-SP8-aarch64-recovery.iso -e images/efiboot.img -no-emul-boot ./iso/`

- `lspci -k`  X1 X2 X3 X4，一条lane就可以双向通信

- `lspci -vvv`

- qemu PCI 拓扑结构

- qemu GICv3仿真

- llvm cpu0

- llvm pass

- qemu nesllvm-mos 6502

- io_uring

- fit image

- 6.s081

- MIT6.824

- cs144

- Nand2Tetris

- csapp labs

- 海思鲲鹏920，Hi1711

- 飞腾

- 鸿蒙PC，HarmonyOS-Next，OpenHarmony

- cs444 Compiler Construction

- container_of

- offset_of

- /dev/pts/*通过path指定路径，会自动创建软链接: `-serial chardev:char0 -chardev pty,id=char0,path=/tmp/pty`

- 创建i2c设备: `echo 0x30 > /sys/bus/i2c/devices/i2c-3/delete_device`, `echo hi1711_mcu_fan 0x30 > new_device`

- imx6ull跑entity_manager、pid controller、dbus-sensors、io_uring等

- device node使用流程，cat cp 调用流程，cp -a 为什么可以复制设备节点

- skip: 输入跳过; seek: 输出跳过，用0填充。`strace dd if=/dev/sda of=test.img bs=10 skip=10 seek=10 count=10`

- `smartctl -a /dev/sda`

- 查看chardev-vc的backend: `grep -rn chardev-vc qemu-9.2.0/ui/`

- 使用 pty: `-serial pty`, `minicom -D /dev/pts/*`,  `tio /dev/pts/*`

- vnc 5900端口显示串口日志: `-serial mon:vc -vnc :0`

- 串口共享前端(Since 10.0): 

  - `-chardev pty,path=/tmp/pty,id=pty0 -chardev vc,id=vc0 -chardev hub,id=hub0,chardevs.0=pty0,chardevs.1=vc0 -device virtconsole,chardev=hub0 -vnc :0`
  - `-chardev pty,path=/tmp/pty,id=pty0 -chardev vc,id=vc0 -chardev hub,id=hub0,chardevs.0=pty0,chardevs.1=vc0 -serial chardev:hub0`

- `mux=on`: 一个后端可以被多个前端使用，共享后端

- 查看字符设备后端的属性: `qemu-system-arm -chardev vc,help`

- 查看指定进程打开的文件: `lsof -p [pid]`

- vc后端接口: `vc_chr_open`,  `char_vc_class_init` `, vc_chr_write`

- serial 使用chardev: `-serial chardev:char0 -chardev pty,id=char0`,  `-chardev pty,id=char1 -monitor chardev:char1`

- serial使用vc: `-serial chardev:char0 -chardev vc,id=char0`

- chardev创建后端接口: `qemu_create_late_backends`, `foreach_device_config`, `serial_parse`

- chardev解析: 

  `qemu_chr_new_permit_mux_mon`

  ​     `qemu_chr_new_from_name` 

  ​         `if(strstart(filename, "chardev:", &p))`

  ​             `chr = qemu_chr_find(p); `

  ​            `return chr; `

  ​          `opts = qemu_chr_parse_compat(label, filename, permit_mux_mon);`

- 指定linux、qemu、llvm版本

- nfs流量转发

- pagemap获取物理地址

- Makefile使用的atime ctime mtime总结

- dtc转换:

  - `build/scripts/dtc/dtc -I dtb -O dts -o *.dts *.dtb`
  - `build/scripts/dtc/dtc -I dts -o dtb -o *.dtb *.dts`

- ko加载校验

- ko构建Makefile模板

- kernel避免有.git编译出来的kernel版本号结尾有+:

  - `make LOCALVERSION= ARCH=arm64 CROSS_COMPILE=aarch64-target-linux-gnu- O=build zImage`

- arm64的Image改为zImage

- 编译单个ko，ko依赖?

  `make CONFIG_DRM_HISI_HIBMC=m -C kernel/build M=kernel/drivers/gpu/drm/hisilicon/hibmc modules`

- ko调试获取加载地址:

  - `b do_init_module`
  - `p /x mod->sect_attrs->attrs[1]->address`
  - `add-symbol-file drivers/net/virtio_net.ko 0xffffffc040e000 -s .data 0xffffc0413000 -s .bss * -s .rodata *`

- 调试ko

  - `make scripts_gdb`
  - `lx-symbols`

- gdb调试命令及示例，可操作/复现:

  - `info file` 查看加载的文件
  - `info line mousedev_create` 查看符号所在的文件行号

- local.conf配置示例

  - `VIRTUAL-RUNTIME_obmc-sensors-hwmon ?= "dbus-sensors"`
  - `VIRTUAL-RUNTIME_obmc-inventory-manager = "entity-manager"`

- overleaf直接上传从github下载的zip，用Makefile直接构建

- docker搭建nfs服务器: `erichough/nfs-server`，docker映射qemu端口

- [overleaf安装中文字体](https://www.bilibili.com/opus/638167200353484805)

- tex编译

  - `apt install texlive-xetex`
  - `apt install latexmk`
  - `latexmk -f -xelatex main.tex`
  - 网页点击后查看执行的命令: `while [ 1 ]; do ps -ef | grep tex; sleep 0.1; done`

- LaTeX Error: File ctexart.cls not found.

  - `apt install texlive-lang-chinese`

- overleaf部署

  - `git clone https://github.com/overleaf/toolkit.git`
  - 初始化: `./bin/init`
  - 第一次启动: `./bin/up`
  - 进入web: `http://localhost/launchpad`
  - 后台运行: `./bin/start`
  - 停止: `./bin/stop`

- overleaf配置完整版

  - `docker exec -it sharelatex bash`
  - `export https_proxy=http://192.168.0.111:10809`
  - `export https_proxy=http://192.168.0.111:10809`
  - `tlmgr option repository https://ftp.math.utah.edu/pub/tex/historic/systems/texlive/2024/tlnet-final/`
  - `tlmgr update --self`
  - `tlmgr install scheme-full`

- 查看systemd启动流程

  - `systemd-analyze plot > plot.svg`

- qemu支持vnc

  - `ui/console-vc.c`
  - `meson.build -> pixman`
  - `apt install libpixman-1-dev`
  - `apt install tio minicom libslirp-dev`
  - `pip3 install tomli`

- evb-ast2600修改obmc-phosphor-image的密码为0penBmc

- ROS2

- iproute2

- legit

- agibot_x1_infer

- ss

- netlink

- af_packet

- rclcpp

- rlwrap

- stripcc

- bear

- `ss -x`查看对端inode: `unix_diag.ko`

- run/tap示例

- iptables处理

- pty原理与示例

- qemu tap配置网络

  - `ifconfig eth0 192.168.7.2 netmask 255.255.255.0 up`
  - `route add default gw 192.168.7.1 netmask 0.0.0.0`

- qemu tap配置

  - tapN(Host): ip = N * 2 + 1, gateway= N * 2 + 2 = ip + 1

  - Guest: client = gateway + 1 = tapnum * 2 + 2, gateway = tapnum * 2 + 1

  - Host和Guest相反，刚好互相走网关

  - `openbmc/scripts/runqemu-ifup 0`

    ```shell
    # 查看tap
    ip tuntap list
    # 创建tap
    ip tuntap add tap0 mode tap group 0
    # 配置tap的IP
    ip addr add 192.168.7.1/32 broadcast 192.168.7.255 dev tap0
    # up tap0
    ip link set dev tap0 up
    # 配置网关 - IP地址配置为 192.168.7.1/32，不会生成 192.168.7.0/24的路由表，所以必须手动配置路由表
    ip route add to 192.168.7.2 dev tap0
    # 配置转发 - 只需要配置guest的转发即可，不需要配置host的；iptables就是为了guest能够访问外网
    # iptables -A POSTROUTING -t nat -j MASQUERADE -s 192.168.7.1/32
    iptables -A POSTROUTING -t nat -j MASQUERADE -s 192.168.7.2/32
    
    # 配置
    echo 1 > /proc/sys/net/ipv4/ip_forward
    echo 1 > /proc/sys/net/ipv4/conf/tap0/proxy_arp
    
    iptables -P FORWARD ACCEPT
    ```

  - `openbmc/scripts/runqemu-ifdown tap0`

    ```shell
    # 删除tap
    ip tuntap del tap0 mode tap
    # 显示tap0
    ip link show tap0
    # 删除tap0
    ip link del tap0
    # 删除转发
    iptables -D POSTROUTING -t nat -j MASQUERADE -s 192.168.7.1/32
    iptables -D POSTROUTING -t nat -j MASQUERADE -s 192.168.7.2/32
    ```

  - 查看nat转发表: `iptables -t nat -L`

- if 根据命令的执行结果决定，不是根据输出决定

  - `if ./a.out; then ls; fi`

- test - check file types and compare values

  - [ 本质上是 test 命令
  - `test -n ""` -> $? 为 1
  - `test -n "hello"` -> $? 为 0
  - `if test -n "hello"; then ls; fi` 等同于 `if [ -n "hello" ]; then ls; fi`

- 从一堆rpm中查找某个文件

  - `find . -name "*.rpm" -exec sh -c "if rpm -qlp {} | grep "lsblk" > /dev/null; then echo {}; fi" \;`

- 正则表达式 - grep

- runqemu命令行显示串口

  - `runqemu serialstdio`

- runqemu使用user网络

  - `runqemu slirp` -> `which runqemu`: 查看check_args可知能配置哪些参数

- busybox启动ftp和telnet服务

  - `busybox tcpsvd -vE 0.0.0.0 21 busybox ftpd -w /tmp`
  - `busybox telnetd`

- losetup扫描一个.img的文件的多个分区

  - `losetup -P /dev/loop10 emmc.img`

- qemu快照

  - 保存快照: `(monitor) savevm hi1711`
  - 使用快照: `-loadvm hi1711`
  - snapshot保存在.qcow2中，因此，更换.qcow2没有以前的snapshot

- qemu-img转换raw和qcow2

  - raw转换为qcow2: `qemu-img convert -f raw -O qcow2 input.raw output.qcow2`
  - qcow2转换为raw: `qemu-img convert -f qcow2 -O raw input.qcow2 output.raw`

- 退出qemu的monitor后，qemu整体就会退出，所以，不能直接退出，退出连接即可

- telnet回到命令行: `ctrl + ]`

- `minicom -D /dev/pts/5`退出: `ctrl + a, x`

- obmc-phosphor-image密码配置

  - meta-phosphor/conf/distro/include/phosphor-defaults.inc
  - `usermod -p ${DEFAULT_OPENBMC_PASSWORD} root;`

- Yocto使用patch报错: `# OA Issue: Missing Upstream-Status in patch`

  - patch的Subject下添加: `Upstream-Status: Pending`

- qemu的 -kernel 和 -append 必须一起使用

- C++类的静态成员必须要在class外定义

- 静态变量调用函数初始化的时机

  - ```c
    int test
    {
        static int a = add(1, 2);
    }
    ```

- ssh转发

  - -g: 监听 0.0.0.0，不是127.0.0.1
  - -N: 不执行远程命令，只转发端口
  - -f: 后台运行
  - -L: 配置转发
  - `ssh -g -f -N -L proxy_port:server_ip:server_port root@[跳板机ip] -22`
  - `ssh -g -f -N -L 2221:192.168.3.30:23 root@10.21.199.146 -22`
  - `ssh -g -f -N -L 12345:/var/run/dbus/system_dbus_socket root@10.21.198.112 -22`
    - 可以转发，但是需要peer的sshd的支持，RTOS208的不支持

- Nginx的反向代理

  - Nginx接收客户端请求，然后将请求转发给后端的服务器，最后将后端服务器的响应返回给客户端

- FTP

  - 服务端: 21端口用于控制信息传输，发送命令
  - 被动模式：客户端发送PASV命令，服务端随机起数据端口，客户端连接服务端
  - 主动模式：客户端发送PORT命令，客户端发送数据端口，服务端主动连接

- 查看strace使用的所有系统调用

  - `cat task.* | cut -d"(" -f1 | sort | uniq`

- uniq

  - 查看出现一次的行：`sort file | uniq -u`
  - 查看出现多次的行：`sort file | uniq -d`
  - 去掉重复的行：`sort file | uniq`

- 列出支持的device

  - `qemu-system-arm --device help`

- 列出该device的配置选项

  - `qemu-system-arm --device e1000,help`

- 设备直通 -> Device Pass Through

- `/dev/kvm`: `apt install qemu-kvm`

- qemu运行ubuntu22.04: `initramfs unpacking failed: write error`

  - 分配内存过小，分配更多内存即可。

- webvirtcloud启动串口

  ```shell
  # /lib/systemd/system/serial-getty@.service
  systemctl enable serial-getty@ttyS0.service
  systemctl start serial-getty@ttyS0.service
  
  minicom -D /dev/pts/*
  ```

- openEuler kernel cmdline: 

  - `cgroup_disable=files apparmor=0 crashkernel=2048M,high logo.nologo selinux=0 selinux=0 console=ttyAMA0,115200`

- ttyS0和tty0同时显示内核日志

  - `/etc/default/grub`: `GRUB_CMDLINE_LINUX_DEFAULT="console=ttyS0 console=tty0 maybe-ubiquity"`
  - `update-grub`

- VGA 一般显示动画，所以不显示日志也可以

- OpenEuler plymouth

  - 查看当前系统支持的theme: `plymouth-set-default-theme -l`
  - 设置theme: `plymouth-set-default-theme longsight`
  - 需要修改的服务:
    - `plymouth-halt.service`
    - `plymouth-kexec.service`
    - `plymouth-poweroff.service`
    - `plymouth-reboot.service`
    - `plymouth-start.service`
  - 服务ExecStart添加参数: `--ignore-serial-consoles --kernel-command-line="splash quiet" --debug`

- libvirt默认iso/qcow2存在位置 /var/lib/libvirt/images

- taskset - 设置cpu亲和性

  - 亲和性绑定: `taskset -cp 2-3 <vhost-pid>`

- exec -> 替换shell的代码段，而不是fork新进程

  - `exec 84<> /dev/net/tun`: 打开文件到shell进程84文件描述符
  - `exec 84<> [file]`
  - `read -r -u 84 data`   =>  `-r`: 不解析转义字符  `-u`: 指定文件描述符读取，而不是从标准输入
  - `exec 84>&-` => 关闭shell的文件描述符 84

- virtio、vhost=on、vhost-user 优缺点

  - virtio: 兼容性好，性能低
  - vhost=on: 性能提升50%，需要内核支持
  - vhost-user: DPDK兼容，配置复杂

- webvirtcloud默认使用 NAT (TAP)网络

  - 网络后端配置
    - `-netdev tap,fd=84,id=hostnet0,vhost=on,vhostfd=86`
  - 192.168.122.* 网段
    - NAT转发表: `iptables -t nat -L`
      - `MASQUERADE tcp -- 192.168.122.0/24 ! 192.168.122.0/24 masq ports: 1024-65535`
      - `MASQUERADE udp -- 192.168.122.0/24 ! 192.168.122.0/24 masq ports: 1024-65535`
      - `MASQUERADE all -- 192.168.122.0/24 ! 192.168.122.0/24 masq ports: 1024-65535`
    - 网桥: `brctl show`
      - `virbr0 -> yes -> vnet19 vnet26`
    - TAP: `ip tuntap list`
      - `vnet19: tap vnet_hdr`
      - `vnet26: tap vnet_hdr`

- docker compose 版本问题

  - does not match any of the regexes: \`^x-\`
  - Not supported URL scheme http+docker
  - 解决方案：从docker-compose github下载最新的可执行文件，放到`/usr/bin/`即可

- 查看systemd的After参数：`man systemd | grep After`

- webvirtcloud的compute必须安装libvirtd，controller可以通过ssh隧道在compute节点执行命令；compute节点通过访问`/run/libvirt/libvirt-sock`管理虚拟机

- ip命令

  ```shell
  # 清除网络接口IP
  ip addr flush dev eth0
  # 配置IP
  ip addr add 172.17.0.3/16 dev eth0
  # 删除IP
  ip addr del 172.17.0.3/16 dev eth0
  # 查看路由表
  ip route show
  # enable/disable eth0
  ip link set eth0 down
  ip link set eth0 up
  # 清除ARP缓存
  ip neigh flush dev eth0
  # 清除路由表相关条目
  ip route flush dev eth0
  # 配置默认网关，默认网关掩码为 0.0.0.0
  ip route add default via 172.17.0.1
  # 删除默认网关
  ip route del default
  # 添加子网网关
  ip route add 10.8.0.0/24 via 10.8.0.1 dev eth0
  # 删除子网网关
  ip route del 10.8.0.0/24 via 10.8.0.1 dev eth0
  # 添加路由表
  ip route add 192.168.2.0/24 dev eth1
  # 删除路由表
  ip route del 192.168.2.0/24 dev eth1
  # 查看一个网络接口的所有ip
  ip addr show dev br-ext
  # 查看所有网络接口的ip
  ip a
  # 查看所有虚拟网卡接口(虚拟以太网设备 veth-pairs)
  ip addr show type veth
  # 查看VLAN接口
  ip addr show eth0.100
  ```

- 查看apt包的内容: `dpkg -L iputils-ping`

- nmcli

  - `nmcli device`
  - `nmcli connection show`

- 查看系统的socket连接，可与 `netstat -tunp 对照` -> `conntrack -L`

- 查看网口接口类型

  - `ethtool -i tap0`
  - `ethtool -i docker0`
  - `ethtool -i eth0`
  - `ethtool -S eth0` -> 网口报文统计，查看veth对端网口的 index

- strace 传递环境变量: `strace -ff -o task -E TEST_ENV=1 ls`

- 网络流量NAT可视化

- iptables

- plymouth

- hwmon

- 网桥配置vlan

- wireshark使用

- bpftrace

- 去掉wireguard的加密

- wireguard安装

  - `apt install wireguard`
  - 查看包依赖：`apt-cache depends wireguard` -> `Depends: wireguard-tools`

- 查看tar.gz里面的文件列表和权限：`tar -tvf plymouth.tar.gz`

- `ls -l /`查不到根目录的权限：`stat /`

- yocto所有ko都打到image

  ```shell
  # conf/local.conf
  INIT_MANAGER = "systemd"
  IMAGE_INSTALL:append = "iptables kernel-modules openssh bash bash-completion"
  ```

- 查看ulimit使用的系统调用

  ```shell
  type ulimit
  # ulimit is a shell builtin
  
  # 在shell内部执行命令，strace的是bash本身，因为是内建命令
  strace bash -c "ulimit -a"
  # 执行一个脚本
  bash xxx.sh
  
  # prlimit64(0, RLIMIT_NOFILE, NULL, {rlim_cur=1024, rlim_max=1024*1024}) -> open files
  # prlimit64(0, RLIMIT_NPROC, NULL, {rlim_cur=96103, rlim_max=96103}) -> max user processes
  # prlimit64(0, RLIMIT_STACK, NULL, {rlim_cur=8192*1024, rlim_max=RLIM64_INFINITY}) -> stack size
  # prlimit64(0, RLIMIT_CORE, NULL, {rlim_cur=0, rlim_max=RLIM64_INFINITY}) -> core file size 
  ```

- kvm

  - virt/lib/irqbypass.ko
  - kvm.ko
  - kvm-intel.ko

- 关闭内核基地址随机化

  - CONFIG_RANDOMIZE_BASE
  - nokaslr

- linux-yocto添加调试信息，生成vmlinux-gdb.py

  ```shell
  bitbake -c do_menuconfig linux-yocto
  bitbake linux-yocto
  bitbake core-image-minimal
  bitbake -c devshell linux-yocto
  make scripts_gdb
  ```

- MASQUERADE需要该ko: `xt_MASQUERADE`

- yocto拆分包全流程：以kernel-module为例

- `iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT`

- `iptables -A FORWARD -i br0 -o br0 -s 192.168.1.1 -d 192.168.1.2 -j ACCEPT`

- `iptables -A FORWARD -i br0 -o br0 -s 192.168.1.1 -d 192.168.1.3 -j DROP`

- `tcpdump -i br0 -nn -e vlan`

- `bridge monitor fdb`

- `bridge fdb show br br-ext`

- NAT是为了三层能收到报文响应，在同一个网段的一定能收到响应报文

- 虚拟机应用 -> eth0(Guest) -> 虚拟化层 -> tap0(Host) -> 宿主网络栈

- 通过vm1调试ip_forward网络拓扑

  ```shell
  # 目标: host <-> vm1; vm1 <-> vm2; host x> vm2
  br-ext: (192.168.33.2)
     veth0-qemu (不配IP)
     tap0 (不配IP)           -> vm1 (eth0 192.168.33.3)
     
  br-int: (不配IP)
     tap1 (不配IP)           -> vm1 (eth1 192.168.7.1)
     tap2 (不配IP)           -> vm2 (eth0 192.168.7.2)
  
  # qemu
  vm1: net0 -> tap0, net1 -> tap1
  vm2: net0 -> tap2
  ```
  
- yocto总流程
  - 组件: `do_build -> do_install -> image -> package -> *.rpm`
  - image: `core-image-minimal -> do_build -> 根据依赖关系从rpm包构建image`
  
- yocto根据编译出来的ko目录生成 `kernel-module-*`目标，所有 `kernel-module-*`都被设置为`kernel-modules`的依赖

- 修改 /etc/passwd，将/bin/bash改为/usr/bin/ipython3，即可登录使用ipython命令行

- wsl -d ubuntu不能重新启动，必须 wsl --shutdown

- perf 监控报文被丢弃

  ```shell
  # 1、生成 perf.data
  perf record -g -a -e skb:free_skb
  # 2、解析perf.data
  perf script
  # 3、查看支持的trace
  perf list
  ```

- 负数的小数表示

  - -0.5表示
  - 负数：表面值越大，负数越大(负号后面的数越小)
  - 正数：表面值越大，正数越大
  - 11111.0 -> -1
  - 11111.1 -> -0.5
  - 1 1111.1 = -2^4 + 2^3 + 2^2 + 2^1 + 2^0 + 2^-1 = -16+8+4+2+1+0.5 = -0.5

- so2的hvc*不能自动创建解决办法

  - `-chardev pty,id=ttyS0 -serial chardev:ttyS0`
  - `console=ttyS0`
  - `minicom -D /dev/pts/*`

- linux v3.0 so2调试配置

  - 需打开内核选项
    - X86_32, LGUEST_GUEST, PARAVIRT_GUEST, VIRTIO_PCI, VIRTIO_BLK, VIRTIO_NET
  - 构建内核
    - `make ARCH=i386 defconfig`
    - `make ARCH=i386 menuconfig`
    - `make  ARCH=i386 -j$(nproc)`
  - 有git仓需要加LOCALVERSION，否则内核版本号会多一个+
    - `make LOCALVERSION= ARCH=i386 -j$(nproc)`
  - 创建 hvc* 设备节点
    - `mknod /dev/hvc0 c 229 0`
    - `mknod /dev/hvc0 c 229 7`
  - 注意点
    - linux v3.0缺乏一些ext4特性，所以使用的是ext3的老应用，不能自动创建hvc*设备节点
    - linux v2.6.19-rc2开始即支持ext4，但是v3.0内核有不兼容的特性，即使ext4文件系统内容全部拷到ext3会提示内核太旧
  - yocto x86文件系统生成件 链接
    - [yocto-1.1.1 ext3](https://downloads.yoctoproject.org/releases/yocto/yocto-1.1.1/machines/qemu/qemux86/core-image-minimal-qemux86.ext3)
    - [yocto-3.4.4 ext4](https://downloads.yoctoproject.org/releases/yocto/yocto-3.4.4/machines/qemu/qemux86/core-image-minimal-qemux86.ext4)

- `udevadm info -a /dev/hvc0`

- so2命令

  - `./local.sh docker interactive --privileged`
  - `make boot`

- 设置cpu亲和性

  ```shell
  # 启动新进程
  taskset -c 0,1 [cmd]
  taskset -c 0-10 [cmd]
  numactl --physcpubind=0,1,2 [cmd]
  
  # 设置已有进程
  taskset -pc 0,1 [pid]
  
  # 查看进程的cpu亲和性
  taskset -cp [pid]
  cat /proc/[pid]/status
  # 第 0、20 个cpu
  # Cpus_allowed:   0100001
  # Cpus_allowed_list:      0,20
  
  # 第 0-27 个cpu
  # Cpus_allowed:   fffffff
  # Cpus_allowed_list:      0-27
  
  # 查看进程当前运行的cpu
  ps -o psr [pid]
  ```

- 条件编译
  - Makefile 引入 .config 配置：控制文件/目录参与编译
  - 生成的宏 include/generated/autoconf.h：控制代码片段参与编译
  
- win键的锁定和解锁

  - fn + win
  - 若 锁定，则 win键不能使用，比如 win + l 不能锁屏

- dpkg

  - 列出包状态：`dpkg -l | grep [pkg]`
  - 列出包安装到系统的内容和路径：`dpkg -L [pkg]`

- rpm

  - 查看系统安装的rpm包：`rpm -qa`
  - 查看一个rpm包的内容：`rpm -qlp [*.rpm]`
  - 安装一个rpm包：`rpm -ivh [*.rpm]`
  - 强制安装一个rpm包：`rpm --force --ivh [*.rpm]`
  - 卸载一个rpm包：`rpm -e [rpm包名]`
  - openEuler查看哪个包提供一个文件：`yum provides busybox`
  
- 查看当前系统的glibc版本

  - `getconf GNU_LIBC_VERSION`

- 解决 `grantpt` 告警问题

  - `man grantpt`

  - ```shell
    Since glibc 2.24:
        _XOPEN_SOURCE >= 500 || (_XOPEN_SOURCE && _XOPEN_SOURCE_EXTENDED)
    
    Glibc 2.23 and earlier:
    	_XOPEN_SOURCE
    
    # 例子 
    #  - 必须在文件的开头，否则 stdlib.h 可能会在其他头文件 include，_XOPEN_SOURCE 还未定义
    #define _XOPEN_SOURCE 600
    #include <stdlib.h>
    #define __USE_MISC
    #include <sys/ioctl.h>
    #include <termios.h>
    ```
  
- qemu串口使用telnet做后端

  - qemu 配置：`-serial telnet::55555,server,nowait,nodelay`
  - 连接：`telnet 127.0.0.1 55555`
  
- qemu使用用户层协议栈 - 映射guest端口到host

  - `qemu-system-arm -m 1024 -M ast2600-evb -nographic -drive file=./obmc-phosphor-image-evb-ast2600.static.mtd,format=raw,if=mtd -net nic -net user,hostfwd=:127.0.0.1:2222-:22,hostfwd=:127.0.0.1:2443-:443,hostfwd=udp:127.0.0.1:2623-:623,hostname=qemu`

- 使用runqemu运行openbmc的evb-ast2600

  ```shell
  # 1、配置网络
  # host
  runqemu
  ip addr add 192.168.7.1/32 broadcast 192.168.7.255 dev tap0
  ip link set dev tap0 up
  # 使用 192.168.7.1/32 不会自动生成路由表，必须手动配置
  ip route add to 192.168.7.2 dev tap0
  
  # guest
  ip addr add 192.168.7.2/32 broadcast 192.168.7.255 dev eth0
  ip link set dev eth0 up
  # 必须要配置路由表，配置默认网关前必须要先能通
  ip route add to 192.168.7.1 dev eth0
  ip route add default via 192.168.7.1
  
  # 2、访问dbus
  # guest
  socat TCP-LISTEN:12345,fork UNIX-CONNECT:/var/run/dbus/system_bus_socket &
  
  # host
  d-feet &
  # Connect to other bus -> tcp:host=192.168.7.2,port=12345
  ```

  