# FAQ

### 1、双显示屏问题

```shell
qemu-system-aarch64 -M virt,gic-version=3 -m 16G -cpu cortex-a72  -smp 4 -kernel Image -append "console=ttyAMA0 nokaslr root=/dev/vda rw" -device e1000e,netdev=tap0 -netdev tap,id=tap0,ifname=tap0,script=no,downscript=no -serial stdio -monitor none -drive format=raw,file=ubuntu22_arm64.img -device qemu-xhci -device usb-mouse -device usb-kbd -device usb-tablet -s  -display gtk,gl=on -device virtio-gpu,id=gpu,edid=on,xres=1920,yres=1080,max_outputs=2
```

- 问题：[双显示屏只有一个屏幕enabled，另一个必须在gtk屏幕的view选项使能，不能用sdl](https://gitlab.com/qemu-project/qemu/-/issues/1107)

- 补丁: [virtio-gpu: Add an option to connect all outputs on startup](https://lists.nongnu.org/archive/html/qemu-devel/2023-03/msg03149.html)