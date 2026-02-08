# debuild

### 1、从源码构建deb包

```shell
# 0) 安装构建工具链
apt install devscripts

# 1) 下载源代码
apt source gdbserver

# 2) 安装依赖
apt build-dep -y .

# 3) 构建
debuild -j$(proc)
```

### 2、dpkg命令

```shell
dpkg -l           # 查看系统已安装的包
dpkg -L gdbserver # 查看已安装的包的内容
```

### 3、构建流程

```shell
# debuild 调用 debian/rules的build目标
```

### 4、从源码构建deb

```shell
# gdb
#   - gdb_12.1-0ubuntu1~22.04.2.debian.tar.xz
#   - gdb_12.1-0ubuntu1~22.04.2.dsc 
#   - gdb_12.1.orig.tar.xz
#   - gdb-12.1
```

### 5、镜像源的结构和搭建

```shell

```

### 6、参考链接

- [debian维护者指南](https://www.debian.org/doc/manuals/debmake-doc/debmake-doc.zh-cn.pdf)