# obmc_console

## 一、overview

- 将 unix socket、socketpair、本地TTY 的输入输出重定向到 指定 TTY

## 二、obmc-console-server 3种通信方式

1. unix socket

   (1) socket-handler.c: socket_init 使用 console-id 创建 unix socket，并监听，注册 socket_poll； 

   (2) socket_poll: 接受client连接，console_mux_activate切换到当前console；注册client_poll，并将客户端的fd作为输入输出；

   (3) client_poll: 将客户端的fd输入输出通过 console_data_out 转发到 server->tty.fd。

2. socketpair

   (1) console-dbus.c: method_connect -> console_mux_activate 切换到当前console，调用 dbus_create_socket_consumer；

   (2) dbus_create_socket_consumer -> socketpair，注册 client_poll，将 fd[0] 作为 输入输出；将fd[1]通过systemd使用`SCM_RIGHTS`将文件描述符传递给另一个dbus进程；

   (3) client_poll: 将fd[0]输入输出通过 console_data_out 转发到 server->tty.fd。

3. 本地TTY

   (1) tty-handler.c: tty_init 通过console的 local-tty配置获取当前console的tty，打开该tty，注册tty_poll；

   (2) tty_poll: 将打开的tty作为输入输出，通过console_data_out转发到 server->tty.fd。

4. 注意：

   (1) 始终只有一个目标 tty，通过切换gpio达到连接同一串口的不同通道的效果；

   (2) 每个console是一个可用gpio切换的通道；

   (3) 多种通信方式连接到同一个console，回像串口服务器一样同时显示。

## 三、obmc-console-client工作流程

1. 根据命令行参数传递的 console-id 获取 sun_path，没有指定默认为 default；
2. 标准输入输出的TTY和 socket 之间转发数据；
3. 使用的就是 server 的 unix socket 的通信方式。

## 四、示例

1. 通过 ubuntu 的 ttyS1 使用 ttyS0 的shell

   ```shell
   # qemu配置
   ubuntu: 
   -serial pty (ttyS0 -> /dev/pts/3)
   -serial pty (ttyS1 -> /dev/pts/8)
   
   # ubuntu在ttyS0启动shell
   (ubuntu) /sbin/agetty -L 115200 ttyS0 vt100
   
   # obmc-console-server配置 - 将 /dev/pts/8 转发到 /dev/pts/3
   test.conf
   local-tty = pts/8
   
   # obmc-console-server运行
   export DBUS_SESSION_BUS_ADDRESS='unix:path=/run/user/0/bus'
   export DBUS_STARTER_BUS_TYPE='system'
   ./obmc-console-server -c test.conf /dev/pts/3
   
   # 同shell交互
   # 从 目标 TTY(/dev/pts/3) 的输入输出，都会同步到所有通信方式的连接
   方式1: (host) obmc-console-client -i default
   方式2: (ubuntu) minicom -D /dev/ttyS1
   方式3: (host) minicom -D /dev/pts/8
   
   # 数据流向
   ubuntu: /dev/ttyS1 -> host: /dev/pts/8 -> obmc-console-server转发 -> host: /dev/pts/3 -> ubuntu: /dev/ttyS0
   ```

 2. 三种通信方式数据流向

    (1) unix socket (obmc-console-client)

    - STDIN -> socketfd -> obmc-console-server 转发 -> (host) /dev/pts/3 -> (ubuntu) /dev/ttyS0

    (2) tty

    - (ubuntu) /dev/ttyS1 -> (host) /dev/pts/8 -> obmc-console-server 转发 -> (host) /dev/pts/3 -> (ubuntu) /dev/ttyS0

    (3) socketpair

    - fds[1] -> fds[0] -> obmc-console-server转发 -> (host) /dev/pts/3 -> (ubuntu) /dev/ttyS0

 3. 多个console配置

    ```shell
    # 使用修改过的代码
    test.conf
    # 设置初始的console，会调用console_mux_activate切换通道；
    # 若不指定，则使用第一个console
    active-console = console1
    [console1]
    tty = /dev/pts/3
    
    [console2]
    tty = /dev/pts/8
    
    [debug]
    log_level = 2
    
    # 运行 - bmc_clid 也要添加这两个环境变量，且不能有 单引号，否则会报错
    export DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/0/bus
    export DBUS_STARTER_BUS_TYPE=system
    
    ./obmc-console-server -c test.conf
    ./obmc-console-client -i debug
    ./obmc-console-client -i console1
    ```