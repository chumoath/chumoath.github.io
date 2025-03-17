# docker

## 1、basic

``` shell
# 构建、运行镜像
docker build -t so2 .
docker run -it -v $(pwd):/root so2

# 带参数构建
docker build -t tianyi/bmc --build-arg ARG_GID=1000 --build-arg ARG_UID=2000 .

# 映射 volume、net
docker run --rm -it --cap-add=NET_ADMIN --device /dev/net/tun:/dev/net/tun -v /root:/root --workdir "/root" so2

# 执行单独命令
docker run --rm -it --cap-add=NET_ADMIN --device /dev/net/tun:/dev/net/tun -v /root:/root --workdir "/root" so2 ls

docker ps

# 删除所有image
docker image prune -a

# 删除所有container
docker container prune

# 查看所有container
docker container list
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

## 4、misc

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

## 5、参考

- [kernel labs docs](https://github.com/linux-kernel-labs/linux/blob/master/tools/labs/docker/docs)
- [kernel labs kernel_build](https://github.com/linux-kernel-labs/linux/tree/master/tools/labs/docker/kernel)
- [openharmony](https://gitee.com/openharmony/docs/tree/OpenHarmony-4.0-Release/docker)
- [ros2](https://github.com/ros2/ros2_documentation/tree/rolling/docker/image)
