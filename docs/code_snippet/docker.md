# docker
- 添加 controller - 控制节点
  
    - docker使用systemd启动
    
        - wsl映射的端口在windows上是127.0.0.1，只能通过localhost访问，不能通过192.168.39.45访问
    
        - 403: CSRF verification failed. Request aborted.
    
        - 出现502: systemctl restart supervisor nginx
    
        - ```shell
          FROM ubuntu:22.04
          RUN sed -i s#archive.ubuntu.com#mirrors.tuna.tsinghua.edu.cn#g /etc/apt/sources.list
          RUN sed -i s#security.ubuntu.com#mirrors.tuna.tsinghua.edu.cn#g /etc/apt/sources.list
          # 禁止交互
          ENV DEBIAN_FRONTEND=noninteractive
          RUN apt update
          RUN apt install -y systemd systemd-sysv locales sudo vim make curl wget net-tools
          RUN apt install -y libvirt-daemon-system  virt-manager
          RUN apt install -y qemu-kvm libvirt-clients bridge-utils libguestfs-tools virt-viewer virtinst
          RUN apt install -y python3-pip python3-libvirt python3-libxml2 python3-lxml novnc python3-dev virtualenv python3-virtualenv python3-guestfs
          RUN apt install -y lsb-release pkg-config gcc git supervisor nginx
          RUN apt install -y libvirt-dev zlib1g-dev libxslt1-dev libsasl2-modules libsasl2-dev libldap2-dev libssl-dev
          RUN ln -sf /usr/bin/python3 /usr/bin/python
          RUN ln -sf /usr/bin/bash /usr/bin/sh
          
          RUN usermod -p "\$6\$UGMqyqdG\$GqTb3tXPFx9AJlzTw/8X5RoW2Z.100dT.acuk8AFJfNQYr.ZRL8itMIgLqsdq46RNHgiv78XayOSl.IbR4DFU." root;
          
          ENV https_proxy=http://192.168.0.111:10809
          RUN cd /root && wget https://raw.githubusercontent.com/retspen/webvirtcloud/master/webvirtcloud.sh
          RUN sed -i "s@20.04@22.04@g" /root/webvirtcloud.sh
          RUN chmod 755 /root/webvirtcloud.sh
          RUN (echo n; sleep 0.1; echo; sleep 0.1; echo; sleep 0.1) | /root/webvirtcloud.sh
          
          CMD ["/sbin/init"]
          ```
        
        - 构建
        
          ```shell
          # 构建 weblibvirt 镜像
          docker build -t weblibvirt .
          
          # 后台特权运行镜像，创建container
          docker run --name webvirt --privileged=true -itd --rm --device /dev/kvm -v $(pwd)/run:/run weblibvirt
          
          # 不映射 /run
          docker run --name webvirt --privileged=true -d --rm --device /dev/kvm -p 80:80 -p 6080:6080 weblibvirt
          
          # 不需要映射6080也可以，只要访问localhost即可；映射images
          docker run --name webvirt --privileged=true -d --rm --device /dev/kvm -p 80:80 -v $(pwd)/images:/var/lib/libvirt/images/ weblibvirt
          
          # 进入container执行bash
          docker exec -it webvirt bash
          
          # 复制文件到container
          docker cp webvirtcloud.sh webvirt:/root
          
          # 停止container
          docker stop webvirt
          
          # 清除container
          echo y | docker container prune
          
          # 进入virsh命令行
          virsh -c qemu:///system?socket=$(pwd)/run/libvirt/libvirt-sock
          
          # 进入虚拟机管理GUI
          virt-manager -c qemu:///system?socket=$(pwd)/run/libvirt/libvirt-sock
          ```
    
- 添加compute - 计算节点

  - docker compute1

      - host: `apt install qemu-kvm`

      - docker

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
          RUN apt install -y openssh-server
          RUN ln -sf /usr/bin/python3 /usr/bin/python
          RUN ln -sf /usr/bin/bash /usr/bin/sh
          RUN sed -i 's@^#\?PermitRootLogin.*@PermitRootLogin yes@' /etc/ssh/sshd_config
          RUN usermod -p "\$6\$UGMqyqdG\$GqTb3tXPFx9AJlzTw/8X5RoW2Z.100dT.acuk8AFJfNQYr.ZRL8itMIgLqsdq46RNHgiv78XayOSl.IbR4DFU." root;
          
          CMD ["/sbin/init"]
          ```
      
      - 构建
      
          ```shell
          # 构建 libvirt 镜像
          docker build -t libvirtd .
          
          # 后台运行 compute1
          docker run --name compute1 --privileged=true -d --rm --device /dev/kvm -p 2222:22 libvirtd
          
          # 连接到 container，密码为 0penBmc
          ssh -p 2222 root@127.0.0.1
          
          # 停止 container
          docker stop compute1
          
          # 查看所有docker container
          docker ps -a
          
          # 删除 container
          docker container prune
          ```
      
      - 使用 PPGUI 将 windows的 0.0.0.0:2222 转发到 192.168.39.45:2222
      
      - 在windows的防火墙的入站、出站规则 开放 2222 TCP端口
      
  - 从webvirtcloud推送公钥到compute1

      ```shell
      # 进入 webvirtcloud 的 docker 命令行
      chown www-data -R ~www-data
      sudo -u www-data ssh-keygen
      cat > ~www-data/.ssh/config << EOF
      Host *
      StrictHostKeyChecking no
      EOF
      chown www-data -R ~www-data/.ssh/config
      
      # 密码为 0penBmc
      sudo -u www-data ssh-copy-id -p 2222 root@192.168.0.159
      
      # web 使用ssh添加计算节点: 192.168.0.159:2222  root
      ```
  