# chumoath's website

## 工具 + 框架 + 业务 = 市场

## 平台做厚，业务做薄

## 优先级排序

- 人生思考：避免一直在学习，而没有做出有价值的东西。[martins3](martins3.github.io)、志辉君的bilibili收藏的视频和项目(思考相关材料、模仿细化技术点)。
- 做头脑风暴的工具和时间，细化知识(协议/工具参数/使用场景/业界标杆/方案) visio-思维导图
- 整理技术树、系统设计场景、工具使用场景、
- shell、makefile、yocto模板、各类技术demo
- 各类标准、工具官方文档
- 接口整理，demo
- 各类参考项目、需求整理(SE输出)、客户市场反馈
- 画图(visio)、学会提前做出整理各类资源和需求的列表和清单，研究的项目是否能满足需求
- 需求 > 技术，有需求才会有技术方向，才能进步
- 结合需求和场景，再做出框架
- 测试：测试框架(CI)、流水线、代码覆盖率、函数调用深度工具
- 需求：问题单和开发需求对接，量化投入产出比
- 软件：工具 + 框架 + 业务 = 市场
- 硬件：核心板(兼容高低端CPU) + 底板(不变)
- PCIe、USB抓包工具，协议分析工具
- 系统：现有复杂系统的集成，比如 redis、mysql等其他数据库(使用场景、优缺点、业界最佳 实践列表)、图形界面

## 系统资源

- cpu (算力、频率、L1/L2/L3、系统调用开销量化、中断响应时间量化)
- gpu (显示接口 HDMI VGA DP eDP DSI)
- 硬盘 (SATA、NVMe)，SATA控制器在PCIe卡(固态硬盘)里
- 内存 (SRAM、DRAM、ROM)
- 网口 (GE? sgmii/rmii/mii? 带内、带外、PHY、交换芯片?)
- 高速总线：PCIe(x1/2/4/8 3.0/4.0)、USB(host、device、OTG、ABC口、USB phy)、FPGA(AXI DMA 中断)
- 低速总线：i2c spi can uart cpld cpld_spi pca9545 pca9555 mdio
- 外设布局：platform(soc)/PCIe (网口、硬盘、显卡)
- 功耗
- 尺寸

## 软件资源

- ROM
- BOOT(uboot/grub/uefi(可以用dts))
- OS(uefi资源使用、系统层面功能支持(mstsc/print/虚拟机))
- 图形桌面 (xfc4-session/gnome-session、Xorg、lightdm/gdm)
- 业务软件/管理软件/系统软件 隔离
- 构建子系统/目录结构 -> 业务软件构建 -> 烧片/升级 (可扩展性/稳定性/性能)

## 工具

- kernel版本
- glibc版本
- binutils版本
- gcc版本
- 系统服务 (systemd/init/openharmony)
- 版本管理/软件包管理

## 机电管理

- 上下电
- 电源
- 风扇
- 传感器告警(温度、电压、功率)
- 升级
- Web显示/操作
- 网络管理
- 命令行调试

## 需求

- 模拟级联中断控制器
- uart做网口物理层通信(DMA)
- i2c模拟控制器
- USB device: 使用imx6ull key做USB HID键盘，通过收集设置按钮代表的按键
- PCIe device: 使用 fpga/arm 做 PCIe device，通过转接线连到 PC
- linux 中断处理流程(GIC寄存器列表详细)、slab内存分配流程、内核启动流程、构建流程、升级接口兼容、驱动系统探测/设备创建、PCIe/USB枚举热插拔、CPU节能、设备工作流程  画图、列清单
- 协议标准、设备寄存器及工作流程(寄存器配置流程) 、linux内核接口、工具文档、内核文档 细化
- wireshark、vscode、lua、qemu、gem5 插件
- pybind11、nand2tetris、fpga(nand2tetris)、fpga-pcie、fpga-usb、fpga-网卡、fpga-显示器、llvm(cpu0、nand2tetris、clang代码提示)、北大编译器、jyyos、51单片机投板、蓝牙小车(ROS2)
- 参考资料和项目搜集整理
- M3模块(不带硬盘和DRAM)使用，M3模块集成了 CPU、SRAM、ROM、晶振，看不到地址线和数据线
- i2c传感器模块、eeprom
- plymouth通用插件 (学习编译原理 和 drm的简单使用)
- Xorg、openbmc、boost、linux 的异步
- gdb命令和高级实用功能的整理
- 一个工具的使用场景，快速搭建实验环境  (docker/qemu/yocto/buildroot/wsl)
- IO多路复用 内核态和用户态，需求和业界最佳实践。(select、poll、epoll)
- 功能 or 算法
- v2ray如何处理多种协议？如何转发流量？
- 实用工具用法整理 (socat、sokit、v2ray、tcpview、vscode、wireshark、visio、onenote、HE、jude、usbview)