# 常用命令

1. 扫描指定主机服务

   `nmap -Pn [IP]`
   
2. python调试

   `python3 -m pdb bitbake.py`

3. python http服务器

   `python3 -m http.server 8888 -d /opt/html`
   
4. bash查看内建命令

   * `man bash`
   * `type ls`, `type echo`, `type cat`, `type type`
   * `help history`, `help bash`
   
5. Xming、Mobaxterm的Xserver失败

   - 使用了全局代理，Mobaxterm经常性卡住也是如此
   - Mobaxterm: `TCP :6000, export DISPLAY="localhost:10.0"`
   - Xming：`TCP :6000, export DISPLAY=:0`
   
6. socat转发x11vnc的流量

   - `socat TCP-LISTEN:12345,fork TCP:192.168.3.10:5900 &`
   
7. 增加swap分区

   - `dd if=/dev/zero of=swap.img bs=1G count=100`
   - `mkswap swap.img`
   - `swapoff -a`
   - `swapon swap.img`
   
8. 查看文件的加载，验证外部ko的Makefile加载2次

   - `make -d V=1`
   - `scripts/Makefile.build`, `scripts/Makefile.lib`

9. qemu使用user网络模式(slirp)，进入系统后的配置

   - qemu配置: `--enable-slirp`
   - `ifconfig eth0 10.0.2.15 netmask 255.255.255.0 up`
   - `route add default gw 10.0.2.2 netmask 0.0.0.0`
   
10. 查看devm的符号

   - `grep -rn EXPORT_SYMBOL drivers/ | grep devm`

11. win11 wsl不能访问host网络

    ```shell
    # 查看wsl网口别名
    Get-NetAdapter | Format-Table Name, InterfaceAlias, InterfaceDescription, Status
    
    # 允许WSL网口通过防火墙
    New-NetFirewallRule -DisplayName "Allow WSL" -Direction Inbound -InterfaceAlias "vEthernet (WSL (Hyper-V firewall))" -Action Allow
    ```

12. sshfs挂载根目录: `此电脑 -> 添加一个网络位置： \\sshfs.r\root@172.25.74.30`

13. typora将复制的图片存到指定目录: `偏好设置 -> 图像 -> 插入图片时，选择路径即可；设置优先使用相对路径`

14. 像welink一样截图：`截图软件snipaste，配置截图并复制快捷键为 Ctrl + ALT + A；截图快捷键设置为空`

15. 使用vmware下载openharmony依赖 / vmware使用VHD硬盘：

    ```shell
    # 1、磁盘管理 -> 更多操作 -> 附加VHD -> 选择wsl的ext4.vhdx; 一定不要格式化
    # 2、powershell -> Get-CimInstance -Query "SELECT * from Win32_DiskDrive"，获取vhdx的物理硬盘号
    # 3、管理员运行vmware -> 添加硬盘 -> 选择物理硬盘，启动虚拟机
    # 4、mount /dev/sdb /mnt
    # 5、cd /mnt/home/gxh/opc; docker run -it -v $(pwd):/home/openharmony swr.cn-south-1.myhuaweicloud.com/openharmony-docker/openharmony-docker:1.0.0
    ```