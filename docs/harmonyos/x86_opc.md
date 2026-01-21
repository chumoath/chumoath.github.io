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

# 7、参考链接

- [openharmony-dropbear](https://gitee.com/ohos-porting-communities/openharmony-dropbear)

- [vendor_opc](https://gitcode.com/ohos-porting-communities/vendor_opc)