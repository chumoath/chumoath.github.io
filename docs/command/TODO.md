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

- docker配置代理

  - `mkdir -p /etc/systemd/system/docker.service.d`

  - `touch /etc/systemd/system/docker.service.d/proxy.conf`

  - ```shell
    [Service]
    Environment="http_proxy=http://192.168.0.111:10809"
    Environment="https_proxy=http://192.168.0.111:10809"
    Environment="no_proxy=localhost,127.0.0.1,mirrors.tuna.tsinghua.edu.cn"
    ```

  - `systemctl daemon-reload`

  - `systemctl restart docker`

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

- nfs依赖的ko: `fscache.ko grace.ko lockd.ko nfs.ko nfs_acl.ko nfsv3.ko sunrpc.ko`

- nfs转发:

  - 转发TCP的2049端口即可
  - nfs权限问题，使用: `/etc/exports` => `/nfs *(rw,sync,no_subtree_check,insecure)`
  - `mount.nfs4 10.21.205.129:/nfs /root/nfs`

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
    # 配置网关
    ip route add to 192.168.7.2 dev tap0
    # 配置转发
    iptables -A POSTROUTING -t nat -j MASQUERADE -s 192.168.7.1/32
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

- webvirtcloud自动安装

  - ```shell
    wget https://raw.githubusercontent.com/retspen/webvirtcloud/master/install.sh
    chmod 744 install.sh
    # run with sudo or root user
    ./install.sh
    ```

  - 修改webvirtcloud.sh，添加ubuntu22.04，重新执行即可

  - 这些依赖必须都安装：`apt -y install git virtualenv python3-virtualenv python3-dev python3-lxml libvirt-dev zlib1g-dev libxslt1-dev nginx supervisor libsasl2-modules gcc pkg-config python3-guestfs libsasl2-dev libldap2-dev libssl-dev`

  - systemctl restart nginx

  - 出现503: systemctl restart supervisor