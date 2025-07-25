# harmonyos_build

### 一、构建

```shell
mkdir ~/bin
curl https://gitee.com/oschina/repo/raw/fork_flow/repo-py3 -o ~/bin/repo 
chmod a+x ~/bin/repo
pip3 install -i https://repo.huaweicloud.com/repository/pypi/simple requests

vim ~/.bashrc               # 编辑环境变量
export PATH=~/bin:$PATH     # 在环境变量的最后添加一行repo路径信息
source ~/.bashrc            # 应用环境变量

cd ~
mkdir ohos_master       #在根目录下新建ohos_master路径
cd ohos_master          #进入ohos_master路径拉取代码

repo init -u https://gitee.com/openharmony/manifest.git -b master --no-repo-verify
repo sync -c
repo forall -c 'git lfs pull'

bash build/prebuilts_download.sh

# build.sh 运行过程中会配置 /root/.ohpm/.ohpmrc
#     registry=https://repo.harmonyos.com/ohpm/
#     publish_registry=https://ohpm.openharmony.cn/ohpm/

#  ohpm config set registry https://repo.harmonyos.com/ohpm/

./build.sh --product-name rk3568 --ccache

docker run --rm -it -v $(pwd)/openharmony:/home/openharmony openharmony

build/prebuilts_download.sh
调用 build/prebuilts_download.py

build/prebuilts_download.py
下载工具

指定 npm 的镜像源
parser.add_argument('--npm-registry', default='https://repo.huaweicloud.com/repository/npm/',

不自动安装，手动安装
    # while retry_times <= max_retry_times:
    #     result, error = _npm_install(args)
    #     if result:
    #         break
    #     print("npm install error, error info: %s" % error)
    #     args.parallel_install = False
    #     if retry_times == max_retry_times:
    #         for error_info in error.split('\n'):
    #             if error_info.endswith('debug.log'):
    #                 log_path = error_info.split()[-1]
    #                 cmd = ['cat', log_path]
    #                 process_cat = subprocess.Popen(cmd)
    #                 process_cat.communicate(timeout=60)
    #                 raise Exception("npm install error with three times, prebuilts download exit")
    #     retry_times += 1

build/prebuilts_download_config.json
各类工具配置及路径

# 配置nodejs的版本
  "file_handle_config": [
    {
      "src":"/prebuilts/build-tools/common/nodejs",
      "dest":"/prebuilts/build-tools/common/nodejs",
      "rename": "true",
      "symlink_src":"/node-v14.21.1-linux-x64",
      "symlink_dest":"/current"
    },

# 手动执行 npm install
cd developtools/ace_ets2bundle/compiler                                   &&  /home/gxh/openharmony/prebuilts/build-tools/common/nodejs/node-v14.21.1-linux-x64/bin/npm install && cd -
cd developtools/ace_js2bundle/ace-loader                                  &&  /home/gxh/openharmony/prebuilts/build-tools/common/nodejs/node-v14.21.1-linux-x64/bin/npm install && cd -
cd third_party/jsframework                                                &&  /home/gxh/openharmony/prebuilts/build-tools/common/nodejs/node-v14.21.1-linux-x64/bin/npm install && cd -
cd third_party/parse5/packages/parse5                                     &&  /home/gxh/openharmony/prebuilts/build-tools/common/nodejs/node-v14.21.1-linux-x64/bin/npm install && cd -
cd third_party/weex-loader                                                &&  /home/gxh/openharmony/prebuilts/build-tools/common/nodejs/node-v14.21.1-linux-x64/bin/npm install && cd -
cd arkcompiler/ets_frontend/legacy_bin/api8                               &&  /home/gxh/openharmony/prebuilts/build-tools/common/nodejs/node-v14.21.1-linux-x64/bin/npm install && cd -
cd interface/sdk-js/build-tools                                           &&  /home/gxh/openharmony/prebuilts/build-tools/common/nodejs/node-v14.21.1-linux-x64/bin/npm install && cd -
cd arkcompiler/ets_frontend/arkguard                                      &&  /home/gxh/openharmony/prebuilts/build-tools/common/nodejs/node-v14.21.1-linux-x64/bin/npm install && cd -
cd arkcompiler/ets_frontend/ets2panda/driver/build_system                 &&  /home/gxh/openharmony/prebuilts/build-tools/common/nodejs/node-v14.21.1-linux-x64/bin/npm install && cd -
cd arkcompiler/ets_frontend/ets2panda/linter                              &&  /home/gxh/openharmony/prebuilts/build-tools/common/nodejs/node-v14.21.1-linux-x64/bin/npm install && cd -
cd arkcompiler/ets_frontend/ets2panda/bindings                            &&  /home/gxh/openharmony/prebuilts/build-tools/common/nodejs/node-v14.21.1-linux-x64/bin/npm install && cd -
cd arkcompiler/runtime_core/static_core/plugins/ets/tools/declgen_ts2sts  &&  /home/gxh/openharmony/prebuilts/build-tools/common/nodejs/node-v14.21.1-linux-x64/bin/npm install && cd -
cd developtools/ace_ets2bundle/koala-wrapper                              &&  /home/gxh/openharmony/prebuilts/build-tools/common/nodejs/node-v14.21.1-linux-x64/bin/npm install && cd -
cd developtools/ace_ets2bundle/arkui-plugins                              &&  /home/gxh/openharmony/prebuilts/build-tools/common/nodejs/node-v14.21.1-linux-x64/bin/npm install && cd -
```

### 二、repo

```shell
# 初始化 repo (使用默认 manifest)
repo init -u <manifest_url> -b <branch_name>

# 或使用特定 manifest 文件
repo init -u <manifest_url> -m <manifest_file.xml> -b <branch_name>

# 同步代码
repo sync -j4  # -j 指定并行任务数

# 查看当前分支
repo branches

# 在所有仓库创建新分支
repo start <branch_name> --all

# 或在特定项目创建分支
repo start <branch_name> <project_path>

# 查看修改状态
repo status

# 查看未提交的修改
repo diff

# 常规 git 操作
git add .
git commit -m "commit message"

# 上传代码评审
repo upload

# 上传特定项目
repo upload <project_path>

# 查看上传记录
repo upload --reviewers=<reviewer_email> --cc=<cc_email>

# 同步最新代码
repo sync -j4

# 同步并删除已合并的分支
repo sync -j4 --prune

# 解决冲突后继续同步
repo sync -j4 --force-sync

# 列出所有分支
repo branches

# 切换分支
repo checkout <branch_name>

# 删除分支
repo abandon <branch_name>

# 列出所有项目
repo list

# 显示项目路径
repo list -p

# 显示项目详细信息
repo list -f

# 丢弃所有本地修改
repo forall -c 'git reset --hard'

# 清理未跟踪文件
repo forall -c 'git clean -fdx'

# 在所有仓库执行命令
repo forall -c 'git status'

# 在特定项目执行命令
repo forall <project_path> -c 'git log -1'

# 强制同步
repo sync --force-sync 
```

### 三、参考链接

- [Docker编译环境](https://gitee.com/openharmony/docs/blob/master/zh-cn/device-dev/get-code/gettools-acquire.md)
- [快速入门](https://gitee.com/openharmony/docs/blob/master/zh-cn/device-dev/quick-start/Readme-CN.md)
- [整机启动流程](https://gitee.com/openharmony/docs/blob/master/zh-cn/device-dev/subsystems/subsys-boot-deviceboot.md)
- [编译构建指导](https://gitee.com/openharmony/docs/blob/master/zh-cn/device-dev/subsystems/subsys-build-all.md)

### 四、Dockerfile

```dockerfile
FROM ubuntu:20.04
WORKDIR /home/openharmony

ENV DEBIAN_FRONTEND=noninteractive

RUN sed -i "s@http://.*archive.ubuntu.com@http://repo.huaweicloud.com@g" /etc/apt/sources.list \
        && sed -i "s@http://.*security.ubuntu.com@http://repo.huaweicloud.com@g" /etc/apt/sources.list \
        && apt-get update -y \
        && apt-get install -y apt-utils binutils bison flex bc build-essential make mtd-utils gcc-arm-linux-gnueabi u-boot-tools python3.8 python3-pip git zip unzip curl wget gcc g++ ruby dosfstools mtools default-jre default-jdk scons python3.8-distutils perl openssl libssl-dev cpio git-lfs m4 ccache zlib1g-dev tar rsync liblz4-tool genext2fs binutils-dev device-tree-compiler e2fsprogs git-core gnupg gnutls-bin gperf lib32ncurses5-dev libffi-dev zlib* libelf-dev libx11-dev libgl1-mesa-dev lib32z1-dev xsltproc x11proto-core-dev libc6-dev-i386 libxml2-dev lib32z-dev libdwarf-dev \
        && apt-get install -y grsync xxd libglib2.0-dev libpixman-1-dev kmod jfsutils reiserfsprogs xfsprogs squashfs-tools  pcmciautils quota ppp libtinfo-dev libtinfo5 libncurses5 libncurses5-dev libncursesw5 libstdc++6 python2.7 gcc-arm-none-eabi \
        && apt-get install -y vim ssh locales \
        && apt-get install -y doxygen \
        && locale-gen "en_US.UTF-8" \
        && rm -rf /bin/sh /usr/bin/python /usr/bin/python3 /usr/bin/python3m \
        && ln -s /bin/bash /bin/sh \
        && ln -s /usr/bin/python3.8 /usr/bin/python3 \
        && ln -s /usr/bin/python3.8 /usr/bin/python3m \
        && ln -s /usr/bin/python3.8 /usr/bin/python

RUN apt install -y libxml2-dev libxslt1-dev libpython3.8-dev
RUN curl https://gitee.com/oschina/repo/raw/fork_flow/repo-py3 > /usr/bin/repo \
        && chmod +x /usr/bin/repo \
        && pip3 install --trusted-host https://repo.huaweicloud.com -i https://repo.huaweicloud.com/repository/pypi/simple requests setuptools pymongo kconfiglib pycryptodome ecdsa ohos-build pyyaml prompt_toolkit==1.0.14 redis json2html yagmail python-jenkins \
        && pip3 install esdk-obs-python --trusted-host pypi.org \
        && pip3 install six --upgrade --ignore-installed six \
        && cd /home/openharmony

ENV LANG=en_US.UTF-8 LANGUAGE=en_US.UTF-8 LC_ALL=en_US.UTF-8
```

### 五、命令

```shell
docker run --rm -it -v $(pwd)/openharmony:/home/openharmony openharmony

./build.sh -p rk3568
./build.sh -p rk3568 --gn-args linux_kernel_version="linux-5.10"
./build.sh -p rk3568 --fast-rebuild --gn-args linux_kernel_version="linux-5.10"

./build.sh -p qemu-arm64-linux-min --gn-args linux_kernel_version="linux-5.10"
./build.sh -p qemu-arm64-linux-min --gn-args linux_kernel_version="linux-5.10" --fast-rebuild

# debug - use true/false instead of 0/1 
./build.sh -p qemu-arm64-linux-min --gn-args linux_kernel_version="linux-5.10" enable_profiling=true is_debug=true

repo init -u https://gitee.com/openharmony/manifest.git -b OpenHarmony-5.1.0-Release --no-repo-verify
repo sync -j24
repo forall -c 'git lfs pull'

bash build/prebuilts_download.sh

# no enough disk space
(host) docker run --rm -it -v $(pwd)/openharmony:/home/openharmony -v /home/gxh/images:/home/openharmony/openharmony/out/qemu-arm64-linux/packages/phone/images openharmony
# or
(host) docker run --rm -it -v $(pwd)/openharmony:/home/openharmony -v /home/xxx/images:/home/openharmony/images openharmony
(docker) ln -s /home/openharmony/images /home/openharmony/openharmony/out/qemu-arm64-linux/packages/phone/images

# run
(host) ln -s /home/xxx/images /mnt/openharmony/openharmony/out/qemu-arm64-linux/packages/phone/images
(host) ./vendor/ohemu/qemu-arm64-linux-min/qemu_run.sh

# segmentation fault: samgr/foundation/render_service reboot
service_control stop foundation
service_control stop render_service
service_control stop appspawn
dmesg -n 1

# gdbserver
export PATH=/mnt/openharmony/openharmony/prebuilts/gcc/linux-x86/aarch64/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu/bin:$PATH

apt install -y make
wget https://mirrors.tuna.tsinghua.edu.cn/gnu/gdb/gdb-8.0.1.tar.xz
tar -xf gdb-8.0.1.tar.xz && cd gdb-8.0.1
mkdir build && cd build
LDFLAGS="-static" ../gdb/gdbserver/configure --host=aarch64-linux-gnu
make -j$(nproc)
# dynamically linked, interpreter /lib/ld-linux-aarch64.so.1,
file gdbserver

# /system/bin/sh: /bin/gdbserver: No such file or directory => /lib/ld-linux-aarch64.so.1
# (shell) /lib/ld-musl-aarch64.so.1 --list /bin/gdbserver                              
# /lib/ld-musl-aarch64.so.1: /bin/gdbserver: Not a valid dynamic program

# static
rm -f gdbserver
make

# delete -Wl,--dynamic-list=../gdb/gdbserver/proc-service.list
# statically linked
file gdbserver

mount system.img /mnt
cp gdbserver /mnt/system/bin/

# strace
apt install -y autoconf
apt install -y gcc-aarch64-linux-gnu
git clone https://github.com/strace/strace.git
cd strace
./bootstrap
mkdir build && cd build
LDFLAGS="-static" ../configure --host=aarch64-linux-gnu --enable-mpers=no
make -j$(nproc)
mount system.img /mnt
cp src/strace /mnt/system/bin/
# no: export PATH=/mnt/openharmony/openharmony/prebuilts/gcc/linux-x86/aarch64/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu/bin:$PATH
#   => configure: error: failed to find timer_create

# must: --enable-mpers=no
#   => configure: error: Cannot enable m32 personality support

# network
# 1. qemu
-net nic,netdev=tap0,model=virtio -netdev tap,id=tap0,ifname=tap0,script=no,downscript=no
-serial telnet::55555,server,nowait,nodelay

# 2. host
ip tuntap add tap0 mode tap group 0
ip link set dev tap0 up
ip addr add 192.168.33.1/24 dev tap0
iptables -A POSTROUTING -t nat -j MASQUERADE -s 192.168.33.0/24
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -P FORWARD ACCEPT

# 3. guest
ip addr add 192.168.33.2/24 dev eth0
ip route add to 192.168.33.1 dev eth0
ip route add default via 192.168.33.1

ifconfig eth0 192.168.33.2 netmask 255.255.255.0
toybox route add default gw 192.168.33.1
ping 192.168.33.1

# debug
apt install gdb-multiarch

telnet loalhost 55555
(guest) ps -ef | grep composer_host
(guest) gdbserver --attach :1234 [pid]
(host) gdb-multiarch hdf_devhost
(host gdb) target remote 192.168.33.2:1234
```

