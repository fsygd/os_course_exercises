# lec 3 SPOC Discussion

## 第三讲 启动、中断、异常和系统调用-思考题

## 3.1 BIOS

- BIOS从磁盘读入的第一个扇区是是什么内容？为什么没有直接读入操作系统内核映像？

  第一个扇区是Boot Loader，因为磁盘上有多种多样的文件系统，机器出厂的时候并不能限制只能只用某种文件系统，BIOS也不能加上能识别所有文件系统的代码，识别文件系统的工作交给加载程序

- 比较UEFI和BIOS的区别。

  BIOS会通过后期的修改完善对后续的支持，但这种支持总是会受到前边的制约。UEFI提供所有平台一致的操作系统启动服务。

## 3.2 系统启动流程

- 分区引导扇区的结束标志是什么？

  0x55AA

- 在UEFI中的可信启动有什么作用？

  BIOS启动以后在读磁盘上的引导记录的时候，会对引导记录的可信性进行一个检查，只有满足签名的引导记录才会读进来

## 3.3 中断、异常和系统调用比较

- 什么是中断、异常和系统调用？

  系统调用：应用程序主动向操作系统发出的服务请求

  异常：非法指令或者其他原因导致当前指令执行失败

  中断：来自硬件设备的处理请求

- 中断、异常和系统调用的处理流程有什么异同？

  #####源头

  中断：外设

  异常：应用程序意想不到的行为

  系统调用：应用程序请求操作提供服务

  ##### 响应方式

  中断：异步

  异常：同步

  系统调用：异步或同步

  ##### 处理机制

  中断：持续，对用户应用程序是透明的

  异常：杀死或者重新执行意想不到的应用程序指令

  系统调用：等待和持续

  ​

- 以ucore lab8的answer为例，uCore的系统调用有哪些？大致的功能分类有哪些？

```

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

大致分为进程管理，I/O，文件系统

## 3.6 请分析函数调用和系统调用的区别

- 系统调用与函数调用的区别是什么？

  系统调用

  ​	INT和IRET指令用于系统调用

  ​	系统调用时，堆栈切换和特权级的切换

  ​	系统调用更安全，但开销也会变大，需要有切换的引导机制，可能需要建立内核堆栈，对传递的参数会有合法性上的安全验证，内核态访问用户态的地址空间需要有地址映射，这个时候页表、TLB可能会失效

  函数调用

  ​	CALL和RET用于常规调用

  ​	常规调用时没有堆栈切换

- 通过分析`int`、`iret`、`call`和`ret`的指令准确功能和调用代码，比较函数调用与系统调用的堆栈操作有什么不同？

  由于内核受保护，内核和应用程序使用不同的堆栈，INT和IRET用于系统调用时会有堆栈的切换，特权级也进行了切换，这时我们就能对设备进行控制了