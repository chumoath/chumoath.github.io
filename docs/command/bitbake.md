# bitbake

## 1、basic

```shell
# 只删除生成件，不删除中间件
bitbake -c clean obmc-phosphor-image

# 列出一个目标的所有任务
bitbake -c listtasks virtual/kernel

# 无视时间戳
bitbake -c deploy virtual/kernel -f

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

# python调试
python3 -m pdb bitbake.py -e

bitbake-layers create-layer meta-imxcustom
bitbake-layers add-layer meta-imxcustom

# 查看packagegroup -> packagegroup-core-boot | packagegroup-base | packagegroup-obmc-apps
bitbake-layers show-recipes | grep packagegroup
# 查看image
bitbake-layers show-recipes | grep image

# 可视化查看任务依赖关系
bitbake -g -u taskexp obmc-phosphor-image
bitbake -g -u taskexp_ncurses obmc-phosphor-image

# 查看包的内容及依赖关系
oe-pkgdata-browser

# local.conf中指定terminal: OE_TERMINAL = "tmux"
# 进入构建上下文
bitbake -c devshell file

# 查找生成该包的recipe
oe-pkgdata-util lookup-recipe busybox

# 列出指定包的文件
oe-pkgdata-util list-pkg-files busybox

# 搜索哪个包有指定文件
oe-pkgdata-util find-path /bin/busybox

# 执行指定recipe的fetch任务
bitbake linux-yocto -c fetch

# 生成指定任务的依赖
bitbake -g obmc-phosphor-image

# 查看最终的recipes的层结构，最终的软件版本
bitbake-layers flatten output

# 查每个recipes属于某层
bitbake-layers show-recipes | grep llvm
```

## 2、生成件目录结构

## 3、devtool添加补丁工作流

1. 在工作区生成备份
   - `devtool modify dbus-sensors` 
   -  `devtool add hello [src]`

2. 修改源代码 或修改recipe
   - `devtool edit-recipe`

3. 构建recipe
   - `devtool build dbus-sensors`

4. 部署到目标环境
   - `devtool deploy-target dbus-sensors root@x.x.x.x`
   - 指定ssh端口；配置覆盖存在的生成件，否则存在会报错：`devtool deploy-target socat root@127.0.0.1 -P 2222 -s -p`
   - 只列出要拷贝的文件，不做实际拷贝：`devtool deploy-target socat root@127.0.0.1 -P 2222 -s -n`

5. 源代码提交到git
   - `git commit`

6. 用 git 补丁生成 *.bbappend
   - `devtool finish dbus-sensors /home/xxx/openbmc/meta-evbcustom`
   - `devtool update-recipe dbus-sensors -a /home/xxx/openbmc/meta-evbcustom`

7. 清除
   - `devtool reset dbus-sensors`

## 4、总结

1. BBPATH是每一层的conf的上层目录，build目录也在BBPATH
2. 先在环境变量$(BBPATH)/conf中找到bblayers.conf，然后必须在某一层找到bitbake.conf

## 5、依赖

- `pip3 install pyobject`
- `apt install libgirepository1.0-dev libgtk-3-dev`

## 6、参考

- [yocto](https://docs.yoctoproject.org/)
- [bootlin](https://bootlin.com/training/yocto/)
- [oebuild && crosstools](https://pages.openeuler.openatom.cn/embedded/docs/build/html/openEuler-24.03-LTS/index.html)