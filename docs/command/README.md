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