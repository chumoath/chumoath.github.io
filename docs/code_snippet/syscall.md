# x86 syscall

- `gcc -no-pie test.S -g`
- `gcc -c test.S -o test.o -g && ld -e main test.o -o a.out`
- x86_64系统调用号：`/usr/include/x86_64-linux-gnu/asm/unistd_64.h`

```asm
str:
	.string "Hello World!\n"

// 必须有 .data，否则用 strace，会有 EFAULT (Bad address)
.data
buf:
	.zero 1024

.text
.globl main

main:
	mov $1, %rax  // write
	mov $1, %rdi  // stdout
	lea str(%rip), %rsi
	mov $13, %rdx
	syscall
	
	mov $0, %rax  // read
	mov $0, %rdi  // stdin
	lea buf(%rip), %rsi
	mov $5, %rdx
	syscall
	
	mov $1, %rax  // write
	mov $1, %rdi  // stdout
	mov buf(%rip), %rsi
	mov $5, %rdx
	syscall
	
	mov $60, %rax  // exit
	xor %rdi, %rdi
	syscall
```

