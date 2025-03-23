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

- `mux=on`: 一个后端被多个前端使用，共享后端

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