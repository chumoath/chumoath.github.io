# Qemu PCI

### 1、qemu配置PCI

```shell
# 全部使用pcie-root-port
qemu-system-aarch64 -M virt,gic-version=3 -m 16G -cpu cortex-a72 -smp 4 \
    -kernel Image -append "console=ttyAMA0 nokaslr root=/dev/vda rw video=Virtual-1:1920x1080@60me" \
    -device pcie-root-port,bus=pcie.0,id=seat0,addr=1.0,chassis=1 \
    -device pcie-root-port,bus=pcie.0,id=seat1,addr=2.0,chassis=2 \
    -device pcie-root-port,bus=pcie.0,id=seat2,addr=3.0,chassis=3 \
    -device pcie-root-port,bus=pcie.0,id=seat3,addr=4.0,chassis=4 \
    -device pcie-root-port,bus=pcie.0,id=seat4,addr=5.0,chassis=5 \
    -device virtio-gpu-pci,bus=seat0 \
    -device qemu-xhci,bus=seat1 \
    -device edu,dma_mask=0xffffffffffffffff,bus=seat2 \
    -device e1000e,netdev=tap0,bus=seat3 -netdev tap,id=tap0,ifname=tap0,script=no,downscript=no \
    -device virtio-blk-pci,bus=seat4,drive=rootfs -blockdev driver=file,node-name=rootfs,filename=ubuntu22_arm64.img \
    -device usb-mouse -device usb-kbd -device usb-tablet \
    -monitor none -serial telnet::55555,server,nowait,nodelay -s
```

![pci](../../assets/image-20260330195404342.png)

```shell
# 1、pcie-root-port，只能连接一个设备
# 2、xio3130-downstream、x3130-upstream：switch设备
# 3、bus使用pcie.0：Root Complex Integrated Endpoint
qemu-system-aarch64 -M virt,gic-version=3 -m 16G -cpu cortex-a72 -smp 4 \
    -kernel Image -append "console=ttyAMA0 nokaslr root=/dev/vda rw video=Virtual-1:1920x1080@60me" \
    -device pcie-root-port,bus=pcie.0,id=seat0,addr=1.0 \
    -device x3130-upstream,id=upstream_port1,bus=seat0 \
    -device xio3130-downstream,id=downstream_port1,bus=upstream_port1,chassis=1,slot=0 \
    -device xio3130-downstream,id=downstream_port2,bus=upstream_port1,chassis=1,slot=1 \
    -device xio3130-downstream,id=downstream_port3,bus=upstream_port1,chassis=1,slot=2 \
    -device xio3130-downstream,id=downstream_port4,bus=upstream_port1,chassis=1,slot=3 \
    -device virtio-gpu-pci,bus=downstream_port1 \
    -device qemu-xhci,bus=downstream_port2 \
    -device edu,dma_mask=0xffffffffffffffff,bus=downstream_port3 \
    -device e1000e,netdev=tap0,bus=downstream_port4 -netdev tap,id=tap0,ifname=tap0,script=no,downscript=no \
    -device virtio-blk-pci,bus=pcie.0,drive=rootfs,addr=2.0 -blockdev driver=file,node-name=rootfs,filename=ubuntu22_arm64.img \
    -device usb-mouse -device usb-kbd -device usb-tablet \
    -monitor none -serial stdio -s
```

![image-20260330202621832](../../assets/image-20260330202621832.png)

```shell
# 使用pxb-pcie，只能直接接 root-port或pci-bridge，但是当前不生效
qemu-system-aarch64 -M virt,gic-version=3 -m 16G -cpu cortex-a72 -smp 4 \
    -kernel Image -append "console=ttyAMA0 nokaslr root=/dev/vda rw video=Virtual-1:1920x1080@60me" \
    -device pxb-pcie,id=pcie.1,bus=pcie.0,bus_nr=70 \
    -device pcie-root-port,bus=pcie.1,id=seat1,addr=1.0,chassis=1,slot=1 \
    -device pcie-root-port,bus=pcie.1,id=seat2,addr=2.0,chassis=2,slot=2 \
    -device pcie-root-port,bus=pcie.1,id=seat3,addr=3.0,chassis=3,slot=3 \
    -device pcie-root-port,bus=pcie.1,id=seat4,addr=4.0,chassis=4,slot=4 \
    -device qemu-xhci,bus=seat1 \
    -device edu,dma_mask=0xffffffffffffffff,bus=seat2 \
    -device e1000e,netdev=tap0,bus=seat3 -netdev tap,id=tap0,ifname=tap0,script=no,downscript=no \
    -device virtio-blk-pci,bus=seat4,drive=rootfs,addr=2.0 -blockdev driver=file,node-name=rootfs,filename=ubuntu22_arm64.img \
    -device usb-mouse -device usb-kbd -device usb-tablet \
    -monitor none -serial stdio -s
```

### 2、Qemu参考文档

- [Qemu PCIe文档](https://github.com/qemu/qemu/blob/master/docs/pcie.txt)

![image-20260330195951183](../../assets/image-20260330195951183.png)

![image-20260330200028090](../../assets/image-20260330200028090.png)
