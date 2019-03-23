# Embedded-Programming-with-the-GNU-Toolchain
## Vijay Kumar B.
<vijaykumar@bravegnu.org>

# 目录
[1.介绍](#1介绍)

[2.建立ARM实验室](#2建立arm实验室)
* [2.1.Qemu ARM](#21qemu-arm)
* [2.2.在Debian中安装Qemu](#22在debian中安装qemu)
* [2.3.安装ARM GNU工具链](#23安装arm-gnu工具链)

[3.Hello ARM](#3hello-arm)
* [3.1.构建二进制文件](#31构建二进制文件)
* [3.2.在Qemu里面执行](#32在qemu里面执行)
* [3.3.更多的监视器命令](#33更多的监视器命令)

[4.更多的汇编器指令](#4更多的汇编器指令)
* [4.1.数组求和](#41数组求和)
	* [4.1.1. .byte指令](#411-byte指令)
	* [4.1.2. .align指令](#412-align指令)
* [4.2.字符串长度](#42字符串长度)
	* [4.2.1. .asciz 指令](#421-asciz指令)
	* [4.1.2. .align指令](#412-align指令)
	
[5.使用RAM](#5使用ram)

[6.链接器](#6链接器)
* [6.1符号解析](#61符号解析)

# 1.介绍
GNU工具链越来越多地用于深度嵌入式软件开发。这种类型的软件开发也称为独立C语言编程和裸机C语言编程。独立的C语言编程带来了新的问题，处理这些问题需要对GNU工具链有更深入的理解。GNU工具链的手册提供了关于工具链的优秀信息，但是是从工具链的角度，而不是从问题的角度。不管怎样，手册就是这样写的。其结果是对常见问题的答案分散在各地，GNU工具链的新用户感到困惑。

本教程试图通过从问题的角度解释这些工具来弥补这一差距。希望这能使更多的人在他们的嵌入式项目中使用GNU工具链。

本教程使用Qemu模拟了一个基于ARM的嵌入式系统。有了它，您可以从舒适的桌面环境中学习GNU工具链，而无需在硬件上进行投资。本教程本身并不教授ARM指令集。它应该与其他书籍和在线教程一起使用，比如:
* [ARM Assembler]( http://www.heyrick.co.uk/assembler/)
* [ARM Assembly Language Programming](http://www.arm.com/miscPDFs/9658.pdf)

但是为了方便读者，附录中列出了常用的ARM指令。

# 2.建立ARM实验室
本节展示如何使用Qemu和GNU工具链在您的PC中设置一个简单的ARM开发和测试环境。Qemu是一个机器模拟器，能够模拟各种机器，包括基于ARM的机器。您可以编写ARM汇编程序，使用GNU工具链编译它们，并在Qemu中执行和测试它们。
## 2.1.Qemu ARM
Qemu将用于模拟Gumstix基于PXA255的connex板。您应该至少拥有0.9.1版本的Qemu来使用本教程。

PXA255有一个ARMv5TE指令集的ARM内核。PXA255也有几个片上外设。本教程将介绍一些外围设备。
## 2.2.在Debian中安装Qemu
本教程要求qemu版本0.9.1或更高。Debian Squeeze/Wheezy中提供的qemu包满足这一要求。使用`apt-get`安装`qemu`。

	$ apt-get install qemu

## 2.3.安装ARM GNU工具链

1. CodeSourcery (Mentor Graphics的一部分)提供了可用于各种体系架构的GNU工具链。下载用于ARM的GNU工具链，可从：
	http://www.mentor.com/embedded-software/sourcery-tools/sourcery-codebench/editions/lite-edition/

2. 解压至`~/toolchains`。

    	$ mkdir ~/toolchains
    	$ cd ~/toolchains
    	$ tar -jxf ~/downloads/arm-2008q1-126-arm-none-eabi-i686-pc-linux-gnu.tar.bz2


3. 工具链路径添加至环境变量`PATH`

		$ PATH=$HOME/toolchains/arm-2008q1/bin:$PATH

4. 在`.bashrc`中添加并导出`PATH`。

# 3.Hello ARM
在本节中，您将学习如何汇编一个简单的ARM程序，并用Qemu模拟connex裸机板进行测试。

汇编程序源文件由一系列语句组成，每行一个。每个语句都具有以下格式。
```asm
label:    instruction         @ comment
```

每个部分都是可选的。

**label:**

* 标签是一种方便的方法来引用指令在内存中的位置。标签可以用于任何地址出现的地方，例如作为分支指令的操作数（`b label`）。标签由字母，数字，_和$组成。

**comment:**
* 注释以@开头，在@号之后出现的字符会被编译器忽略。

**instruction:**
* 指令可以是ARM指令集里面的指令或者汇编器的指令。汇编器的指令是给汇编器的命令。汇编器指令由`.`号打头。

下面是一个非常简单的ARM汇编程序，实现2个数相加。
**Listing 1. Adding Two Numbers**
```asm
        .text
start:                       @ Label, not really required
        mov   r0, #5         @ Load register r0 with the value 5
        mov   r1, #4         @ Load register r1 with the value 4
        add   r2, r1, r0     @ Add r0 and r1 and store in r2

stop:   b stop               @ Infinite loop to stop execution

```
`.text`是一个汇编器指令，是说接下来的指令必须汇编到代码段（`code section`）,而不是数据段（`data section`）。`段(sections)`这个概念会在后面的教程中详细介绍。

## 3.1.构建二进制文件
将程序保存至文件`add.s`中。要汇编此文件，需要调用GNU工具链的汇编器`as`,命令如下。

	$ arm-none-eabi-as -o add.o add.s

-o选项指定了输出文件的名字。
>![](https://github.com/bravegnu/gnu-eprog/blob/master/images/note.png)
>注意：交叉工具链总是以构建它们的目标体系结构为前缀，以避免与主机工具链的名称冲突。为了可读性，我们将在文本中引用不带前缀的工具。

要生成可执行文件，需要调用GNU工具链的连接器`ld`，命令如下。

	$ arm-none-eabi-ld -Ttext=0x0 -o add.elf add.o

这里，-o选项再次指定了输出文件名。`-Ttext=0x0`,指定了分配给标签的地址，以便指令从地址0x0开始。可以通过`nm`命令查看各标签的地址分配情况。

	$ arm-none-eabi-nm add.elf 
	00008010 T __bss_end__
	00008010 T _bss_end__
	00008010 T __bss_start
	00008010 T __bss_start__
	00008010 T _edata
	00008010 T _end
	00008010 T __end__
	00000000 t start
	         U _start
	0000000c t stop

现在只关注标签`start`和`stop`。注意标签`start`和`stop`的地址分配。分配给`start`的地址是`0x0`。因为它是第一条指令的标签。标签`stop`在3条指令之后。每条指令占4个字节。因此`stop`分配的地址是`12(0xC)`。

为指令链接不同的基址将导致为标签分配一组不同的地址。

	$ arm-none-linux-gnueabi-ld -Ttext=0x20000000 add.o -o add.elf 
	$ arm-none-linux-gnueabi-nm add.elf
	20008010 T __bss_end__
	20008010 T _bss_end__
	20008010 T __bss_start
	20008010 T __bss_start__
	20008010 T _edata
	20008010 T _end
	20008010 T __end__
	20000000 t start
	         U _start
	2000000c t stop

`ld`生成的输出文件格式是`ELF`。有多种文件格式可用于存储可执行代码。`ELF`格式在有操作系统的情况下工作得很好，但是由于我们要在裸机上运行程序，所以必须将其转换为更简单的文件格式，称为`二进制格式`。

二进制格式的文件包含特定内存地址的连续字节。该文件中不存储其他附加信息。这对于Flash编程工具来说很方便，因为编程时所要做的就是将文件中的每个字节复制到从内存中指定的基址开始的连续地址。

GNU工具链的`objcopy`命令能用于转换不同的目标文件格式。常见的用法如下。

	objcopy -O <output-format> <in-file> <out-file>

下面的命令用于将add.elf转换为二进制格式。

	$ arm-none-eabi-objcopy -O binary add.elf add.bin

检查文件的大小。该文件将恰好是16个字节。因为有4条指令，每条指令占用4个字节。

	$ ls -la add.bin 
	-rwxrwxr-x 1 thomas thomas 16 3月  23 17:56 add.bin

## 3.2.在Qemu里面执行
当ARM处理器复位时，它就从`0x0`地址开始执行。在`connex板`上有一个16MB的Flash位于地址`0x0`。在Flash开始处的指令将被执行。

当用`qemu`仿真connex板时，必须指定将哪个文件当作Flash使用。Flash文件给谁非常简单。为了从Flash中的地址X获取字节，`qemu`从文件中的偏移量X读取字节。事实上，这与二进制文件格式相同。

为了测试这个程序，我们首先在模拟的`Gumstix connex`板上创建一个16MB的文件来表示Flash。我们使用`dd`命令将16MB的0从`/dev/zero`复制到文件`flash.bin`。数据以4K块的形式复制。

	$ dd if=/dev/zero of=flash.bin bs=4096 count=4096

然后使用以下命令将`add.bin`文件复制到Flash的开头。

	$ dd if=add.bin of=flash.bin bs=4096 conv=notrunc

这相当于将`bin`文件烧录到`Flash`上。

复位之后，处理器将从地址`0x0`开始执行，程序中的指令将被执行。下面给出了调用qemu的命令。

	$ qemu-system-arm -M connex -pflash flash.bin -nographic -serial /dev/null

`-M connex`选项指定要模拟的机器是`connex`。`-pflash`选项指定`Flash .bin`文件表示闪存。`-nographic`指定不需要模拟图形显示。`-serial /dev/null`指定将connex板的串口连接到`/dev/null`，以便丢弃串口数据。

系统执行指令，完成后，在`stop: b stop`指令中无限循环。要查看寄存器的内容，可以使用`qemu`的监视器接口。监控接口是一个命令行接口，通过它可以控制仿真系统并查看系统状态。当`qemu`使用上述命令启动时，监视器接口在`qemu`的标准I/O中提供(就是指可以以敲命令的方式交互)。

要查看寄存器的内容，可以使用`info register`命令。

	(qemu) info registers
	R00=00000005 R01=00000004 R02=00000009 R03=00000000
	R04=00000000 R05=00000000 R06=00000000 R07=00000000
	R08=00000000 R09=00000000 R10=00000000 R11=00000000
	R12=00000000 R13=00000000 R14=00000000 R15=0000000c
	PSR=400001d3 -Z-- A svc32

注意寄存器R02中的值。寄存器包含加法的结果，并且应该与期望值9匹配。

## 3.3.更多的监视器命令
表列出了一些有用的qemu监视器命令。

命令|用途
:--:|:--:
help|列出可用的命令
quit|退出模拟器
xp /fmt addr|从addr转储物理内存
system_reset|复位系统

详细解释下`xp`命令。`fmt`参数指定如何显示内存内容。`fmt`的语法是`<count><format><size>`。
**count**
* 指定count个要转储的数据项。

**size**
* 指定每个数据项的大小。`b`代表8位，`h`代表16位，`w`代表32位，`g`代表64位。

**format**
* 指定显示格式。`x`表示十六进制，`d`表示有符号十进制数，`u`表示无符号十进制数，`o`表示八进制，`c`表示char, `i`表示asm指令。

使用带有`i`格式的`xp`命令，可以用来反汇编内存中的指令。要反汇编位于`0x0`的指令，可以使用`xp`命令，将`fmt`指定为`4iw`。`4`指定要显示4项，`i`指定要打印的项作为指令(相当于一个内置的反汇编器!)，`w`指定这些项的大小为32位。命令的输出如下所示。

	(qemu) xp /4iw 0x0
	0x00000000:  mov        r0, #5  ; 0x5
	0x00000004:  mov        r1, #4  ; 0x4
	0x00000008:  add        r2, r1, r0
	0x0000000c:  b  0xc

# 4.更多的汇编器指令
在此章节，我们通过2个示例程序介绍常用的汇编器指令。
1. 对数组求和的程序
2. 计算字符串长度的程序

## 4.1.数组求和
下面的代码对一个数组求和，并将结果存储在r3中。
**Listing 2. Sum an Array**
```asm
        .text
entry:  b start                 @ Skip over the data
arr:    .byte 10, 20, 25        @ Read-only array of bytes
eoa:                            @ Address of end of array + 1

        .align
start:
        ldr   r0, =eoa          @ r0 = &eoa
        ldr   r1, =arr          @ r1 = &arr
        mov   r3, #0            @ r3 = 0
loop:   ldrb  r2, [r1], #1      @ r2 = *r1++
        add   r3, r2, r3        @ r3 += r2
        cmp   r1, r0            @ if (r1 != r2)
        bne   loop              @    goto loop
stop:   b stop
```
该代码引入了两个新的汇编器指令--`.byte`和`.align`。下面将描述这些汇编器指令。

### 4.1.1. .byte指令
`.byte`的字节大小参数被汇编成内存中的连续字节。对于存储16位值和32位值，有类似的`.2byte`和`.4byte`指令。一般语法如下所示。

	.byte   exp1, exp2, ...
	.2byte  exp1, exp2, ...
	.4byte  exp1, exp2, ...

参数可以是简单的整数，表示为二进制(前缀为`0b`或`0B`)、八进制(前缀为`0`)、十进制(无前缀，不要以`0`开头，避免被当做八进制处理)或十六进制(前缀为`0x`或`0X`)。整数也可以表示为字符常量(由单引号包围的字符)，在这种情况下将使用字符的ASCII值。

参数也可以是由文字和其他符号构造的C表达式。示例如下所示。

	pattern:  .byte 0b01010101, 0b00110011, 0b00001111
	npattern: .byte npattern - pattern
	halpha:   .byte 'A', 'B', 'C', 'D', 'E', 'F'
	dummy:    .4byte 0xDEADBEEF
	nalpha:   .byte 'Z' - 'A' + 1

### 4.1.2. .align指令
ARM要求指令出现在32位对齐的内存位置。指令中4个字节中的第一个字节的地址应该是4的倍数。要做到这一点，可以使用`.align`指令插入填充字节，直到下一个字节地址是4的倍数。只有当在代码中插入数据字节或半字时才需要这样做。
## 4.2.字符串长度
下面的代码计算字符串的长度，并将长度存储在寄存器r1中。
**Listing 3. String Length**
```asm
        .text
        b start

str:    .asciz "Hello World"

        .equ   nul, 0

        .align
start:  ldr   r0, =str          @ r0 = &str
        mov   r1, #0

loop:   ldrb  r2, [r0], #1      @ r2 = *(r0++)
        add   r1, r1, #1        @ r1 += 1
        cmp   r2, #nul          @ if (r1 != nul)
        bne   loop              @    goto loop

        sub   r1, r1, #1        @ r1 -= 1
stop:   b stop
```
该代码引入了两个新的汇编器指令- `.asciz`和`.equ`。下面描述汇编器指令。

### 4.2.1. .asciz指令
`.asciz`指令接受字符串文本作为参数。字符串文字是双引号中的字符序列。字符串文字被汇编成连续的内存位置。汇编器在每个字符串后面自动插入一个`nul`字符(\0字符)。

`'ascii`指令与`.asciz`相同，但是汇编器不会在每个字符串后面插入`nul`字符。
### 4.2.2. .equ指令
汇编器维护称为符号表的东西。符号表将标签名称映射到地址。每当汇编器遇到标签定义时，汇编器在符号表中做一个入口点。每当汇编器遇到标签引用时，它就用符号表中对应地址替换标签。

使用汇编器的指令`.equ`，还可以手动插入符号表中的条目，将名称映射到不一定是地址的值。每当汇编器遇到这些名称时，它就用它们对应的值替换它们。这些名称和标签名称一起称为符号名。

该指令的一般语法如下所示。
```asm
.equ name, expression
```
`name`是一个符号名称，并且具有与标签名称相同的限制。`expression`可以是简单的字符，也可以是`.byte`指令解释的表达式。
>![](https://github.com/bravegnu/gnu-eprog/blob/master/images/note.png)
>注意：
>与`.byte`指令不同，`.equ`指令本身不分配任何内存。它们只是在符号表中创建条目。

# 5.使用RAM
存储前面示例程序的闪存是一种EEPROM。它是一个有用的辅助存储，就像硬盘一样，但是不方便在Flash中存储变量。变量应该存储在RAM中，这样就可以很容易地修改它们。

connex板有一个64 MB的RAM，从地址`0xA0000000`开始，其中可以存储变量。connex板的内存映射如下图所示。

**Figure 1. Memory Map**

![](http://www.bravegnu.org/gnu-eprog/flash-ram-mm.png)

必须进行必要的设置才能将变量放在这个地址。要理解必须做什么设置，必须理解汇编器和链接器的角色。

# 6.链接器
当编写一个多源文件的程序时，每个文件被单独汇编为目标文件。链接器将这些目标文件组合起来形成最终的可执行文件。

**Figure 2. Role of the Linker**

![](http://www.bravegnu.org/gnu-eprog/linker.png)

当组合目标文件在一起时，链接器执行了如下操作。
1. 符号解析
2. 重定位

在本节中，我们将详细研究这些操作。
## 6.1符号解析
在单文件程序中，在生成目标文件时，所有对标签的引用都由汇编器用它们的对应地址替换。但在多文件程序中，如果有对另一个文件中定义的标签的任何引用，则汇编器将这些引用标记为“未解析(unresolved)”。当这些目标文件传递给链接器时，链接器将从其他目标文件确定这些引用的值，并使用正确的值对代码进行调整（patch）。

`sum of array`示例被分成两个文件，以演示链接器执行符号解析。这两个文件将被汇编起来，并检查它们的符号表，以显示未解析引用的存在。

文件`sum-sub.s`包含`sum`子程序，文件`main.s`传入所需的参数调用子程序。这些文件的源代码如下所示。

**Listing 4. main.s - Subroutine Invocation**
```asm
        .text
        b start                 @ Skip over the data
arr:    .byte 10, 20, 25        @ Read-only array of bytes
eoa:                            @ Address of end of array + 1

        .align
start:
        ldr   r0, =arr          @ r0 = &arr
        ldr   r1, =eoa          @ r1 = &eoa

        bl    sum               @ Invoke the sum subroutine

stop:   b stop
```
**Listing 5. sum-sub.s - Subroutine Definition**
```asm
        @ Args
        @ r0: Start address of array
        @ r1: End address of array
        @
        @ Result
        @ r3: Sum of Array

        .global sum

sum:    mov   r3, #0            @ r3 = 0
loop:   ldrb  r2, [r0], #1      @ r2 = *r0++    ; Get array element
        add   r3, r2, r3        @ r3 += r2      ; Calculate sum
        cmp   r0, r1            @ if (r0 != r1) ; Check if hit end-of-array
        bne   loop              @    goto loop  ; Loop
        mov   pc, lr            @ pc = lr       ; Return when done
```