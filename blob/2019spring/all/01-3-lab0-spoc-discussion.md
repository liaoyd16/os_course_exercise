# lec2：lab0 SPOC思考题

## **提前准备**
（请在上课前完成，option）

- 完成lec2的视频学习
- git pull ucore_os_lab, os_tutorial_lab, os_course_exercises  in github repos。这样可以在本机上完成课堂练习。
- 了解代码段，数据段，执行文件，执行文件格式，堆，栈，控制流，函数调用,函数参数传递，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在不同操作系统（如linux, ucore,etc.)与不同硬件（如 x86, riscv, v9-cpu,etc.)中是如何相互配合来体现的。
- 安装好ucore实验环境，能够编译运行ucore labs中的源码。
- 会使用linux中的shell命令:objdump，nm，file, strace，gdb等，了解这些命令的用途。
- 会编译，运行，使用v9-cpu的dis,xc, xem命令（包括启动参数），阅读v9-cpu中的v9\-computer.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
- 了解基于v9-cpu的执行文件的格式和内容，以及它是如何加载到v9-cpu的内存中的。
- 在piazza上就学习中不理解问题进行提问。

---

## 思考题

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？

> 中央处理器，存储设备，输出输出设备等以及其中的总线；
>
> 存取特殊寄存器、进行页从磁盘的载入、仲裁中断、执行IO、中止处理器流水等。

- 你理解的x86的实模式和保护模式有什么区别？物理地址、线性地址、逻辑地址的含义分别是什么？

> 实模式和保护模式的区别主要在于访存的空间大小和访存方式上：实模式下，x86仅能够访问2^20个地址，而保护模式下x86能够访问全部2^32的地址空间；
> 另一方面，保护模式下的访存要通过虚实地址转换，从而保护了系统级内存不被应用级程序篡改。
>
> 物理地址指的是，插在卡槽内的存储卡中的地址，即CPU地址总线传来的地址
> 线性地址是段中的偏移地址加上段对应的基地址得到的地址，需要经过页表查询到对应的页，并在页加载到主存中后对应到主存中的物理地址
> 逻辑地址是程序员直接交互的地址，即访存指令给出的地址；逻辑地址分为段标志和段内偏移两部分

- 你理解的risc-v的特权模式有什么区别？不同模式在地址访问方面有何特征？

> M-模式的访存方式是简单的嵌入式系统的访存方式，没有地址转换，而系统地址和应用、数据地址不加区别；
>
> U-模式需要和M-模式同时启用，同样没有地址转换；不过用户级程序运行在U-模式下而系统级运行在M-模式下
>
> S-模式需要和前两者同时使用，此时运行在M模式以外的程序缺省地都没有访问内存的权限。通过PMP-region的配置，可以令程序对任意由一系列2^k的内存块组合成的内存获得访存权限。

- 理解ucore中list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）

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

> 指的是该变量所占的空间为n位

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

> 组合为如下程序：
>
> ```C
> #include <stdint.h>
> 
> #define STS_IG32 0xE
> #define STS_TG32 0xF
> 
> #define SETGATE(gate, istrap, sel, off, dpl) {            \
>    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
>    (gate).gd_ss = (sel);                                \
>    (gate).gd_args = 0;                                    \
>    (gate).gd_rsv1 = 0;                                    \
>    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
>    (gate).gd_s = 0;                                    \
>    (gate).gd_dpl = (dpl);                                \
>    (gate).gd_p = 1;                                    \
>    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
> }
> 
> /* Gate descriptors for interrupts and traps */
> struct gatedesc {
>    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
>    unsigned gd_ss : 16;            // segment selector
>    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
>    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
>    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
>    unsigned gd_s : 1;                // must be 0 (system)
>    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
>    unsigned gd_p : 1;                // Present
>    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
> };
> 
> int main(void) {
>     unsigned intr;
>     intr=8;
>     SETGATE(*(struct gatedesc*)(&intr), 1,2,3,0);
>     printf("%u", intr);
> }
> ```
>
> gcc编译、运行结果为：131075

### 课堂实践练习

#### 练习一

1. 请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

2. (option)请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。

#### reference
 - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
 - [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
 - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
 - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
 - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)
