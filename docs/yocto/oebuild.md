# oebuild

# 1、镜像构建

```shell
pip3 install oebuild

su gxh
oebuild init -b openEuler-22.03-LTS-SP3 workdir_2203-sp3
cd workdir_2203-sp3/

groupadd docker
usermod -a -G docker chumoath
systemctl daemon-reload && systemctl restart docker
chmod o+rw /var/run/docker.sock

oebuild update
oebuild generate

cd build/qemu-aarch64
oebuild bitbake

bitbake openeuler-image-tiny
```

