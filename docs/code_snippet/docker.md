# docker

- docker使用systemd启动

  ```shell
  FROM ubuntu:22.04
  RUN sed -i s#archive.ubuntu.com#mirrors.tuna.tsinghua.edu.cn#g /etc/apt/sources.list
  RUN sed -i s#security.ubuntu.com#mirrors.tuna.tsinghua.edu.cn#g /etc/apt/sources.list
  # 禁止交互
  ENV DEBIAN_FRONTEND=noninteractive
  RUN apt update
  RUN apt install -y systemd systemd-sysv locales sudo vim make curl wget net-tools
  RUN apt install -y libvirt-daemon-system  virt-manager
  RUN apt install -y qemu-kvm libvirt-clients bridge-utils libguestfs-tools virt-viewer virtinst
  RUN apt install -y git python3-pip python3-libvirt python3-libxml2 novnc supervisor nginx
  RUN ln -sf /usr/bin/python3 /usr/bin/python
  RUN ln -sf /usr/bin/bash /usr/bin/sh
  RUN usermod -p "\$6\$UGMqyqdG\$GqTb3tXPFx9AJlzTw/8X5RoW2Z.100dT.acuk8AFJfNQYr.ZRL8itMIgLqsdq46RNHgiv78XayOSl.IbR4DFU." root;
  CMD ["/sbin/init"]
  ```

- 构建

  ```shell
  # 构建libvirt镜像
  docker build -t libvirt .
  
  # 后台特权运行镜像，创建container
  docker run --name webvirt --privileged=true -itd --rm --device /dev/kvm -v $(pwd)/run:/run libvirt
  
  # 进入container执行bash
  docker exec -it webvirt bash
  
  # 停止container
  docker stop webvirt
  
  # 进入virsh命令行
  virsh -c qemu:///system?socket=$(pwd)/run/libvirt/libvirt-sock
  
  # 进入虚拟机管理GUI
  virt-manager -c qemu:///system?socket=$(pwd)/run/libvirt/libvirt-sock
  ```

  