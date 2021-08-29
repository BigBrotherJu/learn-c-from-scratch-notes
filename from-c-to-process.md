# 从源代码到进程的整个过程

> 一个 C 源代码文件要先被编译器翻译成汇编程序，再被汇编器翻译成机器指令，最后还要经过链接器的处理才能成为可执行文件。
>
> 编译器的工作分为两个阶段，先是预处理（Preprocess）阶段，然后才是编译阶段。

## 预处理阶段：C 代码到预处理后的 C 代码

### 预处理命令

可以用这两种方式对 C 代码 main.c 进行预处理，得到预处理后的 C 代码：

- `$gcc -E main.c`

- `$cpp main.c`

  cpp 是 C preprocessor 的意思。

### 预处理器的工作

预处理器将源代码中的预处理指令进行预处理，得到预处理后的 C 代码。

## 编译阶段：预处理后的 C 代码到汇编代码

### 编译命令

要查看编译后的汇编代码，其实还有一种办法是gcc -S main.c，这样只生成汇编代码main.s，而不生成二进制的目标文件。

## 汇编阶段：汇编代码到可重定位的目标文件

### 汇编命令

汇编器 as 将汇编代码 hello.s 生成可重定位的目标文件 hello.o，hello.s 是 17.3 节中的返回最大数的汇编代码：

``` console
as hello.s -o hello.o
```

### 汇编器的工作

汇编器对汇编代码进行以下操作，得到可重定位的目标文件：

- 汇编器将汇编代码中的助记符翻译成机器指令。

- 汇编器将汇编代码中所有的符号替换成对应的地址，放入可重定位的目标文件。

  符号和地址的对应关系在 symbol table 里，这里的地址是相对符号所在段的地址。

- 汇编器将汇编代码中的 section 搬入可重定位的目标文件，另外还会往可重定位的目标文件中添加另外一些 section（比如符号表）。

## 可重定位的目标文件

目标文件由若干个Section组成，我们在汇编程序中声明的.section会成为目标文件中的Section，此外汇编器还会自动添加一些Section（比如符号表）。

### readelf 结果

可重定位的目标文件可以通过 `readelf -a hello.o` 命令查看其中的信息，显示的信息有：

#### ELF Header

``` console
$ readelf -a max.o
ELF Header:
  Magic:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF32
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              REL (Relocatable file)
  Machine:                           Intel 80386
  Version:                           0x1
  Entry point address:               0x0
  Start of program headers:          0 (bytes into file)
  Start of section headers:          200 (bytes into file)
  Flags:                             0x0
  Size of this header:               52 (bytes)
  Size of program headers:           0 (bytes)
  Number of program headers:         0
  Size of section headers:           40 (bytes)
  Number of section headers:         8
  Section header string table index: 5
...
```

- `Entry point address`

  程序的入口地址是0x0，因为目标文件的入口地址还没确定，链接成可执行文件时才能确定入口地址。

- Program headers

  `Start of program headers`、`Size of program headers`、`Number of program headers` 都是 0。

  因为目标文件没有Program Header Table，链接成可执行文件时才会有Program Header Table；

- Section headers

  `Start of section headers` 表示 section header table 中第一个 section header 的起始地址。

  `Size of section headers` 表示 section header table 中每一个 section header 占用的大小。

  `Number of section headers` 表示 section header table 中 section header 的数量。

  `Section header string table index` 表示 section header string table（.shstrtab）（它也是一个 section）在 section header table 中的位置。这个 section 是汇编器自己加的。

#### Section Headers

```
...
Section Headers:
  [Nr] Name              Type            Addr     Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            00000000 000000 000000 00      0   0  0
  [ 1] .text             PROGBITS        00000000 000034 00002a 00  AX  0   0  4
  [ 2] .rel.text         REL             00000000 0002b0 000010 08      6   1  4
  [ 3] .data             PROGBITS        00000000 000060 000038 00  WA  0   0  4
  [ 4] .bss              NOBITS          00000000 000098 000000 00  WA  0   0  4
  [ 5] .shstrtab         STRTAB          00000000 000098 000030 00      0   0  1
  [ 6] .symtab           SYMTAB          00000000 000208 000080 10      7   7  4
  [ 7] .strtab           STRTAB          00000000 000288 000028 00      0   0  1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings)
  I (info), L (link order), G (group), x (unknown)
  O (extra OS processing required) o (OS specific), p (processor specific)

There are no section groups in this file.

There are no program headers in this file.
...
```

ELF 文件是以 ELF header 开头的，ELF header 给了 ELF 文件的所有信息。我们可以通过 ELF header 中记录的 `Start of section headers`、`Size of section headers`、`Number of section headers`、`Section header string table index` 在 ELF 文件中找到 section header 的所有信息。

- 所有 section

  其中.text和.data是我们在汇编程序中声明的Section，而其他Section是汇编器自动添加的。

  - .text

    见 [.text](#.text)。

  - .rel.text

    见 [Relocation section](#relocation-section)。

  - .data

  - .bss ???

    我们知道，C语言的全局变量如果在代码中没有初始化，就会在程序加载时用0初始化。

    这种数据属于.bss段，在加载时它和.data段一样都是可读可写的数据，但是在ELF文件中.data段需要占用一部分空间保存初始值，而.bss段则不需要。

    也就是说，.bss段在文件中只占一个Section Header而没有对应的Section，程序加载时.bss段占多大内存空间在Section Header中描述。

    在我们这个例子中没有用到.bss段，在第18.3节会看到这样的例子。

  - .shstrtab

    .shstrtab 段以 ASCII 码形式保存着各 Section 的名字，每个 section 名字以 \0 结尾。

  - .symtab

    见 [Symbol table](#symbol-table)。

  - .strtab

    .strtab 段以 ASCII 码形式保存着程序中用到的符号的名字，每个名字以 \0 结尾。

- section header 的属性

  - Name

    section header 中并不存储 section 的 name，而是存储相对于 .shstrtab 段起始地址的 offset。

    比如 .text 段的 section header 中存放的 offset 是 1f，表示 .text 段的名字从 .shstrtab 起始地址加 1f 开始，到 \0 结束。这个范围内的字符就是 .text\0。

  - Addr

    Addr列指出 Section 加载到内存中的地址（虚拟地址），目标文件中各 Section 的加载地址是待定的，所以是00000000，到链接时再确定这些地址。

  - Off 和 Size

    Off 和 Size 列指出各 Section 的起始文件地址和长度。

  - Flg

    Flg 列指出各 section 的权限。

    .data 段可读可写，相当于C程序的全局变量。

    .text 段保存代码，是只读和可执行的。

#### Relocation section

``` console
...
Relocation section '.rel.text' at offset 0x2b0 contains 2 entries:
 Offset     Info    Type            Sym.Value  Sym. Name
00000008  00000201 R_386_32          00000000   .data
00000017  00000201 R_386_32          00000000   .data
...
```

.rel.text 告诉链接器 .text 的哪些地方需要做重定位。只有 .rel.text 列出的地方需要做重定位，其他地方不需要。

- Offset

  第一列Offset的值就是.text段需要改的地方，在.text段中的相对地址是8和0x17，正是这两条指令中00 00 00 00的位置。

- Info

  Info 的高 6 位是 symbol table 的 index，低 2 位是 relocation type。可以看到 index 为 2 的 symbol 是在 symbol table 中是一个 section，它的 section 编号是 3，所以这个 symbol 就是 .data 段。

#### Symbol table

``` console
Symbol table '.symtab' contains 8 entries:
   Num:    Value  Size Type    Bind   Vis      Ndx Name
     0: 00000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 00000000     0 SECTION LOCAL  DEFAULT    1
     2: 00000000     0 SECTION LOCAL  DEFAULT    3
     3: 00000000     0 SECTION LOCAL  DEFAULT    4
     4: 00000000     0 NOTYPE  LOCAL  DEFAULT    3 data_items
     5: 0000000e     0 NOTYPE  LOCAL  DEFAULT    1 start_loop
     6: 00000023     0 NOTYPE  LOCAL  DEFAULT    1 loop_exit
     7: 00000000     0 NOTYPE  GLOBAL DEFAULT    1 _start
```

symbol table，也就是符号表是从 .symtab 段中读出来的。

- Name

  和 section header table 一样，symbol table 不存储 symbol 的 name，而是存储相对于 .strtab 段起始地址的 offset。

- Ndx

  Ndx列是每个符号所在的Section编号，例如符号data_items在第3个Section里（也就是.data段），各Section的编号见Section Header Table。

- Value

  Value列是每个符号所代表的地址，在目标文件中，符号地址都是相对于该符号所在Section的相对地址，比如data_items位于.data段的开头，所以地址是0，_start位于.text段的开头，所以地址也是0，但是start_loop和loop_exit相对于.text段的地址就不是0了。

- Bind

  从Bind这一列可以看出_start这个符号是GLOBAL的，而其他符号是LOCAL的，在汇编程序中用.globl指示声明过的符号会成为全局符号，否则成为局部符号。

  _start就像C程序的main函数一样特殊，是整个程序的入口，链接器在链接时会查找目标文件中的_start符号代表的地址，把它设置为整个程序的入口地址，所以每个汇编程序都要提供一个_start符号并且用.globl声明。

  如果一个符号没有用.globl声明，就表示这个符号只在汇编代码内部使用，不会被链接器用到。

### .text

``` console
$ objdump -d max.o

max.o:     file format elf32-i386


Disassembly of section .text:

00000000 <_start>:
   0:   bf 00 00 00 00          mov    $0x0,%edi
   5:   8b 04 bd 00 00 00 00    mov    0x0(,%edi,4),%eax
   c:   89 c3                   mov    %eax,%ebx

0000000e <start_loop>:
   e:   83 f8 00                cmp    $0x0,%eax
  11:   74 10                   je     23 <loop_exit>
  13:   47                      inc    %edi
  14:   8b 04 bd 00 00 00 00    mov    0x0(,%edi,4),%eax
  1b:   39 d8                   cmp    %ebx,%eax
  1d:   7e ef                   jle    e <start_loop>
  1f:   89 c3                   mov    %eax,%ebx
  21:   eb eb                   jmp    e <start_loop>

00000023 <loop_exit>:
  23:   b8 01 00 00 00          mov    $0x1,%eax
  28:   cd 80                   int    $0x80
```

目标文件中的指令中的符号全部被替换为地址，这个地址就是 symbol table 里每个符号对应的地址，是相对于 section 的地址。

实际上只有符号 data_items 被替换了。data_items 被替换为相对地址，下一步链接器根据 .rel.text 的内容将 data_items 换成绝对的虚拟地址。

跳转指令中的符号不用替换，因为这些跳转实际上都是相对跳转，指令没有 reference 符号。

## 库文件

在链接过程中还用-l选项指定了一些库文件，有libc、libgcc、libgcc_s，

有些库是共享库，需要动态链接，所以用-dynamic-linker选项指定动态链接器是/lib/ld-linux.so.2。

## 链接阶段：可重定位的目标文件到可执行文件

### 链接命令

用链接器（Linker，或Link Editor）ld 把可重定位的目标文件 hello.o 链接成可执行文件 hello：

``` console
ld hello.o -o hello
```

### 链接器的工作

链接器对可重定位的目标文件进行以下操作，得到可执行文件：

- 链接器把ELF文件看成是Section的集合。将目标文件中的Section合并成几个Segment，生成可执行文件。

  对链接器来说，ELF 文件中的 program header table 在链接过程中用不到，可有可无；section header table 中保存了所有Section的描述信息，通过Section Header Table可以找到每个Section在文件中的位置。

  一个或多个 section 组成一个 segment，一个 section 有单独的权限，一个 segment 也有单独的权限。

  有些Section只对链接器有意义，在运行时用不到，也不需要加载到内存，那么就不属于任何Segment，但仍然留在可执行文件中。

  还有一些 section 没有存在的必要，在可执行文件中被删除了。

- 链接器修改目标文件中的信息，对地址做重定位

  链接器要修改指令，把其中的相对地址都改成加载时的内存地址，这些指令才能正确执行。

- 链接器可以把多个目标文件合并成一个可执行文件，在第18.2节详细解释。如果只有一个目标文件，也需要经过链接才能成为可执行文件。

- 我的理解 ???

  链接器先把目标文件中没用的 section（比如 .bss）删掉，然后根据需要加载运行的 section，声明一些 segment，并指明每一个 segment 都包含哪些需要加载运行的 section。

  然后链接器找到合适的虚拟地址，更新每个 segment 的地址，顺便更新和这些 segment 有关的 section 的地址。再更新 symbol table 里的 symbol 对应地址。

  然后把 symbol table 中 _start 的地址设成 entry point address。

  然后链接器根据 .rel.text 在 .text 中把需要的重定位的 symbol 的地址给更新。

  最后把 .rel.text 删掉，在 symbol table 中加上 __bss_start、_edata、_end，生成可执行文件。

- `-l` 选项

  链接库文件时，需要根据库文件文件名用相应的选项执行 GCC。如链接 `libm.so`，命令为 `$gcc main.c -lm`。`-lc` 不用加上，这是 GCC 默认选项。

## 可执行文件

### readelf 结果

可执行文件可以通过 `readelf -a hello` 命令查看其中的信息，hello 是 17.3 节中汇编代码生成的可执行文件。显示的信息有：

#### ELF header

``` console
$ readelf -a max
ELF Header:
  Magic:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF32
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           Intel 80386
  Version:                           0x1
  Entry point address:               0x8048074
  Start of program headers:          52 (bytes into file)
  Start of section headers:          256 (bytes into file)
  Flags:                             0x0
  Size of this header:               52 (bytes)
  Size of program headers:           32 (bytes)
  Number of program headers:         2
  Size of section headers:           40 (bytes)
  Number of section headers:         6
  Section header string table index: 3
...
```

- `Entry point address`

  Entry point address 不再是0x0，改成了0x8048074，这是_start符号的地址。这个地址是绝对的虚拟地址。

- Program headers

  可执行文件中存在 program header table，所以 `Start of program headers`、`Size of program headers`、`Number of program headers` 都是有值的。

  Number of program headers 为 2，表示有两个 program header。

- Section headers

  `Start of section headers` 会发生变动。

  `Size of section headers` 不变。

  `Number of section headers` 小了 2。因为 .rel.text 段和 .bss 段被删掉了。

  `Section header string table index` 也会变。

#### Section Headers

``` console
Section Headers:
  [Nr] Name              Type            Addr     Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            00000000 000000 000000 00      0   0  0
  [ 1] .text             PROGBITS        08048074 000074 00002a 00  AX  0   0  4
  [ 2] .data             PROGBITS        080490a0 0000a0 000038 00  WA  0   0  4
  [ 3] .shstrtab         STRTAB          00000000 0000d8 000027 00      0   0  1
  [ 4] .symtab           SYMTAB          00000000 0001f0 0000a0 10      5   6  4
  [ 5] .strtab           STRTAB          00000000 000290 000040 00      0   0  1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings)
  I (info), L (link order), G (group), x (unknown)
  O (extra OS processing required) o (OS specific), p (processor specific)
```

在Section Header Table中，.text和.data段的加载地址分别改成了0x08048074和0x080490a0。

.bss段没有用到，所以被删掉了。.rel.text段就是用于链接过程的，做完链接就没用了，所以也删掉了。

#### Program Headers

``` console
Program Headers:
  Type           Offset   VirtAddr   PhysAddr   FileSiz MemSiz  Flg Align
  LOAD           0x000000 0x08048000 0x08048000 0x0009e 0x0009e R E 0x1000
  LOAD           0x0000a0 0x080490a0 0x080490a0 0x00038 0x00038 RW  0x1000

 Section to Segment mapping:
  Segment Sections...
   00     .text
   01     .data
```

我们可以通过 ELF header 中记录的 `Start of program headers`、`Size of program headers`、`Number of program headers`、`program header string table index` 在 ELF 文件中找到 program header 的所有信息。

hello 中有两个 segment，.text段和前面的ELF Header、Program Header Table一起组成一个Segment（FileSiz指出总长度是0x9e），.data段组成另一个Segment（总长度是0x38），以后我们把这两个Segment分别叫做Text Segment和Data Segment。

- FileSiz

  FileSiz 指出 segment 的总长度。

- VirtAddr 和 PhysAddr

  VirtAddr列指出Text Segment加载到虚拟地址0x08048000（注意在x86平台上后面的PhysAddr列是没有意义的，并不代表实际的物理地址），Data Segment加载到地址0x080490a0。

  - 可执行文件的两个 segment 必须被加载到内存的不同页面

    两个Segment必须加载到内存中两个不同的页面，因为MMU的权限保护机制是以页为单位的，一个页面只能设置一种权限。即使可执行文件的大小小于一页。

    所以Text Segment加载到虚拟地址0x08048000～0x08048fff的内存页面，而Data Segment加载到虚拟地址0x08049000～0x08049fff的内存页面。

  - segment 在文件页面中的偏移和内存页面中的偏移一样

    此外，为了简化链接器和加载器的实现，还规定每个Segment在文件的页面中偏移多少加载到内存页面也要偏移多少。

    Text Segment在文件的第0个页面开头，加载到内存页面也是从首地址0x08048000开始，由于Text Segment包含了文件名和.text段，所以.text段的加载地址是0x08048074，_start符号位于.text段的开头，所以_start符号的地址也是0x08048074，从符号表中可以验证这一点。

    Data Segment在文件的第0个页面中的偏移是0xa0，在内存页面中的偏移也是0xa0，所以从0x080490a0开始。

- Flg

  Flg列指出Text Segment的访问权限是可读可执行，Data Segment的访问权限是可读可写。

  一个 segment 只能有一种权限。

- Align

  最后一列Align的值0x1000（4K）是x86平台的内存页面大小。在加载时文件也要按内存页面大小分成若干页，文件中的一页对应内存中的一页。

#### Symbol table

``` console
Symbol table '.symtab' contains 10 entries:
   Num:    Value  Size Type    Bind   Vis      Ndx Name
     0: 00000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 08048074     0 SECTION LOCAL  DEFAULT    1
     2: 080490a0     0 SECTION LOCAL  DEFAULT    2
     3: 080490a0     0 NOTYPE  LOCAL  DEFAULT    2 data_items
     4: 08048082     0 NOTYPE  LOCAL  DEFAULT    1 start_loop
     5: 08048097     0 NOTYPE  LOCAL  DEFAULT    1 loop_exit
     6: 08048074     0 NOTYPE  GLOBAL DEFAULT    1 _start
     7: 080490d8     0 NOTYPE  GLOBAL DEFAULT  ABS __bss_start
     8: 080490d8     0 NOTYPE  GLOBAL DEFAULT  ABS _edata
     9: 080490d8     0 NOTYPE  GLOBAL DEFAULT  ABS _end
```

原来目标文件符号表中的Value都是相对地址，现在都改成绝对地址了。

此外还多了三个符号__bss_start、_edata和_end，这些符号在链接脚本中定义，被链接器添加到可执行文件中，链接脚本在第19.1节介绍。

### .text

``` console
$ objdump -d max

max:     file format elf32-i386


Disassembly of section .text:

08048074 <_start>:
 8048074:   bf 00 00 00 00          mov    $0x0,%edi
 8048079:   8b 04 bd a0 90 04 08    mov    0x80490a0(,%edi,4),%eax
 8048080:   89 c3                   mov    %eax,%ebx

08048082 <start_loop>:
 8048082:   83 f8 00                cmp    $0x0,%eax
 8048085:   74 10                   je     8048097 <loop_exit>
 8048087:   47                      inc    %edi
 8048088:   8b 04 bd a0 90 04 08    mov    0x80490a0(,%edi,4),%eax
 804808f:   39 d8                   cmp    %ebx,%eax
 8048091:   7e ef                   jle    8048082 <start_loop>
 8048093:   89 c3                   mov    %eax,%ebx
 8048095:   eb eb                   jmp    8048082 <start_loop>

08048097 <loop_exit>:
 8048097:   b8 01 00 00 00          mov    $0x1,%eax
 804809c:   cd 80                   int    $0x80
```

指令中只有 data_items 对应的相对地址根据 .rel.text 改成了绝对地址。

跳转指令中指定的是相对于当前指令向前或向后跳多少字节，而不是指定一个完整的内存地址，内存地址有32位，这些跳转指令只有16位，显然也不可能指定一个完整的内存地址，这称为相对跳转。这种相对跳转指令只有16位，只能在当前指令前后的一个小范围内跳转，不可能跳得太远，也有的跳转指令指定一个完整的内存地址，可以跳到任何地方，称为绝对跳转。

- 汇编代码和 C 代码穿插显示

  ``` console
  gcc main.c -g
  objdump -dS a.out
  ```

  如果在编译时加上-g选项（在第10章讲过-g选项），那么用objdump反汇编时可以把C代码和汇编代码穿插起来显示，这样C代码和汇编代码的对应关系看得更清楚。

## 去除符号信息的可执行文件

用strip命令去除可执行文件中的符号信息，这样可以有效减小文件的尺寸而不影响运行：

``` console
$ strip max
$ readelf -a max
...
```

注意不要对目标文件和共享库使用strip命令，因为链接器需要利用目标文件和共享库中的符号信息来做链接。

## 加载：可执行文件到进程

保存在硬盘上的程序是不能被CPU直接取指令执行的，操作系统在执行程序时会把它从硬盘拷贝到内存，这样CPU才能取指令执行，这个过程称为加载（Load）。

程序加载到内存之后，成为操作系统调度执行的一个任务，就称为进程（Process）。

> .section指示把代码划分成若干个段（Section），程序被操作系统加载执行时，每个段被加载到不同的地址，操作系统对不同的页面设置不同的读、写、执行权限。

### 加载器的工作

加载器把保存在硬盘上的可执行文件（hello）加载到内存，得到进程：

- 加载器把ELF文件看成是Segment的集合。加载器（Loader）根据可执行文件中的Segment信息加载运行这个程序。

  对加载器来说，ELF 文件中的 Section Header Table 在加载过程中用不到，可有可无；program header table 中保存了所有segment的描述信息，通过 program header table 可以找到每个 segment 在文件中的位置。

## 执行：CPU 执行可执行文件

CPU 只重复做一件事：根据程序计数器寄存器中保存的下一条指令地址，从内存中取得这条指令，执行这条指令。

可执行文件中的地址是虚拟地址，CPU 执行单元在工作时要访问一个虚拟地址，读这个地址上数据，或者写数据到这个地址上。

CPU 执行单元发出虚拟地址后，先在 cache 中进行查找：

- 如果 cache 中缓存了这个虚拟地址，需要检查对应的访问权限。

  - 如果允许访问，就不需要访问物理内存了：

    - 如果是读操作，直接将数据传给 CPU 寄存器；
    - 如果是写操作就直接把数据写到 cache 中（涉及 cache 如何把数据更新到内存中）。

  - 如果不允许访问，cache 会产生一个异常，CPU 处理异常。

- 如果 cache 中没有缓存这个虚拟地址，就需要访问物理内存。虚拟地址翻译成物理地址，可能成功，可能失败：

  - 如果成功，访问物理内存时进行相应的读写操作，另外还要在这个物理内存周围取一个 cache line 放到 cache 中。

    虚拟内存如何翻译为物理内存？虚拟内存通过 MMU 翻译为物理内存。

    MMU 通过 page table 将一个虚拟内存转换为物理内存。page table 中保存的是虚拟内存的 page 到物理内存的 page frame 的映射关系。

    操作系统在初始化或分配、释放内存时会执行一些指令在物理内存中填写 page table，然后用指令设置 MMU，告诉 MMU page table 在物理内存中的什么位置。设置好之后，CPU 每次执行访问内存的指令都会自动引发 MMU 做查表和地址转换操作，地址转换操作由硬件自动完成，不需要用指令控制 MMU 去做。

  - 如果失败，MMU 产生一个异常，CPU 处理异常。

    为什么会失败？

    操作系统在填写 page table 时，还设置了每个 page 的权限（可读、可写、可执行；用户模式、特权模式）。当 CPU 要访问一个 VA 时，MMU 会检查 CPU 当前处于用户模式还是特权模式，访问内存的目的是读数据、写数据还是取指令，如果和操作系统设定的页面权限相符，就允许访问，把它转换成 PA，否则不允许访问，产生一个异常（Exception）。

    实际上，段错误就是因为程序访问了无权访问的虚拟地址。



编译、汇编、链接

> 总结一下编译执行的过程，首先你用文本编辑器写一个C程序，然后保存成一个文件，例如program.c（通常C程序的文件名后缀是.c），这称为源代码（Source Code）或源文件，然后运行编译器对它进行编译，编译的过程并不执行程序，而是把源代码全部翻译成机器指令，再加上一些描述信息，生成一个新的文件，例如a.out，这称为可执行文件，可执行文件可以被操作系统加载运行，计算机执行该文件中由编译器生成的指令，如图1.1所示。

描述信息是什么？
