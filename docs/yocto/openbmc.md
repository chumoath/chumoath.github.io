# openbmc

## 1、phosphor-objmgr各方法的使用

1. objmgr所有接口的封装
   - `bmcweb/src/dbus_utility.cpp`
   - `bmcweb/include/dbus_utility.hpp`
2. 使用dbus-send
2. [dbus-send源码](https://dbus.freedesktop.org/releases/dbus/): `dbus/tools/dbus-send.c`

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

4. FAQ
   - boost大多数时候只是用头文件，不怎么用动态库
   - `CMAKE_EXE_LINKER_FLAGS`不能添加 `-L`选项，否则链接时会报Scrt1.o未定义的符号：main
   - 模板一般在头文件中，否则会出现链接找不到符号问题；若要在.cpp文件中定义，则先要声明指定类型的模板存在
   - `dbus_request_name`只是取一个别名；sd_bus_default的时候就把app注册到bus上了，通过`busctl list`即可查看；只有app提供服务的时候，才需要注册别名，使用boost异步接口。

## 3、传感器数据上报web全流程

## 4、dbus上接口创建全流程

- 使用 phosphor-dbus-interfaces

## 5、boost库交叉编译

## 6、obmc-console

- `bmcweb/include/obmc_console.hpp`

- `xyz.openbmc_project.Console.Access`

- obmc-console的dbus返回的是一个文件描述符，供web的websocket使用

  ```c
  // obmc-console/console-dbus.c
  int method_connect(sd_bus_message *msg, void *userdata, sd_bus_error *err)
      socket_fd = dbus_create_socket_consumer(console);
      rc = sd_bus_reply_method_return(msg, "h", socket_fd);
  
  obmc-console/socket-handler.c
      /* Create a socketpair */
      rc = socketpair(AF_UNIX, SOCK_STREAM, 0, fds);
  ```