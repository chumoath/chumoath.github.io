# gem5

### 1、gem5构建

```shell
git clone https://github.com/gem5/gem5
scons build/ALL/gem5.opt -j$(nproc)
scons build/ARM/gem5.opt -j$(nproc)
scons build/X86/gem5.opt -j$(nproc)
```

### 2、x86运行ubuntu

1. 原始，登录立刻退出

```shell
# 1. 内核、文件系统自动下载
# workload = obtain_resource("x86-ubuntu-24.04-boot-with-systemd")
# board.set_workload(workload)
# 会自动下载到 
# - /root/.cache/gem5/x86-linux-kernel-6.8.0-52-generic
# - /root/.cache/gem5/x86-ubuntu-24.04-img

# 2. 运行
./build/ALL/gem5.opt configs/example/gem5_library/x86-ubuntu-run-with-kvm-no-perf.py
```

2. 关闭自动退出

```shell
# 1. 文件系统镜像修改
cp /root/.cache/gem5/x86-ubuntu-24.04-img /root/.cache/gem5/x86-ubuntu-24.04-img.bak
losetup -P /dev/loop0 /root/.cache/gem5/x86-ubuntu-24.04-img.bak
mount /dev/loop0p2 /mnt
# /mnt/home/gem5/.bashrc 删除 /home/gem5/after_boot.sh
umount /mnt

# 2. python 配置 - configs/example/gem5_library/x86-ubuntu-run-with-kvm-no-perf.py
from gem5.resources.resource import CustomResource, CustomDiskImageResource
kernel = CustomResource("/root/.cache/gem5/x86-linux-kernel-6.8.0-52-generic")
disk_image = CustomDiskImageResource("/root/.cache/gem5/x86-ubuntu-24.04-img.bak")
kernel_args = ["earlyprintk=ttyS0", "console=ttyS0", "lpj=7999923", "root=/dev/sda2"]
board.set_kernel_disk_workload(kernel=kernel, disk_image=disk_image, kernel_args=kernel_args)

# 3. 运行
./build/ALL/gem5.opt configs/example/gem5_library/x86-ubuntu-run-with-kvm-no-perf.py
```

3. 查看日志

```shell
# 编译 m5term
cd gem5/util/term/
make

# 查看日志
./m5term localhost 3456  # ./m5term 3456

# 日志文件 - m5out/board.pc.com_1.device
# SimObject配置文件 - m5out/config.json m5out/config.ini
# 统计信息 (Tick、SimObject) - m5out/stats.txt
```

4. gem5 vscode调试

```shell
unset https_proxy
unset http_proxy
bear -- scons build/ALL/gem5.opt -j$(nproc)
```

5. FAQ

   - 必须使用kvm-no-perf，不能用gem5模拟，太慢了
   - 调试python: 在调试位置添加 `import pdb; pdb.set_trace()`
   - `./build/ALL/gem5.opt --list-sim-objects`:  List all built-in SimObjects, their params and default values.
   - `SConscript: DebugFlag('PciDevice')`: 
     - `./build/ALL/gem5.opt --debug-flags=PciEndpoint,PciDevice,PciHost configs/example/gem5_library/x86-ubuntu-run-with-kvm-no-perf.py`