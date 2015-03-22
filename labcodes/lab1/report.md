1.1
```
生成ucore.img的代码是第181行到185行，
其中第181行说明ucore.img依赖于kernel和bootblock，

生成bootblock的代码在第161行到第167行，
依赖于bootasm.o，bootmain.o和sign。

bootasm.o依赖于bootasm.S，
编译命令为gcc -Iboot/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Os -nostdinc -c boot/bootasm.S -o obj/boot/bootasm.o。

其中-I添加头文件搜索路径，
-Wall启用大部分警告信息，
-fno-builtin除非以__builtin_作为前缀否则不适用内置函数，
-ggdb生成GDB可用的调试信息，
-m32生成32位环境使用的代码，
-gstabs生成stabs格式的调试信息，
-nostdinc搜索头文件时忽略标准系统目录，
-fno-stack-protector不生成用户检测缓冲区溢出的代码，
-Os优化代码大小。

bootmain.o依赖于bootmain.c，
编译命令为gcc -Iboot/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc -fno-stack-protector -Ilibs/ -Os -nostdinc -c boot/bootmain.c -o obj/boot/bootmain.o。

生成sign的命令为
gcc -Itools/ -g -Wall -O2 -c tools/sign.c -o obj/sign/tools/sign.o
gcc -g -Wall -O2 obj/sign/tools/sign.o -o bin/sign

生成bootblock.o的命令为
ld -m    elf_i386 -nostdlib -N -e start -Ttext 0x7C00 obj/boot/bootasm.o obj/boot/bootmain.o -o obj/bootblock.o
其中-m模拟i386的链接器，
-nostdlib只搜索命令中显式指定的库目录，
-N设置代码段和数据段可读可写，
-e指定入口，
-Ttext指定代码段的开始位置。

之后命令
objcopy -S -O binary obj/bootblock.o obj/bootblock.out
拷贝二进制代码bootblock.o到bootblock.out，
其中参数-S不拷贝重定位和符号信息。

使用sign工具处理bootblock.out生成bootblock。
bin/sign obj/bootblock.out bin/bootblock

生成kernel的代码为第145行到第150行。
kernel依赖于kernel.ld和一堆.o文件。
ld -m    elf_i386 -nostdlib -T tools/kernel.ld -o bin/kernel...
链接生成kernel，其中-T指定脚本。

命令
dd if=/dev/zero of=bin/ucore.img count=10000
dd if=bin/bootblock of=bin/ucore.img conv=notrunc
dd if=bin/kernel of=bin/ucore.img seek=1 conv=notrunc
生成一个10000个块的文件，每个块默认512个字节，将bootblock写到第一个块，将kernel写到后续的块。


```

1.2
```
由sign.c中可以看出，引导扇区大小为512字节，以0x55AA结尾。
```

2.1
```
通过改写Makefile文件在调用qemu时增加-d in_asm -D q.log参数，
便可以将汇编指令保存在q.log中。
```

2.2
```
运行make debug后，在gdb中输入b *0x7c00设置断点，
运行c继续执行，停止后输出x /i $pc查看当前指令。
可以看到当前指令为cli。
```

2.3
```
一致
```

2.4
```
已测试
```

3
```
从$cs=0, $pc=0x7c00进入后首先将flag寄存器置0，段寄存器置0，从第15行到第24行。
开启A20，通过将键盘控制器上的A20线置于高电位，全部32条地址线可用， 可以访问4G的内存空间。代码从第29行到第44行。
初始化GDT表，一个简单的GDT表和其描述符已经静态储存在引导区中，通过指令lgdt gdtdesc载入。
进入保护模式，通过将cr0寄存器PE位置1便开启了保护模式，代码第50行到第53行。
通过长跳转更新cs的基地址，代码第56行。
设置段寄存器，并建立堆栈，代码第61行到第67行。
已经进入保护模式，调用bootmain函数。
```

4
```
readsect函数能够读取磁盘上一个指定的扇区，
readseg函数通过调用readsect能够读取任意长度的内容。
在bootmain函数中首先读取前8个扇区共4KB，检查符合ELF格式，
之后根据ELF相关信息，读取每个程序，最后跳转到入口处执行。

```

5
```
ebp寄存器保存调用者的ebp，由此可以找到所有ebp，ebp+4指向调用时的eip，ebp+8，ebp+12，ebp+16，ebp+20是可能的参数。
输出的最后一行是栈底函数调用，其ebp是0x7bf8，是因为bootloader设置堆栈从0x7c00开始，通过call指令压栈的bootmain函数。
```

6.1
```
中断向量表一个表项占用8字节，其中2-3字节是段选择子，0-1字节和6-7字节拼成位移， 两者联合便是中断处理程序的入口地址。
```

6.2
```
见代码
```

6.3
```
见代码
```




