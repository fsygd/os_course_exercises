# lec 3 SPOC Discussion

## **提前准备**
（请在上课前完成）


 - 完成lec3的视频学习和提交对应的在线练习
 - git pull ucore_os_lab, v9_cpu, os_course_spoc_exercises  　in github repos。这样可以在本机上完成课堂练习。
 - 仔细观察自己使用的计算机的启动过程和linux/ucore操作系统运行后的情况。搜索“80386　开机　启动”
 - 了解控制流，异常控制流，函数调用,中断，异常(故障)，系统调用（陷阱）,切换，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在linux, ucore, v9-cpu中的os*.c中是如何具体体现的。
 - 思考为什么操作系统需要处理中断，异常，系统调用。这些是必须要有的吗？有哪些好处？有哪些不好的地方？
 - 了解在PC机上有啥中断和异常。搜索“80386　中断　异常”
 - 安装好ucore实验环境，能够编译运行lab8的answer
 - 了解Linux和ucore有哪些系统调用。搜索“linux 系统调用", 搜索lab8中的syscall关键字相关内容。在linux下执行命令: ```man syscalls```
 - 会使用linux中的命令:objdump，nm，file, strace，man, 了解这些命令的用途。
 - 了解如何OS是如何实现中断，异常，或系统调用的。会使用v9-cpu的dis,xc, xem命令（包括启动参数），分析v9-cpu中的os0.c, os2.c，了解与异常，中断，系统调用相关的os设计实现。阅读v9-cpu中的cpu.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
 - 在piazza上就lec3学习中不理解问题进行提问。

## 第三讲 启动、中断、异常和系统调用-思考题

## 3.1 BIOS
- x86中BIOS从磁盘读入的第一个扇区是是什么内容？为什么没有直接读入操作系统内核映像？

  主引导扇区，因为内核映像很大，且内核映像可能有很多不同的格式，无法用统一的指令来读取

- 比较UEFI和BIOS的区别。

  BIOS后续的改进需要兼容早期的版本，而UEFI的目标是提供一个统一的接口标准

## 3.2 系统启动流程

- x86中分区引导扇区的结束标志是什么？

  0x55AA

- x86中在UEFI中的可信启动有什么作用？

  保证启动的安全性，只有数字签名合法才能正常启动

## 3.3 中断、异常和系统调用比较
- 什么是中断、异常和系统调用？

  系统调用：应用程序主动向操作系统发出的服务请求

  异常：指令非法或者因其他原因导致指令执行失败

  中断：来自外部硬件设备的处理请求

- 中断、异常和系统调用的处理流程有什么异同？

  ##### 响应方式

  中断：异步

  异常：同步

  系统调用：异步或同步

  ##### 处理机制

  中断：持续，对应用程序透明

  异常：强制终止或者重新执行之前执行失败的指令

  系统调用：等待和持续

- 以ucore/rcore lab8的answer为例，ucore的系统调用有哪些？大致的功能分类有哪些？

  ```c
  static int (*syscalls[])(uint32_t arg[]) = {
  
        [SYS_exit]              sys_exit,
        [SYS_fork]              sys_fork,
        [SYS_wait]              sys_wait,
        [SYS_exec]              sys_exec,
        [SYS_yield]             sys_yield,
        [SYS_kill]              sys_kill,
        [SYS_getpid]            sys_getpid,
        [SYS_putc]              sys_putc,
        [SYS_pgdir]             sys_pgdir,
        [SYS_gettime]           sys_gettime,
        [SYS_lab6_set_priority] sys_lab6_set_priority,
        [SYS_sleep]             sys_sleep,
        [SYS_open]              sys_open,
        [SYS_close]             sys_close,
        [SYS_read]              sys_read,
        [SYS_write]             sys_write,
        [SYS_seek]              sys_seek,
        [SYS_fstat]             sys_fstat,
        [SYS_fsync]             sys_fsync,
        [SYS_getcwd]            sys_getcwd,
        [SYS_getdirentry]       sys_getdirentry,
        [SYS_dup]               sys_dup,
    };
  
  ```

  大致分为进程管理、文件系统和I/O


## 3.6 请分析函数调用和系统调用的区别
- 系统调用与函数调用的区别是什么？

  系统调用时会有特权级的切换以及对应堆栈的切换，带来了额外的开销

  常规的函数调用没有堆栈的切换

- 通过分析x86中函数调用规范以及`int`、`iret`、`call`和`ret`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？

  内核的内容需要保护，在系统调用时会有堆栈的切换，从用户态的堆栈切换到内核态的堆栈

