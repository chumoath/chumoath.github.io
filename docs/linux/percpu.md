# percpu

## 1、percpu变量

- percpu变量列表

  ```shell
  cpu_number
  this_cpu_off
  cpu_current_top_of_stack
  current_task
  __preempt_count
  irq_count
  cpu_info
  irq_regs
  hardirq_stack
  softirq_stack
  nmi_state
  ```

## 2、x86_32 cpu_current_top_of_stack 的设置

- 获取当前进程的PCB

  ```c
  include/asm-generic/current.h: current -> get_current(current_thread_info()->task)
      arch/x86/include/asm/thread_info.h: current_thread_info
          return (struct thread_info *)(current_top_of_stack() - THREAD_SIZE);
          arch/x86/include/asm/processor.h: current_top_of_stack
              return this_cpu_read_stable(cpu_current_top_of_stack);
  ```

- 设置 percpu cpu_current_top_of_stack

  ```c
  # cpu_current_top_of_stack 初始化
  
  init/init_task.c
      struct task_struct init_task = INIT_TASK(init_task);
      union thread_union init_thread_union __init_task_data = { INIT_THREAD_INFO(init_task) };
  
  // THREAD_SIZE = (((1UL) << 12) << 1) = 0x2000;
  arch/x86/kernel/cpu/common.c
      DEFINE_PER_CPU(unsigned long, cpu_current_top_of_stack) = (unsigned long)&init_thread_union + THREAD_SIZE;
  
  # smp cpu的设置
  kernel/cpu.c: _cpu_up
      struct task_struct *idle;
      idle = idle_thread_get(cpu);
          kernel/smpboot.c: idle_thread_get
              struct task_struct *tsk = per_cpu(idle_threads, cpu);
              init_idle(tsk, cpu);
      __cpu_up(cpu, idle);
      arch/x86/include/asm/smp.h: __cpu_up
          smp_ops.cpu_up(cpu, tidle);
          arch/x86/kernel/smpboot.c: native_cpu_up
              common_cpu_up(cpu, tidle);
                  per_cpu(cpu_current_top_of_stack, cpu) = (unsigned long)task_stack_page(idle) + THREAD_SIZE;
              do_boot_cpu(apicid, cpu, tidle);
  
  # 进程切换
  arch/x86/kernel/process_32.c
  __switch_to
      this_cpu_write(cpu_current_top_of_stack, (unsigned long)task_stack_page(next_p) + THREAD_SIZE);
      this_cpu_write(current_task, next_p);
  ```

- 设置 percpu idle_threads

  ```c
  kernel/smpboot.c
      static DEFINE_PER_CPU(struct task_struct *, idle_threads);
  
  kernel/sched/core.c: sched_init
      kernel/sched/core.c: init_idle(current, smp_processor_id());
          kernel/sched/sched.h: __set_task_cpu(idle, cpu);
              task_thread_info(p)->cpu = cpu;
      kernel/smpboot.c: idle_thread_set_boot_cpu
          per_cpu(idle_threads, smp_processor_id()) = current;
          
  kernel/smp.c: smp_init
      kernel/smpboot.c: idle_threads_init
          for_each_possible_cpu(cpu) if (cpu != boot_cpu) idle_init(cpu);
              kernel/smpboot.c: idle_init
              struct task_struct *tsk = per_cpu(idle_threads, cpu);
              if (!tsk) tsk = fork_idle(cpu); per_cpu(idle_threads, cpu) = tsk;
              kernel/fork.c: fork_idle
                  task = copy_process(CLONE_VM, 0, 0, NULL, &init_struct_pid, 0, 0);
                  init_idle_pids(task->pids);
                  init_idle(task, cpu);
  
  kernel/cpu.c: _cpu_up
      struct task_struct *idle;
      idle = idle_thread_get(cpu);
          kernel/smpboot.c: idle_thread_get
              struct task_struct *tsk = per_cpu(idle_threads, cpu);
              init_idle(tsk, cpu);
  ```

## 3、gdb查看percpu变量

- 当前cpu的offset的设置

  ```c
  arch/x86/kernel/setup_percpu.c
  DEFINE_PER_CPU_READ_MOSTLY(unsigned long, this_cpu_off) = BOOT_PERCPU_OFFSET;
  
  unsigned long __per_cpu_offset[NR_CPUS] __read_mostly = {
          [0 ... NR_CPUS-1] = BOOT_PERCPU_OFFSET,
  };
  
  // __per_cpu_offset 和 this_cpu_off 的设置
  include/asm-generic/percpu.h
      #define per_cpu_offset(x) (__per_cpu_offset[x])
  
  arch/x86/kernel/setup_percpu.c
      setup_per_cpu_areas
          delta = (unsigned long)pcpu_base_addr - (unsigned long)__per_cpu_start;
          for_each_possible_cpu(cpu) {
              per_cpu_offset(cpu) = delta + pcpu_unit_offsets[cpu];
              per_cpu(this_cpu_off, cpu) = per_cpu_offset(cpu);
              per_cpu(cpu_number, cpu) = cpu;
              // 将offset设置为GDT_ENTRY_PERCPU段的基址，GDT_ENTRY_PERCPU段的选择子放在%fs
              setup_percpu_segment(cpu);
                  pack_descriptor(&gdt, per_cpu_offset(cpu), 0xFFFFF, 0x2 | DESCTYPE_S, 0x8);
                  gdt.s = 1;
                  write_gdt_entry(get_cpu_gdt_table(cpu), GDT_ENTRY_PERCPU, &gdt, DESCTYPE_S);
  
  arch/x86/include/asm/segment.h
      # define __KERNEL_PERCPU                (GDT_ENTRY_PERCPU*8)
      
  // 主核和smp核的%fs的设置
  arch/x86/kernel/head_32.S: 
      startup_32_smp -> default_entry
      startup_32 -> default_entry
      movl $(__KERNEL_PERCPU), %eax
      movl %eax,%fs
  
  // 子进程的%fs的设置
  arch/x86/kernel/process_32.c: copy_thread_tls
      childregs->fs = __KERNEL_PERCPU;
  ```

- gdb查看percpu变量的值

  ```c
  // 获取指定cpu的offset 
  (gdb) p __per_cpu_offset[cpu_id]
  
  // 获取变量的类型 - 获取每个cpu的cpu_number变量
  (gdb) p &cpu_number
  (gdb) p *(int *)((unsigned long)&cpu_number+__per_cpu_offset[cpu_id])
  
  // 获取每个cpu的this_cpu_off变量
  (gdb) p &this_cpu_off
  (gdb) p *(unsigned long *)((unsigned long)&this_cpu_off+__per_cpu_offset[cpu_id])
  ```

- 查看系统中所有的task