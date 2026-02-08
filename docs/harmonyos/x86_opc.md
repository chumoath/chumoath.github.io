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

# 使用 tap 网口
qemu-system-x86_64 --enable-kvm -machine q35,accel=kvm,vmport=off -m 16G -smp 8 -drive if=none,file=updater.img,format=raw,id=updater,index=0 -device virtio-blk-pci,drive=updater -drive if=none,file=system.img,format=raw,id=system,index=1 -device virtio-blk-pci,drive=system -drive if=none,file=vendor.img,format=raw,id=vendor,index=2 -device virtio-blk-pci,drive=vendor -drive if=none,file=userdata.img,format=raw,id=userdata,index=3 -device virtio-blk-pci,drive=userdata -append "loglevel=1 ip=192.168.33.2:255.255.255.0::eth0:off sn=0023456789 console=tty0 console=ttyS0 init=/bin/init ohos.boot.hardware=x86_general root=/dev/ram0 rw ohos.required_mount.system=/dev/block/vdb@/usr@ext4@ro,barrier=1@wait,required ohos.required_mount.vendor=/dev/block/vdc@/vendor@ext4@ro,barrier=1@wait,required ohos.required_mount.misc=/dev/block/vda@/misc@none@none=@wait,required" -kernel bzImage -initrd ramdisk.img -nographic -vga none -device virtio-gpu-pci,max_outputs=1,xres=1024,yres=768,addr=08.0  -device virtio-mouse-pci -device virtio-keyboard-pci -device es1370  -k en-us  -display gtk,gl=off -device e1000e,netdev=tap0 -netdev tap,id=tap0,ifname=tap0,script=no,downscript=no -serial telnet::55555,server,nowait,nodelay -monitor none
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
# host网络配置
ip tuntap add tap0 mode tap group 0
ip link set dev tap0 up
ip addr add 192.168.33.1/24 dev tap0
iptables -A POSTROUTING -t nat -j MASQUERADE -s 192.168.33.0/24
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -P FORWARD ACCEPT

# guest配置网络
ifconfig eth0 192.168.33.2/24 up

# host连接guest
ssh 192.168.33.2  # 密码 123456

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

# 8、DELL 3579 G3适配鸿蒙

```shell
# DELL 3579固态硬盘为NVME nvmen0，机械硬盘为 sda; 使用固态硬盘的USB为 sdb，但是不能引导，U盘未知
# 1、挂载物理硬盘
wsl --shutdown
wsl --mount \\.\PHYSICALDRIVE1 --bare
wsl -d Ubuntu-22.04

# 2、分区/镜像烧录 - 同qemu_x86_64，最大问题是grub的root，当前仍为 hd0,gpt1，可以使用

# 3、配置显示 - 否则缩放太过
# /mnt/etc/window/resources/display_manager_config.xml
<dpi>240</dpi>

# 4、BIOS - F2进入BIOS配置，禁用 Secure boot
# 5、启动  - F12，选择机械硬盘启动
# 6、显卡为集成显卡(Intel(R) UHD Graphics 630)，N卡没有驱动；内核显卡驱动为 i915，vendor:device (8086:3e9b)
驱动配置：CONFIG_DRM_I915
驱动路径：drivers/gpu/drm/i915/
驱动pciidlist: 
    drivers/gpu/drm/i915/i915_pci.c
    	static const struct pci_device_id pciidlist[] = {
    		INTEL_CFL_H_GT2_IDS(&cfl_gt2_info),
    		{0, 0, 0}
    	};
    include/drm/i915_pciids.h
        #define INTEL_CFL_H_GT2_IDS(info) \
        	INTEL_VGA_DEVICE(0x3E9B, info), /* Halo GT2 */ \
        	INTEL_VGA_DEVICE(0x3E94, info)  /* Halo GT2 */
```

# 9、A卡适配鸿蒙

- 添加内核启动参数，开启 DRM_ATOMIC

```shell
# radeon.si_support=0 amdgpu.si_support=1 radeon.cik_support=0 amdgpu.cik_support=1: 直接让radeon的驱动返回，使用amdgpu驱动
# amdgpu.dc=1：使能amdgpu的DRM_ATOMIC
radeon.si_support=0 amdgpu.si_support=1 radeon.cik_support=0 amdgpu.cik_support=1 amdgpu.dc=1
```

# 10、命令

```shell
# 1、重启桌面
service_control stop foundation
service_control stop render_service
service_control stop composer_host

service_control start composer_host
service_control start render_service
service_control start foundation        # (/system/bin/sa_main)

# 2、重新挂载为读写
mount -o remount,rw /vendor
mount -o remount,rw /chipset

# 3、测试没有virtio_gpu_dri文件，桌面是否亮
# 1) 桌面亮
mv /vendor/lib64/chipsetsdk/virtio_gpu_dri.so /vendor/lib64/chipsetsdk/virtio_gpu_dri.so.bak
mv /chipset/lib64/chipsetsdk/virtio_gpu_dri.so /chipset/lib64/chipsetsdk/virtio_gpu_dri.so.bak

# 2) 桌面不亮
mv /vendor/lib64/chipsetsdk/libgallium_dri.so /vendor/lib64/chipsetsdk/libgallium_dri.so.bak

# 4、手动启动服务
# 停止服务
service_control stop foundation
service_control stop render_service
service_control stop composer_host

# 手动启动
/vendor/bin/hdf_devhost -i 12 -n composer_host -p -8 -s 1 &
# /vendor/lib64/chipsetsdk/virtio_gpu_dri.so 不存在，加载 /vendor/lib64/chipsetsdk/libgallium_dri.so 
render_service &
service_control start foundation

# 停止手动启动
service_control stop foundation
killall render_service
killall composer_host
```

# 11、openharmony-6.0调试drm

```shell
# 0、启动composer_host，render_service启动后才会调用接口
# 1、构建静态链接的gdbserver
wget https://mirrors.tuna.tsinghua.edu.cn/gnu/gdb/gdb-12.1.tar.xz
LDFLAGS="-static" ../configure
make -j$(nproc)
file gdbserver/gdbserver

# 2、openharmony运行待调试程序 - 以composer_host为例
# host
sshpass -p 123456 scp gdbserver 192.168.33.2:/data/

# qemu
ifconfig eth0 192.168.33.2 up

service_control stop foundation
service_control stop render_service
service_control stop composer_host

/data/gdbserver 0.0.0.0:1234 /vendor/bin/hdf_devhost -i 12 -n composer_host -p -8 -s 1

# 3、host使用
ln -s /root/opc /home/openharmony
cd /opc/out/x86_general
gdb exe.unstripped/hdf/hdf_core/hdf_devhost
> target remote 192.168.33.2:1234
> b DrmDevice::Init
> b DrmDisplay::Init
> b DrmVsyncWorker::Init
> b DrmVsyncWorker::WaitNextVBlank
> c
```

# 12、参考链接

- [openharmony-dropbear](https://gitee.com/ohos-porting-communities/openharmony-dropbear)
- [vendor_opc](https://gitcode.com/ohos-porting-communities/vendor_opc)

# 13、openharmony-6.0调用链

- DrmDevice::Init

```shell
Thread 2 "OS_IPC_1_3987" hit Breakpoint 1, 0x00007fff755dd4a0 in OHOS::HDI::DISPLAY::DrmDevice::Init() () from target:/vendor/lib64/libdisplay_composer_vendor.z.so
(gdb) bt
#0  0x00007fff755dd4a0 in OHOS::HDI::DISPLAY::DrmDevice::Init() () from target:/vendor/lib64/libdisplay_composer_vendor.z.so
#1  0x00007fff755d48db in OHOS::HDI::DISPLAY::HdiDeviceInterface::DiscoveryDevice() () from target:/vendor/lib64/libdisplay_composer_vendor.z.so
#2  0x00007fff755d9573 in OHOS::HDI::DISPLAY::HdiSession::Init() () from target:/vendor/lib64/libdisplay_composer_vendor.z.so
#3  0x00007ffff6d0b7d0 in std::__h::__call_once(unsigned long volatile&, void*, void (*)(void*)) () from target:/lib64/chipset-sdk-sp/libc++.so
#4  0x00007fff755d92f6 in OHOS::HDI::DISPLAY::HdiSession::GetInstance() () from target:/vendor/lib64/libdisplay_composer_vendor.z.so
#5  0x00007fff7558639f in OHOS::HDI::DISPLAY::DisplayComposerVdiImpl::RegHotPlugCallback(void (*)(unsigned int, bool, void*), void*) () from target:/vendor/lib64/libdisplay_composer_vdi_impl.z.so
#6  0x00007fff75697560 in OHOS::HDI::Display::Composer::DisplayComposerService::RegHotPlugCallback(OHOS::sptr<OHOS::HDI::Display::Composer::V1_0::IHotPlugCallback> const&) () from target:/vendor/lib64/libdisplay_composer_service_1.3.z.so
#7  0x00007fff75993191 in OHOS::HDI::Display::Composer::V1_0::DisplayComposerStub::DisplayComposerStubRegHotPlugCallback_(OHOS::MessageParcel&, OHOS::MessageParcel&, OHOS::MessageOption&, OHOS::sptr<OHOS::HDI::Display::Composer::V1_0::IDisplayComposer>) ()
   from target:/vendor/lib64/libdisplay_composer_stub_1.0.z.so
#8  0x00007fff7580f074 in OHOS::HDI::Display::Composer::V1_3::DisplayComposerStub::OnRemoteRequest(unsigned int, OHOS::MessageParcel&, OHOS::MessageParcel&, OHOS::MessageOption&) () from target:/vendor/lib64/libdisplay_composer_stub_1.3.z.so
#9  0x00007ffff718aaa4 in OHOS::IPCObjectStub::SendRequestInner(unsigned int, OHOS::MessageParcel&, OHOS::MessageParcel&, OHOS::MessageOption&) () from target:/system/lib64/platformsdk/libipc_single.z.so
#10 0x00007ffff718b181 in OHOS::IPCObjectStub::SendRequest(unsigned int, OHOS::MessageParcel&, OHOS::MessageParcel&, OHOS::MessageOption&) () from target:/system/lib64/platformsdk/libipc_single.z.so
#11 0x00007fff7590466a in DisplayComposerDriverDispatch(HdfDeviceIoClient*, int, HdfSBuf*, HdfSBuf*) () from target:/vendor/lib64/libdisplay_composer_driver_1.0.z.so
#12 0x00007ffff6f0e1d6 in DeviceServiceStubDispatch () from target:/vendor/lib64/libhdf_host.z.so
#13 0x00007ffff6f4e34c in HdfRemoteServiceStub::OnRemoteRequest(unsigned int, OHOS::MessageParcel&, OHOS::MessageParcel&, OHOS::MessageOption&) () from target:/system/lib64/platformsdk/libhdf_ipc_adapter.z.so
#14 0x00007ffff718aaa4 in OHOS::IPCObjectStub::SendRequestInner(unsigned int, OHOS::MessageParcel&, OHOS::MessageParcel&, OHOS::MessageOption&) () from target:/system/lib64/platformsdk/libipc_single.z.so
#15 0x00007ffff718b181 in OHOS::IPCObjectStub::SendRequest(unsigned int, OHOS::MessageParcel&, OHOS::MessageParcel&, OHOS::MessageOption&) () from target:/system/lib64/platformsdk/libipc_single.z.so
#16 0x00007ffff71a726e in OHOS::BinderInvoker::GeneralServiceSendRequest(binder_transaction_data const&, OHOS::MessageParcel&, OHOS::MessageParcel&, OHOS::MessageOption&) () from target:/system/lib64/platformsdk/libipc_single.z.so
#17 0x00007ffff71a769c in OHOS::BinderInvoker::Transaction(binder_transaction_data_secctx&) () from target:/system/lib64/platformsdk/libipc_single.z.so
#18 0x00007ffff71a7df6 in OHOS::BinderInvoker::HandleCommandsInner(unsigned int) () from target:/system/lib64/platformsdk/libipc_single.z.so
#19 0x00007ffff71a6312 in OHOS::BinderInvoker::HandleCommands(unsigned int) () from target:/system/lib64/platformsdk/libipc_single.z.so
#20 0x00007ffff71a614a in OHOS::BinderInvoker::StartWorkLoop() () from target:/system/lib64/platformsdk/libipc_single.z.so
#21 0x00007ffff71a7f16 in OHOS::BinderInvoker::JoinThread(bool) () from target:/system/lib64/platformsdk/libipc_single.z.so
#22 0x00007ffff719dc05 in OHOS::IPCWorkThread::JoinThread(int, int) () from target:/system/lib64/platformsdk/libipc_single.z.so
#23 0x00007ffff719e0f3 in OHOS::IPCWorkThread::ThreadHandler(void*) () from target:/system/lib64/platformsdk/libipc_single.z.so
#24 0x00007ffff7fd04ee in start () from target:/lib/ld-musl-x86_64.so.1
#25 0x00007ffff7f4384b in __clone () from target:/lib/ld-musl-x86_64.so.1
#26 0x00007ffff719dcf0 in ?? () from target:/system/lib64/platformsdk/libipc_single.z.so
```

- DrmVsyncWorker::Init

```shell
#0  0x00007fff755e5f60 in OHOS::HDI::DISPLAY::DrmVsyncWorker::Init(int)@plt () from target:/vendor/lib64/libdisplay_composer_vendor.z.so
#1  0x00007fff755e2ec4 in void std::__h::__call_once_proxy[abi:v15004]<std::__h::tuple<OHOS::HDI::DISPLAY::DrmVsyncWorker::GetInstance()::$_1&&> >(void*) () from target:/vendor/lib64/libdisplay_composer_vendor.z.so
#2  0x00007ffff6d0b7d0 in std::__h::__call_once(unsigned long volatile&, void*, void (*)(void*)) () from target:/lib64/chipset-sdk-sp/libc++.so
#3  0x00007fff755e2785 in OHOS::HDI::DISPLAY::DrmVsyncWorker::GetInstance() () from target:/vendor/lib64/libdisplay_composer_vendor.z.so
#4  0x00007fff755e1661 in OHOS::HDI::DISPLAY::DrmDisplay::RegDisplayVBlankCallback(void (*)(unsigned int, unsigned long, void*), void*) () from target:/vendor/lib64/libdisplay_composer_vendor.z.so
#5  0x00007fff75586a6e in int OHOS::HDI::DISPLAY::HdiSession::CallDisplayFunction<unsigned int*, OHOS::HDI::Display::Composer::V1_0::DisplayModeInfo*>(unsigned int, int (OHOS::HDI::DISPLAY::HdiDisplay::*)(unsigned int*, OHOS::HDI::Display::Composer::V1_0::DisplayModeInfo*), unsigned int*, OHOS::HDI::Display::Composer::V1_0::DisplayModeInfo*) () from target:/vendor/lib64/libdisplay_composer_vdi_impl.z.so
#6  0x00007fff75587bce in OHOS::HDI::DISPLAY::DisplayComposerVdiImpl::RegDisplayVBlankCallback(unsigned int, void (*)(unsigned int, unsigned long, void*), void*) () from target:/vendor/lib64/libdisplay_composer_vdi_impl.z.so
#7  0x00007fff756993de in OHOS::HDI::Display::Composer::DisplayComposerService::RegDisplayVBlankCallback(unsigned int, OHOS::sptr<OHOS::HDI::Display::Composer::V1_0::IVBlankCallback> const&) () from target:/vendor/lib64/libdisplay_composer_service_1.3.z.so
#8  0x00007fff75993664 in OHOS::HDI::Display::Composer::V1_0::DisplayComposerStub::DisplayComposerStubRegDisplayVBlankCallback_(OHOS::MessageParcel&, OHOS::MessageParcel&, OHOS::MessageOption&, OHOS::sptr<OHOS::HDI::Display::Composer::V1_0::IDisplayComposer>) ()
   from target:/vendor/lib64/libdisplay_composer_stub_1.0.z.so
#9  0x00007fff7580f0ab in OHOS::HDI::Display::Composer::V1_3::DisplayComposerStub::OnRemoteRequest(unsigned int, OHOS::MessageParcel&, OHOS::MessageParcel&, OHOS::MessageOption&) () from target:/vendor/lib64/libdisplay_composer_stub_1.3.z.so
#10 0x00007ffff718aaa4 in OHOS::IPCObjectStub::SendRequestInner(unsigned int, OHOS::MessageParcel&, OHOS::MessageParcel&, OHOS::MessageOption&) () from target:/system/lib64/platformsdk/libipc_single.z.so
#11 0x00007ffff718b181 in OHOS::IPCObjectStub::SendRequest(unsigned int, OHOS::MessageParcel&, OHOS::MessageParcel&, OHOS::MessageOption&) () from target:/system/lib64/platformsdk/libipc_single.z.so
#12 0x00007fff7590466a in DisplayComposerDriverDispatch(HdfDeviceIoClient*, int, HdfSBuf*, HdfSBuf*) () from target:/vendor/lib64/libdisplay_composer_driver_1.0.z.so
#13 0x00007ffff6f0e1d6 in DeviceServiceStubDispatch () from target:/vendor/lib64/libhdf_host.z.so
#14 0x00007ffff6f4e34c in HdfRemoteServiceStub::OnRemoteRequest(unsigned int, OHOS::MessageParcel&, OHOS::MessageParcel&, OHOS::MessageOption&) () from target:/system/lib64/platformsdk/libhdf_ipc_adapter.z.so
#15 0x00007ffff718aaa4 in OHOS::IPCObjectStub::SendRequestInner(unsigned int, OHOS::MessageParcel&, OHOS::MessageParcel&, OHOS::MessageOption&) () from target:/system/lib64/platformsdk/libipc_single.z.so
#16 0x00007ffff718b181 in OHOS::IPCObjectStub::SendRequest(unsigned int, OHOS::MessageParcel&, OHOS::MessageParcel&, OHOS::MessageOption&) () from target:/system/lib64/platformsdk/libipc_single.z.so
#17 0x00007ffff71a726e in OHOS::BinderInvoker::GeneralServiceSendRequest(binder_transaction_data const&, OHOS::MessageParcel&, OHOS::MessageParcel&, OHOS::MessageOption&) () from target:/system/lib64/platformsdk/libipc_single.z.so
#18 0x00007ffff71a769c in OHOS::BinderInvoker::Transaction(binder_transaction_data_secctx&) () from target:/system/lib64/platformsdk/libipc_single.z.so
#19 0x00007ffff71a7df6 in OHOS::BinderInvoker::HandleCommandsInner(unsigned int) () from target:/system/lib64/platformsdk/libipc_single.z.so
#20 0x00007ffff71a6312 in OHOS::BinderInvoker::HandleCommands(unsigned int) () from target:/system/lib64/platformsdk/libipc_single.z.so
#21 0x00007ffff71a614a in OHOS::BinderInvoker::StartWorkLoop() () from target:/system/lib64/platformsdk/libipc_single.z.so
#22 0x00007ffff71a7f16 in OHOS::BinderInvoker::JoinThread(bool) () from target:/system/lib64/platformsdk/libipc_single.z.so
#23 0x00007ffff719dc05 in OHOS::IPCWorkThread::JoinThread(int, int) () from target:/system/lib64/platformsdk/libipc_single.z.so
#24 0x00007ffff719e0f3 in OHOS::IPCWorkThread::ThreadHandler(void*) () from target:/system/lib64/platformsdk/libipc_single.z.so
#25 0x00007ffff7fd04ee in start () from target:/lib/ld-musl-x86_64.so.1
#26 0x00007ffff7f4384b in __clone () from target:/lib/ld-musl-x86_64.so.1
#27 0x00007ffff719dcf0 in ?? () from target:/system/lib64/platformsdk/libipc_single.z.so
```

- DrmVsyncWorker::WaitNextVBlank正常显示时未被调用