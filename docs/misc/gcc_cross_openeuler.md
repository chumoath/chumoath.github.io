# gcc_cross_openeuler

### 一、overview

- openeuler为arm64，为发行版的arm64构建在x86运行的交叉编译工具链
- 只升级gcc，不升级 内核头文件、binutils、glibc

### 二、构建binutils

- 若 编译机为 arm64，则可以找到 arm64 的binutils、内核头文件、glibc，不需要单独构建 sysroot
- 若 编译机为 x86，就需要手动构建 binutils和sysroot

```shell
# 查看binutils版本
(openeuler) as --version
(x86) wget https://mirrors.tuna.tsinghua.edu.cn/gnu/binutils/binutils-2.37.tar.xz
(x86) tar -xf binutils-2.37.tar.xz && cd binutils-2.37
(x86) mkdir build && cd build
(x86) ../configure --host=x86_64-pc-linux-gnu --target=aarch64-target-linux-gnu --with-multilib-list=lp64 --with-arch=armv8 --prefix=/root/openeuler/openeuler_arm64le
(x86) make -j$(nproc)
(x86) make install
```

### 二、构建sysroot

- [openeuler 22.03 SP3 rpm](https://mirrors.tuna.tsinghua.edu.cn/openeuler/openEuler-22.03-LTS-SP3/OS/aarch64/Packages/)

- 通过 `yum provides /usr/include/stdio.h` 获取需要的文件所在包

- `rpm2cpio *.rpm | cpio -idmv` 放到 sysroot 即可

- 不需要手动构建，直接用openeuler的rpm包即可；包括：内核头文件、C运行库(glibc)、必要的其他头文件(编译过程中会报错提示)

- ```shell
  # 创建rpm包的目录
  (x86) mkdir -p /root/openeuler/rpm
  
  # 需要的rpm包
  # 内核头文件
  kernel-headers-5.10.0-182.0.0.95.oe2203sp3.aarch64.rpm
  
  # glibc(C运行库)
  glibc-2.34-143.oe2203sp3.aarch64.rpm
  glibc-devel-2.34-143.oe2203sp3.aarch64.rpm
  
  # crypt.h
  libxcrypt-4.4.26-5.oe2203sp3.aarch64.rpm
  libxcrypt-devel-4.4.26-5.oe2203sp3.aarch64.rpm
  
  # 在 binutils的安装目录创建 sysroot 目录
  (x86) mkdir -p /root/openeuler/openeuler_arm64le/sysroot && cd /root/openeuler/openeuler_arm64le/sysroot
  (x86) find ../../rpm -name *.rpm | xargs -I{} sh -c "rpm2cpio {} | cpio -idmv"
  ```

### 三、构建高版本gcc

```shell
# 获取 openeuler原始的 gcc编译选项
# 必须去掉 --build=aarch64-linux-gnu，否则gcc的配置 GCC_FOR_TARGET会出错，通过config.status可验证
(openeuler) gcc -v

# 将binutils添加到环境变量
(x86) export PATH=/root/openeuler/openeuler_ram64le/bin:$PATH

# 下载依赖
(x86) wget https://mirrors.tuna.tsinghua.edu.cn/gnu/gcc/gcc-12.3.0/gcc-12.3.0.tar.xz
(x86) tar -xf gcc-12.3.0.tar.xz && cd gcc-12.3.0
(x86) ./contrib/download_prerequisites
(x86) mkdir build && cd build
# 必须有 --with-sysroot，不能只有 --with-build-sysroot，会报 stdio.h 找不到
(x86) ../configure --host=x86_64-linux-gnu --target=aarch64-target-linux-gnu --prefix=/root/openeuler/openeuler_arm64le \
--enable-shared --with-arch=armv8-a --enable-threads=posix --enable-checking=release --with-system-zlib --enable-__cxa_atexit \
--disable-libunwind-exceptions --enable-gnu-unique-object --enable-linker-build-id --with-linker-bash-style=gnu --enable-languages=c,c++ \
--enable-plugin --enable-initfini-array --disable-libgcj --with-isl --without-cloog --enable-gnu-indirect-function --with-stage1-ldflags='-Wl,-z,relro,-z,now' \
--with-boot-ldflags='-Wl,-z,relro,-z,now' --disable-bootstrap \
--with-multilib-list=lp64 --enable-bolt --with-build-sysroot=/root/openeuler/openeuler_arm64le/sysroot \
--with-sysroot=/root/openeuler/openeuler_arm64le/sysroot \
--with-build-time-tools=/root/openeuler/openeuler_arm64le/bin
(x86) make -j$(nproc)
(x86) make install
```

### 四、可能的报错

1. `libgcc_s.so: cannot find crti.o: No such file or directory`

   - 修改 crti.o、crtn.o和动态链接库的路径: `lib`为`lib64`

   ```shell
   (x86) gcc-12.3.0/build/gcc/xgcc -print-file-name=crti.o
   # libraries: ...:/root/openeuler/openeuler_arm64le/sysroot/lib/:/root/openeuler/openeuler_arm64le/sysroot/usr/lib/
   # 但是 openeuler的crti.o和crtn.o在 sysroot/usr/lib64
   (x86) gcc-12.3.0/build/gcc/xgcc -print-search-dirs
   
   # 解决
   # 1、cp -rfa sysroot/usr/lib64 sysroot/usr/lib
   # 2、修改gcc源代码，添加补丁
   gcc/config/aarch64/aarch64-linux.h
   
   - #define GLIBC_DYNAMIC_LINKER "/lib/ld-linux-aarch64%{mbig-endian:_be}%{mabi=ilp32:_ilp32}.so.1"
   
   + #undef STANDARD_STARTFILE_PREFIX_2
   + #define STANDARD_STARTFILE_PREFIX_2 "/usr/lib64/"
   + #define GLIBC_DYNAMIC_LINKER "/lib%{mabi=lp64:64}%{mabi=ilp32:ilp32}/ld-linux-aarch64%{mbig-endian:_be}%{mabi=ilp32:_ilp32}.so.1"
   ```

### 五、gcc构建步骤概述

```shell
core: C语言 到 汇编代码，没有周边依赖  => xgcc xg++
libgcc: libgcc.a libgcc_s.so
c++: libstdc++.so

最终就是将 xgcc 重命名为 aarch64-taarget-linux-gnu-gcc 
```

### 六、使用

```shell
export PATH=/root/openeuler/openeuler_arm64le/bin:$PATH
aarch64-target-linux-gnu-gcc --sysroot=/root/openeuler/openeuler_arm64le/sysroot test.c -o test --verbose
# -dynamic-linker /lib64/ld-linux-aarch64.so.1
```

