# bitbake

## 1、bitbake命令

```shell
locale-gen en_US.UTF-8

# python调试
python3 -m pdb bitbake.py -e

# 只删除生成件，不删除中间件
bitbake -c clean obmc-phosphor-image

# 列出一个目标的所有任务
bitbake -c listtasks virtual/kernel

# 无视时间戳
bitbake -c deploy virtual/kernel -f

# local.conf中指定terminal: OE_TERMINAL = "tmux"
# 进入构建上下文
bitbake -c devshell file

# 执行指定recipe的fetch任务
bitbake -c fetch linux-yocto

# 配置内核
bitbake -c do_menuconfig virtual/kernel
# 进入内核构建上下文
bitbake -c devshell virtual/shell

# 默认task是do_build
bitbake printhello
bitbake -c do_build printhello

# 构建完成后，生成包管理索引
bitbake package-index

bitbake-getvar BBPATH
bitbake-getvar BBFILES
bitbake-getvar BBLAYERS
# 查看指定recipe的变量
bitbake-getvar -r core-image-minimal IMAGE_INSTALL

bitbake-getvar -r obmc-phosphor-image OVERRIDES
bitbake-getvar -r nativesdk-llvm OVERRIDES
bitbake-getvar -r llvm-native OVERRIDES

bitbake-getvar -r file-native PN
bitbake-getvar -r file-native OVERRIDES
bitbake-getvar -r file CLASSOVERRIDE

bitbake-getvar -r obmc-phosphor-image IMGCLASSES
bitbake-getvar -r obmc-phsophor-image IMAGE_CLASSES
bitbake-getvar -r obmc-phsophor-image KERNEL_CLASSES
bitbake-getvar -r obmc-phsophor-image IMAGE_FSTYPES
bitbake-getvar -r obmc-phsophor-image IMAGE_ROOTFS

bitbake-getvar -r file-native PACKAGE_ARCH
bitbake-getvar -r file PACKAGE_ARCH
bitbake-getvar -r webui-vue PACKAGE_ARCH
bitbake-getvar -r linux-aspeed PACKAGE_ARCH

bitbake -e | grep BBPATH

# 生成指定任务的依赖
bitbake -g obmc-phosphor-image

# Xserver显示任务依赖关系图
bitbake -g -u taskexp obmc-phosphor-image
# 字符界面显示任务依赖关系图
bitbake -g -u taskexp_ncurses obmc-phosphor-image
# 详细显示构建流程
bitbake -g -u teamcity obmc-phosphor-image

# 查看recipe的版本：RecipeName LatestVersion PreferredVersion   RequiredVersion
bitbake --show-versions
```

## 2、oe-pkgdata命令

```shell
# 浏览生成的包的内容及依赖关系：左边是recipe，右边是pkg
oe-pkgdata-browser

# 查找生成该pkg的recipe
oe-pkgdata-util lookup-recipe busybox-dbg

# 列出指定包的文件
oe-pkgdata-util list-pkg-files busybox

# 搜索哪个包提供指定文件
oe-pkgdata-util find-path /usr/bin/busybox

# 查看包的版本、recipe、大小信息
oe-pkgdata-util package-info busybox-dbg

# 将recipe包名转换为运行时包名
oe-pkgdata-util lookup-pkg glibc
```

## 3、bitbake-layers命令

```shell
# 查看最终的recipes的层结构，最终的软件版本；bbappend也会被添加
bitbake-layers flatten output

# 列出当前配置支持的machine
bitbake-layers show-machines

# 列出当前配置的layers
bitbake-layers show-layers

# 从bblayers.conf中删除layer
bitbake-layers remove-layer meta-evbcustom

# 显示配置的layers中是否有重复的recipe
bitbake-layers show-overlayed

# 列出当前配置的bbappend
bitbake-layers show-appends

# 创建layer，添加到bblayers.conf
bitbake-layers create-layer meta-evbcustom

# 添加layer到bblayers.conf
bitbake-layers add-layer meta-evbcustom

# 查每个recipes属于某层
bitbake-layers show-recipes | grep -A1 llvm
bitbake-layers show-recipes | grep -A1 linux-libc-headers

# 查看packagegroup -> packagegroup-core-boot | packagegroup-base | packagegroup-obmc-apps
bitbake-layers show-recipes | grep packagegroup

# 查看image
bitbake-layers show-recipes | grep image

# 查看每个recipe依赖的.bb和.bbclass
bitbake-layers show-cross-depends -f

# 保存当前配置到模板
bitbake-layers save-build-conf /home/gxh/openbmc/meta-evbcustom evbcustom
```

## 4、生成件目录结构

## 5、devtool添加补丁工作流

1. 在工作区生成备份
   - `devtool modify dbus-sensors` 
2. 修改源代码 或修改recipe
   - `devtool edit-recipe dbus-sensors`
3. 构建recipe / image
   - `devtool build dbus-sensors`
   - `devtool build-image obmc-phosphor-image`
4. 部署到目标环境

   - `devtool deploy-target dbus-sensors root@x.x.x.x`
     - `-s: 查看状态`
     - `-p 覆盖存在的文件`
     - `-S 去掉符号信息, 从/tmp/devtool_deploy.list可以看到文件大小, -S 先后比较大小`

   - 指定ssh端口；配置覆盖存在的生成件，否则存在会报错
     - `devtool deploy-target socat root@127.0.0.1 -P 2222 -s -p`
   - 只列出要拷贝的文件，不做实际拷贝
     - `devtool deploy-target socat root@127.0.0.1 -P 2222 -s -n`
   - 去掉符号信息，防止空间不足
     - `devtool deploy-target socat root@127.0.0.1 -P 2222 -s -p -S`
5. 源代码提交到git
   - `git commit`
6. 用 git 补丁生成 *.bbappend
   
   - `devtool update-recipe dbus-sensors -a /home/xxx/openbmc/meta-evbcustom`
7. 清除
   - `devtool reset dbus-sensors`
8. 生成 *.bbappend 并清除 (同 6 + 7)

   - `devtool finish dbus-sensors /home/xxx/openbmc/meta-evbcustom`
9. 查看当前状态
   - `devtool status`
10. 参考
    - [devtool](https://docs.yoctoproject.org/ref-manual/devtool-reference.html)

## 6、bitbake-layer添加layer工作流(以bmc-clid为例)

1. 创建层
   - `bitbake-layers create-layer meta-evbcustom`
   
2. 添加到 bblayers.conf
   - `bitbake-layers add-layer meta-evbcustom`
   
3. 从远程仓库创建recipe - 源代码路径默认为 `build/evb-ast2600/workspace/sources`
   
   - `devtool add bmc-clid https://github.com/chumoath/bmc_clid.git -B main`
   
3. 修改recipe
   
   - `devtool edit-recipe bmc-clid`
   
4. recipe发布到指定layer
   - `mkdir -p /home/gxh/openbmc/meta-evbcustom/recipes-phosphor`
   - `devtool finish bmc-clid /home/gxh/openbmc/meta-evbcustom/recipes-phosphor`
   
5. 从bblayers.conf删除layer
   - `bitbake-layers remove-layer meta-evbcustom`
   
6. 常用命令

   ```shell
   # 一、调试
   devtool edit-recipe bmc-clid
   bitbake obmc-phosphor-image
   
   bitbake-getvar -r obmc-phosphor-image PACKAGE_INSTALL
   bitbake-getvar -r obmc-phosphor-image IMAGE_INSTALL
   bitbake-getvar -r bmc-clid DEPENDS
   bitbake-getvar -r bmc-clid RDEPENDS:bmc-clid
   bitbake-getvar -r bmc-clid PACKAGES
   bitbake -s
   bitbake -g
   bitbake -e
   oe-pkgdata-browser
   bitbake package-index
   # 查看打出了哪些包
   ls openbmc/build/evb-ast2600/tmp/deploy/rpm/armv7ahf_vfpv4d16 | grep bmc-clid
   # 查看生成文件系统的日志
   openbmc/build/evb-ast2600/tmp/work/evb_ast2600-openbmc-linux-gnueabi/obmc-phosphor-image/1.0/temp/log.do_rootfs
   # 查看rpm包推荐的
   rpm -q --recommends bmc-clid-1.0+git-r0.armv7ahf_vfpv4d16.rpm
   # 查看rpm包提供的
   rpm -q --provides bmc-clid-1.0+git-r0.armv7ahf_vfpv4d16.rpm
   # 查看 rpm 包的内容
   rpm -ql bmc-clid-1.0+git-r0.armv7ahf_vfpv4d16.rpm
   # 查看rpm包的依赖
   rpm -qR bmc-clid-1.0+git-r0.armv7ahf_vfpv4d16.rpm
   
   # 二、运行 qemu
   runqemu publicvnc
   runqemu serialstdio
   
   # guest
   ip addr add 192.168.7.2/24 dev eth0
   ip route add default via 192.168.7.1
   # host
   telnet 192.168.7.2 8888
   
   # 三、分析
   # 一个 *.bb 的继承流程
   meta/conf/bitbake.conf(变量配置等) -> meta/classes-global/base.bbclass -> meta/classes-recipe/cmake.bbclass -> *.bb
   
   # 一个*.bb文件默认的PACKAGES
   meta/conf/bitbake.conf
   PACKAGES = "${PN}-src ${PN}-dbg ${PN}-staticdev ${PN}-dev ${PN}-doc ${PN}-locale ${PACKAGE_BEFORE_PN} ${PN}"
   
   # openbmc/meta/lib/oe/rootfs.py
   IMAGE_ROOTFS
   IMAGE_PKGTYPE
   ROOTFS_PREPROCESS_COMMAND
   ROOTFS_POSTPROCESS_COMMAND
   ROOTFS_POSTINSTALL_COMMAND
   
   # openbmc/meta/lib/oe/package_manager/rpm/rootfs.py
   bb.note(str())
   bb.warn(str())
   bb.error(str())
   
   # 依赖
   DEPENDS -> 构建时依赖
   RDEPENDS -> 运行时依赖，例如：设置为自定义的 PACKAGES
   ```

## 7、recipetool工作流

1. 参考
   - [Writing a New Recipe](https://docs.yoctoproject.org/dev/dev-manual/new-recipe.html)

## 8、保存当前配置到模板

1. 保存配置的 conf/local.conf、conf/bblayers.conf 到 meta-evbcustom层，templatename为evbcustom

   - `bitbake-layers save-build-conf /home/gxh/openbmc/meta-evbcustom evbcustom`

2. 保存的位置

   - `/home/gxh/openbmc/meta-evbcustom/conf/templates/evbcustom/bblayers.conf.sample`
   - `/home/gxh/openbmc/meta-evbcustom/conf/templates/evbcustom/local.conf.sample`

3. 使用模板

   - local.conf会指定MACHINE，bblayers.conf会指定包含的层

   - `TEMPLATECONF=/home/gxh/openbmc/meta-evbcustom/conf/templates/evbcustom . /home/gxh/openbmc/oe-init-build-env build/evb-ast2600`

## 9、openbmc runqemu

1. 启动vnc服务器
   - `runqemu publicvnc`
   - `runqemu qemuparams="-vnc :0"`
   - `-vnc :0 -> 5900`, `-vnc :1 -> 5901`
   - `-serial mon:vc -vnc :0`
   - `-serial chardev:char0 -chardev vc,id=char0 -vnc :0`
   - VNC是将VGA显示接口重定向到VNC，因此暂时无法启动多个，可看代码后尝试修改
   - 查看 qemu chardev vc 的属性：`qemu-system-arm -chardev vc,help`
2. 指定内核启动参数
   - `runqemu bootparams="earlyprintk"`
3. 映射端口
   - `qemu-system-arm -m 1024 -M ast2600-evb -nographic -drive file=./obmc-phosphor-image-evb-ast2600.static.mtd,format=raw,if=mtd -net nic -net user,hostfwd=:127.0.0.1:2222-:22,hostfwd=:127.0.0.1:2443-:443,hostfwd=:127.0.0.1:12345-:12345,hostfwd=udp:127.0.0.1:2623-:623,hostname=qemu`
   - `qemu-system-arm -machine ast2600-evb -m 1G -net nic -net user,hostfwd=:127.0.0.1:2222-:22,hostfwd=:127.0.0.1:2443-:443,hostfwd=:127.0.0.1:12345-:12345,hostfwd=udp:127.0.0.1:2623-:623,hostname=qemu -drive file=./core-image-minimal-evb-ast2600.static.mtd,if=mtd,format=raw -serial mon:vc -serial null -vnc :0`
4. 参考
   - [qemu选项](https://www.qemu.org/docs/master/system/qemu-manpage.html)
   - `runqemu --help`

## 10、总结

1. BBPATH是每一层的conf的上层目录，build目录也在BBPATH
2. 先在环境变量$(BBPATH)/conf中找到bblayers.conf，然后必须在某一层找到bitbake.conf

## 11、依赖

- `pip3 install pyobject`
- `apt install libgirepository1.0-dev libgtk-3-dev`

## 12、参考

- [yocto](https://docs.yoctoproject.org/)
- [bootlin](https://bootlin.com/training/yocto/)
- [oebuild && crosstools](https://pages.openeuler.openatom.cn/embedded/docs/build/html/openEuler-24.03-LTS/index.html)
- [bitbake helloworld](https://docs.yoctoproject.org/bitbake/2.10/bitbake-user-manual/bitbake-user-manual-hello.html)
- [bitbake 变量-BBPATH...](https://docs.yoctoproject.org/bitbake/2.10/bitbake-user-manual/bitbake-user-manual-ref-variables.html)
- [bitbake VariableFlags](https://docs.yoctoproject.org/bitbake/2.10/bitbake-user-manual/bitbake-user-manual-metadata.html#variable-flags)
- [jia.je openbmc qemu](https://jia.je/system/2023/08/11/openbmc-qemu/)
- [qemu选项](https://www.qemu.org/docs/master/system/qemu-manpage.html)
- [Yocto quick start](https://docs.yoctoproject.org/brief-yoctoprojectqs/index.html)
- [github poky](https://github.com/yoctoproject/poky.git)