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