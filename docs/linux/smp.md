# x86_32 smp

```shell
v4.3

1、arch/x86/boot/header.S 的 hdr 重点字段: jump (_start，装载程序入口点 -> main -> 进入保护模式 -> code32_start)，code32_start(保护模式入口点)， pref_address (内核装载地址)
2、setup.elf (arch/x86/boot/) 包括 16位 和 32位 代码，且没有开启分页模式，都是物理地址，用于装载内核； vmlinux.bin 为自解压代码和压缩过的内核，setup装载后调用即可。 setup.bin、vmlinux.bin -> bzImage
3、bzImage包含了内核装载代码，efi代码待分析，但是和 hdr 脱不了关系
4、执行路径：
  1) bootloader设置 hdr 的 code32_start 为 arch/x86/boot/compressed/head_32.S:startup_32，此时进入保护模式
  2) bootloader执行 hdr(boot_params) 的 jump 字段的指令，跳转到 arch/x86/boot/header.S: start_of_setup，开始装载压缩过的内核 -> main -> go_to_protected_mode -> protected_mode_jump(code32_start)
  3) arch/x86/boot/compressed/head_32.S:startup_32 开始解压内核到 pref_address，并正式开始执行内核代码

	b *0x100000   # 进入保护模式后跳转的地址，放自解压代码的入口
	b *0x1000000  # 自解压代码将内核解压的位置，为内核的入口

# 编译选项
arch/x86/Makefile
CODE16GCC_CFLAGS := -m32 -Wa,$(srctree)/arch/x86/boot/code16gcc.h
M16_CFLAGS       := $(call cc-option, -m16, $(CODE16GCC_CFLAGS))
REALMODE_CFLAGS := $(M16_CFLAGS) -g -Os -D__KERNEL__

# 实模式进入保护模式
code32_start 被设置为 arch/x86/boot/compressed/head_32.S:startup_32 

# 查看 pref_address:  0x1000000  -> 代码为arch/x86/kernel/head_32.S:startup_32
arch/x86/boot/header.S

make  ARCH=i386 arch/x86/boot/header.o V=1

pref_address： ((0x1000000 + (0x200000 - 1)) & ~(0x200000 - 1)) = 0x1000000

# 内核链接脚本
__HEAD                .section        ".head.text","ax"

arch/x86/kernel/head_32.S
	__HEAD
	ENTRY(startup_32)

// 内核ELF文件入口点为 加载的物理地址 0x1000000
readelf -e vmlinux -> Entry point address: 0x1000000

// 虚拟地址 和 物理地址 不一样
Program Headers:
  Type           Offset   VirtAddr   PhysAddr   FileSiz MemSiz  Flg Align
  LOAD           0x001000 0xc1000000 0x01000000 0xac6000 0xac6000 R E 0x1000
  LOAD           0xac7000 0xc1ac6000 0x01ac6000 0x157000 0x317000 RW  0x1000

// 此处 0x1000000 即为 内核物理地址， 加上 0xC0000000 即为虚拟地址
// 链接脚本中： . 指定虚拟地址， AT 指定 section 的物理地址
arch/x86/kernel/vmlinux.lds
	ENTRY(phys_startup_32)
	SECTIONS
	{
	 . = 0xC0000000 + ((0x1000000 + (0x200000 - 1)) & ~(0x200000 - 1));
	 phys_startup_32 = startup_32 - 0xC0000000;
	 /* Text and read-only data */
	 .text : AT(ADDR(.text) - 0xC0000000) {
	  _text = .;
	  /* bootstrapping code */
	  *(.head.text)
	  
# 调试 arch/x86/kernel/head_32.S: startup_32_smp 代码

# objdump -d vmlinux | grep startup_32_smp
c10000f0 <startup_32_smp>:

gdb -> add-symbol-file vmlinux 0x1000000 -> target remote :1234 -> b *0x10000f0 即可

# smp cpu启动
1) startup_32_smp 入口 初始化
init/main.c:start_kernel -> arch/x86/kernel/setup.c： setup_arch -> arch/x86/realmode/init.c: reserve_real_mode, setup_real_mode

   在低1M分配real_mode_header
   memcpy(base, real_mode_blob, size) => 将 arch/x86/realmode/rm 编译出来的生成件拷贝到该头
   
   # trampoline_header 为指针，实际定义在 arch/x86/realmode/rm/trampoline_32.S
   trampoline_header = (struct trampoline_header *) __va(real_mode_header->trampoline_header);
   
   # 将 startup_32_smp 物理地址放到 real_mode_header 的 trampoline_header 指针指向的地址的 tr_start
   trampoline_header->start = __pa_symbol(startup_32_smp);
   
2) arch/x86/realmode/rm/header.S  被编成 bin，然后拷贝到内存
	# 定义的全部都是指针，
    .section ".header", "a"
        .balign 16
GLOBAL(real_mode_header)
        .long   pa_text_start
        .long   pa_ro_end
        /* SMP trampoline */
        .long   pa_trampoline_start  # 被设置为 arch/x86/realmode/rm/trampoline_32.S 的 trampoline_start的地址
        .long   pa_trampoline_status
        .long   pa_trampoline_header
	
	arch/x86/realmode/rm/trampoline_32.S
	    .text
        .code16
		ENTRY(trampoline_start)
			movl    tr_start, %eax
			ljmpl   $__BOOT_CS, $pa_startup_32
		
		.section ".text32","ax"
		.code32
		ENTRY(startup_32)                       # note: also used from wakeup_asm.S
			jmp     *%eax
		
        GLOBAL(trampoline_header)
            tr_start:               .space  4
            tr_gdt_pad:             .space  2
            tr_gdt:                 .space  6
        END(trampoline_header)

3) trampoline_start被设置
	ret_from_kernel_thread -> kernel_init -> kernel_init_freeable -> smp_init -> cpu_up -> _cpu_up -> __cpu_up -> native_cpu_up -> do_boot_cpu
		unsigned long start_ip = real_mode_header->trampoline_start;
		smpboot_setup_warm_reset_vector(start_ip);
			*((volatile unsigned short *)phys_to_virt(TRAMPOLINE_PHYS_HIGH)) = start_eip >> 4;
			*((volatile unsigned short *)phys_to_virt(TRAMPOLINE_PHYS_LOW)) = start_eip & 0xf;

# 调试 trampoline_start
1） 找到 real_mode_header 加载的地址，然后获取物理地址即可，只调试汇编代码，不用源代码

# 0x7c00 的代码只提示用户使用bootloader，不直接将内核放到 MBR; qemu只需要作为bootloader加载内核代码即可，没必要模拟BIOS启动MBR(可能都没有执行到0x7c00)
        .code16
        .section ".bstext", "ax"

        .global bootsect_start
bootsect_start:
        # Normalize the start address
        ljmp    $BOOTSEG, $start2

start2:
        movw    %cs, %ax
        movw    %ax, %ds
        movw    %ax, %es
        movw    %ax, %ss
        xorw    %sp, %sp
        sti
        cld

        movw    $bugger_off_msg, %si

msg_loop:
        lodsb
        andb    %al, %al    // 不是 xor，不改变 al的值； 只测试 al 是否是 0，即字符串结尾
        jz      bs_die
        movb    $0xe, %ah
        movw    $7, %bx
        int     $0x10
        jmp     msg_loop

        .section ".bsdata", "a"
bugger_off_msg:
        .ascii  "Use a boot loader.\r\n"
        .ascii  "\n"
        .ascii  "Remove disk and press any key to reboot...\r\n"
        .byte   0


        .section ".header", "a"
        .globl  sentinel
sentinel:       .byte 0xff, 0xff        /* Used to detect broken loaders */

        .globl  hdr
hdr:
setup_sects:    .byte 0                 /* Filled in by build.c */
root_flags:     .word ROOT_RDONLY
syssize:        .long 0                 /* Filled in by build.c */
ram_size:       .word 0                 /* Obsolete */
vid_mode:       .word SVGA_MODE
root_dev:       .word 0                 /* Filled in by build.c */
boot_flag:      .word 0xAA55

        # offset 512, entry point

        .globl  _start
_start:
                # Explicitly enter this as bytes, or the assembler
                # tries to generate a 3-byte jump here, which causes
                # everything else to push off to the wrong offset.
                .byte   0xeb            # short (2-byte) jump
                .byte   start_of_setup-1f

                .ascii  "HdrS"          # header signature
                .word   0x020d          # header version number (>= 0x0105)
				
code32_start:                           # here loaders can put a different
                                        # start address for 32-bit code.
                .long   0x100000        # 0x100000 = default for big kernel

        .section ".entrytext", "ax"
start_of_setup:
# Force %es = %ds
        movw    %ds, %ax
        movw    %ax, %es
        cld

# Jump to C code (should not return)
        calll   main

arch/x86/boot/main.c
arch/x86/boot/pm.c
	main
		copy_boot_params
			# arch/x86/boot/header.S 里的全局变量 hdr 和 boot_params结构体里的参数一一对应
			memcpy(&boot_params.hdr, &hdr, sizeof hdr);
			
		go_to_protected_mode
			enable_a20()
			reset_coprocessor();
			mask_all_interrupts();
			setup_idt();
			setup_gdt();
			protected_mode_jump(boot_params.hdr.code32_start,
                            (u32)&boot_params + (ds() << 4));
			
			// 第一个参数 code32_start: %eax，第二个参数 boot_params: %edx
			movw %ds,%dx
			movzwl  %dx, %edx
			sall    $4, %edx
			addl    $boot_params, %edx
			movl    boot_params+532, %eax
			call    protected_mode_jump

arch/x86/boot/pmjump.S
        .text
        .code16

/*
 * void protected_mode_jump(u32 entrypoint, u32 bootparams);
 */
GLOBAL(protected_mode_jump)
        movl    %edx, %esi              # Pointer to boot_params table

        xorl    %ebx, %ebx
        movw    %cs, %bx
        shll    $4, %ebx
        addl    %ebx, 2f                # 将 %ebx 加到 2f 指向的内存
        jmp     1f                      # Short jump to serialize on 386/486
1:

        movw    $__BOOT_DS, %cx
        movw    $__BOOT_TSS, %di

        movl    %cr0, %edx
        orb     $X86_CR0_PE, %dl        # Protected mode
        movl    %edx, %cr0

        # Transition to 32-bit mode
        .byte   0x66, 0xea              # ljmpl opcode
2:      .long   in_pm32                 # offset
        .word   __BOOT_CS               # segment
ENDPROC(protected_mode_jump)

		# 此处进入32位模式
        .code32
        .section ".text32","ax"
GLOBAL(in_pm32)
        # Set up data segments for flat 32-bit mode
        movl    %ecx, %ds
        movl    %ecx, %es
        movl    %ecx, %fs
        movl    %ecx, %gs
        movl    %ecx, %ss
        # The 32-bit code sets up its own stack, but this way we do have
        # a valid stack if some debugging hack wants to use it.
        addl    %ebx, %esp

        # Set up TR to make Intel VT happy
        ltr     %di

        # Clear registers to allow for future extensions to the
        # 32-bit boot protocol
        xorl    %ecx, %ecx
        xorl    %edx, %edx
        xorl    %ebx, %ebx
        xorl    %ebp, %ebp
        xorl    %edi, %edi

        # Set up LDTR to make Intel VT happy
        lldt    %cx
		// %eax 为 protected_mode_jump 传递的 code32_start
        jmpl    *%eax                   # Jump to the 32-bit entrypoint
ENDPROC(in_pm32)

arch/x86/include/uapi/asm/bootparam.h
struct setup_header {
		__u32   code32_start;
};
struct boot_params {
        struct setup_header hdr;    /* setup header */  /* 0x1f1 */
};

arch/x86/boot/setup.ld
	ENTRY(_start)
	. = 0;
	.bstext         : { *(.bstext) }
	.bsdata         : { *(.bsdata) }

	. = 495;
	.header         : { *(.header) }
	.entrytext      : { *(.entrytext) }
	.inittext       : { *(.inittext) }
	.initdata       : { *(.initdata) }
	__end_init = .;

	.text           : { *(.text) }
	.text32         : { *(.text32) }

arch/x86/boot/bzImage
arch/x86/boot/setup.bin arch/x86/boot/vmlinux.bin

arch/x86/boot/setup.bin
arch/x86/boot/header.S  arch/x86/boot/setup.ld
arch/x86/boot/*

# 查看汇编代码的好方法
make  ARCH=i386 arch/x86/boot/pm.o V=1
-c 换为 -S，pm.o 换为 pm.s
-m16

arch/x86/boot/vmlinux.bin
/vmlinux -> arch/x86/boot/compressed/vmlinux.bin -> arch/x86/boot/compressed/vmlinux.bin.gz -> arch/x86/boot/compressed/piggy.o -> arch/x86/boot/compressed/vmlinux -> arch/x86/boot/vmlinux.bin
                                                                                               arch/x86/boot/compressed/*

看一下汇编代码即可
16、32、64bit 的 函数调用参数传递、系统调用参数传递
```

## 内核启动流程

- 主核启动流程

  ```shell
  qemu/pc-bios/optionrom/linuxboot_dma.c: load_kernel(cs:ip设置为0x1020:0, setup.bin加载地址为0x10000) -> arch/x86/boot/header.S: _start,start_of_setup (setup.bin,0x10000) -> arch/x86/boot/compressed/head_32.S: startup_32(compressed_vmlinux,code32_start,0x100000) -> arch/x86/kernel/head_32.S: startup_32(/vmlinux,pref_address,CONFIG_PHYSICAL_START,0x1000000) -> default_entry -> arch/x86/kernel/head32.c: i386_start_kernel(initial_code) -> init/main.c: start_kernel -> rest_init 
  
  rest_init -> kernel/fork.c: kernel_thread(kernel_init -> _do_fork -> copy_process -> arch/x86/kernel/process_32.c: copy_thread_tls { child: [ childregs->bx = sp(kernel_init); p->thread.ip = ret_from_kernel_thread; ] parent: [p->thread.ip = ret_from_fork; ] } -> ret_from_kernel_thread -> init/main.c: kernel_init -> kernel_init_freeable
  
  rest_init -> kernel_thread(kthreadd ...
  
  // 主核创建完内核线程后，和其他smp核一样，都进入idle状态，谓：SMP；内核线程也都在所有cpu调度
  rest_init -> cpu_startup_entry -> kernel/sched/idle.c: cpu_startup_entry -> cpu_idle_loop -> while (!need_resched()) cpuidle_idle_call -> default_idle_call -> arch/x86/kernel/process.c: arch_cpu_idle -> default_idle(x86_idle) -> include/linux/irqflags.h: safe_halt(raw_safe_halt,arch_safe_halt) -> arch/x86/include/asm/paravirt.h: arch_safe_halt(pv_irq_ops.safe_halt) -> arch/x86/include/asm/irqflags.h: native_safe_halt("sti; hlt")
  ```

- smp核启动流程

  - 主核设置smp核的tr_start 为 startup_32_smp

    ```shell
    init/main.c: start_kernel -> arch/x86/kernel/setup.c: setup_arch -> arch/x86/realmode/init.c: setup_real_mode(trampoline_header->start = __pa_symbol(startup_32_smp), arch/x86/realmode/rm/trampoline_32.S 的 trampoline_header->tr_start)
    ```

  - 主核设置smp核的trampoline入口为trampoline_header. trampoline_start，设置initial_code为start_secondary，启动smp核

    ```shell
    init/main.c: start_kernel -> rest_init -> kernel/fork.c: kernel_thread(kernel_init -> _do_fork -> copy_process -> arch/x86/kernel/process_32.c: copy_thread_tls { child: [ childregs->bx = sp(kernel_init); p->thread.ip = ret_from_kernel_thread; ] parent: [p->thread.ip = ret_from_fork; ] } -> ret_from_kernel_thread -> init/main.c: kernel_init -> kernel_init_freeable -> kernel/smp.c: smp_init(for_each_present_cpu(cpu)) -> kernel/cpu.c: cpu_up -> _cpu_up -> arch/x86/include/asm/smp.h: __cpu_up(smp_ops.cpu_up) -> arch/x86/kernel/smp.c: (.cpu_up = native_cpu_up) -> arch/x86/kernel/smpboot.c: native_cpu_up -> do_boot_cpu
    
    arch/x86/include/asm/apic.h
    #define TRAMPOLINE_PHYS_LOW             0x467
    #define TRAMPOLINE_PHYS_HIGH            0x469
    
    arch/x86/kernel/smpboot.c: do_boot_cpu
    start_ip = real_mode_header->trampoline_start;
    initial_code = (unsigned long)start_secondary;
    
    // 设置 IPI 的启动地址为arch/x86/realmode/rm/trampoline_32.S的trampoline_start  - start_ip: 0x9c000 
    smpboot_setup_warm_reset_vector(start_ip);
        *((volatile unsigned short *)phys_to_virt(TRAMPOLINE_PHYS_HIGH)) = start_eip >> 4;
        *((volatile unsigned short *)phys_to_virt(TRAMPOLINE_PHYS_LOW)) = start_eip & 0xf;
        
    wakeup_cpu_via_init_nmi
        wakeup_secondary_cpu_via_init
            // 设置APIC，Send IPI 
    
    // 恢复
    smpboot_restore_warm_reset_vector();
        *((volatile u32 *)phys_to_virt(TRAMPOLINE_PHYS_LOW)) = 0;
    ```

  - smp执行流程

  ```shell
  // cpu_stop_threads 和 softirq_threads 的调用 - 都是主核创建
  // 1. kernel_init -> do_pre_smp_initcalls
  // 2. kernel_init -> smp_init -> cpu_up -> smpboot_create_threads 
  
  // 设置 x86_idle 为 default_idle
  arch/x86/kernel/smpboot.c: start_secondary -> smp_callin -> smp_store_cpu_info -> arch/x86/kernel/cpu/common.c: identify_secondary_cpu -> identify_cpu -> arch/x86/kernel/process.c: select_idle_routine(x86_idle = default_idle;)
  
  // pv_irq_ops.safe_halt 设置
  arch/x86/kernel/paravirt.c
  pv_irq_ops pv_irq_ops = {
      ...
      .safe_halt = native_safe_halt;
      ...
  };
  
  // smp idle
  arch/x86/realmode/rm/trampoline_32.S: trampoline_start(0x9c000) -> arch/x86/kernel/head_32.S: startup_32_smp(trampoline_header.tr_start) -> arch/x86/kernel/smpboot.c: start_secondary(initial_code) -> kernel/sched/idle.c: cpu_startup_entry -> cpu_idle_loop -> while (!need_resched()) cpuidle_idle_call -> default_idle_call -> arch/x86/kernel/process.c: arch_cpu_idle -> default_idle(x86_idle) -> include/linux/irqflags.h: safe_halt(raw_safe_halt,arch_safe_halt) -> arch/x86/include/asm/paravirt.h: arch_safe_halt(pv_irq_ops.safe_halt) -> arch/x86/include/asm/irqflags.h: native_safe_halt("sti; hlt")
  
  // 注册每个cpu内核线程 -> 为什么注册两次？
  include/linux/init.h
  #define early_initcall(fn) __define_initcall(fn, early)
  __section__(".initcallearly.init")
  
  // 链接脚本
  arch/x86/kernel/vmlinux.lds: section(.init.data)
  
  // early_initcall 的调用
  init/main.c: kernel_init -> kernel_init_freeable -> do_pre_smp_initcalls
      for (fn = __initcall_start; fn < __initcall0_start; fn++)
          do_one_initcall(*fn);
  
  // 其他 initcall 的调用
  init/main.c: kernel_init -> kernel_init_freeable -> do_basic_setup -> do_initcalls
      for (level = 0; level < ARRAY_SIZE(initcall_levels) - 1; level++)
          do_initcall_level(level);
              for (fn = initcall_levels[level]; fn < initcall_levels[level+1]; fn++)
                  do_one_initcall(*fn);
  
  // initcall 顺序
      __initcall_start: early
  initcall_levels
      __initcall0_start: pure
      __initcall1_start: core
      __initcall2_start: postcore
      __initcall3_start: arch
      __initcall4_start: subsys
      __initcall5_start: fs
      (.initcallrootfs.init: rootfs)
      __initcall6_start: device
      __initcall7_start: late
      __initcall_end
  
  kernel/stop_machine.c 
  smp_hotplug_thread cpu_stop_threads = {
      ...
      .thread_fn              = cpu_stopper_thread,
      .thread_comm            = "migration/%u",
      ...
  };
  cpu_stop_init
      for_each_possible_cpu(cpu)
          include/linux/smpboot.h: smpboot_register_percpu_thread(&cpu_stop_threads)
          kernel/smpboot.c: smpboot_register_percpu_thread_cpumask
              for_each_online_cpu(cpu) __smpboot_create_thread(plug_thread, cpu);
              list_add(&plug_thread->list, &hotplug_threads);
  early_initcall(cpu_stop_init);
  
  kernel/softirq.c
  struct smp_hotplug_thread softirq_threads = {
      ...
      .thread_fn              = run_ksoftirqd,
      .thread_comm            = "ksoftirqd/%u",
      ...
  };
  spawn_ksoftirqd
      smpboot_register_percpu_thread(&softirq_threads);
  early_initcall(spawn_ksoftirqd);
  
  // 在启动smp核创建的线程
  init/main.c: kernel_init -> kernel_init_freeable -> kernel/smp.c: smp_init(for_each_present_cpu(cpu)) -> kernel/cpu.c: cpu_up -> _cpu_up -> kernel/smpboot.c: smpboot_create_threads
      // softirq_threads 和 cpu_stop_threads
      list_for_each_entry(cur, &hotplug_threads, list) {
          __smpboot_create_thread(cur, cpu);
      }
  
  // smp 调度
  kernel/sched/idle.c:
  ```

- 虚拟地址

  - x86_32 的 低内存空间，直接映射到虚拟地址的高1G，即0xC0000000

  ```shell
  CONFIG_PAGE_OFFSET=0xC0000000
  arch/x86/include/asm/page_32_types.h
  #define __PAGE_OFFSET  _AC(CONFIG_PAGE_OFFSET, UL)
  
  arch/x86/include/asm/page_32.h
  #define __phys_addr(x)  __phys_addr_nodebug(x)
  
  arch/x86/include/asm/page_32.h
  #define __phys_addr_nodebug(x) ((x) - PAGE_OFFSET)
  
  arch/x86/include/asm/page.h
  #define __pa(x)         __phys_addr((unsigned long)(x))
  #define __va(x)         ((void *)((unsigned long)(x)+PAGE_OFFSET))
  
  // 通过虚拟地址查看SMP的IPI的内容
  (gdb) x/10x 0xC0000000
  (gdb) watch *(unsigned short*)0xC0000467    // 0x9C00
  (gdb) watch *(unsigned short*)0xC0000467    // 0x0000
  ```

  
