# openharmony_drm

- [gdb调试openharmony进程](https://laval.csdn.net/6646f9fd931dbe49ec6d8528.html)
- [gdb工具-使用静态链接的gdbserver](git clone https://gitee.com/qq2820/buildroot-tool-ohos.git)
- 在openharmony使用glibc的两种方法：
  1. 全部使用静态链接 (静态链接的，只要内核特性兼容，在哪里都可以跑)
  2. 显示使用glibc的动态链接器执行可执行程序(和glibc系统实际执行方式一致)

```shell
# 调试信息：is_debug=true use_unstripped_as_runtime_output=true

ifconfig eth0 192.168.33.2 up

device/soc/opc/x86_general/hardware/display/src/display_device/core/hdi_session.cpp
# Init的调用 - 单例模式
HdiSession &HdiSession::GetInstance()
	std::call_once(once, [&]() { instance.Init(); });
	return instance;

HdiSession::Init
    HdiDeviceInterface::DiscoveryDevice                         # 打开drm
	auto displays = device->DiscoveryDisplay();                 # 基于drm设备创建HdiDisplay(DrmDisplay)
	mHdiDisplays.insert(displays.begin(), displays.end());
	
HdiDeviceInterface::DiscoveryDevice
	std::shared_ptr<HdiDeviceInterface> drmDevice = DrmDevice::Create();  # 打开drm
	ret = drmDevice->Init();                                              
		drmSetClientCap(GetDrmFd(), DRM_CLIENT_CAP_UNIVERSAL_PLANES, 1);
		drmSetClientCap(GetDrmFd(), DRM_CLIENT_CAP_ATOMIC, 1);
		drmSetMaster(GetDrmFd());
		drmIsMaster(GetDrmFd());
	
DrmDevice::DiscoveryDisplay
	drmModeGetResources
	FindAllCrtc
	FindAllEncoder
	FindAllConnector
	FindAllPlane
	使用 crtc 和 DrmDevice 创建 DrmDisplay
	调用 DrmDisplay::Init -> HdiDisplay::Init
	std::shared_ptr<HdiDisplay> display = std::make_shared<DrmDisplay>(connector, crtc, mInstance);
	DISPLAY_LOGD();
	display->Init();
	mDisplays.emplace(display->GetId(), std::move(display));
	
device/soc/opc/x86_general/hardware/display/src/display_device/drm/drm_vsync_worker.cpp
DrmVsyncWorker - 单例模式
	DrmVsyncWorker::GetInstance()
		static DrmVsyncWorker instance;
		int ret = instance.Init(DrmDevice::GetDrmFd());
		return instance;
	
	DrmVsyncWorker::Init
		mThread = std::make_unique<std::thread>([this]() { WorkThread(); });
		mRunning = true;
		
	DrmVsyncWorker::WorkThread()
		WaitSignalAndCheckRuning();
		uint64_t time = WaitNextVBlank(seq);
		mCallBack->Vsync(seq, time);
		
	DrmVsyncWorker::WaitSignalAndCheckRuning()
		mCondition.wait(ul, [this]() { return (mEnable || !mRunning); });
	
	DrmVsyncWorker::WaitNextVBlank
		drmWaitVBlank(mDrmFd, &vblank);
	
	DrmVsyncWorker::EnableVsync(bool enable)
		mEnable = enable;
		mCondition.notify_one();
```

