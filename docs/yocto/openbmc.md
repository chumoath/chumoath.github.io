# openbmc

## 1、phosphor-objmgr各方法的使用

1. objmgr所有接口的封装
   - `bmcweb/src/dbus_utility.cpp`
   - `bmcweb/include/dbus_utility.hpp`
2. 使用dbus-send

## 2、Ubuntu22.04使用sdbusplus和boost

1. sdbusplus的编译

   - gcc-11对c++23支持不全，会有语法问题
   - `/usr/include/boost/asio/awaitable.hpp`添加 `#include <utility>`，否则会因为没有定义`std::exchange`而提示boost未找到
   - sdbusplus的`include/sdbusplus/bus/match.hpp`的`interfacesAddedAtPath`和`interfacesRemovedAtPath`的`constexpr`改为`inline`
   - `module 'jsonschema' has no attribute 'Draft202012Validator'`报错：`rm -rf /usr/lib/python3/dist-packages/jsonschema*; pip3 install jsonschema`

   ```shell
   # 环境准备
   apt remove -y gcc-11 gcc g++
   apt install -y gcc-12 g++-12 libboost-* libsystemd-dev
   pip3 install meson inflection mako
   
   # 构建
   git clone https://github.com/openbmc/sdbusplus.git && cd sdbusplus
   // 使用旧版本，否则需要编译高版本的boost库
   git checkout 956e87a2ed85f7b415e22ba4ad610f3e9e94ccb8
   meson setup _build --prefix=/usr
   ninja -C _build
   
   # 安装路径为 DESTDIR/prefix/include、DESTDIR/prefix/etc、DESTDIR/prefix/bin
   DESTDIR=/ ninja -C _build install
   ```

2. clion在安装boost、sdbusplus、g++11改为g++12后c++等头文件不提示问题
   - 原因：linux头文件在windows缓存没有更新
   - 措施：
     1. 删除clion的远程头文件：`C:\Users\xxx\AppData\Local\JetBrains\CLion2022.2\.remote\192.168.39.45_22`
     2. clion点击`File->Invalidate Caches`
   
3. 启动sdbusplus的example的流程

   ```shell
   # 1、配置dbus总线的环境变量
   # 解决：sdbusplus::bus::bus sdbusplus::bus::new_default()：org.freedesktop.DBus.Error.FileNotFound: No such file or directorys
   export DBUS_SESSION_BUS_ADDRESS='unix:path=/run/user/0/bus'
   export DBUS_STARTER_BUS_TYPE='system'
   
   # 2、添加配置文件
   # 解决：sd_bus_request_name: org.freedesktop.DBus.Error.AccessDenied: Permission denied
   cp openbmc/meta-phosphor/recipes-phosphor/dbus/dbus-perms/org.openbmc.conf /usr/share/dbus-1/system.d/
   
   <busconfig>
     <policy context="default">
       <allow own="*"/>
       <allow send_destination="*"/>
     </policy>
   </busconfig>
   
   # 3、执行sdbusplus的example
   ./sdbusplus/_build/example/asio-example
   ```

## 3、传感器数据上报web全流程

## 4、dbus上接口创建全流程

- 使用 phosphor-dbus-interfaces

## 5、boost库交叉编译