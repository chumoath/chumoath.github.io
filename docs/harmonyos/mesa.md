# mesa

### 1、OH

```shell
# LLVMpipe（常见于无GPU或驱动问题的Linux系统）
OpenGL renderer string: llvmpipe (LLVM 15.0.6, 256 bits)

# Softpipe（更基本的软件渲染器）
OpenGL renderer string: Softpipe

# SVGA（虚拟机中的软件渲染）
OpenGL renderer string: SVGA3D



Intel Corporation UHD Graphics 630

硬件渲染：

OpenGL vendor string: NVIDIA Corporation
OpenGL renderer string: NVIDIA GeForce RTX 3060/PCIe/SSE2

OpenGL vendor string: Intel
OpenGL renderer string: Mesa Intel(R) UHD Graphics (TGL GT1)

OpenGL vendor string: AMD
OpenGL renderer string: AMD Radeon RX 6700 XT


apt install mesa-utils

apt install mesa-utils mesa-va-drivers libva-drm2 vainfo

glxinfo -display :10 -B


如果输出中包含No，则表示硬件加速可用；如果包含Yes，则表示硬件加速不可用，可能启用了llvmpipe。

/usr/share/X11/xorg.conf.d/20-intel.conf

Section "Device"
   Identifier  "Intel Graphics"
   Driver      "modesetting"
   Option      "AccelMethod" "none"
   Option      "TearFree"    "true"
EndSection



/etc/apt/source.list

# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ noble main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ noble main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ noble-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ noble-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ noble-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ noble-backports main restricted universe multiverse


ubuntu-24.04: 
Xorg :0 -retro &
glxinfo -display :0 -B


systemctl restart gdm3


su ubuntu
glxinfo -display :0 -B



glxinfo -display :0 -B
name of display: :0
display: :0  screen: 0
direct rendering: Yes
Extended renderer info (GLX_MESA_query_renderer):
    Vendor: Intel (0x8086)
    Device: Mesa Intel(R) UHD Graphics 630 (CFL GT2) (0x3e9b)
    Version: 25.0.7
    Accelerated: yes
    Video memory: 3900MB
    Unified memory: yes
    Preferred profile: core (0x1)
    Max core profile version: 4.6
    Max compat profile version: 4.6
    Max GLES1 profile version: 1.1
    Max GLES[23] profile version: 3.2
OpenGL vendor string: Intel
OpenGL renderer string: Mesa Intel(R) UHD Graphics 630 (CFL GT2)
OpenGL core profile version string: 4.6 (Core Profile) Mesa 25.0.7-0ubuntu0.24.04.1
OpenGL core profile shading language version string: 4.60
OpenGL core profile context flags: (none)
OpenGL core profile profile mask: core profile

OpenGL version string: 4.6 (Compatibility Profile) Mesa 25.0.7-0ubuntu0.24.04.1
OpenGL shading language version string: 4.60
OpenGL context flags: (none)
OpenGL profile mask: compatibility profile

OpenGL ES profile version string: OpenGL ES 3.2 Mesa 25.0.7-0ubuntu0.24.04.1
OpenGL ES profile shading language version string: OpenGL ES GLSL ES 3.20



apt install glmark2-x11

glmark2 -> CPU占用率 6%


vblank_mode=0 glxgears


查看GPU利用率:
apt install intel-gpu-tools

intel_gpu_top # 用不了



   550.491] (II) modeset(0): [DRI2]   DRI driver: iris
[   550.491] (II) modeset(0): [DRI2]   VDPAU driver: va_gl


/dev/dri/card1  -  英伟达显卡 150Ti    nouveau
/dev/dri/card2  -  intel UHD 630显卡  iris



# 禁止英伟达显卡驱动加载
mv /lib/modules/6.14.0-27-generic/kernel/drivers/gpu/drm/nouveau/nouveau.ko.zst /lib/modules/6.14.0-27-generic/kernel/drivers/gpu/drm/nouveau/nouveau.ko.zst.bak

grub选项 -> 按 e，并添加：
modprobe.blacklist=nouveau

# 只有一个card1
ls -l /dev/dri/


update-grub
update-initramfs -u

Ubuntu-24.04

网络：
apt install -y openssh-server

/etc/ssh/sshd_config
PermitRootLogin yes

apt install mesa-utils

su ubuntu

glxinfo -display :0 -B
    Vendor: Intel (0x8086)
    Device: Mesa Intel(R) UHD Graphics 630 (CFL GT2) (0x3e9b)
    Version: 25.0.7
    Accelerated: yes
	OpenGL vendor string: Intel
	OpenGL renderer string: Mesa Intel(R) UHD Graphics 630 (CFL GT2)

vblank_mode=0 glxgears -display :0
	30579 frames in 5.0 seconds = 6115.745 FPS
	31092 frames in 5.0 seconds = 6218.308 FPS
	30922 frames in 5.0 seconds = 6184.319 FPS
	30903 frames in 5.0 seconds = 6180.597 FPS
	
CPU占用率: 12%

Ubuntu-24.04会自动下载debug信息：
su ubuntu

systemctl stop gdm3

export http_proxy=http://192.168.0.111:7897
export https_proxy=http://192.168.0.111:7897

gdb glxinfo
set args -display :0 -B
b dlopen
run

libGLX_mesa.so.0

gdb /usr/lib/xorg/Xorg
set args :0 -retro
b dlopen
run

InitOutput -> xf86LoadSubModule -> LoadSubModule: /usr/lib/xorg/modules/libglamoregl.so

InitOutput -> /usr/lib/xorg/modules/drivers/modesetting_drv.so -> /usr/lib/xorg/modules/libglamoregl.so: glamor_egl_init -> gbm_create_device -> /usr/lib/x86_64-linux-gnu/gbm/tls/i915_gbm.so
                                                                                                                                              -> /usr/lib/x86_64-linux-gnu/gbm/i915_gbm.so
																																			  -> /usr/lib/x86_64-linux-gnu/gbm/tls/dri_gbm.so
																																			  -> /usr/lib/x86_64-linux-gnu/gbm/dri_gbm.so
																					-> epoxy_has_egl_extension -> epoxy_egl_dlsym(eglQueryString) -> epoxy_load_egl -> get_dlopen_handle -> libEGL.so.1
																																															    (constructor)-> __eglInit
																												-> eglQueryString -> LoadVendors -> LoadVendor -> libEGL_mesa.so.0																																											-> epoxy_internal_gl_version -> epoxy_gl_dlsym(glGetString) -> epoxy_load_gl -> libGL.so.1
										
InitExtensions -> 	/usr/lib/xorg/modules/extensions/libglx.so -> /usr/lib/x86_64-linux-gnu/dri/iris_dri.so	-> 	gbm_create_device: 	/usr/lib/x86_64-linux-gnu/gbm/tls/i915_gbm.so
																																	/usr/lib/x86_64-linux-gnu/gbm/i915_gbm.so
																																	/usr/lib/x86_64-linux-gnu/gbm/dri_gbm.so
																																
/usr/lib/xorg/modules/extensions/libglx.so
/usr/lib/xorg/modules/drivers/modesetting_drv.so
/usr/lib/xorg/modules/drivers/fbdev_drv.so
/usr/lib/xorg/modules/drivers/vesa_drv.so
/usr/lib/xorg/modules/libfbdevhw.so
/usr/lib/xorg/modules/libglamoregl.so
/usr/lib/x86_64-linux-gnu/gbm/tls/i915_gbm.so
/usr/lib/x86_64-linux-gnu/gbm/i915_gbm.so
/usr/lib/x86_64-linux-gnu/gbm/tls/dri_gbm.so
/usr/lib/x86_64-linux-gnu/gbm/dri_gbm.so
libEGL.so.1
libEGL_mesa.so.0
libGL.so.1
/usr/lib/x86_64-linux-gnu/dri/iris_dri.so
libEGL.so.1
/usr/lib/x86_64-linux-gnu/gbm/tls/i915_gbm.so
/usr/lib/x86_64-linux-gnu/gbm/i915_gbm.so
/usr/lib/x86_64-linux-gnu/gbm/tls/dri_gbm.so
/usr/lib/x86_64-linux-gnu/gbm/dri_gbm.so

真正入口点： xserver-xorg-core:/usr/lib/xorg/modules/libglamoregl.so

libegl1: libEGL.so.1
libegl-mesa0: libEGL_mesa.so.0
libgl1: libGL.so.1
libgl1-mesa-dri: /usr/lib/x86_64-linux-gnu/dri/iris_dri.so
libgbm1: /usr/lib/x86_64-linux-gnu/gbm/dri_gbm.so

apt source libgl1-mesa-dri

重命名libGL.so.1 libEGL.so.1 直接崩溃
重命名 /usr/lib/x86_64-linux-gnu/dri/iris_dri.so 不影响
/usr/lib/x86_64-linux-gnu/dri/iris_dri.so链接到/usr/lib/x86_64-linux-gnu/dri/libdril_dri.so
重命名/usr/lib/x86_64-linux-gnu/dri/libdril_dri.so，可以显示，但是glxinfo glxgears 执行失败

# llvmpipe渲染
mv /usr/lib/xorg/modules/libglamoregl.so /usr/lib/xorg/modules/libglamoregl.so.bak
systemctl restart gdm3

su ubuntu
ubuntu@ubuntu:/root$ glxinfo -display :0 -B
    Vendor: Mesa (0xffffffff)
    Device: llvmpipe (LLVM 19.1.1, 256 bits) (0xffffffff)
	Accelerated: no
	OpenGL vendor string: Mesa
	OpenGL renderer string: llvmpipe (LLVM 19.1.1, 256 bits)

ubuntu@ubuntu:/root$ vblank_mode=0 glxgears -display :0
	12754 frames in 5.0 seconds = 2550.652 FPS
	13924 frames in 5.0 seconds = 2784.608 FPS
	13684 frames in 5.0 seconds = 2736.719 FPS

# 显卡渲染
mv /usr/lib/xorg/modules/libglamoregl.so.bak /usr/lib/xorg/modules/libglamoregl.so
systemctl restart gdm3

su ubuntu
glxinfo -display :0 -B
	OpenGL vendor string: Intel
	OpenGL renderer string: Mesa Intel(R) UHD Graphics 630 (CFL GT2)

ubuntu@ubuntu:/root$ vblank_mode=0 glxgears -display :0
	30585 frames in 5.0 seconds = 6116.914 FPS
	30979 frames in 5.0 seconds = 6195.707 FPS
	
strings /usr/lib/x86_64-linux-gnu/dri/iris_dri.so | grep iris
	__driDriverGetExtensions_iris

mesa-25.2.8/src/gallium/targets/dril/dril_target.c
	#define DEFINE_LOADER_DRM_ENTRYPOINT(drivername)                          \
	const __DRIextension **__driDriverGetExtensions_##drivername(void);       \
	PUBLIC const __DRIextension **__driDriverGetExtensions_##drivername(void) \
	{                                                                         \
	   return __driDriverExtensions;                                   \
	}
	
	DEFINE_LOADER_DRM_ENTRYPOINT(i915)
	DEFINE_LOADER_DRM_ENTRYPOINT(iris)
	DEFINE_LOADER_DRM_ENTRYPOINT(virtio_gpu)

dri_search_path
DEFAULT_DRIVER_DIR

oh5.1 - mesa22
mesa3d/src/gallium/targets/dri/target.c
	DEFINE_LOADER_DRM_ENTRYPOINT(virtio_gpu)
	
mesa3d/meson.build
	with_gallium_softpipe = gallium_drivers.contains('swrast')
	with_gallium_vc4 = gallium_drivers.contains('vc4')
	with_gallium_v3d = gallium_drivers.contains('v3d')
	with_gallium_panfrost = gallium_drivers.contains('panfrost')
	with_gallium_etnaviv = gallium_drivers.contains('etnaviv')
	with_gallium_tegra = gallium_drivers.contains('tegra')
	with_gallium_crocus = gallium_drivers.contains('crocus')
	with_gallium_iris = gallium_drivers.contains('iris')
	with_gallium_i915 = gallium_drivers.contains('i915')
	with_gallium_svga = gallium_drivers.contains('svga')
	with_gallium_virgl = gallium_drivers.contains('virgl')

	with_tools = get_option('tools')  设置为空即可

mesa3d/build-ohos/src/gallium/targets/dri
	libgallium_dri.so
	panfrost_dri.so
	rockchip_dri.so
	是一样的
	
mesa3d/src/gallium/meson.build
	with_gallium_softpipe = gallium_drivers.contains('swrast')
	
	if with_gallium_softpipe
	  subdir('drivers/softpipe')
	  if draw_with_llvm
		subdir('drivers/llvmpipe')
	  endif
	else
	  driver_swrast = declare_dependency()
	endif

	with_gallium_virgl = gallium_drivers.contains('virgl')
	if with_gallium_virgl
	  subdir('winsys/virgl/common')
	  subdir('winsys/virgl/drm')
	  subdir('winsys/virgl/vtest')
	  subdir('drivers/virgl')
	else
	  driver_virgl = declare_dependency()
	endif
	
ohos_load_driver

src/egl/drivers/dri2/platform_ohos.c
	dri2_dpy->driver_name = strdup("kms_swrast");

	static EGLBoolean
	
ohos_load_driver(_EGLDisplay *disp, bool swrast)
	{
		struct dri2_egl_display *dri2_dpy = dri2_egl_display(disp);

		dri2_dpy->driver_name = loader_get_driver_for_fd(dri2_dpy->fd);
		if (dri2_dpy->driver_name == NULL) {
			return false;
		}

	#ifdef HAVE_DRM_GRALLOC
		/* Handle control nodes using __DRI_DRI2_LOADER extension and GEM names
		 * for backwards compatibility with drm_gralloc. (Do not use on new
		 * systems.) */
		dri2_dpy->loader_extensions = ohos_dri2_loader_extensions;
		if (!dri2_load_driver(disp)) {
			goto error;
		}
	#else
		if (swrast) {
			/* Use kms swrast only with vgem / virtio_gpu.
			 * virtio-gpu fallbacks to software rendering when 3D features
			 * are unavailable since 6c5ab.
			 */
			if (strcmp(dri2_dpy->driver_name, "vgem") == 0 ||
				strcmp(dri2_dpy->driver_name, "virtio_gpu") == 0) {
				free(dri2_dpy->driver_name);
				dri2_dpy->driver_name = strdup("kms_swrast");
			} else {
				goto error;
			}
		}

		dri2_dpy->loader_extensions = ohos_image_loader_extensions;
		if (!dri2_load_driver_dri3(disp)) {
			goto error;
		}
	#endif

		return true;

	error:
		free(dri2_dpy->driver_name);
		dri2_dpy->driver_name = NULL;
		return false;
	}

mesa编译：
./ohos/build_mesa3d.py /home/openharmony/openharmony/out/x86_general
	文件一摸一样
	virtio_gpu_dri.so
	kms_swrast_dri.so
	swrast_dri.so
	
	nm virtio_gpu_dri.so | grep __driDriverGetExtensions_
	
	libgallium_dri.so
	
#0  0x00007ffff7fc56fd in strlen () from /lib/ld-musl-x86_64.so.1
#1  0x00007ffff45b6708 in OHOS::Rosen::RenderContext::SetUpGpuContext(std::__h::shared_ptr<OHOS::Rosen::Drawing::GPUContext>) ()
   from /system/lib64/platformsdk/librender_service_base.z.so
#2  0x00007ffff0863441 in OHOS::Rosen::RSBaseRenderEngine::Init(bool) ()
   from /system/lib64/librender_service.z.so
#3  0x00007ffff07a94bf in OHOS::Rosen::RSMainThread::Init() ()
   from /system/lib64/librender_service.z.so
#4  0x00007ffff07f27b8 in OHOS::Rosen::RSRenderService::Init() ()
   from /system/lib64/librender_service.z.so
#5  0x000055555555dc77 in main[cfi] ()
#6  0x00007ffff7f3f638 in libc_start_main_stage2 ()
   from /lib/ld-musl-x86_64.so.1
#7  0x000055555555a068 in _start_c ()
#8  0x000055555555a016 in _start ()

# 必须保证composer_host进程启动，render_service 服务退出
service_control start composer_host
service_control stop render_service

mount /dev/block/sda /mnt
export LD_LIBRARY_PATH=/mnt/usr/lib/x86_64-linux-gnu/
/mnt/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2 /mnt/usr/bin/gdb render_service

loader_open_driver

loader_open_driver_lib
先从 /system/lib64/tls/ 搜索，再从/system/lib64/搜索

659    for (const char *p = search_paths; p < end; p = next + 1) {
660       int len;
661       next = strchr(p, ':');
662       if (next == NULL)
663          next = end;
664
665       len = next - p;
666       snprintf(path, sizeof(path), "%.*s/tls/%s%s.so", len,
667                p, driver_name, lib_suffix);
668       driver = dlopen(path, RTLD_NOW | RTLD_GLOBAL);
669       if (driver == NULL) {
670          snprintf(path, sizeof(path), "%.*s/%s%s.so", len,
671                   p, driver_name, lib_suffix);
672          driver = dlopen(path, RTLD_NOW | RTLD_GLOBAL);
673          if (driver == NULL) {
674             dl_error = dlerror();
675             log_(_LOADER_DEBUG, "MESA-LOADER: failed to open %s: %s\n",
676                  path, dl_error);
677          }
678       }
679       /* not need continue to loop all paths once the driver is found */
680       if (driver != NULL)
681          break;
682    }

715    void *driver = loader_open_driver_lib(driver_name, "_dri", search_path_vars,
716                                          DEFAULT_DRIVER_DIR, true);
717

733    if (!extensions)
734       extensions = dlsym(driver, __DRI_DRIVER_EXTENSIONS);
735    if (extensions == NULL) {
736       log_(_LOADER_WARNING,
737            "MESA-LOADER: driver exports no extensions (%s)\n", dlerror());
738       dlclose(driver);
739       driver = NULL;
740    }

/home/openharmony/openharmony/third_party/mesa3d/build-ohos/src/gallium/targets/dri/libgallium_dri.so

libEGL.so.1.0.0 libgbm.so.1.0.0 libglapi.so.0.0.0 libGLESv1_CM.so.1.1.0 libGLESv2.so.2.0.0

cd /data/

# 必须使用 -s指定字符串显示长度，使用 -v 和 -e writev 都不行
/mnt/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2 /mnt/usr/bin/strace  -ff -o task -v  -s 65536 render_service

grep -rn dlopen task*

b LoadEgl

# 因为把环境变量设置到了ubuntu，使用了ubuntu的libdrm.so
task.5114:8808:writev(3, [{iov_base="^\0\30&B*\217i\360\v\35,\265\4\0\0\0\0\0\0\372\23\0\0\0\24\0\r", iov_len=28}, {iov_base="OHOS::RS\0", iov_len=9}, {iov_base="RsFrameReport:[Init] dlopen libframe_ui_intf.so success!\0", iov_len=57}], 3) = 94
task.5114:9066:writev(3, [{iov_base="\235\0\30;B*\217i0c>-\265\4\0\0\0\0\0\0\372\23\0\0\0\24\0\r", iov_len=28}, {iov_base="OpenGLWrapper\0", iov_len=14}, {iov_base="dlopen failed. error: Error relocating /mnt/usr/lib/x86_64-linux-gnu//libdrm.so: __vfprintf_chk: symbol not found.\0", iov_len=115}], 3) = 157
task.5114:9208:writev(3, [{iov_base="\235\0\30;B*\217i\246\314\244-\265\4\0\0\0\0\0\0\372\23\0\0\0\24\0\r", iov_len=28}, {iov_base="OpenGLWrapper\0", iov_len=14}, {iov_base="dlopen failed. error: Error relocating /mnt/usr/lib/x86_64-linux-gnu//libdrm.so: __vfprintf_chk: symbol not found.\0", iov_len=115}], 3) = 157
task.5114:9335:writev(3, [{iov_base="\235\0\30;B*\217i\2205\2.\265\4\0\0\0\0\0\0\372\23\0\0\0\24\0\r", iov_len=28}, {iov_base="OpenGLWrapper\0", iov_len=14}, {iov_base="dlopen failed. error: Error relocating /mnt/usr/lib/x86_64-linux-gnu//libdrm.so: __vfprintf_chk: symbol not found.\0", iov_len=115}], 3) = 157

task.5280:10892:open("/system/lib64/tls/virtio_gpu_dri.so", O_RDONLY|O_LARGEFILE|O_CLOEXEC) = -1 ENOENT (No such file or directory)
task.5280:10895:writev(18, [{iov_base="\232\0\30+\233*\217i\353\271\302\36\16\5\0\0\240\24\0\0\240\24\0\0\0?\0\r", iov_len=28}, {iov_base="MUSL-LDSO\0", iov_len=10}, {iov_base="Open absolute_path library: check ns accessible failed, pathname /system/lib64/tls/virtio_gpu_dri.so namespace ndk.\0", iov_len=116}], 3) = 154
task.5280:10896:writev(18, [{iov_base="\207\0\30+\233*\217i7(\304\36\16\5\0\0\240\24\0\0\240\24\0\0\0?\0\r", iov_len=28}, {iov_base="MUSL-LDSO\0", iov_len=10}, {iov_base="Error loading header /system/lib64/tls/virtio_gpu_dri.so, namespace ndk has no inherits, errno=2\0", iov_len=97}], 3) = 135
task.5280:10897:writev(18, [{iov_base="\211\0\30+\233*\217id\351\304\36\16\5\0\0\240\24\0\0\240\24\0\0\0?\0\r", iov_len=28}, {iov_base="MUSL-LDSO\0", iov_len=10}, {iov_base="Error loading header: can't find library /system/lib64/tls/virtio_gpu_dri.so in namespace: default\0", iov_len=99}], 3) = 137
task.5280:10898:writev(18, [{iov_base="u\0\30+\233*\217i\r\267\305\36\16\5\0\0\240\24\0\0\240\24\0\0\0?\0\r", iov_len=28}, {iov_base="MUSL-LDSO\0", iov_len=10}, {iov_base="dlopen_impl load library header failed for /system/lib64/tls/virtio_gpu_dri.so\0", iov_len=79}], 3) = 117
task.5280:11025:writev(18, [{iov_base="\330\2\30;\233*\217i\313\16&\37\16\5\0\0\240\24\0\0\240\24\0\0\0?\0\r", iov_len=28}, {iov_base="MUSL-SIGCHAIN\0", iov_len=14}, {iov_base="signal_chain_handler call 2 rd sigchain action for signal: 11 sca_sigaction=7f5077f78590 noreturn=0 FREEZE_signo_11 thread_list_lock_status:-1 tl_lock_count=0 tl_lock_waiters=0 tl_lock_tid_fail=-1 tl_lock_count_tid=-1 tl_lock_count_fail=-10000 tl_lock_count_tid_sub=-1 thread_list_lock_after_lock=5280 thread_list_lock_pre_unlock=5280 thread_list_lock_pthread_exit=-1 thread_list_lock_tid_overlimit=-1 tl_lock_unlock_count=0 __pthread_gettid_np_tl_lock=0 __pthread_exit_tl_lock=0 __pthread_create_tl_lock=0 __pthread_key_delete_tl_lock=0 __synccall_tl_lock=0 __membarrier_tl_lock=0 install_new_tls_tl_lock=0 set_syscall_hooks_tl_lock=0 set_syscall_hooks_linux_tl_lock=0 fork_tl_lock=0 \0", iov_len=686}], 3) = 728

通过使用beyond-compare对比得知，oh6.0的x86预构建的mesa，使用的是mesa22版本，不是mesa25。 比如 ohos_logger 这个符号，是mesa22的，mesa25没有。

通过gdb调试，loader_open_driver 行号信息也是和oh5.0的mesa一模一样。

# 在 build-ohos 目录进行构建

meson_cmd = [
    'meson',
    'setup',
    mesa3d_dir,
    'build-ohos',
    '-Dplatforms=ohos',
    '-Degl-native-platform=ohos',
    '-Ddri-drivers=',
    '-Dgallium-drivers=virgl,swrast',
    '-Dvulkan-drivers=',
    '-Dgbm=enabled',
    '-Degl=enabled',
    '-Dcpp_rtti=false',
    '-Dglx=disabled',
    '-Dtools=',
    '-Ddri-search-path=/system/lib64',
    '--cross-file=cross_file',
    F'--prefix={mesa3d_dir}/build-ohos/install',
]

OH6.0的 render_service正常日志

libEGL warning: <message truncated>
libEGL warning: loader_open_device /dev/dri/renderD128
libEGL warning: egl: failed to create dri2 screen
libEGL warning: DRI2: failed to create screen
libEGL warning: <message truncated>
libEGL warning: <message truncated>
libEGL warning: loader_open_device /dev/dri/card0

#0  0x00007fff69fe8800 in ohos_load_driver (disp=<optimized out>,
    swrast=<optimized out>) at ../src/egl/drivers/dri2/platform_ohos.c:1035
#1  ohos_probe_device (disp=<optimized out>, swrast=<optimized out>)
    at ../src/egl/drivers/dri2/platform_ohos.c:1112
#2  ohos_open_device (disp=<optimized out>, swrast=<optimized out>)
    at ../src/egl/drivers/dri2/platform_ohos.c:1216
#3  dri2_initialize_ohos (disp=<optimized out>, disp@entry=0x7ffff711c360)
    at ../src/egl/drivers/dri2/platform_ohos.c:1254
#4  0x00007fff69fe5693 in dri2_initialize (disp=disp@entry=0x7ffff711c360)
    at ../src/egl/drivers/dri2/egl_dri2.c:1198
#5  0x00007fff69fd889e in eglInitialize (dpy=0x7ffff711c360,
    major=0x7ffff3f217f4 <OHOS::gWrapperHook+16132>,
    minor=0x7ffff3f217f8 <OHOS::gWrapperHook+16136>)
    at ../src/egl/main/eglapi.c:644
#6  0x00007ffff3efa4e5 in OHOS::EglWrapperDisplay::Init(int*, int*) ()
   from /system/lib64/platformsdk/libEGL.so
#7  0x00007ffff3eee327 in eglInitialize ()
   from /system/lib64/platformsdk/libEGL.so
#8  0x00007ffff2e27743 in OHOS::Rosen::RenderContext::InitializeEglContext()
    () from /system/lib64/platformsdk/librender_service_base.z.so
#9  0x00007ffff66f9a36 in OHOS::Rosen::RSBaseRenderEngine::Init() ()
   from /system/lib64/librender_service.z.so
#10 0x00007ffff663a1fb in OHOS::Rosen::RSMainThread::Init() ()

./src/loader/pci_id_driver_map.h

./src/egl/drivers/dri2/platform_ohos.c
ohos_load_driver

vim ./src/loader/loader.c +92

pci_ids/virtio_gpu_pci_ids.h

virtio_gpu -> 1af4:1050

fd: 12                %d
vendor_id: 0x1af4     %04x
chip_id: 0x1050       %04x
driver: virtio_gpu    %s

段错误：
(gdb) p log_string
$14 = "pci id for fd -11872: 1050:f1d0de10, driver ", '\000' <repeats 660 times>...

(gdb) p fmt
$12 = 0x7fff66dbd035 "pci id for fd %d: %04x:%04x, driver %s\n"
(gdb) p fd
$6 = 17
(gdb) p /x vendor_id
$8 = 0x1af4
(gdb) p /x chip_id
$9 = 0x1050
(gdb) p /x driver
$10 = 0x7ffff1d0de10
(gdb) p (char*)driver
$11 = 0x7ffff1d0de10 "virtio_gpu"

./obj/third_party/musl/usr/lib/x86_64-linux-ohos/libc.so

./lib.unstripped/obj/third_party/musl/usr/lib/x86_64-linux-ohos/libc.so

-fno-emulated-tls

stdarg.h:12:#define va_start(v,l)   __builtin_va_start(v,l)

0  0x00007ffff7fc48e6 in memchr () from /lib/ld-musl-x86_64.so.1
#1  0x00007ffff7fba488 in printf_core () from /lib/ld-musl-x86_64.so.1
#2  0x00007ffff7fb9a54 in vfprintf () from /lib/ld-musl-x86_64.so.1
#3  0x00007ffff7fb86f7 in snprintf () from /lib/ld-musl-x86_64.so.1
#4  0x00007fff6633e891 in ohos_logger (level=3,
    fmt=0x7fff6597d035 "pci id for fd %d: %04x:%04x, driver %s\n")
    at ../src/loader/loader.c:92
#5  0x00007fff6633e770 in loader_get_pci_driver (fd=17)
    at ../src/loader/loader.c:566
#6  loader_get_driver_for_fd (fd=fd@entry=17) at ../src/loader/loader.c:594
#7  0x00007fff6633e0e3 in pipe_loader_drm_probe_fd_nodup (
    dev=dev@entry=0x7ffff713f418, fd=17)
    at ../src/gallium/auxiliary/pipe-loader/pipe_loader_drm.c:141
#8  0x00007fff6633e052 in pipe_loader_drm_probe_fd (
    dev=dev@entry=0x7ffff713f418, fd=<optimized out>)
    at ../src/gallium/auxiliary/pipe-loader/pipe_loader_drm.c:193
#9  0x00007fff65c5751e in dri2_init_screen (sPriv=0x7ffff6e69850)
    at ../src/gallium/frontends/dri/dri2.c:2468
#10 0x00007fff65c5462d in driCreateNewScreen2 (scrn=<optimized out>,
    fd=<optimized out>, extensions=<optimized out>,
    driver_extensions=<optimized out>, driver_configs=0x7ffff5e895d0,
    data=<optimized out>) at ../src/gallium/frontends/dri/dri_util.c:142
#11 0x00007fff680e4074 in dri2_create_screen (disp=disp@entry=0x7ffff71e2c80)
--Type <RET> for more, q to quit, c to continue without paging--
    at ../src/egl/drivers/dri2/egl_dri2.c:1071
#12 0x00007fff680e8435 in ohos_probe_device (disp=0x7ffff71e2c80,
    swrast=<optimized out>) at ../src/egl/drivers/dri2/platform_ohos.c:1113
#13 ohos_open_device (disp=0x7ffff71e2c80, swrast=<optimized out>)
    at ../src/egl/drivers/dri2/platform_ohos.c:1213
#14 dri2_initialize_ohos (disp=disp@entry=0x7ffff71e2c80)
    at ../src/egl/drivers/dri2/platform_ohos.c:1251
#15 0x00007fff680e5243 in dri2_initialize (disp=disp@entry=0x7ffff71e2c80)
    at ../src/egl/drivers/dri2/egl_dri2.c:1195
#16 0x00007fff680d72f3 in eglInitialize (dpy=0x7ffff71e2c80,
    major=0x7ffff4a6013c <OHOS::gWrapperHook+16132>,
    minor=0x7ffff4a60140 <OHOS::gWrapperHook+16136>)
    at ../src/egl/main/eglapi.c:639
#17 0x00007ffff4a395cf in OHOS::EglWrapperDisplay::Init(int*, int*) ()
   from /system/lib64/platformsdk/libEGL.so
#18 0x00007ffff4a2daf7 in eglInitialize ()
   from /system/lib64/platformsdk/libEGL.so
#19 0x00007ffff69b60a3 in OHOS::Rosen::RenderContext::InitializeEglContext()
    () from /system/lib64/platformsdk/librender_service_base.z.so
#20 0x00007fffef0233e3 in OHOS::Rosen::RSBaseRenderEngine::Init(bool) ()
   from /system/lib64/librender_service.z.so
#21 0x00007fffeef694bf in OHOS::Rosen::RSMainThread::Init() ()
   from /system/lib64/librender_service.z.so
--Type <RET> for more, q to quit, c to continue without paging--[  113.659689] [pid=1][LoopEvent][WARNING][le_epoll.c:120]timeout:134040
[  113.664917] sysrq: Show Blocked State
[  114.667373] sysrq: Show backtrace of all active CPUs
[  185.643557] [pid=1][LoopEvent][WARNING][le_epoll.c:120]timeout:62056
[  185.652282] sysrq: Show Blocked State
[  186.668692] sysrq: Show backtrace of all active CPUs

#22 0x00007fffeefb27b8 in OHOS::Rosen::RSRenderService::Init() ()
   from /system/lib64/librender_service.z.so
#23 0x000055555555dc77 in main[cfi] ()
#24 0x00007ffff7f3f638 in libc_start_main_stage2 ()
   from /lib/ld-musl-x86_64.so.1
#25 0x000055555555a068 in _start_c ()
#26 0x000055555555a016 in _start ()
```



### 2、补丁

```shell
root@chumoath:~/opc510/openharmony/third_party/mesa3d# git diff .
diff --git a/ohos/build_mesa3d.py b/ohos/build_mesa3d.py
index 16b466becd3..b070d407b15 100755
--- a/ohos/build_mesa3d.py
+++ b/ohos/build_mesa3d.py
@@ -38,7 +38,7 @@ mesa3d_install_dir = os.path.join(out_dir, 'packages', 'phone', 'mesa3d')
 product = os.path.basename(out_dir)
 project_dir = os.path.dirname(os.path.dirname(out_dir))
 project_dir = os.path.abspath(project_dir)
-mesa3d_dir = os.path.dirname(os.path.dirname(__file__))
+mesa3d_dir = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
 mesa3d_dir = os.path.abspath(mesa3d_dir)
 pkgconf_dir = os.path.join(mesa3d_dir, './pkgconfig')
 os.environ['PKG_CONFIG_PATH'] = pkgconf_dir
@@ -60,14 +60,14 @@ meson_cmd = [
     '-Dplatforms=ohos',
     '-Degl-native-platform=ohos',
     '-Ddri-drivers=',
-    '-Dgallium-drivers=panfrost',
+    '-Dgallium-drivers=virgl,swrast',
     '-Dvulkan-drivers=',
     '-Dgbm=enabled',
     '-Degl=enabled',
     '-Dcpp_rtti=false',
     '-Dglx=disabled',
-    '-Dtools=panfrost',
-    '-Ddri-search-path=/system/lib',
+    '-Dtools=',
+    '-Ddri-search-path=/system/lib64',
     '--cross-file=cross_file',
     F'--prefix={mesa3d_dir}/build-ohos/install',
 ]
diff --git a/ohos/meson_cross_process.py b/ohos/meson_cross_process.py
index 83e23488211..0a0e2bf45d3 100644
--- a/ohos/meson_cross_process.py
+++ b/ohos/meson_cross_process.py
@@ -33,39 +33,31 @@ corss_file_content='''
 needs_exe_wrapper = true

 c_args = [
-    '-march=armv7-a',
-    '-mfloat-abi=softfp',
-    '-mtune=generic-armv7-a',
-    '-mfpu=neon',
-    '-mthumb',
-    '--target=arm-linux-ohosmusl',
+    '--target=x86_64-linux-ohosmusl',^M
     '--sysroot=sysroot_stub',
     '-fPIC']

 cpp_args = [
-    '-march=armv7-a',
-    '--target=arm-linux-ohosmusl',
+    '--target=x86_64-linux-ohosmusl',^M
     '--sysroot=sysroot_stub',
     '-fPIC']

 c_link_args = [
-    '-march=armv7-a',
-    '--target=arm-linux-ohosmusl',
+    '--target=x86_64-linux-ohosmusl',^M
     '-fPIC',
     '--sysroot=sysroot_stub',
-    '-Lsysroot_stub/usr/lib/arm-linux-ohos',
-    '-Lproject_stub/prebuilts/clang/ohos/linux-x86_64/llvm/lib/clang/10.0.1/lib/arm-linux-ohos',
-    '-Lproject_stub/prebuilts/clang/ohos/linux-x87_64/llvm/lib/arm-linux-ohos/c++',
+    '-Lsysroot_stub/usr/lib/x86_64-linux-ohos',^M
+    '-Lproject_stub/prebuilts/clang/ohos/linux-x86_64/llvm/lib/clang/15.0.4/lib/x86_64-linux-ohos',^M
+    '-Lproject_stub/prebuilts/clang/ohos/linux-x86_64/llvm/lib/x86_64-linux-ohos',^M
     '--rtlib=compiler-rt',
     ]

 cpp_link_args = [
-    '-march=armv7-a',
-    '--target=arm-linux-ohosmusl',
+    '--target=x86_64-linux-ohosmusl',^M
     '--sysroot=sysroot_stub',
-    '-Lsysroot_stub/usr/lib/arm-linux-ohos',
-    '-Lproject_stub/prebuilts/clang/ohos/linux-x86_64/llvm/lib/clang/10.0.1/lib/arm-linux-ohos',
-    '-Lproject_stub/prebuilts/clang/ohos/linux-x87_64/llvm/lib/arm-linux-ohos/c++',
+    '-Lsysroot_stub/usr/lib/x86_64-linux-ohos',^M
+    '-Lproject_stub/prebuilts/clang/ohos/linux-x86_64/llvm/lib/clang/15.0.4/lib/x86_64-linux-ohos',^M
+    '-Lproject_stub/prebuilts/clang/ohos/linux-x86_64/llvm/lib/x86_64-linux-ohos',^M
     '-fPIC',
     '-Wl,--exclude-libs=libunwind_llvm.a',
     '-Wl,--exclude-libs=libc++_static.a',
@@ -83,11 +75,6 @@ cpp_ld = 'lld'
 strip = 'project_stub/prebuilts/clang/ohos/linux-x86_64/llvm/bin/llvm-strip'
 pkgconfig = '/usr/bin/pkg-config'

-[host_machine]
-system = 'linux'
-cpu_family = 'arm'
-cpu = 'armv7'
-endian = 'little'
 '''

 def generate_cross_file(project_stub_in, sysroot_stub_in):
diff --git a/ohos/pkgconfig_template/expat.pc b/ohos/pkgconfig_template/expat.pc
index afb95404ba0..c9fa74cf885 100644
--- a/ohos/pkgconfig_template/expat.pc
+++ b/ohos/pkgconfig_template/expat.pc
@@ -1,6 +1,6 @@
 ohos_project_dir=ohos_project_directory_stub
-libdir=${ohos_project_dir}/out/ohos-arm-release/obj/third_party/expat
-includedir=${ohos_project_dir}/out/ohos-arm-release/third_party_expat/expat-2.4.1/lib
+libdir=${ohos_project_dir}/out/x86_general/obj/third_party/skia/third_party/expat/^M
+includedir=${ohos_project_dir}/third_party/skia/third_party/externals/expat/expat/lib/^M

 Name: expat
 Version: 2.4.1
diff --git a/ohos/pkgconfig_template/libsurface.pc b/ohos/pkgconfig_template/libsurface.pc
index af31e40d6ba..e423cd7e1c1 100644
--- a/ohos/pkgconfig_template/libsurface.pc
+++ b/ohos/pkgconfig_template/libsurface.pc
@@ -1,6 +1,6 @@
 ohos_project_dir=ohos_project_directory_stub
-libdir=${ohos_project_dir}/out/ohos-arm-release/graphic/graphic_surface/
-includedir=${ohos_project_dir}/out/ohos-arm-release/innerkits/ohos-arm/graphic_surface/surface/include/
+libdir=${ohos_project_dir}/out/x86_general/graphic/graphic_surface/
+includedir=${ohos_project_dir}/out/x86_general/innerkits/ohos-x86_64/graphic_surface/surface/include/

 Name: libsurface
 Version: 2.4.1
diff --git a/src/loader/loader.c b/src/loader/loader.c
index e843fc7af32..2355a13ddaf 100644
--- a/src/loader/loader.c
+++ b/src/loader/loader.c
@@ -110,7 +110,7 @@ static void ohos_logger(int level, const char *fmt, ...)
     DISPLAY_LOGI();
 }

-static loader_logger *log_ = ohos_logger;
+static loader_logger *log_ = default_logger;

 int
 loader_open_device(const char *device_name)
 
 # 使用 ohos_logger snprintf 会出现 va的错乱问题，原因待分析，可以使用带调试信息的C库查看。
```

