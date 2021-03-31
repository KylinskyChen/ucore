# 一、关注点

系统软件启动过程。

# 二、学习目标

bootloader 使用特定的机制来加载并运行操作系统这个软件。

- 能够切换到 x86 的保护模式并显示字符；
- 为启动操作系统 ucore 做准备；

- 读磁盘并加载 ELF 执行文件格式，并显示字符。

- 处理时钟中断和显示字符；

该部分实现的 bootloader 代码小于 512 字节，能放到硬盘的主引导扇区上。

## 2.1 计算机原理

- CPU 的编址与寻址：基于分段机制的内存管理；
- CPU 的中断机制；
- 外设：串口、并口、CGA，时钟，硬盘；

## 2.2 bootloader 软件

- 编译运行 bootloader 的过程；
- 调试 bootloader 的方法；
- PC 启动 bootloader 的过程；
- ELF 执行文件的格式和加载；
- 外设访问：读硬盘、在 CGA 上显示字符串；

## 2.3 ucore OS 软件

- 编译运行 ucore OS 的过程；
- ucore OS 的启动过程；
- 调试 ucore OS 的方法；
- 函数调用关系：在汇编级了解函数调用栈的结构和处理过程；
- 中断管理：与软件相关的中断处理；
- 外设管理：时钟。

# 三、关注点

通过静态分析代码来深入理解。

## 3.1 理解通过 make 生成执行文件的过程

make 执行时，一般只显示输出，使用以下命令显示 make 具体执行了哪些命令。

```bash
make "V="
```

### 3.1.1 操作系统镜像文件 ucore.img 生成的过程

### 3.1.2 符合系统规范的硬盘主引导扇区的特征

## 3.2 使用 qemu 执行并调试程序

### 3.2.1 从 CPU 加电后的第一条指令开始单步跟踪 BIOS 的执行

#### 3.2.1.1 gdb

`/home/moocos/moocos/ucore_lab/labcodes_answer/lab1_result/tools/lab1init` 文件中的指令。

```bash
file bin/kernel
target remote :1234
set architecture i8086
b *0x7c00
continue
x /2i $pc
```

`/home/moocos/moocos/ucore_lab/labcodes_answer/lab1_result/tools/moninit` 文件中的指令。

```bash
file bin/kernel
target remote :1234
break kern_init
```

使用如下指令可以进行 bootloader 的调试。

```bash
cd labcodes_answer/lab1_result/
make lab1-mon
```

##### 3.2.1.1.1 qemu 和 gdb 联调

qemu 和 gdb 之间使用网络端口 `1234` 进行通讯。

打开 qemu 进行模拟之后，执行 gdb 并输入以下指令可以连接 qemu，此时 qemu 会进入停止状态，等待响应 gdb 的命令。

```bash
target remote localhost:1234
```

我们需要 qemu 在一开始便进入等待模式，则我们不再使用 `make qemu` 开始系统的运行，而使用 `make debug` 来完成这项工作。这样 qemu 便不会在 gdb 尚未连接的时候擅自运行了。

##### 3.2.1.1.2 gdb 的地址断点

在 gdb 命令行中，使用以下指令便可以在指定 **内存地址** 设置断点，当 qemu 中的 cpu 执行到指定地址时，便会将控制权交给 gdb。

```bash
b *[地址]
```

##### 3.2.1.1.3 反汇编

有可能 gdb 无法正确获取当前 qemu 执行的汇编指令，通过如下配置可以在每次 gdb 命令行前强制反汇编当前的指令，在 gdb 命令行或配置文件中添加如下指令。

```bash
define hook-stop
x/i $pc
end
```

##### 3.2.1.1.4 单步调试

区别在于单步的跨度上。

- next ：单步到程序源代码的下一行，不进入函数；       
- nexti：单步一条机器指令，不进入函数；              
- step ：单步到下一个不同的源代码行（包括进入函数）；  
- stepi：单步一条机器指令；                        

### 3.2.2 在初始化位置 0x7C00 设实地址断点进行测试

### 3.2.3 从 0x7C00 开始跟踪代码，将单步跟踪反汇编的代码与 bootasm.S 和 bootasm.asm 比较

### 3.2.4 找一个 bootloader 或内核中的位置设断点并调试

## 3.3 分析 bootloader 进入保护模型的过程

BIOS 将通过读取硬盘主引导扇区到内存，并转跳到对应内存中的位置执行 bootloader。

bootloader 是如何完成从实模式进入保护模式的？

关注 `保护模式和分段机制` 和 `lab1/boot/bootasm.S` 源码。

### 3.3.1 开启 A20 的原因和方法

### 3.3.2 初始化 GDT 表的方法

### 3.3.3 使能并进入保护模式的方法

## 3.4 分析 bootloader 加载 ELF 格式的 OS 的过程

关注 `bootmain.c` 来了解 bootloader 如何加载 ELF 文件。

通过分析源码和使用 qemu 来运行调试 bootloader 和 OS。

### 3.4.1 bootloader 如何读取硬盘扇区

关注 `硬盘访问概述` 小节。

### 3.4.2 bootloader 如何加载 ELF 格式的 OS

关注 `ELF 执行文件格式概述` 小节。

## 3.5 实现函数调用堆栈跟踪函数

关注 `kdebug.c 中函数 print_stackframe 的实现`，通过函数 print_stackframe 来跟踪函数调用堆栈中记录的返回地址。

```bash
ebp:0x00007b28 eip:0x00100992 args:0x00010094 0x00010094 0x00007b58 0x00100096
    kern/debug/kdebug.c:305: print_stackframe+22
ebp:0x00007b38 eip:0x00100c79 args:0x00000000 0x00000000 0x00000000 0x00007ba8
    kern/debug/kmonitor.c:125: mon_backtrace+10
ebp:0x00007b58 eip:0x00100096 args:0x00000000 0x00007b80 0xffff0000 0x00007b84
    kern/init/init.c:48: grade_backtrace2+33
ebp:0x00007b78 eip:0x001000bf args:0x00000000 0xffff0000 0x00007ba4 0x00000029
    kern/init/init.c:53: grade_backtrace1+38
ebp:0x00007b98 eip:0x001000dd args:0x00000000 0x00100000 0xffff0000 0x0000001d
    kern/init/init.c:58: grade_backtrace0+23
ebp:0x00007bb8 eip:0x00100102 args:0x0010353c 0x00103520 0x00001308 0x00000000
    kern/init/init.c:63: grade_backtrace+34
ebp:0x00007be8 eip:0x00100059 args:0x00000000 0x00000000 0x00000000 0x00007c53
    kern/init/init.c:28: kern_init+88
ebp:0x00007bf8 eip:0x00007d73 args:0xc031fcfa 0xc08ed88e 0x64e4d08e 0xfa7502a8
<unknow>: -- 0x00007d72 –
```

解释最后一行各个数值的含义。

关注 `函数堆栈` 小节，了解编译器如何建立函数调用关系的。

关注 `lab1/obj/bootblock.asm`，了解 bootloader 源码与机器码的语句和地址等的对应关系。

关注 `lab1/obj/kernel.asm`，了解 ucore OS 源码与机器码的语句和地址等的对应关系。

## 3.6 关注中断初始化和处理的方法

关注小节 `中断与异常`。

### 3.6.1 中断描述符表的大小和中断处理代码的入口

中断描述符表（也可简称为保护模式下的中断向量表）中一个表项占多少字节？

其中哪几位代表中断处理代码的入口？

### 3.6.2 中断初始化

`kern/trap/trap.c` 中的`idt_init` 函数用于对中断向量表进行初始化，依次对所有中断入口进行初始化。

使用 `mmu.h` 中的 `SETGATE` 宏，填充 idt 数组内容。

每个中断的入口由 `tools/vectors.c` 生成，使用 `trap.c` 中声明的 `vectors` 数组即可。

### 3.6.3 中断处理函数 trap

`trap.c` 中的中断处理函数 `trap`，使操作系统每遇到 100 次时钟中断后，调用 `print_ticks` 子程序，向屏幕上打印一行文字 `100 ticks`。

除了系统调用中断 `T_SYSCALL` 使用陷阱门描述符且权限为用户态权限以外，其它中断均使用特权级 `DPL` 为０的中断门描述符，权限为内核态权限。

ucore 的应用程序处于特权级 ３，需要采用 `int 0x80` 指令操作（这种方式称为软中断，软件中断，Tra 中断）来发出系统调用请求，并要能实现从特权级 ３ 到特权级 ０ 的转换，所以系统调用中断 `T_SYSCALL` 所对应的中断门描述符中的特权级 `DPL` 需要设置为 ３。

## 3.7 模式切换

实现在 `mooc_os_lab` 中的 `mooc_os_2014` branch 中的 `labcodes_answer/lab1_result` 目录下。

### 3.7.1 实现用户态函数

[扩展练习 Challenge 1（需要编程）](https://objectkuan.gitbooks.io/ucore-docs/content/lab1/lab1_2_1_7_ex7.html)

增加 syscall 功能，即增加一用户态函数（可执行一特定系统调用：获得时钟计数值），当内核初始完毕后，可从内核态返回到用户态的函数，而用户态的函数又通过系统调用得到内核态的服务。

### 3.7.2 用键盘实现用户模式内核模式切换

[扩展练习 Challenge 2（需要编程）](https://objectkuan.gitbooks.io/ucore-docs/content/lab1/lab1_2_1_7_ex7.html)

键盘输入 3 时切换到用户模式，键盘输入 0 时切换到内核模式。

# 四、项目组成

```bash
.
├── boot
│   ├── asm.h
│   ├── bootasm.S
│   └── bootmain.c
├── kern
│   ├── debug
│   │   ├── assert.h
│   │   ├── kdebug.c
│   │   ├── kdebug.h
│   │   ├── kmonitor.c
│   │   ├── kmonitor.h
│   │   ├── panic.c
│   │   └── stab.h
│   ├── driver
│   │   ├── clock.c
│   │   ├── clock.h
│   │   ├── console.c
│   │   ├── console.h
│   │   ├── intr.c
│   │   ├── intr.h
│   │   ├── kbdreg.h
│   │   ├── picirq.c
│   │   └── picirq.h
│   ├── init
│   │   └── init.c
│   ├── libs
│   │   ├── readline.c
│   │   └── stdio.c
│   ├── mm
│   │   ├── memlayout.h
│   │   ├── mmu.h
│   │   ├── pmm.c
│   │   └── pmm.h
│   └── trap
│       ├── trap.c
│       ├── trapentry.S
│       ├── trap.h
│       └── vectors.S
├── libs
│   ├── defs.h
│   ├── elf.h
│   ├── error.h
│   ├── printfmt.c
│   ├── stdarg.h
│   ├── stdio.h
│   ├── string.c
│   ├── string.h
│   └── x86.h
├── Makefile
└── tools
    ├── function.mk
    ├── gdbinit
    ├── grade.sh
    ├── kernel.ld
    ├── sign.c
    └── vector.c

10 directories, 48 files
```

## 4.1 bootloader 

- boot/bootasm.S
    - 定义并实现了 bootloader 最先执行的函数 start；
    - start 函数进行了一定的初始化，完成了从实模式到保护模式的转换；
    - 调用 bootmain.c 中的 bootmain 函数；

- boot/bootmain.c
    - 定义并实现了 bootmain 函数实现了通过屏幕、串口和并口显示字符串；
    - bootmain 函数加载 ucore 操作系统到内存，然后跳转到 ucore 的入口处执行；

- boot/asm.h
    - 是 bootasm.S 汇编文件所需要的头文件；
    - 主要是一些与 X86 保护模式的段访问方式相关的宏定义；

## 4.2 ucore

### 4.2.1 系统初始化

- kern/init/init.c
    - ucore 操作系统的初始化启动代码；

### 4.2.2 内存管理

- kern/mm/memlayout.h
    - ucore 操作系统有关段管理（段描述符编号、段号等）的一些宏定义；

- kern/mm/mmu.h
    - ucore 操作系统有关 X86 MMU 等硬件相关的定义；
        - EFLAGS 寄存器中各位的含义；
        - 应用、系统段类型；
        - 中断门描述符定义；
        - 段描述符定义；
        - 任务状态段定义；
        - NULL 段声明的宏 SEG_NULL；
        - 特定段声明的宏 SEG；
        - 设置中断门描述符的宏 SETGATE；

- kern/mm/pmm.[ch]
    - 设定了 ucore 操作系统在段机制中要用到的全局变量；
        - 任务状态段 ts；
        - 全局描述符表 gdt[]；
        - 加载全局描述符表寄存器的函数 lgdt；
        - 临时的内核栈stack0；
        - 对全局描述符表和任务状态段的初始化函数 gdt_init；

### 4.2.3 外设驱动

- kern/driver/intr.[ch]
    - 实现了通过设置 CPU 的 eflags 来屏蔽和使能中断的函数；

- kern/driver/picirq.[ch]
    - 实现了对中断控制器 8259A 的初始化和使能操作；

- kern/driver/clock.[ch]
    - 实现了对时钟控制器 8253 的初始化操作；

- kern/driver/console.[ch]
    - 实现了对串口和键盘的中断方式的处理操作；

### 4.2.4 中断处理

- kern/trap/vectors.S
    - 包括 256 个中断服务例程的入口地址和第一步初步处理实现；
    - 注意，此文件是由 tools/vector.c 在编译 ucore 期间动态生成的；

- kern/trap/trapentry.S
    - 紧接着第一步初步处理后，进一步完成第二步初步处理；
    - 并且有恢复中断上下文的处理，即中断处理完毕后的返回准备工作；

- kern/trap/trap.[ch]
    - 紧接着第二步初步处理后，继续完成具体的各种中断处理操作；

### 4.2.5 内核调试

- kern/debug/kdebug.[ch]
    - 提供源码和二进制对应关系的查询功能；
    - 用于显示调用栈关系；
    - 其中补全 print_stackframe 重点关注，其他实现部分目前不必深究；

- kern/debug/kmonitor.[ch]
    - 实现提供动态分析命令的 kernel monitor，便于在 ucore 出现 bug 或问题后，能够进入kernel monitor 中，查看当前调用关系；
    - 实现部分不必深究；

- kern/debug/panic.c | assert.h
    - 提供了 panic 函数和 assert 宏；
    - 便于在发现错误后，调用 kernel monitor；
    - 可在编程实验中充分利用 assert 宏和 panic 函数，提高查找错误的效率；

### 4.2.6 公共库

- libs/defs.h
    - 包含一些无符号整型的缩写定义；

- Libs/x86.h
    - 一些用 GNU C 嵌入式汇编实现的 C 函数（由于使用了 inline 关键字，所以可以理解为宏）；

### 4.2.7 工具

- Makefile | function.mk
    - 指导 make 完成整个软件项目的编译，清除等工作；

- sign.c
    - 一个 C 语言小程序，是辅助工具，用于生成一个符合规范的硬盘主引导扇区；

- tools/vector.c
    - 生成 vectors.S，此文件包含了中断向量处理的统一实现；

### 4.2.8 make 生成的文件

- bin/ucore.img
    - 一个包含了 bootloader 或 OS 的硬盘镜像；
