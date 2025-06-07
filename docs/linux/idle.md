# idle

## 1、cpu初始化后idle过程

```shell
# swapper 进程 (process 0)

# kernel初始化之后主核idle
init/main.c: start_kernel -> rest_init -> kernel/sched/idle.c: cpu_startup_entry(CPUHP_ONLINE) -> cpu_idle_loop

# smp核idle
arch/x86/kernel/head_32.S: startup_32_smp -> arch/x86/kernel/smpboot.c: start_secondary(initial_code) -> cpu_start_entry(CPUHP_ONLINE) -> cpu_idle_loop
```

## 2、swapper进程资源

```shell
# process 0 进程资源
init_task
init_stack
init_mm      -> INIT_MM
init_mmap    -> INIT_MMAP
init_fs      -> INIT_FS
init_files   -> INIT_FILES
init_signals -> INIT_SIGNALS
```

## 3、init进程启动流程

```shell
init/main.c: start_kernel -> rest_init -> kernel_init -> run_init_process
    argv_init[0] = init_filename;
    do_execve (getname_kernel(init_filename), ...)
```

