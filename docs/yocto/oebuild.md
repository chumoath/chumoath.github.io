# oebuild

### 1、镜像构建

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

### 2、使用外部工具链

```shell
/usr1/openeuler/src/yocto-meta-openeuler/meta-openeuler/lib/oe/external.py
	run

/usr1/openeuler/src/yocto-meta-openeuler/meta-openeuler/classes/external_global.bbclass
	external_run -> oe.external.run
	
/home/openeuler/build/qemu-aarch64/conf/local.conf
	EXTERNAL_TOOLCHAIN_arm = "/usr1/openeuler/gcc/openeuler_gcc_arm32le"
	EXTERNAL_TOOLCHAIN_aarch64 = "/usr1/openeuler/gcc/openeuler_gcc_arm64le"
	EXTERNAL_TOOLCHAIN_x86-64 = "/usr1/openeuler/gcc/openeuler_gcc_x86_64"
	EXTERNAL_TOOLCHAIN_riscv64 = "/usr1/openeuler/gcc/openeuler_gcc_riscv64"
	EXTERNAL_TARGET_SYS_arm = "arm-openeuler-linux-gnueabi"
	EXTERNAL_TARGET_SYS_aarch64 = "aarch64-openeuler-linux-gnu"
	EXTERNAL_TARGET_SYS_x86-64 = "x86_64-openeuler-linux-gnu"
	EXTERNAL_TARGET_SYS_riscv64 = "riscv64-openeuler-linux-gnu"

/usr1/openeuler/src/yocto-meta-openeuler/meta-openeuler/classes/external-toolchain-cross.bbclass
/usr1/openeuler/src/yocto-meta-openeuler/meta-openeuler/classes/external-toolchain.bbclass

/usr1/openeuler/src/meta-openeuler/recipes-external/gcc/gcc-external-cross.bb
	require recipes-external/gcc/gcc-external.inc
	inherit external-toolchain-cross

	PN .= "-${TARGET_ARCH}"
	DEPENDS += "virtual/${TARGET_PREFIX}binutils"
	PROVIDES += "\
		virtual/${TARGET_PREFIX}gcc-intermediate \
		virtual/${TARGET_PREFIX}gcc \
		virtual/${TARGET_PREFIX}g++ \
	"

	EXTERNAL_CROSS_BINARIES = "${gcc_binaries}"

/usr1/openeuler/src/yocto-meta-openeuler/meta-openeuler/recipes-external/gcc/gcc-external.inc
	PV = "${GCC_VERSION}"
	gcc_binaries = "gcc gcc-${@'${PV}'.replace('${EXTERNAL_PV_SUFFIX}', '')} \
					gcc-ar gcc-nm gcc-ranlib cc gcov gcov-tool c++ g++ cpp gfortran"
```



