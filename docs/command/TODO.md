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

- overleaf

- cs444 Compiler Construction

- `overleaf-toolkit -> ./bin/init ./bin/up  ./bin/start  ./bin/stop`

- `http://localhost/launchpad`

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

- 串口多路显示: `-chardev pty,path=/tmp/pty,id=pty0 -chardev vc,id=vc0 -chardev hub,id=hub0,chardevs.0=pty0,chardevs.1=vc0 -device virtconsole,chardev=hub0 -vnc :0`

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