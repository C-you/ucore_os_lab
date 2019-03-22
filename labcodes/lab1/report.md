#Ucore lab- lab1实验报告
##练习1
###1.	操作系统镜像文件ucore.img是如何一步一步生成的？
输入make v=可得
+ cc kern/init/init.c  //编译所有生成bin/kernel所需的文件
+ cc kern/libs/stdio.c
+ cc kern/libs/readline.c
+ cc kern/debug/panic.c
+ cc kern/debug/kdebug.c
+ cc kern/debug/kmonitor.c
+ cc kern/driver/clock.c
+ cc kern/driver/console.c
+ cc kern/driver/picirq.c
+ cc kern/driver/intr.c
+ cc kern/trap/trap.c
+ cc kern/trap/vectors.S
+ cc kern/trap/trapentry.S
+ cc kern/mm/pmm.c
+ cc libs/string.c
+ cc libs/printfmt.c
+ ld bin/kernel   //链接生成bin/kernel
+ cc boot/bootasm.S  //编译bootasm.S  bootmain.c  sign.c
+ cc boot/bootmain.c
+ cc tools/sign.c
+ ld bin/bootblock
'obj/bootblock.out' size: 488 bytes
build 512 bytes boot sector: 'bin/bootblock' success!
记录了10000+0 的读入
记录了10000+0 的写出
5120000 bytes (5,1 MB, 4,9 MiB) copied, 0,0327003 s, 157 MB/s
记录了1+0 的读入
记录了1+0 的写出
512 bytes copied, 0,000196915 s, 2,6 MB/s
记录了146+1 的读入
记录了146+1 的写出
74816 bytes (75 kB, 73 KiB) copied, 0,00037946 s, 197 MB/s

由上可得，先编译所有生成bin/kernel所需的文件，然后链接生成bin/kernel，
然后编译bootasm.S  bootmain.c  sign.c ，再根据sign规范生成obj/bootblock.o
最后生成ucore.img。

###2.一个被系统认为是符合规范的硬盘主引导扇区的特征是什么？
第一，磁盘主引导扇区只有512字节
第二，磁盘最后两个字节为0x55和0xAA
第三，由不超过466字节的启动代码和不超过64字节的硬盘分区表加上两个字节的结束符组成

##练习2
###1.从CPU加电后执行的第一条指令开始，单步跟踪BIOS的执行
修改lab1/tools/gdbinit里的内容为
```
set architecture i8086
target remote :1234
define hook-stop
x/i $pc
end
```
然后在lab1运行make debug
使用si命令即可进行单步调试
###2.在初始化位置0x7c00设置实地址断点,测试断点正常
修改同样的文件gdbinit为
```
file obj/bootblock.o
set architecture i8086
target remote :1234
b *0x7c00
continue
```
完成
###3.从0x7c00开始跟踪代码运行,将单步跟踪反汇编得到的代码与bootasm.S和 bootblock.asm进行比较
代码均相同

###4.自己找一个bootloader或内核中的代码位置，设置断点并进行测试
在进入保护模式的位置设置断点,运行到断点后就停止
将gdbinit进行修改
```
file obj/bootblock.o
set architecture i8086
target remote :1234
b protcseg
continue
```

##练习3
###分析bootloader进入保护模式的过程
bootbloader首先屏蔽所有中断,之后将段寄存器清零,打开A20地址线,加载GDT的基地址,切换到保护模式,跳转到32位代码. 在32位代码中, bootloader重新设置保护模式下的段寄存器, 然后设置栈顶指针, 之后跳转到C代码。

##练习4
###分析bootloader加载ELF格式的OS的过程
先读取elf文件头,验证文件头中的magic number,读取程序头部，即一个program header结构的数组，从中获得程序各个的信息，且读取到指定的地址处,再跳转执行该文件
下面表示将该地址强制转换成一个参数为空的函数，并执行该函数
```
  ((void (*)(void))(ELFHDR->e_entry & 0xFFFFFF))()
```

##练习5
###实现函数调用堆栈跟踪函数
首先，可以通过read_ebp()和read_eip()函数来获取当前ebp寄存器和eip 寄存器的信息。
然后输出当前栈帧内容，然后再获得前一个栈帧的ebp和eip(用当前的ebp值作为地址)，之后再进行递归到所有栈帧都被访问。
执行 make qemu 得到输出，其中最后一行是：
```
ebp:00007bf8 eip:00007d6e args:c031fcfa c08ed88e 64e4d08e fa7502a8
    <unknow>: -- 0x00007d6d --
```
其对应的是第一个使用堆栈的函数，bootmain.c中的bootmain，bootloader设置的堆栈从0x7c00开始，使用”call bootmain”转入bootmain函数。call指令压栈，所以bootmain中ebp为0x7bf8。

##练习6
###完善中断初始化和处理
####1.中断描述符表（也可简称为保护模式下的中断向量表）中一个表项占多少字节？其中哪 几位代表中断处理代码的入口？
一个表项占8个字节，其中0-1位和6-7位合起来表示地址，2-3位为段选择子。段选择子从GDT或者LDT中取得基址，相加后得到程序入口
####2.请编程完善kern/trap/trap.c中对中断向量表进行初始化的函数idt_init
先在外面声明__vectors，里面存储着每个中断处理代码的偏移，然后将idt中每一项用SETGATE来设置，最后调用lidt()函数即可
####3.请编程完善trap.c中的中断处理函数trap
设置一个变量进行累加，每次累计到100 ticks时打印并清空



