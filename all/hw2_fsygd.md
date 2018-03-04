## 思考题

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？

  硬件设计上需要能执行特权指令，只有CPU在特权态时才可执行的指令

  处理异常需要中断/异常/系统服务等管理指令

  虚存需要TLB/MMU等管理指令

  特权模式需要调整特权级管理指令

  段页式需要分段分页管理指令

- 你理解的x86的实模式和保护模式有什么区别？物理地址、线性地址、逻辑地址的含义分别是什么？

  实模式只有16位的寻址空间，且没有保护机制，保护模式有32位的寻址空间

  实模式：80386加电启动后处于实模式运行状态，在这种状态下软件可访问的物理内存空间不能超过1MB，且无法发挥Intel 80386以上级别的32位CPU的4GB内存管理能力。

  保护模式：支持内存分页机制，提供了对虚拟内存的良好支持。保护模式下80386支持多任务，还支持优先级机制，不同的程序可以运行在不同的优先级上。优先级一共分0~3 4个级别，操作系统运行在最高的优先级0上，应用程序则运行在比较低的级别上，配合良好的检查机制后，既可以在任务间实现数据的安全共享也可以很好地隔离各个任务。



​        物理地址：处理器提交到总线上用于访问计算机系统中的内存和外设的最终地址

​        线性地址：操作系统的虚存管理下每个运行的应用程序能访问的地址空间。每个运行的应用程序都认为自己独享整个计算机系统的地址空间，这样可以让多个运行的应用程序之间相互隔离

​        逻辑地址：逻辑地址空间是应用程序直接使用的地址空间

- 理解list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）
- 对于如下的代码段，请说明":"后面的数字是什么含义

```
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
 };

```

  “：”后面的数字表示无符号数的位数

- 对于如下的代码段，

```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}

```

如果在其他代码段中有如下语句，

```
unsigned intr;
intr=8;
SETGATE(intr, 1,2,3,0);

```

请问执行上述指令后， intr的值是多少？

intr的值为0x20003，前16位为3，后16位为2，采用小端

### 课堂实践练习

#### 练习一

请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

- [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)

- ##### [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)

- ##### [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)

- ##### [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)

- ##### [[IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)]

  ```c
  static inline void
  lgdt(struct pseudodesc *pd) {
  asm volatile ("lgdt (%0)" :: "r" (pd));
  asm volatile ("movw %%ax, %%gs" :: "a" (USER_DS));
  asm volatile ("movw %%ax, %%fs" :: "a" (USER_DS));
  asm volatile ("movw %%ax, %%es" :: "a" (KERNEL_DS));
  asm volatile ("movw %%ax, %%ds" :: "a" (KERNEL_DS));
  asm volatile ("movw %%ax, %%ss" :: "a" (KERNEL_DS));
  // reload cs
  asm volatile ("ljmp %0, $1f\n 1:\n" :: "i" (KERNEL_CS));
  }
  ```

  首先加载全局描述符到pd中，用USER_DS的值初始化gs和fs，用KERNEL_DS的值初始化es、ds和ss，长跳转到基址为KERNEL_CS，偏移量为标号1的地方。

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore中宏定义的用途，并举例描述其含义。

> 利用宏进行复杂数据结构中的数据访问； 利用宏进行数据类型转换；如 to_struct, 常用功能的代码片段优化；如 ROUNDDOWN, SetPageDirty