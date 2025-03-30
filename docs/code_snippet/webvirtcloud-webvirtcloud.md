# webvirtcloud - webvirtcloud

## 1、webvirtcloud controller
- 安装
    ```shell
    git clone https://github.com/webvirtcloud/webvirtcloud.git
    
    # Dockerfile.frontend Dockerfile.backend
    ENV http_proxy="http://192.168.0.111:10809"
    ENV https_proxy="http://192.168.0.111:10809"
    
    # webvirtcloud 的 backend 和 frontend 的 Dockerfile，最后都要把代理去掉；否则，backend 和 compute通过http通信就会被代理转发。
    ENV http_proxy=
    ENV https_proxy=
    
    # 更新 docker-compose 版本
    wget https://github.com/docker/compose/releases/download/v2.34.0/docker-compose-linux-x86_64 -O /usr/bin/docker-compose
    chmod 755 /usr/bin/docker-compose
    
    cp Caddyfile.noncert Caddyfile
    ./webvirtcloud.sh env
    export http_proxy="http://192.168.0.111:10809"
    export https_proxy="http://192.168.0.111:10809"
    
    ./webvirtcloud.sh start
    
    # 出现 webvirtcloud-db-migration 服务启动失败，stop 再 start，直到 admin@webvirt.cloud 被创建
    ./webvirtcloud.sh stop
    
    docker network inspect webvirtcloud_default
    
    # 进入 webvirtcloud 的 backend
    docker exec -it webvirtcloud-backend sh
    ```

- 使用

  ```shell
  # localhost/admin
  # Username: admin@webvirt.cloud
  # Password: admin
  
  # 添加 user
  # test@test.com
  # gxhgxhgxh
  
  # 添加 compute
  # name: compute1
  # Hostname: 172.19.0.9
  # Token: edebec0cca2998964cdb72ecd8d41053edbddf475c627a3056af7de5ee8012d2
  
  # localhost/sign_in
  # Deploy virtance -> ubuntu20.04
  # 进入 vnc，若re-mounted: opts => 重新创建一个内存大的虚拟机即可
  
  # 进入 admin，查看虚拟机创建情况；先要下载对应Distribution的qcow2文件
  ```

## 2、webvirtcloud compute

- Dockerfile

  ```dockerfile
  FROM ubuntu:22.04
  RUN sed -i 's@archive.ubuntu.com@mirrors.ustc.edu.cn@g' /etc/apt/sources.list
  RUN sed -i 's@security.ubuntu.com@mirrors.ustc.edu.cn@g' /etc/apt/sources.list
  # 禁止交互
  ENV DEBIAN_FRONTEND=noninteractive
  RUN apt update
  RUN apt install -y systemd systemd-sysv locales sudo vim make curl wget net-tools
  RUN apt install -y libvirt-clients bridge-utils
  RUN apt install -y network-manager firewalld
  RUN apt install -y prometheus prometheus-node-exporter
  RUN apt install -y linux-image-generic
  RUN apt install -y iputils-ping bind9-dnsutils
  # 防止 ebtables: RULE_APPEND failed
  RUN apt install -y ebtables
  RUN ln -sf /usr/bin/python3 /usr/bin/python
  RUN ln -sf /usr/bin/bash /usr/bin/sh
  
  RUN usermod -p "\$6\$UGMqyqdG\$GqTb3tXPFx9AJlzTw/8X5RoW2Z.100dT.acuk8AFJfNQYr.ZRL8itMIgLqsdq46RNHgiv78XayOSl.IbR4DFU." root;
  
  CMD ["/sbin/init"]
  ```

- 配置

  ```shell
  # 一、运行 compute 的 container
  # webvirtcloud controller 会自动创建 webvirtcloud_default 网桥；
  # controller frontend 把 80 端口映射到 host，但是 backend 在 docker 里，连接到 webvirtcloud_default 网桥；
  # compute 要能和 backend 通信，所以 compute 必须接到 webvirtcloud_default 网桥
  
  # 启动 compute1: 将 compute 部署到 docker(本质和部署在host一致，因为只是隔离资源，host能看到docker container的进程)
  docker build -t ubuntu_system .
  
  # container的 eth0 必须对应 docker0(bridge)，否则 域名会无法访问，和默认网关无关
  # 网桥不能 up，否则 也会导致 域名无法解析；因为影响了 iptables 对 DNS(53端口) 的转发
  
  # docker run --name compute1 --privileged=true --cap-add=NET_ADMIN -d --rm --device /dev/kvm --network=webvirtcloud_default ubuntu_system
  docker run --dns 8.8.8.8 --name compute1 --privileged=true --cap-add=NET_ADMIN -d --rm --device /dev/kvm --network webvirtcloud_default ubuntu_system
  docker network connect bridge compute1
  docker exec -it compute1 bash
  echo "nameserver 8.8.8.8" > /etc/resolv.conf
  # docker stop compute1
  
  # 二、配置网络
  # 添加网桥
  # 清除 eth1 所有IP，被桥接的设备不能配IP，否则网络会不通
  ip addr flush dev eth0
  
  # 创建网桥并配置基础参数
  brctl addbr br-ext
  brctl stp br-ext off
  ip link set br-ext mtu 1500
  
  # 添加物理接口到网桥
  ip link set eth0 down
  brctl addif br-ext eth0
  ip link set eth0 up
  
  # 配置多IP地址，配置IP时的掩码会对应路由表一个表项，匹配就会将报文从该网络接口发出；都不匹配就发给默认网关
  ip addr add 10.255.0.1/16 dev br-ext
  ip addr add 169.254.169.254/16 dev br-ext
  # 同 backend 通信的 IP范围
  ip addr add 172.19.0.9/16 dev br-ext
  
  # 激活网桥 (external)
  ip link set br-ext up
  
  
  # 清除eth0所有IP，被桥接的设备不配IP，否则网会不通
  ip addr flush dev eth1
  brctl addbr br-int
  brctl stp br-int off
  ip link set br-int mtu 1500
  
  ip link set eth1 down
  brctl addif br-int eth1
  ip link set eth1 up
  
  ip addr add 172.17.0.2/16 dev br-int
  
  ip link set br-int up
  brctl show && ip addr show br-int
  
  # 配置 docker container 的 默认网关 (报文离开docker container)
  ip route add default via 172.17.0.1
  ip route show
  
  export https_proxy="http://192.168.0.111:10809"
  # 三、安装 libvirt: 会配置 virsh 的 public、private 网络、nwfilter，virsh 的 images、isos，sysctl ip_forward；所以必须全部执行
  # 如不全部执行，会出现 => Network not found: no network with matching name 'public'
  curl -fsSL https://raw.githubusercontent.com/webvirtcloud/webvirtcompute/master/scripts/libvirt.sh | sed '98a\systemctl restart libvirtd'| bash
  
  # 四、安装 Prometheus
  curl -fsSL https://raw.githubusercontent.com/webvirtcloud/webvirtcompute/master/scripts/prometheus.sh | sed 's@21.04@22.04@g' | bash
  
  # 五、安装 webvirtcompute
  curl -fsSL https://raw.githubusercontent.com/webvirtcloud/webvirtcompute/master/scripts/install.sh | bash
  
  # 确保 compute 能访问 https://cloud-images.webvirt.cloud/ubuntu-22-04-x64.qcow2
  sed -i '8a\Environment="https_proxy=http://192.168.0.111:10809"'  /etc/systemd/system/webvirtcompute.service
  
  systemctl daemon-reload
  systemctl restart webvirtcompute.service
  systemctl cat webvirtcompute.service
  
  # 六、FAQ
  # 1、supermin: failed to find a suitable kernel (host_cpu=x86_64)
  #     apt install linux-image-generic
  # 2、Network not found: no network with matching name 'public'
  #     完全执行 libvirt.sh
  # 3、ebtables --concurrent -t nat -N libvirt-J-vnet0 -> TABLE_ADD
  #     暂时通过去掉 webvirtcompute/src/vrtmgr/libvrt.py 注释掉nwfilter的分支部分规则
  # 4、compute端无法访问 https://cloud-images.webvirt.cloud/ubuntu-22-04-x64.qcow2
  #     webvirtcompute.service 添加 Environment="https_proxy=http://192.168.0.111:10809 代理
  # 5、ebtables --concurrent -t nat -A J-vnet0-ipv4-ip -p ipv4 --ip-source 0.0.0.0 --ip-protocol 17 -j RETURN: RULE_APPEND failed (No such file or directory): rule in chain J-vnet0-ipv4-ip
  #     apt install ebtables
  ```

- 源码构建并手动安装 webvirtcompute

  ```shell
  # 一、构建
  git clone https://github.com/webvirtcloud/webvirtcompute.git
  
  # Dockerfile.ubuntu2204
  RUN sed -i 's@archive.ubuntu.com@mirrors.ustc.edu.cn@g' /etc/apt/sources.list
  RUN sed -i 's@security.ubuntu.com@mirrors.ustc.edu.cn@g' /etc/apt/sources.list
  
  ENV http_proxy="http://192.168.0.111:10809"
  ENV https_proxy="http://192.168.0.111:10809"
  
  ENV http_proxy=
  ENV https_proxy=
  
  # 构建 编译环境的docker镜像
  docker build -t webvirtcompute:ubuntu2204 -f Dockerfile.ubuntu2204 .
  
  # 暂时规避 ebtables --concurrent -t nat -N libvirt-J-vnet0; TABLE_ADD failed. => 这是WSL内核支持的问题，docker本质上是用户态程序的依赖打包而已
  # 解决方案：webvirtcompute/src/vrtmgr/libvrt.py 注释掉nwfilter的分支部分即可
  
  # 确保 compute 可以访问 https://cloud-images.webvirt.cloud/ubuntu-22-04-x64.qcow2
  sed -i '8a\Environment="https_proxy=http://192.168.0.111:10809"' ./src/dist/webvirtcompute.service
  sed -i '8a\Environment="https_proxy=http://192.168.0.111:10809"' ./conf/webvirtcompute.service
  
  make -f Makefile.ubuntu2204 compile
  make -f Makefile.ubuntu2204 package
  
  # 二、host
  docker cp release/webvirtcompute-ubuntu2204-amd64.tar.gz compute1:/tmp/
  
  # 三、docker: 安装 webvirtcompute
  TOKEN=$(echo -n $(date) | sha256sum | cut -d ' ' -f1)
  tar -xvf /tmp/webvirtcompute-ubuntu2204-amd64.tar.gz -C /tmp
  cp /tmp/webvirtcompute/webvirtcompute /usr/local/bin/webvirtcompute
  chmod +x /usr/local/bin/webvirtcompute
  
  mkdir -p /etc/webvirtcompute
  cp /tmp/webvirtcompute/webvirtcompute.ini /etc/webvirtcompute/webvirtcompute.ini
  sed -i "s/token = .*/token = $TOKEN/" /etc/webvirtcompute/webvirtcompute.ini
  cp /tmp/webvirtcompute/webvirtcompute.service /etc/systemd/system/webvirtcompute.service
  systemctl daemon-reload
  systemctl enable --now webvirtcompute
  echo -e "Installing webvirtcompute... - Done!\n"
  
  # Show token
  echo -e "\nYour webvirtcompue connection token is: \n\t\n$TOKEN\n\nPlease add it to admin panel when you add the compute node.\n"
  
  # Cleanup
  rm -rf /tmp/webvirtcompute*
  ```

- compute端使用virsh

    ```shell
    # 进入虚拟机串口： 已经被libvirtd连接了，所以无法使用minicom连接了;monitor也是如此
    virsh console Virtance-12
    
    # 执行 monitor 命令
    virsh qemu-monitor-command Virtance-12 --hmp help
    
    # 查看所有的虚拟机 domain
    virsh list --all
    
    # 查看 libvirtd 的网络配置，会被 domain 的 xml source
    virsh net-list --all
    
    # 查看 default 的网络xml配置
    virsh net-dumpxml default
    
    # 修改 network xml配置
    virsh net-edit public
    
    # 查看 一个命令的 help
    virsh help net-dumpxml
    
    # 注册 libvirt 网络
    virsh net-define public.xml
    # 启动网络
    virt net-start public
    # 设置自启动
    virsh net-autostart public
    
    # 修改 domain 的xml
    virsh edit Virtance-10
    
    # python3-livirt使用 - apt install python3-libvirt
    ipython3
    > import libvirt
    > wvm = libvirt.open("qemu:///system")
    > vm = wvm.lookupByName("Virtance-6")
    > wvm.listAllDomains()[0].name()
    > wvm.listAllDomains()[0].XMLDesc()
    
    # 格式化查看 domain 的 xml
    # 去掉 \n
    sed -i 's@\\n@@g' input.xml
    xmllint --format input.xml -o output.xml
    # 格式化json
    python3 -m json.tool input.json > output.json
    jq . input.json > output.json
    ```

    [virsh](https://www.cnblogs.com/qiuhom-1874/p/13510721.html)

- wsl支持ebtables 和 wireguard(nft)

  ```shell
  # apt install libelf-dev flex bison libssl-dev dwarves
  
  # 编译 wsl 内核
  # 1、下载 linux-msft-wsl-5.15.167.4 源码
  # 2、cat /proc/config.gz | gunzip -dc > .config
  # 3、make -j24 bzImage modules
  # 4、添加 ko 后，最好把 bzImage 也重新加载，因为vmlinux会被重新编译
  # 5、总结：通过重新编译wsl的内核即可满足 ebtalbes 和 wireguard 的需求，不需要加载ko
  
  # wsl --shutdown
  # 用 arch/x86/boot/bzImage 替换 C:\ProgramFiles\WSL\tools\kernel
  
  # 不确定
  BRIDGE_NF_EBTABLES
  -> Networking support (NET [=y])                                             
    -> Networking options                                                     
      -> Network packet filtering framework (Netfilter) (NETFILTER [=y])     
        -> Ethernet Bridge tables (ebtables) support (BRIDGE_NF_EBTABLES [=n]) 
  
  # 进入目录，全部加 m
  
  # 不确定
  BRIDGE
  -> Networking support (NET [=y])                                               
    -> Networking options                                                     
      -> 802.1d Ethernet Bridging (BRIDGE [=n]) 
  
  # 确定，WSL /proc/config.gz CONFIG_NF_TABLES_BRIDGE=m，所以把bzImage编译后拿过来即可
  NF_TABLES_BRIDGE
  -> Networking support (NET [=y])                                             
    -> Networking options 
      -> Network packet filtering framework (Netfilter) (NETFILTER [=y]) 
        -> Ethernet Bridge nf_tables support
  # 进入目录，全部加 m
  
  find . -name ebt*.ko | xargs -I{} insmod {}
  ```

- FAQ

    - Failed to generate BTF for vmlinux -> `apt install dwarves`

    - WSL kernel 位置：C:\ProgramFiles\WSL\tools\kernel

    - WSL配置：C:\Users\xxx\.wslconfig

      ```shell
      [wsl2]
      memory=24G
      kernel=C:\\bzImage
      ```

## 3、wireguard 配置 VPN

- WSL - 中继服务器

  ```shell
  # /etc/wireguard/wg0.conf
  
  [Interface]
  Address = 192.168.3.1/24                 # wsl 的 wg0 配置的 VPN 网段IP
  PrivateKey = 4O6gBgrMBz+nGR0lqEiMdzwq4IzwRiom6T1RYxQbI0M=
  ListenPort = 51820
  PostUp   = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o br-6a7541fe74a1 -j MASQUERADE
  PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o br-6a7541fe74a1 -j MASQUERADE
  
  # windows client
  [Peer]
  PublicKey = 5dYw8RR4dEmP172lV0povGO/hUemFRUfegZU7TvrLG0=
  AllowedIPs = 192.168.3.2/32  # 目的IP在该范围的，通过VPN隧道(wg0网卡)发送；此处指定 VPN IP 192.168.3；此处 windows 所在的子网网段不被开放，也不会被中继服务器转发
  
  # docker client
  [Peer]
  PublicKey = FTt7tPJG8Vi049qKDTnCSl+4ggJmdGJTPbjz2Mjx+XA=
  AllowedIPs = 192.168.3.3/32, 192.168.33.0/24 # 目的IP在该范围的，通过VPN隧道(wg0网卡)发送；此处指定 VPN 网段 192.168.3.3 和 192.168.33.0/24 网段；外部访问 192.168.33.0/24网段的，会被代为转发到 192.168.3.3
  ```

- windows client

  ```shell
  [Interface]
  PrivateKey = gAw98k/7d7i2KTOm8720zL8xt+Ml8f/vsBRy7smBTms=
  Address = 192.168.3.2/24
  
  [Peer]
  PublicKey = JjMr+eFD06oydpYCp2jPtp9PFBfudakwVXDmNE7yzRg=
  AllowedIPs = 192.168.3.0/24, 172.19.0.0/16  # 目的IP在该范围的，通过VPN隧道(wg0网卡)发送；此处指定 VPN 网段 192.168.3.0/24 和 172.19.0.0/16 网段
  Endpoint = 192.168.39.45:51820
  PersistentKeepalive = 25
  ```

- docker client

  ```shell
  Address = 192.168.3.3/24
  PrivateKey = UFaEIO800ipqydm8/6NDtu6ZBbWGgDtjcX8AKeacdlA=
  PostUp   = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o br-ext -j MASQUERADE
  PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o br-ext -j MASQUERADE
  
  [Peer]
  PublicKey = JjMr+eFD06oydpYCp2jPtp9PFBfudakwVXDmNE7yzRg=
  AllowedIPs = 192.168.3.0/24              # 目的IP在该范围的，通过VPN隧道(wg0网卡)发送；此处指定 VPN 网段 192.168.3.0/24 网段；不是开放给别人的子网网段，对本地子网的访问由中继服务器转发
  Endpoint = 192.168.39.45:51820
  PersistentKeepalive = 25
  ```

- command

  ```shell
  apt install wireguard
  
  # 生成服务端密钥对
  cd /etc/wireguard/
  wg genkey | tee privatekey | wg pubkey > publickey
  
  # 生成客户端1私钥
  wg genkey > client1.key
  # 通过私钥生成客户端1密钥
  wg pubkey < client1.key > client1.key.pub
  
  # 生成客户端2私钥
  wg genkey > client2.key
  # 通过私钥生成客户端2密钥
  wg pubkey < client2.key > client2.key.pub
  
  systemcel enable wg-quick@wg0.service
  systemcel start wg-quick@wg0.service
  systemctl list-units --type=service
  
  wg-quick up wg0
  wg-quick down wg0
  wg show
  ```

- 参考

  - [wireguard访问内网](https://xiexiage.com/posts/vpn-wireguard)
  - [wsl使用wireguard](https://medium.com/@emryslvv)
  - [wireguard中继组网](https://blog.csdn.net/networken/article/details/137670459)
  - [ebtables](https://www.cnblogs.com/xuanxuanBOSS/p/11424290.html)
  - [Ethernet Bridging](http://docs.kernel.org/networking/bridge.html)

## 4、虚拟机网络分析

- docker里虚拟机访问外网：

  ```shell
  # slave(虚拟机)
      ip route del default
  	ip route add default via 192.168.33.1
  
  # 必须执行，否则虚拟机就是无法访问代理IP地址
  # docker host:
  	ip addr add 192.168.33.1/24 dev br-ext
  	iptables -A POSTROUTING -t nat -j MASQUERADE -s 192.168.33.1/24
  	# iptables -A POSTROUTING -t nat -j MASQUERADE
  	iptables -t nat -L POSTROUTING --line-number
  	# 删除一条指定的 POSTROUTING规则
  	iptables -t nat -D POSTROUTING [N]
  ```

- slave 虚拟机报文流程(NAT)：

  - 访问外网流程

    ```shell
    # 目的IP(代理所在的 IP): 192.168.0.111
    192.168.33.71(虚拟机IP) -> 路由选择(本地无匹配网络接口) -> 使用默认网关(192.168.33.1) -> docker host
    
    docker host (目标非 docker host IP，需 NAT) -> 路由选择(仍然无本地网络接口可处理) -> 172.19.0.1(报文流出 docker host) -> MASQUERADE(src替换为172.19.0.9，由 docker host 代替 虚拟机 访问外网)
    ```

  - 访问docker host 的 172.17.0.2 流程

    ```shell
    # 目的IP(docker host 的 另一个网口 IP): 172.17.0.2
    192.168.33.71(虚拟机IP) -> 路由选择(本地无匹配网络接口) -> 使用默认网关(192.168.33.1) -> docker host
    
    docker host (目标 为 docker host IP) -> 直接处理
    ```

  - 访问docker host 的 所在子网的其他主机(docker0 的 另一个IP 172.17.0.3)

    ```shell
    # 目的IP(docker0 的 另一个IP): 172.17.0.3
    192.168.33.71(虚拟机IP) -> 路由选择(本地无匹配网络接口) -> 使用默认网关(192.168.33.1) -> docker host
    
    docker host (目标非 docker host IP，需 NAT) -> 路由选择(br-int 172.17.0.2/16) -> br-int(流出docker host) -> MASQUERADE(src替换为172.17.0.2，由 docker host 代替 虚拟机 访问 另一个子网) -> docker0
    
    br-int(docker host) -> eth1(docker host) -> (veth paris) -> veth0 (wsl tap) -> docker0 -> 另一个docker container
    ```

- 虚拟机网络总结(重点是NAT)

  1. 当一台主机的 IP 作为默认网关时，该子网内任何主机访问 网关所在的另一个子网时，网关都必须要配置 NAT，相当于路由器
  2. docker host中的虚拟机，可直接 ping通 172.17.0.2, 10.255.0.1 等 docker host 的 IP，因为是docker host 本地处理；不可以 ping 通 172.17.0.3，只要虚拟机通过网关访问另一个子网，不必是默认网关，都必须配置NAT(MASQUERADE)，网关代替虚拟机访问(因为必须修改 src IP，否则无法收到响应)。
  3. 虚拟机网络：
     - 桥接： 虚拟机网卡 和 host 网卡添加到同一个网桥，虚拟机网卡配置为 和 host 访问外网的同网段IP，直接和外网通信
     - NAT： 虚拟机网卡 和 host 一个网卡组成一个内网网段，添加到一个网桥；host 另一个网卡(或同一个网卡)可访问外网，虚拟机访问外网的流程由 docker host 转发(src IP 替换为 docker host 在另一个子网的 IP)
  4. 路由器就是通过NAT功能代替内网访问外网的
     - 解决ipv4地址不足问题
     - 隐藏内网结构
     - 一有网关(默认网关/掩码网关)，网关必须实现 NAT 转发，即 MASQUERADE
  5. 非网关主机网络
     - 路由表/路由表项，就是为了确定用户程序产生的报文，从哪个网口发出去
     - 默认网关本质上就是一个掩码网关，只不过掩码为 0.0.0.0
     - 非网关主机对 NAT 不感知，只有网关主机才需要关心其子网的 NAT 转发
