# Android build

### 1、host 拉取android代码

```shell
export http_proxy=http://172.25.64.1:7897
export https_proxy=http://172.25.64.1:7897

apt install -y repo

repo init -u https://android.googlesource.com/platform/manifest -b android-2.3.1_r1
repo sync -c
```

### 2、docker 编译android

```shell
docker pull ubuntu:12.04
docker run -it -v $(pwd)/android:/home/android ubuntu:12.04

sed -i "s@http://.*archive.ubuntu.com@http://old-releases.ubuntu.com@g" /etc/apt/sources.list
sed -i "s@http://.*security.ubuntu.com@http://old-releases.ubuntu.com@g" /etc/apt/sources.list

export http_proxy=http://172.25.64.1:7897
export https_proxy=http://172.25.64.1:7897

apt-get update
apt-get install cpp gcc g++ git gnupg flex bison gperf build-essential zip curl libc6-dev \
libncurses5-dev:i386 libreadline6-dev:i386 x11proto-core-dev libx11-dev:i386 \
libgl1-mesa-glx:i386 libgl1-mesa-dev gcc-multilib g++-multilib mingw32 tofrodos python-markdown \
libxml2-utils xsltproc zlib1g-dev:i386 cpp-4.6 libz-dev

jdk-6u45-linux-x64.bin
wget https://www.atteya.net/site/en/downloads/java-jdk?download=48:java-jdk-6u45-linux-x64

mkdir -p /usr/java
chmod +x ./jdk-6u45-linux-x64.bin
cp jdk-6u45-linux-x64.bin /usr/java/
cd /usr/java/ && ./jdk-6u45-linux-x64.bin

update-alternatives --install /usr/bin/java    java     /usr/java/jdk1.6.0_45/bin/java   1061
update-alternatives --install /usr/bin/javac   javac    /usr/java/jdk1.6.0_45/bin/javac   1061
update-alternatives --install /usr/bin/jar     jar      /usr/java/jdk1.6.0_45/bin/jar   1061
update-alternatives --install /usr/bin/javah   javah    /usr/java/jdk1.6.0_45/bin/javah   1061
update-alternatives --install /usr/bin/javadoc javadoc  /usr/java/jdk1.6.0_45/bin/javadoc   1061

update-alternatives --config java
update-alternatives --config javac
update-alternatives --config jar
update-alternatives --config javah
update-alternatives --config javadoc

# 报错修改
# 1) RefBase.cpp
# frameworks/base/libs/utils/Android.mk: LOCAL_CFLAGS += -DLIBUTILS_NATIVE=1 $(TOOL_CFLAGS) 改为 LOCAL_CFLAGS += -DLIBUTILS_NATIVE=1 $(TOOL_CFLAGS) –fpermissive

# 2) _FORTIFY_SOURCE redefined
# build/core/combo/HOST_linux-x86.mk: HOST_GLOBAL_CFLAGS += -D_FORTIFY_SOURCE=0 改为 HOST_GLOBAL_CFLAGS += -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=0
make -j24
make sdk
```

- [RefBase.cpp](https://www.cnblogs.com/googlegis/archive/2012/01/02/2978744.html)
- [_FORTIFY_SOURCE](https://www.cnblogs.com/googlegis/archive/2012/01/07/2978726.html)

### 3、在ubuntu22运行ubuntu12编出的emulator

```shell
# 从ubuntu12复制emulator所需的所有动态库
mkdir ubuntu12_libs
ldd /bin/ls | cut -d' ' -f3 | xargs -I{} sh -c "cp {} ./ubuntu12_libs/"
cp /lib/ld-linux.so.2 ./ubuntu12_libs/

# ubuntu22运行ubuntu12编出的emulator
ANDROID_PRODUCT_OUT=/root/android/out/target/product/generic
LD_LIBRARY_PATH=ubuntu12_libs  ubuntu12_libs/ld-linux.so.2  out/host/linux-x86/bin/emulator
```

### 4、构建分析

```shell
Makefile
   include build/core/main.mk

build/core/main.mk
   TOP := .
   .PHONY: droid
   DEFAULT_GOAL := droid
   $(DEFAULT_GOAL):
   
   # include any Android.mk files we can find.
   subdirs := $(TOP)
   FULL_BUILD := true
   
   # 从 TOP目录(顶级目录，因为Makefile的include)搜索所有Android.mk
   subdir_makefiles := $(shell build/tools/findleaves.py --prune=out --prune=.repo --prune=.git $(subdirs) Android.mk)

   include $(subdir_makefiles)

# emulator构建
external/qemu/Android.mk
   ifeq ($(DEFAULT_GOAL),droid)
      LOCAL_PATH:= $(call my-dir)
      include $(LOCAL_PATH)/Makefile.android
   else

external/qemu/Makefile.android
```

### 5、添加Android组件

```shell
# - external/freg/Android.mk
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE := freg
LOCAL_SRC_FILES := $(call all-subdir-c-files)
include $(BUILD_EXECUTABLE)

# - external/freg/freg.c
#include <stdio.h>

int main(void)
{
        printf ("freg\n");
        return 0;
}
```

