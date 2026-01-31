# Harmony Next laptop

### 1、快捷键(完全同Win)： 
- 最小化所有窗口：win + D
- 锁屏：win + L
- 任务管理器：Ctrl + shift + ESC
- 打开任务管理器的界面：Ctrl + ALT + Del

### 2、虚拟机 - 铠大师(鸿蒙应用商店) windows-arm64

- CPU (stratovirt 6核6线程，3GHZ)
- 网络 (Red Hat Virtio Ethernet Adapter)
- 硬盘 (Red Hat Virtio SCSI Disk Device)
- 显示 （QraylddDriver Device) (SVGAMPWDDM Device) 对应 (NVIDIA GeForce RTX 4060 Ti)

### 3、任务管理器
- 后台进程 (*_host、vm_manager、stratovirt、dev_manager、hdf_manager、render_service、foundation、appspwan、composer_host、allocator_host)
- stratovirt 相当于 Qemu
- vm_manager 相当于 libvirt
- 显示: composer_host、render_service、foundation

### 4、动画
- 类似MAC的窗口动画，很丝滑，用户体验很好

### 5、芯片、OS
- Kirin X90A
- HarmonyOS-Next、微内核
- Openharmony 5.0/6.0 (SDK?)

### 6、显卡
- 未找到，应该是集成显卡

### 7、WIFI
- 显示类似 Win

### 8、优势
- 平板和笔记本系统一样，都可以支持虚拟机，说明可以共系统，比MAC&iPad强；
- 同时支持windows快捷键和MAC的动画效果。

### 9、缺陷
- 开发工具缺失: 只有应用商店有一个 linux shell，相当于一个小的虚拟机；
- 不开虚拟机，空转情况下CPU专用率为 10%；
- 平板的处理器型号为Kirin X90A，但笔记本看不到处理器型号。