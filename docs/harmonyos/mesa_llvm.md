# mesa_llvm

# 1、构建

```shell
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
    '-Ddri-search-path=/vendor/lib64/chipsetsdk',
    '--cross-file=cross_file',
    F'--prefix={mesa3d_dir}/build-ohos/install',

llvm-14.0.6.src.tar.xz

wget https://archive.mesa3d.org/older-versions/22.x/mesa-22.2.4.tar.xz

export PATH=/opt/llvm/bin:$PATH

# 使用llvm静态库
meson setup builddir . -Dplatforms=x11 -Degl-native-platform=auto -Ddri-drivers= -Dgallium-drivers=virgl,swrast \
	-Dvulkan-drivers= -Dgbm=enabled -Degl=enabled  -Dglx=disabled -Dtools= -Ddri-search-path=/vendor/lib64/chipsetsdk \
	-Dshared-llvm=disabled

# 使用llvm动态库，默认使用的动态库，所以只编译静态库会找不到
meson setup builddir . -Dplatforms=x11 -Degl-native-platform=auto -Ddri-drivers= -Dgallium-drivers=virgl,swrast \
	-Dvulkan-drivers= -Dgbm=enabled -Degl=enabled  -Dglx=disabled -Dtools= -Ddri-search-path=/vendor/lib64/chipsetsdk \
	-Dshared-llvm=enabled

查看报错： mesa/builddir/meson-logs/meson-log.txt

DESTDIR=/opt/mesa ninja -C builddir install

cd /opt/
tar -zcvf ./mesa.tar.gz mesa

scp mesa.tar.gz 192.168.33.2:/root/

# 运行
qemu-system-x86_64 -M q35 -cpu max -m 8G -smp 8 -kernel bzImage -enable-kvm -serial stdio \
	-device e1000e,netdev=tap0 -device virtio-gpu -netdev tap,id=tap0,ifname=tap0,script=no,downscript=no \
	-hda ubuntu22.img -append "console=ttyS0 root=/dev/sda rw nokaslr"

dd if=/dev/zero of=ubuntu22.img bs=1G count=40
mkfs.ext4 -d ubuntu-rootfs ubuntu22.img

./src/gallium/auxiliary/gallivm/lp_bld_tgsi_soa.c
	lp_build_tgsi_soa -> lp_build_tgsi_llvm

./src/gallium/auxiliary/gallivm/lp_bld_tgsi.c
	lp_build_tgsi_llvm
		const struct tgsi_full_instruction *instr = bld_base->instructions + bld_base->pc;
		lp_build_tgsi_inst_llvm(bld_base, instr)
			enum tgsi_opcode opcode = inst->Instruction.Opcode;
			const struct lp_build_tgsi_action *action = &bld_base->op_actions[opcode];
			action->emit(action, bld_base, &emit_data);
			
qemu-system-x86_64 -M q35 -cpu max -m 8G -smp 8 -kernel bzImage -enable-kvm -serial stdio \
-device e1000e,netdev=tap0 \
-netdev tap,id=tap0,ifname=tap0,script=no,downscript=no \
-hda ubuntu22.img -append "console=ttyS0 root=/dev/sda rw nokaslr"

# 扩容
truncate -s +20G ubuntu22.img
dd if=/dev/zero bs=1G count=20 >> ubuntu22.img

调整分区大小
e2fsck -f ubuntu22.img
resize2fs ubuntu22.img

apt install -y libglfw3-dev llvm-dev lightdm ukui  mesa-utils glmark2-es2-drm glmark2

# 启动lightdm

apt install ukui lightdm
systemctl stop gdm3
systemctl disable gdm3

systemctl start lightdm
systemctl enable lightdm

/etc/X11/default-display-manager
改为 /usr/sbin/lightdm

# 构建opengles相关的，也可以使用 glmark2-es2 使用编译出的 mesa

apt install -y libx11-dev  libdrm-dev
git clone https://github.com/chumoath/opengles-book-samples.git

opengles-book-samples/LinuxDRM/Makefile

INCDIR=-I./Common -I/usr/include/libdrm -I/root/mesa/usr/local/include/
LIBS=-lGLESv2 -lEGL -lm -lgbm -ldrm  -L/root/mesa/usr/local/lib/x86_64-linux-gnu/

export LD_LIBRARY_PATH=/root/mesa/usr/local/lib/x86_64-linux-gnu

mkdir -p /vendor/lib64/chipsetsdk/
cp -rfa /root/mesa/usr/local/lib/x86_64-linux-gnu/dri/* /vendor/lib64/chipsetsdk/

find . -type f -executable
./Chapter_2/Hello_Triangle/CH02_HelloTriangle
./Chapter_8/Simple_VertexShader/CH08_SimpleVertexShader

glCompileShader

# 调试（在lightdm的terminal中调试）：
apt source glmark2-es2
apt build-dep -y .
dpkg-buildpackage -rfakeroot -b

dpkg -i glmark2-es2_2021.02-0ubuntu1_amd64.deb
dpkg -i glmark2-es2-dbgsym_2021.02-0ubuntu1_amd64.ddeb

gdbserver :1234 glmark2-es2

gdb glmark2-es2
> target remote :1234
> directory /root/mesa-22.2.4/src

#0  llvmpipe_create_context (screen=0x5555556aa0d0, priv=0x0, flags=8) at ../src/gallium/drivers/llvmpipe/lp_context.c:194
#1  0x00007ffff6c3f151 in st_api_create_context (stapi=<optimized out>, smapi=0x5555556a9d50, attribs=0x7fffffffd8a0,
    error=0x7fffffffd89c, shared_stctxi=0x0) at ../src/mesa/state_tracker/st_manager.c:1065
#2  0x00007ffff6aaca7c in dri_create_context (api=api@entry=API_OPENGLES2, visual=visual@entry=0x5555556b6300,
    cPriv=cPriv@entry=0x5555558d8960, ctx_config=ctx_config@entry=0x7fffffffd9a0, error=error@entry=0x7fffffffda3c,
    sharedContextPrivate=sharedContextPrivate@entry=0x0) at ../src/gallium/frontends/dri/dri_context.c:169
#3  0x00007ffff6ab068b in driCreateContextAttribs (screen=0x5555556a7da0, api=<optimized out>, config=0x5555556b6300,
    shared=<optimized out>, num_attribs=<optimized out>, attribs=<optimized out>, error=0x7fffffffda3c, data=0x5555558e95e0)
    at ../src/gallium/frontends/dri/dri_util.c:634
#4  0x00007ffff77ad22a in dri2_create_context (disp=0x555555630300, conf=0x5555556bbab0, share_list=<optimized out>,
    attrib_list=0x5555555d0818 <GLStateEGL::gotValidContext()::context_attribs>) at ../src/egl/drivers/dri2/egl_dri2.c:1623
#5  0x00007ffff77a16ed in eglCreateContext (dpy=0x555555630300, config=0x5555556bbab0, share_list=0x0,
    attrib_list=0x5555555d0818 <GLStateEGL::gotValidContext()::context_attribs>) at ../src/egl/main/eglapi.c:825
#6  0x000055555556c5ce in GLStateEGL::gotValidContext (this=0x7fffffffdda0) at ../src/gl-state-egl.cpp:715
#7  GLStateEGL::gotValidContext (this=0x7fffffffdda0) at ../src/gl-state-egl.cpp:697
#8  GLStateEGL::valid (this=0x7fffffffdda0) at ../src/gl-state-egl.cpp:375
#9  GLStateEGL::valid (this=0x7fffffffdda0) at ../src/gl-state-egl.cpp:364
#10 0x000055555556b5e7 in CanvasGeneric::do_make_current (this=0x7fffffffdeb0) at ../src/canvas-generic.cpp:287
#11 0x000055555556bc72 in CanvasGeneric::reset (this=0x7fffffffdeb0) at ../src/canvas-generic.cpp:59
#12 CanvasGeneric::reset (this=0x7fffffffdeb0) at ../src/canvas-generic.cpp:49
#13 0x00005555555721ac in MainLoop::step (this=0x5555558f1480) at ../src/main-loop.cpp:85
#14 0x0000555555565a66 in do_benchmark (canvas=...) at ../src/main.cpp:123
#15 main (argc=<optimized out>, argv=<optimized out>) at ../src/main.cpp:221

#0  llvmpipe_create_screen (winsys=winsys@entry=0x5555556aa070) at ../src/gallium/drivers/llvmpipe/lp_screen.c:1026
#1  0x00007ffff6aa6e8a in sw_screen_create_named (config=<optimized out>, driver=<optimized out>, winsys=0x5555556aa070)
    at ../src/gallium/auxiliary/target-helpers/sw_helper.h:48
#2  sw_screen_create_vk (winsys=0x5555556aa070, config=<optimized out>, sw_vk=<optimized out>)
    at ../src/gallium/auxiliary/target-helpers/sw_helper.h:103
#3  0x00007ffff713ec8b in pipe_loader_sw_create_screen (dev=0x5555556aa000, config=<optimized out>, sw_vk=<optimized out>)
    at ../src/gallium/auxiliary/pipe-loader/pipe_loader_sw.c:425
#4  0x00007ffff713ebf7 in pipe_loader_create_screen_vk (dev=0x5555556aa000, sw_vk=sw_vk@entry=false)
    at ../src/gallium/auxiliary/pipe-loader/pipe_loader.c:171
#5  0x00007ffff713ec2b in pipe_loader_create_screen (dev=<optimized out>) at ../src/gallium/auxiliary/pipe-loader/pipe_loader.c:177
#6  0x00007ffff6aa7648 in drisw_init_screen (sPriv=0x5555556a7da0) at ../src/gallium/frontends/dri/drisw.c:548
#7  0x00007ffff6ab0b4f in driCreateNewScreen2 (scrn=0, fd=-1, extensions=<optimized out>, driver_extensions=<optimized out>,
    driver_configs=0x555555630e20, data=0x555555630300) at ../src/gallium/frontends/dri/dri_util.c:142
#8  0x00007ffff77ae59f in dri2_create_screen (disp=disp@entry=0x555555630300) at ../src/egl/drivers/dri2/egl_dri2.c:1091
#9  0x00007ffff77b243a in dri2_initialize_x11_swrast (disp=0x555555630300) at ../src/egl/drivers/dri2/platform_x11.c:1451
#10 dri2_initialize_x11 (disp=disp@entry=0x555555630300) at ../src/egl/drivers/dri2/platform_x11.c:1686
#11 0x00007ffff77ac5e8 in dri2_initialize (disp=0x555555630300) at ../src/egl/drivers/dri2/egl_dri2.c:1180
#12 dri2_initialize (disp=disp@entry=0x555555630300) at ../src/egl/drivers/dri2/egl_dri2.c:1147
#13 0x00007ffff77a39fe in eglInitialize (dpy=0x555555630300, major=0x7fffffffdc24, minor=0x7fffffffdc20) at ../src/egl/main/eglapi.c:639
#14 0x00005555555688d4 in GLStateEGL::gotValidDisplay (this=0x7fffffffdda0) at ../src/gl-state-egl.cpp:514
#15 0x0000555555567f2b in CanvasGeneric::init (this=0x7fffffffdeb0) at ../src/canvas-generic.cpp:43
#16 CanvasGeneric::init (this=0x7fffffffdeb0) at ../src/canvas-generic.cpp:38
#17 0x000055555556526c in main (argc=<optimized out>, argv=<optimized out>) at ../src/main.cpp:205

eglInitialize -> dri2_initialize -> dri2_initialize_x11 -> dri2_initialize_x11_swrast -> dri2_create_screen
	-> dri2_dpy->swrast->createNewScreen -> driCreateNewScreen2 -> psp.driver.InitScreen -> drisw_init_screen

context_create

llvmpipe_create_screen
llvmpipe_create_context

./src/gallium/targets/dri/target.c
	#if defined(GALLIUM_SOFTPIPE)

	const __DRIextension **__driDriverGetExtensions_swrast(void);

	PUBLIC const __DRIextension **__driDriverGetExtensions_swrast(void)
	{
	   return galliumsw_driver_extensions;
	}

./src/gallium/frontends/dri/drisw.c
	const struct __DriverAPIRec galliumsw_driver_api = {
	   .InitScreen = drisw_init_screen,
	   .DestroyScreen = dri_destroy_screen,
	   .CreateBuffer = drisw_create_buffer,
	   .DestroyBuffer = dri_destroy_buffer,
	   .SwapBuffers = drisw_swap_buffers,
	   .CopySubBuffer = drisw_copy_sub_buffer,
	};
	
	static const struct __DRIDriverVtableExtensionRec galliumsw_vtable = {
	   .base = { __DRI_DRIVER_VTABLE, 1 },
	   .vtable = &galliumsw_driver_api,
	};

	const __DRIextension *galliumsw_driver_extensions[] = {
		&driCoreExtension.base,
		&driSWRastExtension.base,
		&driSWCopySubBufferExtension.base,
		&gallium_config_options.base,
		&galliumsw_vtable.base,
		NULL
	};

llvm: (llvm-project区别于llvm)
cmake -S ./llvm -B build \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/opt/llvm \
    -DLLVM_ENABLE_PROJECTS="" \
    -DLLVM_TARGETS_TO_BUILD="X86" \
    -DLLVM_INCLUDE_TOOLS=ON \
    -DLLVM_BUILD_TOOLS=OFF \
	-DLLVM_ENABLE_RTTI=ON \
    -DLLVM_INCLUDE_EXAMPLES=OFF \
    -DLLVM_INCLUDE_TESTS=OFF \
	-DLLVM_INCLUDE_BENCHMARKS=OFF \
	-DLLVM_INSTALL_UTILS=ON \
	-DLLVM_INSTALL_MODULEMAPS=ON \
	-DLLVM_INSTALL_BINUTILS_SYMLINKS=ON \
	-DLLVM_INSTALL_CCTOOLS_SYMLINKS=ON \
	-DLLVM_LINK_LLVM_DYLIB=ON                  # 生成 libLLVM-14.so
	
cmake -S ./ -B build \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/opt/llvm \
    -DLLVM_ENABLE_PROJECTS="" \
    -DLLVM_TARGETS_TO_BUILD="X86" \
    -DLLVM_INCLUDE_TOOLS=ON \
    -DLLVM_BUILD_TOOLS=OFF \
	-DLLVM_ENABLE_RTTI=ON \
    -DLLVM_INCLUDE_EXAMPLES=OFF \
    -DLLVM_INCLUDE_TESTS=OFF \
	-DLLVM_INCLUDE_BENCHMARKS=OFF \
	-DLLVM_INSTALL_UTILS=ON \
	-DLLVM_INSTALL_MODULEMAPS=ON \
	-DLLVM_INSTALL_BINUTILS_SYMLINKS=ON \
	-DLLVM_INSTALL_CCTOOLS_SYMLINKS=ON \
	-DLLVM_LINK_LLVM_DYLIB=ON 
	
make -C build llvm-config -j$(nproc)
make -C build all -j$(nproc)
	
	-DBUILD_SHARED_LIBS=ON

cmake --build build -j $(nproc) --target all
cmake --build build -j $(nproc) --target llvm-config

cmake --install build
cp -fa build/bin/llvm-config /opt/llvm/bin/

------------------------------------------------------------------------------------------------------------------
OHOS - 交叉编译使用 qemu-static

/root/opc/device/board/opc/x86_general/test/glmark2
/root/opc/device/board/opc/x86_general/test/native_window_ohos
/root/opc/device/board/opc/x86_general/test/native_window_wrapper

# clang
export PATH=/root/opc/prebuilts/clang/ohos/linux-x86_64/llvm/bin:$PATH

# CC
export CC="clang --target=x86_64-linux-ohosmusl --sysroot=/root/opc/out/x86_general/obj/third_party/musl -fno-emulated-tls"
export CXX="clang++ --target=x86_64-linux-ohosmusl --sysroot=/root/opc/out/x86_general/obj/third_party/musl -fno-emulated-tls"

# LLVM构建
cmake -S ./ -B build-ohos \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/opt/llvm-ohos \
    -DLLVM_ENABLE_PROJECTS="" \
    -DLLVM_TARGETS_TO_BUILD="X86" \
    -DLLVM_INCLUDE_TOOLS=OFF \
    -DLLVM_BUILD_TOOLS=OFF \
	-DLLVM_ENABLE_RTTI=OFF \
    -DLLVM_INCLUDE_EXAMPLES=OFF \
    -DLLVM_INCLUDE_TESTS=OFF \
	-DLLVM_INCLUDE_BENCHMARKS=OFF \
	-DLLVM_INCLUDE_TOOLS=ON

cmake -S llvm -B build-ohos \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/opt/llvm-ohos \
    -DLLVM_ENABLE_PROJECTS="" \
    -DLLVM_TARGETS_TO_BUILD="X86" \
    -DLLVM_INCLUDE_TOOLS=OFF \
    -DLLVM_BUILD_TOOLS=OFF \
	-DLLVM_ENABLE_RTTI=OFF \
    -DLLVM_INCLUDE_EXAMPLES=OFF \
    -DLLVM_INCLUDE_TESTS=OFF \
	-DLLVM_INCLUDE_BENCHMARKS=OFF \
	-DLLVM_INCLUDE_TOOLS=ON

# llvm-tablegen执行所需的libc++.so的动态库：
export LD_LIBRARY_PATH=/root/opc/prebuilts/clang/ohos/linux-x86_64/llvm/lib/x86_64-linux-ohos
ln -sf /root/opc/out/x86_general/obj/third_party/musl/usr/lib/x86_64-linux-ohos/libc.so /lib/ld-musl-x86_64.so.1

make -C build-ohos all -j$(nproc)
make -C build-ohos llvm-config -j$(nproc)

cmake --install build-ohos
cp -fa build-ohos/bin/llvm-config /opt/llvm-ohos/bin/

LLVMPasses  -lLLVMObjCARCOpts

/root/opc/prebuilts/clang/ohos/linux-x86_64/llvm/bin/clang++  -o src/gallium/targets/dri/libgallium_dri.so src/gallium/targets/dri/libgallium_dri.so.p/target.c.o -L/root/opc/out/x86_general/obj/third_party/musl/usr/lib/x86_64-linux-ohos -L/root/opc/prebuilts/clang/ohos/linux-x86_64/llvm/lib/clang/15.0.4/lib/x86_64-linux-ohos -L/root/opc/prebuilts/clang/ohos/linux-x86_64/llvm/lib/x86_64-linux-ohos -Wl,--as-needed -Wl,--no-undefined -fuse-ld=lld -shared -fPIC -Wl,--start-group -Wl,-soname,libgallium_dri.so --target=x86_64-linux-ohosmusl --sysroot=/root/opc/out/x86_general/obj/third_party/musl -fPIC -Wl,--exclude-libs=libunwind_llvm.a -Wl,--exclude-libs=libc++_static.a -Wl,--exclude-libs=libvpx_assembly_arm.a -Wl,--warn-shared-textrel --rtlib=compiler-rt '-Wl,-rpath,$ORIGIN/../../../mapi/shared-glapi:/root/opc/out/x86_general/thirdparty/libdrm:/root/opc/out/x86_general/hiviewdfx/hilog' -Wl,-rpath-link,/root/mesa3d/build-ohos/src/mapi/shared-glapi -Wl,-rpath-link,/root/opc/out/x86_general/thirdparty/libdrm -Wl,-rpath-link,/root/opc/out/x86_general/hiviewdfx/hilog src/gallium/frontends/dri/libdri.a src/util/libmesa_util.a src/util/format/libmesa_format.a src/util/libmesa_util_sse41.a src/c11/impl/libmesa_util_c11.a src/mesa/libmesa.a src/compiler/glsl/libglsl.a src/compiler/glsl/glcpp/libglcpp.a src/compiler/nir/libnir.a src/compiler/libcompiler.a src/mesa/libmesa_sse41.a src/gallium/auxiliary/libgalliumvl.a src/gallium/auxiliary/libgallium.a src/mapi/shared-glapi/libglapi.so.0.0.0 src/gallium/auxiliary/pipe-loader/libpipe_loader_static.a src/loader/libloader.a src/util/libxmlconfig.a src/gallium/winsys/sw/null/libws_null.a src/gallium/winsys/sw/wrapper/libwsw.a src/gallium/winsys/sw/dri/libswdri.a src/gallium/winsys/sw/kms-dri/libswkmsdri.a src/gallium/drivers/llvmpipe/libllvmpipe.a src/gallium/drivers/softpipe/libsoftpipe.a src/gallium/drivers/virgl/libvirgl.a src/gallium/winsys/virgl/drm/libvirgldrm.a src/gallium/winsys/virgl/common/libvirglcommon.a src/gallium/winsys/virgl/vtest/libvirglvtest.a -Wl,--build-id=sha1 -Wl,--gc-sections -Wl,--version-script /root/mesa3d/src/gallium/targets/dri/dri.sym -Wl,--dynamic-list /root/mesa3d/src/gallium/targets/dri/../dri-vdpau.dyn /root/opc/out/x86_general/thirdparty/libdrm/libdrm.so -lrt -ldl -lpthread -lm -L/opt/llvm-ohos/lib -lLLVMCoroutines -lLLVMipo -lLLVMVectorize -lLLVMLinker -lLLVMIRReader -lLLVMAsmParser -lLLVMFrontendOpenMP -lLLVMX86TargetMCA -lLLVMMCA -lLLVMX86Disassembler -lLLVMX86AsmParser -lLLVMX86CodeGen -lLLVMCFGuard -lLLVMGlobalISel -lLLVMX86Desc -lLLVMX86Info -lLLVMSelectionDAG -lLLVMInstrumentation -lLLVMAsmPrinter -lLLVMMCJIT -lLLVMMCDisassembler -lLLVMInterpreter -lLLVMExecutionEngine -lLLVMRuntimeDyld -lLLVMOrcTargetProcess -lLLVMOrcShared -lLLVMCodeGen -lLLVMTarget -lLLVMScalarOpts -lLLVMInstCombine -lLLVMAggressiveInstCombine -lLLVMTransformUtils -lLLVMBitWriter -lLLVMAnalysis -lLLVMProfileData -lLLVMSymbolize -lLLVMDebugInfoPDB -lLLVMDebugInfoMSF -lLLVMDebugInfoDWARF -lLLVMObject -lLLVMTextAPI -lLLVMMCParser -lLLVMMC -lLLVMDebugInfoCodeView -lLLVMBitReader -lLLVMCore -lLLVMRemarks -lLLVMBitstreamReader -lLLVMBinaryFormat -lLLVMSupport -lLLVMDemangle -pthread /root/opc/out/x86_general/obj/third_party/skia/m133/third_party/expat/libexpat.a /root/opc/out/x86_general/obj/third_party/zlib/libz.a -L/opt/llvm-ohos/lib -lLLVMCoroutines -lLLVMipo -lLLVMVectorize -lLLVMLinker -lLLVMIRReader -lLLVMAsmParser -lLLVMFrontendOpenMP -lLLVMX86TargetMCA -lLLVMMCA -lLLVMX86Disassembler -lLLVMX86AsmParser -lLLVMX86CodeGen -lLLVMCFGuard -lLLVMGlobalISel -lLLVMX86Desc -lLLVMX86Info -lLLVMSelectionDAG -lLLVMInstrumentation -lLLVMAsmPrinter -lLLVMMCJIT -lLLVMMCDisassembler -lLLVMInterpreter -lLLVMExecutionEngine -lLLVMRuntimeDyld -lLLVMOrcTargetProcess -lLLVMOrcShared -lLLVMCodeGen -lLLVMTarget -lLLVMScalarOpts -lLLVMInstCombine -lLLVMAggressiveInstCombine -lLLVMTransformUtils -lLLVMBitWriter -lLLVMAnalysis -lLLVMProfileData -lLLVMSymbolize -lLLVMDebugInfoPDB -lLLVMDebugInfoMSF -lLLVMDebugInfoDWARF -lLLVMObject -lLLVMTextAPI -lLLVMMCParser -lLLVMMC -lLLVMDebugInfoCodeView -lLLVMBitReader -lLLVMCore -lLLVMRemarks -lLLVMBitstreamReader -lLLVMBinaryFormat -lLLVMSupport -lLLVMDemangle /root/opc/out/x86_general/hiviewdfx/hilog/libhilog.so -L/opt/llvm-ohos/lib -lLLVMCoroutines -lLLVMipo -lLLVMVectorize -lLLVMLinker -lLLVMIRReader -lLLVMAsmParser -lLLVMFrontendOpenMP -lLLVMX86TargetMCA -lLLVMMCA -lLLVMX86Disassembler -lLLVMX86AsmParser -lLLVMX86CodeGen -lLLVMCFGuard -lLLVMGlobalISel -lLLVMX86Desc -lLLVMX86Info -lLLVMSelectionDAG -lLLVMInstrumentation -lLLVMAsmPrinter -lLLVMMCJIT -lLLVMMCDisassembler -lLLVMInterpreter -lLLVMExecutionEngine -lLLVMRuntimeDyld -lLLVMOrcTargetProcess -lLLVMOrcShared -lLLVMCodeGen -lLLVMTarget -lLLVMScalarOpts -lLLVMInstCombine -lLLVMAggressiveInstCombine -lLLVMTransformUtils -lLLVMBitWriter -lLLVMAnalysis -lLLVMProfileData -lLLVMSymbolize -lLLVMDebugInfoPDB -lLLVMDebugInfoMSF -lLLVMDebugInfoDWARF -lLLVMObject -lLLVMTextAPI -lLLVMMCParser -lLLVMMC -lLLVMDebugInfoCodeView -lLLVMBitReader -lLLVMCore -lLLVMRemarks -lLLVMBitstreamReader -lLLVMBinaryFormat -lLLVMSupport -lLLVMDemangle -lLLVMPasses -lLLVMObjCARCOpts -Wl,--end-group
------------------------------------------------------------------------------------------------------------------

# mesa

export PATH=/opt/llvm-ohos/bin:/root/opc/prebuilts/clang/ohos/linux-x86_64/llvm/bin:$PATH
export LD_LIBRARY_PATH=/root/opc/prebuilts/clang/ohos/linux-x86_64/llvm/lib/x86_64-linux-ohos

export PKG_CONFIG_PATH=/root/mesa3d/pkgconfig

./build_mesa3d.py /root/opc/out/x86_general 

meson setup /root/mesa3d build-ohos -Dplatforms=ohos -Degl-native-platform=ohos -Ddri-drivers=  \
	-Dgallium-drivers=virgl,swrast -Dvulkan-drivers= -Dgbm=enabled -Degl=enabled -Dcpp_rtti=true -Dzstd=disabled \
	-Dglx=disabled -Dtools= -Dshared-llvm=disabled -Ddri-search-path=/vendor/lib64/chipsetsdk --cross-file=cross_file \
	--prefix=/root/mesa3d/build-ohos/install

# llvm-config的指定：使用cross_file必须用 binaries指定
cross_file
	llvm-config = '/opt/llvm-ohos/bin/llvm-config'
	
hilog | grep DfxFaultLogger

/system/bin/bootanimation

glmark2-es2-ohos --data-path /system/lib/glmark2/data/

qemu-system-x86_64 -M q35 -cpu max -m 8G -smp 8 -kernel bzImage -enable-kvm -serial stdio \
-device e1000e,netdev=tap0 \
-netdev tap,id=tap0,ifname=tap0,script=no,downscript=no \
-hda ubuntu22.img -append "console=ttyS0 root=/dev/sda rw nokaslr"

qemu-system-x86_64 --enable-kvm -machine q35,accel=kvm,vmport=off -m 16G \
-smp 8 -drive if=none,file=updater.img,format=raw,id=updater,index=0 \
-device virtio-blk-pci,drive=updater -drive if=none,file=system.img,format=raw,id=system,index=1 \
-device virtio-blk-pci,drive=system -drive if=none,file=vendor.img,format=raw,id=vendor,index=2 \
-device virtio-blk-pci,drive=vendor -drive if=none,file=userdata.img,format=raw,id=userdata,index=3 \
-device virtio-blk-pci,drive=userdata -append "loglevel=1 ip=192.168.33.2:255.255.255.0::eth0:off \
sn=0023456789 console=tty0 console=ttyS0 init=/bin/init ohos.boot.hardware=x86_general root=/dev/ram0 \
rw ohos.required_mount.system=/dev/block/vdb@/usr@ext4@ro,barrier=1@wait,required \
ohos.required_mount.vendor=/dev/block/vdc@/vendor@ext4@ro,barrier=1@wait,required \
ohos.required_mount.misc=/dev/block/vda@/misc@none@none=@wait,required" \
-kernel bzImage -initrd ramdisk.img -nographic -vga none \
-device virtio-gpu-pci,max_outputs=1,xres=1024,yres=768,addr=08.0  \
-device virtio-mouse-pci -device virtio-keyboard-pci -device es1370  -k en-us  \
-display gtk,gl=off -device e1000e,netdev=tap0 -netdev tap,id=tap0,ifname=tap0,script=no,downscript=no \
-serial stdio -monitor none -hda /root/ubuntu/x86/ubuntu22.img

cd ~/opc/out/x86_general/packages/phone/images

service_control stop foundation

mount -t ext4 /dev/block/sda /mnt

ifconfig eth0 192.168.33.2 up

rm -f /root/.ssh/known_hosts
ssh 192.168.33.2

/system/bin/bootanimation

LD_LIBRARY_PATH=/mnt/gdb_lib /mnt/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2   /mnt/usr/bin/gdbserver 0.0.0.0:1234 /system/bin/bootanimation

/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2

# 查看一个可执行文件的所有动态库
ldd /usr/bin/gdb | cut -d">" -f2 | cut -d"(" -f1 | grep -v vdso | xargs -I{} sh -c "\cp {} gdb_lib/"

ln -sf /vendor/lib64/chipsetsdk/libGLESv2.so.2.0.0   /mnt/mesa_lib/libGLESv2.so.2
ln -sf /vendor/lib64/chipsetsdk/libEGL.so.1.0.0      /mnt/mesa_lib/libEGL.so.1

LD_LIBRARY_PATH=/mnt/mesa_lib /mnt/CH02_HelloTriangle

LD_LIBRARY_PATH=/mnt/gdb_lib:/mnt/mesa_lib:/vendor/lib64/chipsetsdk /mnt/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2   /mnt/usr/bin/gdb /mnt/CH02_HelloTriangle

nm libgallium_dri.so | cut -d" " -f3 | sort | uniq > normal.symbols

nm libgallium_dri.so | cut -d" " -f3 | sort | uniq > new.symbols

strings libgallium_dri.so | grep "\.cpp" | wc -l

strings virtio_gpu_dri.so | grep "\.cpp" | sort | uniq > new.files

strings virtio_gpu_dri.so | grep "\.cpp" | sort | uniq > normal.files

git clone https://gitee.com/openharmony/third_party_llvm-project.git -b OpenHarmony-6.0-Release

git clone git@gitee.com:openharmony/third_party_llvm-project.git -b OpenHarmony-6.0-Release

hilog | grep DfxFaultLogger

ip tuntap add tap0 mode tap group 0
ip link set dev tap0 up
ip addr add 192.168.33.1/24 dev tap0
iptables -A POSTROUTING -t nat -j MASQUERADE -s 192.168.33.0/24
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -P FORWARD ACCEPT

LD_LIBRARY_PATH=/mnt/gdb_lib /mnt/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2 /mnt/usr/bin/gdbserver 0.0.0.0:1234 /system/bin/bootanimation

gdb out/x86_general/exe.unstripped/graphic/graphic_2d/bootanimation

 target remote 192.168.33.2:1234
 b glGetString
 b entry_current_get
 directory /root/mesa3d/build-ohos/
 
 b _mesa_initialize_exec_table
 
scp out/x86_general/lib.unstripped/graphic/graphic_2d/libGLESv3.so 192.168.33.2:/system/lib64/platformsdk/

#0  _mesa_initialize_exec_table (ctx=0x7ffff6c110a0) at src/mapi/glapi/gen/api_exec_init.c:50
#1  0x00007fff6348f7d9 in _mesa_initialize_dispatch_tables (ctx=0x7ffff6c110a0) at ../src/mesa/main/context.c:937
#2  0x00007fff63704214 in st_create_context_priv (ctx=0x7ffff6c110a0, pipe=<optimized out>, options=0x7fff68df9118) at ../src/mesa/state_tracker/st_context.c:780
#3  st_create_context (api=api@entry=API_OPENGLES2, pipe=pipe@entry=0x7ffff6fd3060, visual=<optimized out>, share=share@entry=0x0, options=options@entry=0x7fff68df9118, no_error=false, has_egl_image_validate=<optimized out>) at ../src/mesa/state_tracker/st_context.c:877
#4  0x00007fff63719755 in st_api_create_context (stapi=<optimized out>, smapi=0x7ffff7187ce0, attribs=0x7fff68df90f0, error=0x7fff68df90c4, shared_stctxi=0x0) at ../src/mesa/state_tracker/st_manager.c:1078
#5  0x00007fff633e932c in dri_create_context (api=api@entry=API_OPENGLES2, visual=visual@entry=0x7ffff70069d0, cPriv=cPriv@entry=0x7ffff7043850, ctx_config=ctx_config@entry=0x7fff68df91b8, error=error@entry=0x7fff68df9234, sharedContextPrivate=sharedContextPrivate@entry=0x0)
    at ../src/gallium/frontends/dri/dri_context.c:169
#6  0x00007fff633e8770 in driCreateContextAttribs (screen=0x7ffff730a9d0, api=<optimized out>, config=0x7ffff70069d0, shared=<optimized out>, num_attribs=<optimized out>, attribs=<optimized out>, error=0x7fff68df9234, data=0x7ffff6f116a0) at ../src/gallium/frontends/dri/dri_util.c:634
#7  0x00007fff6bea59b0 in dri2_create_context (disp=<optimized out>, conf=0x7fffefd96eb0, share_list=<optimized out>, attrib_list=<optimized out>) at ../src/egl/drivers/dri2/egl_dri2.c:1599
#8  0x00007fff6be98f3b in eglCreateContext (dpy=0x7ffff6a9f030, config=0x7fffefd96eb0, share_list=0x0, attrib_list=0x7fffef50f0b4) at ../src/egl/main/eglapi.c:830
#9  0x00007fffee6bae44 in OHOS::EglWrapperDisplay::CreateEglContext(void*, void*, int const*) () from target:/system/lib64/platformsdk/libEGL.so
#10 0x00007fffee6ad719 in eglCreateContext () from target:/system/lib64/platformsdk/libEGL.so

entry_current_get -> GET_DISPATCH -> 

_glapi_tls_Dispatch
_glapi_get_dispatch

static inline const struct _glapi_table *
entry_current_get(void)
{
   return _glapi_get_dispatch();
}

src/mapi/glapi/glapi.h
	#if DETECT_OS_WINDOWS && !defined(MAPI_MODE_UTIL) && !defined(MAPI_MODE_GLAPI)
	# define GET_DISPATCH() _glapi_get_dispatch()
	# define GET_CURRENT_CONTEXT(C)  struct gl_context *C = (struct gl_context *) _glapi_get_context()
	#else
	# define GET_DISPATCH() _glapi_tls_Dispatch
	# define GET_CURRENT_CONTEXT(C)  struct gl_context *C = (struct gl_context *) _glapi_tls_Context
	#endif


src/mapi/entry.c
	static inline const struct _glapi_table *
	entry_current_get(void)
	{
	   return GET_DISPATCH();
	}

./src/mapi/shared-glapi/libglapi.so.0.0.0.p/.._u_current.c.o
0000000000000000 T _glapi_get_dispatch

__emutls_v._glapi_tls_Dispatch
__emutls_t._glapi_tls_Dispatch

__emutls_get_address
entry_current_get

__tls_get_addr

_GLAPI_EXPORT struct _glapi_table *_glapi_get_dispatch(void);


./src/mapi/u_current.h
	#define u_current_get_table_internal _glapi_get_dispatch
	
	#define u_current_table _glapi_tls_Dispatch
	
./src/mapi/u_current.c
	void
	u_current_set_table(const struct _glapi_table *tbl)
	{
	   stub_init_once();

	   if (!tbl)
		  tbl = (const struct _glapi_table *) table_noop_array;

	   u_current_table = (struct _glapi_table *) tbl;
	}

	void
	u_current_set_table(const struct _glapi_table *tbl)
	{
	   stub_init_once();

	   if (!tbl)
		  tbl = (const struct _glapi_table *) table_noop_array;

	   _glapi_tls_Dispatch = (struct _glapi_table *) tbl;
	}

	struct _glapi_table *
	u_current_get_table_internal(void)
	{
	   return u_current_table;
	}

	struct _glapi_table *
	_glapi_get_dispatch(void)
	{
	   return _glapi_tls_Dispatch;
	}

	__attribute__((visibility("default"))) extern _Thread_local struct _glapi_table * _glapi_tls_Dispatch;
	
	_Thread_local struct _glapi_table *_glapi_tls_Dispatch = (struct _glapi_table *) table_noop_array;

	_glapi_set_dispatch(ctx->CurrentClientDispatch); -> u_current_set_table((const struct _glapi_table *) dispatch);

ninja ./src/mapi/shared-glapi/libglapi.so.0.0.0.p/.._u_current.c.o  -v

__emutls_get_address
__tls_get_addr

libgallium_dri.so

03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: Tid:1076, Name:bootanimation
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #00 pc 0000000000008d4f /vendor/lib64/chipsetsdk/libGLESv2.so.2.0.0(glGetString+15)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #01 pc 000000000000ae8f /system/lib64/platformsdk/libprivacy_communication_adapter_cxx.z.so(9ca36d249711e9aa52e69d11faba2acd)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #02 pc 00000000000b5f72 /system/lib/ld-musl-x86_64.so.1(__libc_malloc_impl+290)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #03 pc 0000000000124e36 /system/lib/ld-musl-x86_64.so.1(malloc+182)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #04 pc 00000000000450d0 /vendor/lib64/chipsetsdk/libglapi.so.0.0.0(__emutls_get_address+32)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #05 pc 0000000000008d4f /vendor/lib64/chipsetsdk/libGLESv2.so.2.0.0(glGetString+15)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #06 pc 0000000000867f04 /system/lib64/platformsdk/librender_service_base.z.so(OHOS::Rosen::RenderContext::SetUpGpuContext(std::__h::shared_ptr<OHOS::Rosen::Drawing::GPUContext>)+212)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #07 pc 0000000000868124 /system/lib64/platformsdk/librender_service_base.z.so(OHOS::Rosen::RenderContext::AcquireSurface(int, int)+68)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #08 pc 000000000002e42e /system/lib64/platformsdk/libEGL.so(eglMakeCurrent+142)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #09 pc 000000000086293a /system/lib64/platformsdk/librender_service_base.z.so(OHOS::Rosen::RSSurfaceFrameOhosGl::CreateSurface()+42)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #10 pc 0000000000008e48 /system/lib64/chipset-sdk-sp/libhitrace_meter.so(FinishTrace+88)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #11 pc 0000000000862876 /system/lib64/platformsdk/librender_service_base.z.so(OHOS::Rosen::RSSurfaceFrameOhosGl::GetCanvas()+38)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #12 pc 000000000002ad31 /system/bin/bootanimation(OHOS::BootPicturePlayer::Draw()+295)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #13 pc 000000000001f8d4 /system/lib64/chipset-sdk-sp/libeventhandler.z.so(OHOS::AppExecFwk::EventHandler::DistributeEvent(std::__h::unique_ptr<OHOS::AppExecFwk::InnerEvent, void (*)(OHOS::AppExecFwk::InnerEvent*)> const&)+1236)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #14 pc 00000000000337dc /system/lib64/chipset-sdk-sp/libeventhandler.z.so(OHOS::AppExecFwk::(anonymous namespace)::EventRunnerImpl::ExecuteEventHandler(std::__h::unique_ptr<OHOS::AppExecFwk::InnerEvent, void (*)(OHOS::AppExecFwk:)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #15 pc 00000000000cb179 /system/lib64/chipset-sdk-sp/libc++.so(std::__h::mutex::unlock()+9)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #16 pc 00000000000278fd /system/lib64/chipset-sdk-sp/libeventhandler.z.so(OHOS::AppExecFwk::EventQueueBase::GetEvent()+717)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #17 pc 0000000000040170 /system/lib64/chipset-sdk-sp/libeventhandler.z.so(OHOS::AppExecFwk::InnerEventPool::Drop(OHOS::AppExecFwk::InnerEvent*)+0)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #18 pc 0000000000040170 /system/lib64/chipset-sdk-sp/libeventhandler.z.so(OHOS::AppExecFwk::InnerEventPool::Drop(OHOS::AppExecFwk::InnerEvent*)+0)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #19 pc 0000000000032afe /system/lib64/chipset-sdk-sp/libeventhandler.z.so(OHOS::AppExecFwk::(anonymous namespace)::EventRunnerImpl::Run()+1006)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #20 pc 00000000000d0cd0 /system/lib64/chipset-sdk-sp/libc++.so(std::__h::basic_string<char, std::__h::char_traits<char>, std::__h::allocator<char>>::append(char const*, unsigned long)+336)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #21 pc 0000000000040170 /system/lib64/chipset-sdk-sp/libeventhandler.z.so(OHOS::AppExecFwk::InnerEventPool::Drop(OHOS::AppExecFwk::InnerEvent*)+0)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #22 pc 0000000000040170 /system/lib64/chipset-sdk-sp/libeventhandler.z.so(OHOS::AppExecFwk::InnerEventPool::Drop(OHOS::AppExecFwk::InnerEvent*)+0)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #23 pc 000000000003647d /system/lib64/chipset-sdk-sp/libeventhandler.z.so(OHOS::AppExecFwk::EventRunner::Run()+429)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #24 pc 000000000002395f /system/bin/bootanimation(OHOS::BootAnimationOperation::StartEventHandler(OHOS::BootAnimationConfig const&)+2549)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #25 pc 00000000001256a0 /system/lib/ld-musl-x86_64.so.1(prctl+528)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #26 pc 00000000000d9730 /system/lib64/chipset-sdk-sp/libc++.so(std::__h::__thread_specific_ptr<std::__h::__thread_struct>::__at_thread_exit(void*)+0)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #27 pc 0000000000040170 /system/lib64/chipset-sdk-sp/libeventhandler.z.so(OHOS::AppExecFwk::InnerEventPool::Drop(OHOS::AppExecFwk::InnerEvent*)+0)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #28 pc 00000000000b4230 /system/lib64/chipset-sdk-sp/libc++.so(__cxa_guard_release+49)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #29 pc 0000000000022eeb /system/bin/bootanimation
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #30 pc 00000000001144ee /system/lib/ld-musl-x86_64.so.1(start+174)
03-08 15:01:05.488  1186  1186 I C02d11/DfxFaultLogger: #31 pc 000000000008784b /system/lib/ld-musl-x86_64.so.1

mount vendor.img /mnt

rm -f /mnt/lib64/chipsetsdk/libEGL.so.1.0.0
rm -f /mnt/lib64/chipsetsdk/libGLESv1_CM.so.1.1.0
rm -f /mnt/lib64/chipsetsdk/libGLESv2.so.2.0.0
rm -f /mnt/lib64/chipsetsdk/libgallium_dri.so
rm -f /mnt/lib64/chipsetsdk/libgbm.so.1.0.0
rm -f /mnt/lib64/chipsetsdk/libglapi.so.0.0.0

cp virtio_gpu/*   /mnt/lib64/chipsetsdk/

glmark2-es2-ohos --data-path /system/lib/glmark2/data/
/system/bin/bootanimation
/data/gl_test

qemu-system-x86_64 --enable-kvm -machine q35,accel=kvm,vmport=off -m 16G -smp 8 \
-drive if=none,file=updater.img,format=raw,id=updater,index=0 -device virtio-blk-pci,drive=updater \
-drive if=none,file=system.img,format=raw,id=system,index=1 -device virtio-blk-pci,drive=system \
-drive if=none,file=vendor.img,format=raw,id=vendor,index=2 -device virtio-blk-pci,drive=vendor \
-drive if=none,file=userdata.img,format=raw,id=userdata,index=3 -device virtio-blk-pci,drive=userdata \
-append "loglevel=7 ip=192.168.33.2:255.255.255.0::eth0:off sn=0023456789 console=tty0 console=ttyS0 \
init=/bin/init ohos.boot.hardware=x86_general root=/dev/ram0 rw ohos.required_mount.system=/dev/block/vdb@/usr@ext4@ro,barrier=1@wait,required \
ohos.required_mount.vendor=/dev/block/vdc@/vendor@ext4@ro,barrier=1@wait,required ohos.required_mount.misc=/dev/block/vda@/misc@none@none=@wait,required" \
-kernel bzImage -initrd ramdisk.img -nographic -vga none -device virtio-gpu-pci,max_outputs=1,xres=1024,yres=768,addr=08.0  \
-device virtio-mouse-pci -device virtio-keyboard-pci -device es1370  -k en-us  -display gtk,gl=off -device e1000e,netdev=tap0 \
-netdev tap,id=tap0,ifname=tap0,script=no,downscript=no -serial stdio -monitor none

service_control stop foundation
service_control stop render_service
service_control stop composer_host

service_control start composer_host
service_control start render_service
service_control start foundation

src/egl/drivers/dri2/platform_ohos.c

static int get_fourcc(int native)
{
    switch (native) {
        case PIXEL_FMT_RGB_565:
            return DRM_FORMAT_RGB565;
        case PIXEL_FMT_BGRA_8888:
            return DRM_FORMAT_ARGB8888;
        case PIXEL_FMT_RGBA_8888:
            return DRM_FORMAT_ABGR8888;
        case PIXEL_FMT_RGBX_8888:
            return DRM_FORMAT_XBGR8888;
        default:
            _eglLog(_EGL_WARNING, "unsupported native buffer format 0x%{public}x", native);
    }
    return -1;
}

LD_LIBRARY_PATH=/mnt/gdb_lib /mnt/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2   /mnt/usr/bin/gdbserver 0.0.0.0:1234 /system/bin/bootanimation

ninja ./src/egl/libEGL.so.1.0.0.p/drivers_dri2_platform_ohos.c.o -v

clang -E 

qemu: native 12
PIXEL_FMT_RGBA_8888

qemu riscv
https://laval.csdn.net/user/discuss/6963b40f6554f1331aa1452c

case PIXEL_FMT_BGRX_8888:

return DRM_FORMAT_XRGB8888;

static int get_fourcc(int native)
{
    switch (native) {
        case PIXEL_FMT_RGB_565:
            return DRM_FORMAT_RGB565;
        case PIXEL_FMT_BGRA_8888:
            return DRM_FORMAT_ARGB8888;
        case PIXEL_FMT_RGBA_8888:
            return DRM_FORMAT_ABGR8888;
        case PIXEL_FMT_RGBX_8888:
            return DRM_FORMAT_XBGR8888;
        case PIXEL_FMT_BGRX_8888:
            return DRM_FORMAT_XRGB8888;
        default:
            _eglLog(_EGL_WARNING, "unsupported native buffer format 0x%{public}x", native);
    }
    return -1;
}

/data/gl_test -l

snapshot_display

cf621eda48

foundation/graphic/graphic_2d/rosen/modules/render_service_client/core/pipeline/rs_render_thread.cpp:122:        ROSEN_LOGD("RSRenderThread DrawFrame(%{public}" PRIu64 ") in %{public}s",

C01406/OHOS::RS: RSRenderThread DrawFrame(2291695855114) in GPU

03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderThread ProcessCommands size: 1
03-11 23:55:00.030  1515  1515 D C01201/EventHandler: RemoveTask enter name ArkUIVsyncTimeoutCheck
03-11 23:55:00.030  1515  1515 D C01201/EventQueueBase: Remove name enter
03-11 23:55:00.030  1515  1515 D C01201/EventQueueBase: Remove filter enter
03-11 23:55:00.030  1515  1515 D C01201/EventHandler: RemoveTask enter name ArkUIVsyncRecover
03-11 23:55:00.030  1515  1515 D C01201/EventQueueBase: Remove name enter
03-11 23:55:00.030  1515  1515 D C01201/EventQueueBase: Remove filter enter
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSTransactionDataCallbackManager trigger callback error, timeStamp: 2291679853842 token: 0
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderThread DrawFrame(2291695855114) in GPU
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: Root SystemUi_VolumePanel: Negative width or height [0.000000 0.000000]
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1515 D C02d66/Hiview-PerfMonitor: OnSceneStop: SceneManager::OnSceneStop scene has not started, scene type: 4
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1515 D C02d66/Hiview-PerfMonitor: OnSceneStop: SceneManager::OnSceneStop scene has not started, scene type: 6
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1515 D C02d66/Hiview-PerfMonitor: OnSceneStop: SceneManager::OnSceneStop scene has not started, scene type: 8
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1515 D C01201/EventQueueBase: time consumer: 0
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: apply legacy property failed, type: 138
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: apply legacy property failed, type: 138
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: apply legacy property failed, type: 138
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: apply legacy property failed, type: 140
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: apply legacy property failed, type: 138
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: apply legacy property failed, type: 140
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: apply legacy property failed, type: 138
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: apply legacy property failed, type: 140
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: Root SystemUi_ControlPanel: Negative width or height [0.000000 0.000000]
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: Root SystemUi_NotificationPanel: Negative width or height [0.000000 0.000000]
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: Root SystemUi_BannerNotice: Negative width or height [0.000000 0.000000]
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RootNode SystemUi_StatusBar: Invisible
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RootNode SystemUi_NavigationBar: Invisible
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSRenderNode::apply modifiers RSRenderNode's dirty is false or dirtyTypes_ is none
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RootNode SystemUi_PrivacyIndicator: Invisible
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: Root SystemUi_VolumePanel: Negative width or height [0.000000 0.000000]
03-11 23:55:00.030   486   974 D C057c1/ProcessSkeleton: AttachInvokerProcInfo 285: 13554192, 1515 1515 10008 4832716538 0
03-11 23:55:00.030   486   974 D C057c1/ProcessSkeleton: AttachInvokerProcInfo 285: 13554192, 486 486 1003 0 0
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: RSSurfaceOhosGl:RequestFrame, width is 640, height is 480
03-11 23:55:00.030  1515  1668 D C01406/OHOS::RS: SetUpGpuContext: Drawing GPUContext has already created!!
03-11 23:55:00.031  1515  1668 D C01406/OHOS::RS: AcquireSurface: CreateCanvas successfully!!!
03-11 23:55:00.031  1515  1668 E C01406/OHOS::RS: QueryEglBufferAge: eglQuerySurface is failed
03-11 23:55:00.031  1515  1668 D C01406/OHOS::RS: ProcessRootRenderNode SetBufferAge with invalid buffer age -1
03-11 23:55:00.031  1515  1668 D C01406/OHOS::RS: UpdateDirtyAndSetEGLDamageRegion dirtyRect = [0, 0, 640, 480]
03-11 23:55:00.031  1515  1668 D C01406/OHOS::RS: RSImage::HDRConvert HDRDraw pixelMap_ IsHdr: 0
03-11 23:55:00.031  1515  1668 D C01406/OHOS::RS: RSRenderThreadVisitor FlushImplicitTransactionFromRT uiTimestamp = 2291679853842
03-11 23:55:00.031  1515  1668 D C01406/OHOS::RS: RSRenderThreadVisitor FlushFrame surfaceNodeId = 6506875453443, uiTimestamp = 2291679853842
03-11 23:55:00.031  1515  1668 D C01406/OHOS::RS: RenderFrame: RenderFrame: Canvas
03-11 23:55:00.031   486   974 D C057c1/ProcessSkeleton: AttachInvokerProcInfo 285: 13554192, 1515 1515 10008 4832716538 0
03-11 23:55:00.032   486   974 E C01400/DISP: [SetMetadata@display_buffer_vdi_impl.cpp:87] SetMetadata is not supported
03-11 23:55:00.032   486   974 E C02515/METADATA_SRV: [SetMetadata:136]  fail
03-11 23:55:00.032   486   974 E C01401/Bufferqueue: <surface_buffer_impl.cpp:750-SetMetadata>: SetMetadata Failed with -5
03-11 23:55:00.032   486   974 D C01400/SyncFence: <289>WriteToMessageParcel: WriteToMessageParcel fence is invalid : -1
03-11 23:55:00.032   486   974 D C057c1/ProcessSkeleton: AttachInvokerProcInfo 285: 13554192, 486 486 1003 0 0
03-11 23:55:00.034  1515  1668 D C01400/SyncFence: <289>WriteToMessageParcel: WriteToMessageParcel fence is invalid : -1
03-11 23:55:00.034   486   974 D C057c1/ProcessSkeleton: AttachInvokerProcInfo 285: 13554192, 1515 1515 10008 4832716538 0
03-11 23:55:00.034   486   974 D C01406/OHOS::RS: RsDebug RSRenderServiceListener::OnBufferAvailable node id:6506875453443
03-11 23:55:00.034   486   974 D C057c1/ProcessSkeleton: AttachInvokerProcInfo 285: 13554192, 486 486 1003 0 0
03-11 23:55:00.034  1515  1668 D C01406/OHOS::RS: SwapBuffers: SwapBuffers successfully
03-11 23:55:00.034  1515  1668 D C01406/OHOS::RS: RSSurfaceOhosGl: FlushFrame, SwapBuffers eglsurface
03-11 23:55:00.034  1515  1668 D C01406/OHOS::RS: Root SystemUi_ControlPanel: Negative width or height [0.000000 0.000000]
03-11 23:55:00.034  1515  1668 D C01406/OHOS::RS: Root SystemUi_NotificationPanel: Negative width or height [0.000000 0.000000]
03-11 23:55:00.034  1515  1668 D C01406/OHOS::RS: Root SystemUi_BannerNotice: Negative width or height [0.000000 0.000000]
03-11 23:55:00.034  1515  1668 D C01406/OHOS::RS: RootNode SystemUi_StatusBar: Invisible
03-11 23:55:00.034  1515  1668 D C01406/OHOS::RS: RootNode SystemUi_NavigationBar: Invisible
03-11 23:55:00.034  1515  1668 D C01406/OHOS::RS: RootNode SystemUi_PrivacyIndicator: Invisible

```

