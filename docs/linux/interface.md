# interface

```c
// 内核符号名字空间
// MODULE_IMPORT_NS
// EXPORT_SYMBOL_NS

// module重定位代码 - arch/arm64/kernel/module.c

// workqueue
// system_wq events schedule_work
// drivers/char/rtc.c -> add_wait_queue remove_wait_queue _wake_up prepare_to_wait

// for_each_task
// find_task_by_pid hash_pid unhash_pid
```

