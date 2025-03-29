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

### 2、webvirtcloud compute

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

  
