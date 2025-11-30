# FAQ

### 1、双显示屏问题

```shell
qemu-system-aarch64 -M virt,gic-version=3 -m 16G -cpu cortex-a72  -smp 4 -kernel Image -append "console=ttyAMA0 nokaslr root=/dev/vda rw" -device e1000e,netdev=tap0 -netdev tap,id=tap0,ifname=tap0,script=no,downscript=no -serial stdio -monitor none -drive format=raw,file=ubuntu22_arm64.img -device qemu-xhci -device usb-mouse -device usb-kbd -device usb-tablet -s  -display gtk,gl=on -device virtio-gpu,id=gpu,edid=on,xres=1920,yres=1080,max_outputs=2
```

- 问题：[双显示屏只有一个屏幕enabled，另一个必须在gtk屏幕的view选项使能，不能用sdl](https://gitlab.com/qemu-project/qemu/-/issues/1107)

- 补丁: [virtio-gpu: Add an option to connect all outputs on startup](https://lists.nongnu.org/archive/html/qemu-devel/2023-03/msg03149.html)

- 解决方案:

 ```shell
 # 核心问题：图形界面无法持久化
 xrandr -display :0 --output Virtual-2 --auto --same-as Virtual-1
 xrandr -display :0 --output Virtual-2 --auto --right-of Virtual-1
 xrandr -display :0 --output Virtual-2 --mode 1920x1080
 xrandr -display :0 --output Virtual-2 --off
 xrandr -display :0 --output Virtual-2 --auto
 
 # 1)添加connect_outputs补丁，启动时自动使能；然后执行xrandr
 ./qemu-6.2.0/build/qemu-system-aarch64 -M virt,gic-version=3 -m 16G -cpu cortex-a72  -smp 4 -kernel Image -append "console=ttyAMA0 nokaslr root=/dev/vda rw" -device e1000e,netdev=tap0 -netdev tap,id=tap0,ifname=tap0,script=no,downscript=no -serial stdio -monitor none -drive format=raw,file=ubuntu22_arm64.img -device qemu-xhci -device usb-mouse -device usb-kbd -device usb-tablet -s  -vnc :0,head=0,display=gpu -vnc :1,head=1,display=gpu -device virtio-gpu,id=gpu,edid=on,xres=1920,yres=1080,max_outputs=2,connect_outputs=on
 
 # 2)在gtk view选项卡手动使能Virtual-2，再执行xrandr
 qemu-system-aarch64 -M virt,gic-version=3 -m 16G -cpu cortex-a72  -smp 4 -kernel Image -append "console=ttyAMA0 nokaslr root=/dev/vda rw" -device e1000e,netdev=tap0 -netdev tap,id=tap0,ifname=tap0,script=no,downscript=no -serial stdio -monitor none -drive format=raw,file=ubuntu22_arm64.img -device qemu-xhci -device usb-mouse -device usb-kbd -device usb-tablet -s  -display gtk -device virtio-gpu,id=gpu,edid=on,xres=1920,yres=1080,max_outputs=2
 
 # 3)运行后使用HMP手动使能？
 ```

  