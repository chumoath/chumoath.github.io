# cross_toolchain

## 1、ct-ng构建aarch64交叉编译工具链

```shell
# 安装依赖
apt install -y texinfo
apt install -y help2man
apt install -y libtool-bin

# 构建ct-ng
wget http://crosstool-ng.org/download/crosstool-ng/crosstool-ng-1.27.0.tar.xz
tar -xvf crosstool-ng-1.27.0.tar.xz
cd crosstool-ng-1.27.0
./configure --prefix=/usr
make -j$(nproc)
make install

# 查看支持的target
ct-ng help

# 打印所有samples配置
ct-ng list-samples

# 显示aarch64-unknown-linux-gnu的配置，各个组件的版本号
ct-ng show-aarch64-unknown-linux-gnu
#    Languages       : C,C++
#    OS              : linux-6.13
#    Binutils        : binutils-2.43.1
#    Compiler        : gcc-14.2.0
#    Linkers         :
#    C library       : glibc-2.41
#    Debug tools     : gdb-16.2
#    Companion libs  : expat-2.5.0 gettext-0.23.1 gmp-6.3.0 isl-0.26 libiconv-1.16 mpc-1.3.1 mpfr-4.2.1 ncurses-6.4 zlib-1.3.1 zstd-1.5.6
#    Companion tools :

# 使用aarch64-unknown-linux-gnu的配置
ct-ng aarch64-unknown-linux-gnu

# 修改配置
ct-ng menuconfig
# 设置为可以root运行：ALLOW_BUILD_AS_ROOT
# CT_TARGET_VENDOR=target
# CT_TARGET_KERNEL=linux
# CT_TARGET_SYS=gnu
# CT_TARGET=aarch64-target-linux-gnu
# scripts/functions
#     CT_TARGET="${CT_TARGET_ARCH}"       
#     CT_TARGET="${CT_TARGET:+${CT_TARGET}-}${CT_TARGET_VENDOR}"
#     CT_TARGET="${CT_TARGET:+${CT_TARGET}-}${CT_TARGET_KERNEL}"
#     CT_TARGET="${CT_TARGET:+${CT_TARGET}-}${CT_TARGET_SYS}"

# 查看当前的配置，各个组件的版本号
ct-ng show-config
# Languages       : C,C++
# OS              : linux-5.10.233
# Binutils        : binutils-2.43.1
# Compiler        : gcc-13.3.0
# C library       : glibc-2.41
# Debug tools     : gdb-16.2

# 构建工具链
ct-ng build
# 默认构建目录：CT_WORK_DIR="${CT_TOP_DIR}/.build          => .build/aarch64-target-linux-gnu/build/
# 默认安装目录：CT_PREFIX_DIR="${CT_PREFIX:-${HOME}/x-tools}/${CT_HOST:+HOST-${CT_HOST}/}${CT_TARGET}"
```

## 2、交叉编译工具链的组件

- binutils
- kernel headers
- glibc
- gcc
- GMP (the GNU Multiple Precision Arithmetic Library)
- MPFR (the C library for multiple-precision floating-point computations with correct rounding)
- MPC (the C library for the arithmetic of complex numbers)

## 3、构建流程

1. GMP
2. MPFR
3. MPC
4. binutils
5. core pass 1 compiler
6. kernel headers
7. C library headers and start files
8. core pass 2 compiler
9. complete C library
10. final compiler

## 4、参考

- [components](https://crosstool-ng.github.io/docs/toolchain-construction/)
- MPC依赖GMP和MPFR，MPFR 依赖GMP，GMP没有依赖