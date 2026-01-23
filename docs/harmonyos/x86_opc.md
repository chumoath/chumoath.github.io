# x86_opc

# 1、构建

```shell
# 1、获取docker镜像
mkdir -p opc/openharmony && cd opc/openharmony             # 防止每次重启docker都要重新下载工具链
docker pull swr.cn-south-1.myhuaweicloud.com/openharmony-docker/openharmony-docker:1.0.0
docker run -it -v $(pwd):/home/openharmony swr.cn-south-1.myhuaweicloud.com/openharmony-docker/openharmony-docker:1.0.0
apt install -y autoconf libtool-bin
pip3 install json5

# 2、拉代码
repo init -u https://gitee.com/ohos-porting-communities/manifest.git -b OpenHarmony-6.0-Release --no-repo-verify
repo sync -c
repo forall -c 'git lfs pull'
bash build/prebuilts_download.sh

# 3、全量构建
#X86 general
./build.sh --product-name x86_general --ccache
./build.sh --product-name x86_general --ccache --fast-rebuild

#X86 qemu
./build.sh --product-name x86_general --ccache --gn-args=is_qemu=true
./build.sh --product-name x86_general --ccache --gn-args=is_qemu=true --fast-rebuild

# 区别
# 1) vendor.img: /etc/fstabxxx sda4 -> vdd
# 2) window/display_manager_config.xml -> dpi
```

# 2、qemu运行

- 使用qemu的显示缩放比例过大：`修改vendor.img:/etc/window/resources/display_manager_config.xml`，dpi 改为 80

```shell
# cat /dev/input/mouse0  查看鼠标响应
# cat /proc/bus/input/devices
# 分辨率调为 1920x1080，速度太慢; gtk, vmport=on，则指针不能动; sdl, vmport=off, 没有指针
# vmport=off -> 禁用vmmouse，使用命令行指定的mouse

qemu-system-x86_64 \
--enable-kvm \
-machine q35,accel=kvm,vmport=off \
-m 16G \
-smp 8 \
-drive if=none,file=updater.img,format=raw,id=updater,index=0 \
-device virtio-blk-pci,drive=updater \
-drive if=none,file=system.img,format=raw,id=system,index=1 \
-device virtio-blk-pci,drive=system \
-drive if=none,file=vendor.img,format=raw,id=vendor,index=2 \
-device virtio-blk-pci,drive=vendor \
-drive if=none,file=userdata.img,format=raw,id=userdata,index=3 \
-device virtio-blk-pci,drive=userdata \
-append "loglevel=1 ip=192.168.33.2:255.255.255.0::eth0:off sn=0023456789 console=tty0 console=ttyS0 init=/bin/init ohos.boot.hardware=x86_general root=/dev/ram0 rw ohos.required_mount.system=/dev/block/vdb@/usr@ext4@ro,barrier=1@wait,required ohos.required_mount.vendor=/dev/block/vdc@/vendor@ext4@ro,barrier=1@wait,required ohos.required_mount.misc=/dev/block/vda@/misc@none@none=@wait,required" \
-kernel bzImage \
-initrd ramdisk.img \
-nographic \
-vga none \
-device virtio-gpu-pci,max_outputs=1,xres=1024,yres=768,addr=08.0  \
-device virtio-mouse-pci \
-device virtio-keyboard-pci \
-device es1370  \
-k en-us  \
-display gtk,gl=off \
-device e1000,netdev=net0 \
-netdev user,id=net0,hostfwd=tcp::2222-:22 \
-serial telnet::55555,server,nowait,nodelay \
-monitor none
```

# 3、单独构建

```shell
# 1、从BUILD.gn获取目标名称
# 2、从opc/out/x86_general/build.ninja确认
# 3、运行ninja

# eg:
# 1、单独构建内核
rm -rf /home/openharmony/out/x86_general/kernel
/home/openharmony/prebuilts/build-tools/linux-x86/bin/ninja -w dupbuild=warn -C /home/openharmony/out/x86_general images

# 2、单独构建libdisplay_buffer_vendor的so
# device/soc/opc/x86_general/hardware/display/BUILD.gn -> libdisplay_buffer_vendor
/home/openharmony/prebuilts/build-tools/linux-x86/bin/ninja -w dupbuild=warn -C /home/openharmony/out/x86_general libdisplay_buffer_vendor

# 3、手动构建内核
export PRODUCT_PATH=vendor/opc/x86_general
# device/board/opc/x86_general/kernel/build_kernel.sh
export ROOT_DIR=/home/openharmony
export PATH=${ROOT_DIR}/prebuilts/clang/ohos/linux-x86_64/llvm/bin/:${ROOT_DIR}/prebuilts/develop_tools/pahole/bin/:$PATH
make LLVM=1 LLVM_IAS=1 -C ${ROOT_DIR}/out/x86_general/kernel/OBJ/linux-6.6/ menuconfig
make LLVM=1 LLVM_IAS=1 -C ${ROOT_DIR}/out/x86_general/kernel/OBJ/linux-6.6/ bzImage -j24
make LLVM=1 LLVM_IAS=1 -C ${ROOT_DIR}/out/x86_general/kernel/OBJ/linux-6.6/ modules -j24
```

# 4、自动补全

```shell
# 生成compile_commands.json
```

# 5、启动日志获取

```shell
# 设置日志缓冲区，最大16M
hilog -G 16M

# 收集日志
hilog > /data/qemu_oh_boot.log
```

# 6、配置dropbear密码

```shell
# openharmony dropbear配置： /etc/init/dropbear.cfg
# 密码修改：修改DROPBEAR_PASSWORD参数

# 手动启动：dropbear_service dropbear -RB
```

# 7、qemu使用grub引导openharmony

- qemu启动

```shell
qemu-system-x86_64 -M q35,accel=kvm,vmport=off  -m 16G -smp 8  \
-bios /usr/share/OVMF/OVMF_CODE.fd \
-device e1000e,netdev=tap0 -netdev tap,id=tap0,ifname=tap0,script=no,downscript=no \
-hda openharmony.img  -nographic -vga none \
-device virtio-gpu-pci,max_outputs=1,xres=1024,yres=768,addr=08.0  \
-device virtio-mouse-pci -device virtio-keyboard-pci  \
-device es1370  -k en-us  -display gtk,gl=off \
-serial stdio  -monitor none -enable-kvm

qemu-system-x86_64 -M q35,accel=kvm,vmport=off  -m 16G -smp 8  \
-bios /usr/share/OVMF/OVMF_CODE.fd \
-device e1000e,netdev=tap0 -netdev tap,id=tap0,ifname=tap0,script=no,downscript=no \
-hda /dev/loop0  -nographic -vga none \
-device virtio-gpu-pci,max_outputs=1,xres=1024,yres=768,addr=08.0  \
-device virtio-mouse-pci -device virtio-keyboard-pci  \
-device es1370  -k en-us  -display gtk,gl=off \
-serial stdio  -monitor none -enable-kvm

qemu-system-x86_64 -M q35,accel=kvm,vmport=off  -m 16G -smp 8  \
-bios /usr/share/OVMF/OVMF_CODE.fd \
-device e1000e,netdev=tap0 -netdev tap,id=tap0,ifname=tap0,script=no,downscript=no \
-hda /dev/sdd  -nographic -vga none \
-device virtio-gpu-pci,max_outputs=1,xres=1024,yres=768,addr=08.0  \
-device virtio-mouse-pci -device virtio-keyboard-pci  \
-device es1370  -k en-us  -display gtk,gl=off \
-serial stdio  -monitor none -enable-kvm
```

- 重新构建内核 - 将硬盘的AHCI驱动打进内核

```shell
# device/board/opc/x86_general/kernel/configs/pocket2_oh_defconfig
-CONFIG_SATA_AHCI=m
+CONFIG_SATA_AHCI=y
 CONFIG_SATA_MOBILE_LPM_POLICY=3
-CONFIG_SATA_AHCI_PLATFORM=m
+CONFIG_SATA_AHCI_PLATFORM=y
 # CONFIG_AHCI_DWC is not set
 CONFIG_SATA_INIC162X=m
-CONFIG_SATA_ACARD_AHCI=m
+CONFIG_SATA_ACARD_AHCI=y
 CONFIG_SATA_SIL24=m
 CONFIG_ATA_SFF=y
```

- 制作镜像 - 使用文件作为磁盘

```shell
dd if=/dev/zero    of=openharmony.img bs=1G count=20

# 1、分区
losetup -P /dev/loop0 openharmony.img

fdisk /dev/loop0
# 创建GPT分区表
> g
# 新建分区
> n
# loop0p1 500M
# loop0p2 2G
# loop0p3 2G
# loop0p4 2G
# loop0p5 2G

# 修改loop0p1的分区类型
> t -> 1(EFI)
# 打印分区表
> p
# 写入分区表
> w

# 2、准备grub启动
# 格式化分区
mkfs.vfat /dev/loop0p1

mount /dev/loop0p1 /mnt
cp -rfa iso/EFI /mnt/

# 复制 bzImage ramdisk.img
cp bzImage /mnt/
cp ramdisk.img /mnt/

# 3、准备其他分区
dd if=updater.img  of=/dev/loop0p2
dd if=system.img   of=/dev/loop0p3
dd if=vendor.img   of=/dev/loop0p4
dd if=userdata.img of=/dev/loop0p5

# 4、适配显示和分区挂载
mount /dev/loop0p4 /mnt
# /mnt/etc/fstab.x86_general - 否则初始化失败

/dev/block/sda5  /data  ext4  discard,noatime,nosuid,nodev,usrquota wait,check,fileencryption=software,quot

# /mnt/etc/window/resources/display_manager_config.xml - 否则桌面会放大
<dpi>80</dpi>
```

- 制作镜像 - 使用物理硬盘

```shell
# 0、wsl挂载物理硬盘
wsl --shutdown
wsl --mount \\.\PHYSICALDRIVE1 --bare
wsl -d Ubuntu-22.04

# 1、准备grub引导
mkfs.vfat /dev/sdd1
cp -rfa iso/EFI /mnt/

# 复制 bzImage ramdisk.img
cp bzImage /mnt/
cp ramdisk.img /mnt/

# 2、准备其他分区
dd if=updater.img  of=/dev/sdd2
dd if=system.img   of=/dev/sdd3
dd if=vendor.img   of=/dev/sdd4
dd if=userdata.img of=/dev/sdd5

# 3、适配显示和分区挂载
mount /dev/sdd4 /mnt
# /mnt/etc/fstab.x86_general - 否则初始化失败

# 虚拟机里面还是sda，所以不变
/dev/block/sda5  /data  ext4  discard,noatime,nosuid,nodev,usrquota wait,check,fileencryption=software,quot

# /mnt/etc/window/resources/display_manager_config.xml - 否则桌面会放大
<dpi>80</dpi>
```

- /mnt/EFI/BOOT/grub.cfg

```shell
set default="0"

function load_video {
  insmod efi_gop
  insmod efi_uga
  insmod video_bochs
  insmod video_cirrus
  insmod all_video
}

load_video
set gfxpayload=keep
insmod gzio
insmod part_gpt
insmod ext2

set timeout=60
### END /etc/grub.d/00_header ###

menuentry 'openharmony 6.0'  --class openharmony --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-6.6.0-72.6.0.56.oe2503.x86_64-advanced-85e4c7d3-83f3-44ae-9137-53b05d7b44ab' {
        load_video
        set gfxpayload=keep
        insmod gzio
        insmod part_gpt
        insmod ext2
        # 必须指定bzImage,ramdisk.img所在的硬盘和分区
        set root='hd0,gpt1'

        echo    'Loading bzImage'
        linux   /bzImage loglevel=1 ip=192.168.33.2:255.255.255.0::eth0:off sn=0023456789 console=tty0 console=ttyS0 init=/bin/init ohos.boot.hardware=x86_general root=/dev/ram0 rw ohos.required_mount.system=/dev/block/sda3@/usr@ext4@ro,barrier=1@wait,required ohos.required_mount.vendor=/dev/block/sda4@/vendor@ext4@ro,barrier=1@wait,required ohos.required_mount.misc=/dev/block/sda2@/misc@none@none=@wait,required
        echo    'Loading initial ramdisk ...'
        initrd  /ramdisk.img
}
```

# 8、参考链接

- [openharmony-dropbear](https://gitee.com/ohos-porting-communities/openharmony-dropbear)

- [vendor_opc](https://gitcode.com/ohos-porting-communities/vendor_opc)