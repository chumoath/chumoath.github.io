# docker

## 1、basic

``` shell
# 构建、运行镜像
docker build -t so2 .
docker run -it -v $(pwd):/root so2

# container停止后自动删除
docker run -it --rm -v $(pwd):/root so2

# 指定container名称
docker run --name=webvirt -it --rm -v $(pwd):/root so2

# 后台特权运行container，然后exec进入；使用stop退出
docker run --name=webvirt --privileged=true -itd --rm --device /dev/kvm -v $(pwd)/run:/run so2
docker exec -it webvirt bash
docker stop webvirt
# 进入virsh命令行
virsh -c qemu:///system?socket=$(pwd)/run/libvirt/libvirt-sock
# 进入虚拟机管理GUI
virt-manager -c qemu:///system?socket=$(pwd)/run/libvirt/libvirt-sock

# 带参数构建
docker build -t tianyi/bmc --build-arg ARG_GID=1000 --build-arg ARG_UID=2000 .

# 映射 volume、net
docker run --rm -it --cap-add=NET_ADMIN --device /dev/net/tun:/dev/net/tun -v /root:/root --workdir "/root" so2

# 执行单独命令，执行完成删除创建的container
docker run --rm -it --cap-add=NET_ADMIN --device /dev/net/tun:/dev/net/tun -v /root:/root --workdir "/root" so2 ls

# 列出正在运行的容器
docker ps

# 查看所有容器，包括停止的容器
docker ps -a

# 删除所有image
docker image prune -a

# 删除所有container
docker container prune

# 查看所有container
docker container list

# 进入正在运行的容器，容器必须启动才能进入
docker exec -it [containerID] bash
# 进入正在运行的容器，执行一个命令
docker exec -it [containerID] ls

# 启动指定container
docker start [containerID]

# 停止指定container
docker stop [containerID]

# 删除指定container
docker rm [containerID]

# 删除正在运行的container
docker rm -f [containerID]

# 统计container的资源使用
docker stats

# 将文件复制到container的指定目录
docker cp winfonts.tar.gz sharelatex:/overleaf

# 将 container 保存为 image
docker commit -a "author" -m "commit massage" [containerID] myimage:1.0

# docker查看volume
docker volume ls

# docker查看volume映射信息
docker volume inspect [volume_id]

# docker删除没有被使用的匿名volume
docker volume prune

# 查看 container 运行日志
docker-compose logs -f [containerID]
docker logs -f [containerID]
```

## 2、发布image

- server

  ```shell
  # 搭建服务
  docker pull registry:2
  docker run -d -p 5000:5000 --name registry registry:2
  
  # 推送镜像，发布前需要打域名前缀才能push
  docker tag tbmc:latest localhost:5000/tbmc:latest
  docker push localhost:5000/tbmc:latest
  ```

- client

  ```shell
  # 解决: http: server gave HTTP response to HTTPS client
  echo '{' > /etc/docker/daemon.json
  echo '    "insecure-registries": ["10.21.2.184:5000"]' >> /etc/docker/daemon.json
  echo '}' >> /etc/docker/daemon.json
  
  systemctl restart docker
  
  # 拉取镜像，拉取后重命名
  docker pull 10.21.2.184:5000/tbmc
  docker tag 10.21.2.184:5000/tbmc tbmc
  ```

## 3、docker compose

```shell
docker-compose up
docker compose up -d
docker compose down

# 构建镜像
docker-compose build docs-build
# 启动服务
docker-compose up -d docs-build
# 执行一条命令
docker-compose exec -u ubuntu docs-build bash -c "cd /linux/tools/labs && make docs"
```

- [kernel labs Makefile](https://github.com/linux-kernel-labs/linux/blob/master/tools/labs/Makefile)

## 4、docker配置代理

- `mkdir -p /etc/systemd/system/docker.service.d`

- `touch /etc/systemd/system/docker.service.d/proxy.conf`

- ```shell
  [Service]
  Environment="http_proxy=http://192.168.0.111:10809"
  Environment="https_proxy=http://192.168.0.111:10809"
  Environment="no_proxy=localhost,127.0.0.1,mirrors.tuna.tsinghua.edu.cn"
  ```

- `systemctl daemon-reload`

- `systemctl restart docker`

## 5、docker nework

- command

    ```shell
    # 查看docker network (bridge): bridge(docker0)、host、none 为默认的
    docker network ls

    # docker container 添加多个网卡
    # 创建网桥 network1 => 会自动创建该网段的MASQUERADE
    docker network create network1
    # 指定网桥子网 和 网桥IP(子网网关)
    docker network create --subnet 10.21.0.0/16 --gateway 10.21.0.1 network1

    # 指定网桥为network1
    docker run --name webvirt --privileged=true -d --rm --device /dev/kvm --network network1 weblibvirt
    # docker 默认network为 bridge，即docker0网桥
    docker run --name webvirt --privileged=true -d --rm --device /dev/kvm --network bridge weblibvirt

    # 将 network1网桥添加到container weblibvirt => 创建veth pairs虚拟以太网设备，并添加到network1网桥
    docker network connect network1 weblibvirt

    # 删除网桥network1
    docker network rm network1

    # 查看网桥network1信息
    docker network inspect network1

    # 查看网桥
    brctl show
    ```

- docker网络流量

  ```shell
  # 访问 wsl 网络
  # docker: ping 192.168.39.45，目的IP 在docker route表未匹配到，走默认网关，进入wsl的路由选择路由接口
  container1 -> eth0 -> veth0 \
  container2 -> eth0 -> veth1 -->(docker default gateway) docker0 -> wsl路由选择 -> 本地处理报文
  
  # 访问外网
  container1 -> eth0 -> veth0 \
  container2 -> eth0 -> veth1 -->(default gateway) docker0 -> wsl路由选择 -> 非WSL内部接口转发 -> wsl 默认网关 -> eth0 (外网接口: ip forward/MASQUERADE)
  
  # brctl show
  docker0 -> veth0
          -> veth1
          
  # 虚拟以太网设备
  # 虚拟以太网设备(veth pairs)成对出现
  ```
  

## 6、misc

```shell
# 打包镜像
docker save -o tbmc_latest.tar tbmc:latest

# 加载镜像
docker load -i tbmc_latest.tar

# 查看私有源镜像
curl 10.21.2.184:5000/v2/_catalog

# 查看私有仓库某个镜像版本
curl 10.21.2.184:5000/v2/tbmc/tags/list
```

## 7、参考

- [kernel labs docs](https://github.com/linux-kernel-labs/linux/blob/master/tools/labs/docker/docs)
- [kernel labs kernel_build](https://github.com/linux-kernel-labs/linux/tree/master/tools/labs/docker/kernel)
- [openharmony](https://gitee.com/openharmony/docs/tree/OpenHarmony-4.0-Release/docker)
- [ros2](https://github.com/ros2/ros2_documentation/tree/rolling/docker/image)
- [so2 assignments](https://gitlab.cs.pub.ro/so2/so2-assignments)
