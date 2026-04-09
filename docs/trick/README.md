# trick

### 1、调试init进程，或者包装其他进程

```shell
# /usr/local/bin/wget
# chmod +x /usr/local/bin/wget
#!/bin/bash
exec /usr/bin/wget "$@" --no-check-certificate
```

### 2、直接指定动态链接器，不需要内核解析ELF文件拿到动态链接器的路径

```shell
# LD_LIBRARY_PATH 可以独立于另一个系统，LD_PRELOAD 可以mock一个动态库的函数(符号解析时从已有符号表找到了就不会从其他动态库找)
LD_LIBRARY_PATH=/ LD_PRELOAD=/lib/x86_64-linux-gnu/libc.so.6 LD_DEBUG=all /lib64/ld-linux-x86-64.so.2 /usr/bin/ls

# ldd /usr/bin/ls
/lib64/ld-linux-x86-64.so.2 --list /usr/bin/ls

# 在openharmony执行ubuntu的strace
# 1) ubuntu
mkdir ubuntu_libs
cp /usr/bin/strace ubuntu_libs/
ldd /usr/bin/strace | cut -d' ' -f3 | xargs -I{} sh -c "cp {} ubuntu_libs"
cp /lib64/ld-linux-x86-64.so.2  ubuntu_libs/

# 2) openharmony
LD_LIBRARY_PATH=ubuntu_libs/ ubuntu_libs/ld-linux-x86-64.so.2 ubuntu_libs/strace /usr/bin/ls
```

### 3、自动创建依赖/动态库路径缓存

```shell
# rpm包经常用
# 1、openeuler用于安装kernel modules后，更新 /boot/initramfs
dracut -f
# 2、查看当前系统的动态库路径缓存
ldconfig -p
# 3、自动更新 /lib/modules/xxx/modules.dep
depmod -a
```

