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
