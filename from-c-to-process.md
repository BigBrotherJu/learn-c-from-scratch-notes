# 从源代码到进程的整个过程

一个 C 代码文件要被预处理器预处理成预处理后的 C 代码文件，然后被编译器编译成汇编代码文件，再被汇编器汇编成可重定位的目标文件，最后被链接器链接系统提供的其他目标文件和库文件成一个可执行文件。

多个 C 代码文件要被预处理器预处理成多个预处理后的 C 代码文件，然后被编译器编译成多个汇编代码文件，再被汇编器汇编成多个可重定位的目标文件，最后被链接器链接系统提供的其他目标文件和库文件成一个可执行文件。

这个过程中 gcc 分别调动 cc1、as、collect2。

## 预处理阶段：C 代码到预处理后的 C 代码

### 预处理命令

可以用这两种方式对 C 代码 main.c 进行预处理，得到预处理后的 C 代码：

- `$gcc -E main.c`

- `$cpp main.c`

  cpp 是 C preprocessor 的意思。

### 预处理器的工作

预处理器对 C 代码进行以下操作，得到预处理后的 C 代码：

1.  把第2.2节提到过的三连符替换成相应的单字符。

2.  把用\斜线续行的多行代码接成一行。

    \斜线后面要紧跟换行（\r\n或\n），中间不能有其他字符（空格或 Tab 也不行）。

3.  把每个注释（不管是单行注释还是多行注释）替换成一个空格。

4.  把得到的每行代码分成 token 和空白字符。

    经过以上两步之后去掉了一些换行，有的换行在续行过程中去掉了，有的换行在多行注释之中，也随着注释一起去掉了，剩下的代码行称为**逻辑代码行**。

    这时的Token称为预处理Token，包括标识符、关键字、整数常量、浮点数常量、字符常量、字符串字面值、运算符和其他标点符号，在划分Token时要遵循第6.3节讲过的最长匹配原则。

    在第2.6节讲过C语言规定的空白字符包括空格、\t、\v、\r、\n、\f。

5.  在Token中识别出宏定义和预处理指示。

    如果遇到宏定义则做宏展开。

    如果遇到预处理指示则做相应的预处理动作。

    如果遇到#include预处理指示，则把相应的源文件包含进来，并对该源文件做以上1～5步预处理。

    预处理指示的定义见下。

6.  找出字符常量或字符串字面值中的转义序列，用相应的字节来替换它。

    比如把\n替换成字节0x0a。

7.  把相邻的字符串字面值连接起来。

8.  经过以上处理之后，把空白字符丢掉，把Token交给C编译器做语法解析，这时就不再是预处理Token，而称为C Token了。

### 预处理指示严格定义

一条预处理指示由一个逻辑代码行组成，第一个预处理Token是#号，后面跟若干个预处理Token。

在逻辑代码行的开头、结尾以及各预处理Token之间可以出现任意多个空格和Tab，但不允许出现其他空白字符（除预处理指示之外的其他逻辑代码行对空白字符没有此限制）。

把一个预处理指示写成多行要用\斜线续行，因为根据定义，一条预处理指示只能由一个逻辑代码行组成，而把C代码写成多行则不必用\斜线续行，因为换行在C代码中只不过是一种空白字符，在做语法解析时所有空白字符都已经丢掉了。

### 例子

一个 C 文件中有以下代码：

``` c
#include <stdio.h>
#define STR "hello, "\
		"world"
int main(void)
{
    printf(
    	STR);
    return 0;
}
```

1.  代码中没有三联符，不用做。

2.  将用\斜线续行的多行代码接成一行，得到：

    ``` c
    #include <stdio.h>
    #define STR "hello, "		"world"
    int main(void)
    {
        printf(
        	STR);
        return 0;
    }
    ```

3.  代码中没有注释，不用做。

4.  第二步得到的代码行称为逻辑代码行，预处理器把每个逻辑代码行分成预处理 token 和空白字符（空格、\t、\v、\r、\n、\f）。

    上面代码的第 2 行被划分成：#，define，空格，STR，空格，"hello,␣"，Tab，Tab，"world"，换行。

    第 5 第 6 行被划分成：printf，(，换行，Tab，STR，)，;，换行。

    其他逻辑代码行也会被分成预处理 token 和空白字符。

5.  在预处理 token 中识别出宏定义和预处理指示。

    第 1 逻辑代码行中有 #include 预处理指示，把 stdio.h 内容包含进来，对 stdio.h 内容做上面一到五步。

    第 2 逻辑代码行中有 #define 预处理指示，预处理器记录下这个宏定义。

    第 6 逻辑代码行中有 STR 宏定义，预处理器做宏展开，第 6 逻辑代码行变成以下预处理 token：printf，(，换行，Tab，"hello, "，Tab，Tab，"world"，)，;，换行。

    ``` c
    ...
    #define STR "hello, "		"world"
    int main(void)
    {
        printf(
        	"hello, "		"world");
        return 0;
    }
    ```

6.  代码中没有转义序列，不用做。

7.  把相邻的字符串字面值连接起来。

    第 6 逻辑代码行的 "hello, "，Tab，Tab，"world" 会被连接成 "hello, world"。

8.  上面 7 步完成后，空白字符被丢掉，预处理 token 交给 C 编译器做语法解析，这时 token 被称为 C token。

    第 6 逻辑代码行最后交给C编译器做语法解析的Token是：printf，(，"hello, world"，)，;。

## 编译阶段：预处理后的 C 代码到汇编代码

### 编译命令

gcc -S main.c 将 main.c 进行预处理，然后编译成汇编代码 main.s。

### 编译器的工作

### 内联汇编

gcc 提供了一种扩展语法可以在 C 代码中使用内联汇编，详见 [C 内联汇编](part-2/chapter-18.md#18.5-C-内联汇编)

## 汇编代码

汇编代码可以自己直接写，如 17.3 节中的汇编代码；也可以通过 `gcc -S` 由 c 代码生成。

## 汇编阶段：汇编代码到可重定位的目标文件

### 汇编命令

- as

  汇编器 as 将汇编代码 hello.s 生成可重定位的目标文件 hello.o，hello.s 是 17.3 节中的返回最大数的汇编代码：

  ``` console
  as hello.s -o hello.o
  ```

- gcc -c hello.c

  gcc -c 也可以将 C 代码 hello.c 生成可重定位的目标文件 hello.o，中间要调动 cc1 和 as。

### 汇编器的工作

汇编器对汇编代码进行以下操作，得到可重定位的目标文件：

- 汇编器将汇编代码中的助记符翻译成机器指令

- 汇编器将汇编代码中每个符号替换成符号代表的地址，放入可重定位的目标文件

  符号和地址的对应关系在 symbol table 里，这里的地址是相对符号所在段的地址。

- 汇编器将汇编代码中的 section 放入可重定位的目标文件，另外还会往可重定位的目标文件中添加另外一些 section（比如符号表）

## 可重定位的目标文件

可重定位的目标文件由若干个Section组成，我们在汇编代码中声明的.section会成为可重定位的目标文件中的Section，此外汇编器还会自动添加一些Section（比如符号表）。

这节展示的可重定位的目标文件是由 17.3 节中的汇编代码直接生成的，如果是由 C 代码生成汇编代码生成可重定位的目标文件，可重定位的目标文件内容可能不一样。

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

  程序的入口地址是0x0，因为可重定位的目标文件的入口地址还没确定，链接成可执行文件时才能确定入口地址。

- Program headers

  `Start of program headers`、`Size of program headers`、`Number of program headers` 都是 0。

  因为可重定位的目标文件没有Program Header Table，链接成可执行文件时才会有Program Header Table。

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

  其中.text和.data是我们在汇编代码中声明的Section，而其他Section是汇编器自动添加的。

  - .text

    见 [.text](#.text)。

  - .rel.text

    见 [Relocation section](#relocation-section)。

  - .data

  - .bss

    我们知道，C语言的全局变量如果在代码中没有初始化，就会在程序加载时用0初始化。

    这种数据属于.bss段，在加载时它和.data段一样都是可读可写的数据，但是在ELF文件中.data段需要占用一部分空间保存初始值，而.bss段则不需要。

    也就是说，.bss段在文件中只占一个Section Header而没有对应的Section，程序加载时.bss段占多大内存空间在Section Header中描述。

    > 在我们这个例子中没有用到.bss段，在第18.3节会看到这样的例子。

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

    Addr列指出 Section 加载到内存中的地址（虚拟地址），可重定位的目标文件中各 Section 的加载地址是待定的，所以是00000000，到链接时再确定这些地址。

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

  Value列是每个符号所代表的地址，在可重定位的目标文件中，符号地址都是相对于该符号所在Section的相对地址，比如data_items位于.data段的开头，所以地址是0，_start位于.text段的开头，所以地址也是0，但是start_loop和loop_exit相对于.text段的地址就不是0了。

- Bind

  从Bind这一列可以看出_start这个符号是GLOBAL的，而其他符号是LOCAL的，在汇编代码中用.globl指示声明过的符号会成为全局符号，否则成为局部符号。

  全局符号会被链接器进行链接；局部符号只在汇编代码内部使用，不会被链接器用到。

  _start就像C程序的main函数一样特殊，是整个程序的入口，链接器在链接时会查找可重定位的目标文件中的_start符号代表的地址，把它设置为整个程序的入口地址，所以每个汇编代码都要提供一个_start符号并且用.globl声明。

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

可重定位的目标文件中的指令中的符号全部被替换为地址，这个地址就是 symbol table 里每个符号对应的地址，是相对于 section 的地址。

实际上只有符号 data_items 被替换了。data_items 被替换为相对地址，下一步链接器根据 .rel.text 的内容将 data_items 换成绝对的虚拟地址。

跳转指令中的符号不用替换，因为这些跳转实际上都是相对跳转，指令没有 reference 符号。详细解释见可执行文件的[.text](#.text-1)

## 库文件

### 库文件名

库文件名都是以 lib 开头的。

- 静态库

  静态库以.a作为后缀，表示Archive。

- 共享库

  按照共享库的命名惯例，每个共享库有三个文件名：real name、soname和linker name。

  - real name

    真正的库文件（而不是符号链接）的名字是real name，包含完整的共享库版本号，例如 libcap.so.2.17、libc-2.11.1.so、libcidn-2.11.1.so等。

    注意动态链接器的名字特殊，不叫libxxx，而叫ld-2.11.1.so。

  - soname：动态链接器需要

    soname是符号链接的名字，只包含共享库的主版本号，主版本号一致即可保证库函数的接口一致。

    可执行文件的.dynamic段只记录共享库的soname，动态链接器只要找到soname一致的共享库就可以加载它做动态链接。

    例如上面的libcap.so.2其实是指向libcap.so.2.17的符号链接，这说明主版本号是2，次版本号是17，其实应用程序并不关心这个符号链接所指向的真正的库文件是libcap.so.2.17还是libcap.so.2.16，应用程序只认libcap.so.2这个soname，只要soname正确，这个共享库就应该提供了应用程序所需要的接口，就应该可以正确链接和运行，这意味着libcap.so.2.17和libcap.so.2.16相比可能增加了一些函数接口，或者修正了一些Bug，但绝不应该修改或删除了一些函数接口。

    使用共享库可以很方便地升级库文件（改一下符号链接的指向就可以了）而不需要重新编译程序，这是静态库所没有的优点。

    注意libc的版本编号有一点特殊，libc-2.11.1.so的主版本号是6而不是2或2.11，这也是由于历史原因，Linux曾经用过另外一套libc的实现，后来才改用glibc，但主版本号仍沿用了原来的。

    另外，动态链接器的soname也很特殊，叫ld-linux.so.2，它指向ld-2.11.1.so。

  - linker name：链接器需要

    linker name 以.so作为后缀，表示 Shared Object。

    linker name仅在编译链接时使用，gcc的-L选项应该指定linker name所在的目录。

    有的linker name是库文件的一个符号链接，有的linker name是一段链接脚本。例如上面的libc.so就是一个linker name，它是一段链接脚本，其中指定了soname的路径和其他需要提供给链接器的信息。

    ``` console
    $ cat /usr/lib/libc.so
    /* GNU ld script
       Use the shared library, but some functions are only in
       the static library, so try that secondarily.  */
    OUTPUT_FORMAT(elf32-i386)
    GROUP ( /lib/libc.so.6 /usr/lib/libc_nonshared.a  AS_NEEDED ( /lib/ld-linux.so.2 ) )
    ```

### 创建静态库

1.  将 C 文件编译成目标文件

    ``` console
    gcc -c stack.c push.c pop.c is_empty.c
    ```

2.  用 ar 命令将目标文件打包成静态库

    ``` console
    $ ar rs libstack.a stack.o push.o pop.o is_empty.o
    ar: creating libstack.a
    ```

    ar命令类似于tar命令，也是用来打包的，但是把目标文件打包成静态库的格式只能用ar命令而不能用tar命令。

    选项r表示将后面的目标文件列表添加到文件包libstack.a，如果libstack.a不存在就创建它，如果libstack.a中已有同名的目标文件就替换成新的。

    选项s表示为静态库创建索引，这个索引被链接器使用。ranlib命令也可以为静态库创建索引，以上命令等价于：

    ``` console
    ar r libstack.a stack.o push.o pop.o is_empty.o
    ranlib libstack.a
    ```

### 创建共享库

1.  先生成 PIC 目标文件

    组成共享库的目标文件和一般的目标文件有所不同，在编译时要加-fPIC选项，例如：

    ``` console
    gcc -c -fPIC stack.c push.c pop.c is_empty.c
    ```

    -f后面跟一些编译选项，PIC是其中一种，表示生成位置无关代码（Position Independent Code）。

2.  将 PIC 目标文件整合成共享库

    不指定 soname 的话，soname 就是 libstack.so，默认的 soname 在共享库中不会记录，生成的可执行文件中记录的是 libstack.so：

    ``` console
    gcc -shared -o libstack.so stack.o push.o pop.o is_empty.o
    ```

    指定 soname 的话，共享库中会记录这个 soname，生成的可执行文件中也记录这个 soname：

    ``` console
    gcc -shared -Wl,-soname,libstack.so.1 -o libstack.so.1.0 stack.o push.o pop.o is_empty.o
    ```

    这样编译生成的库文件是libstack.so.1.0，是real name，但这个库文件中记录了它的soname是libstack.so.1。

    如果把libstack.so.1.0所在的目录加入/etc/ld.so.conf中，然后运行ldconfig命令，ldconfig会在 libstack.so.1.0 的目录中自动创建一个libstack.so.1，它是指向 libstack.so.1.0 的符号链接。

    但是调用 `gcc main.c -L. -lstack` 时会出错，因为 gcc 只认 linker name，所以我们要创建一个 libstack.so，这个 libstack.so 是指向 libstack.so.1.0 的符号链接。再编译就没问题了。

    生成的可执行文件中记录的是 libstack.so.1。

## 链接阶段：可重定位的目标文件到可执行文件

### 链接命令

- ld

  链接器（Linker，或Link Editor）ld 将可重定位的目标文件 hello.o 链接成可执行文件 hello：

  ``` console
  ld hello.o -o hello
  ```

- gcc xxx.c

  用 gcc 将单个 C 文件生成可执行文件时，先调动 cc1 和 as，得到可重定位的目标文件。最后会调动 collect2 将 crt1.o、crti.o、crtbegin.o、crtend.o、crtn.o 、库文件 libc、libgcc、libgcc_s、我们自己的可重定位的目标文件进行链接。collect2 调动 ld，真正进行链接的是 ld。

  ``` console
  $ gcc main.c -o main -v
  ...
   /usr/lib/gcc/i486-linux-gnu/4.4.3/collect2 --build-id --eh-frame-hdr -m elf_i386 --hash-style=both -dynamic-linker /lib/ld-linux.so.2 -o main -z relro /usr/lib/gcc/i486-linux-gnu/4.4.3/../../../../lib/crt1.o /usr/lib/gcc/i486-linux-gnu/4.4.3/../../../../lib/crti.o /usr/lib/gcc/i486-linux-gnu/4.4.3/crtbegin.o -L/usr/lib/gcc/i486-linux-gnu/4.4.3 -L/usr/lib/gcc/i486-linux-gnu/4.4.3 -L/usr/lib/gcc/i486-linux-gnu/4.4.3/../../../../lib -L/lib/../lib -L/usr/lib/../lib -L/usr/lib/gcc/i486-linux-gnu/4.4.3/../../.. -L/usr/lib/i486-linux-gnu /tmp/ccIaCwo7.o -lgcc --as-needed -lgcc_s --no-as-needed -lc -lgcc --as-needed -lgcc_s --no-as-needed /usr/lib/gcc/i486-linux-gnu/4.4.3/crtend.o /usr/lib/gcc/i486-linux-gnu/4.4.3/../../../../lib/crtn.o
  ```

- gcc xxx.c yyy.c zzz.c ...

  用 gcc 将多个 C 文件生成可执行文件时，先对每个 C 文件调动 cc1 和 as，得到多个可重定位的目标文件。最后调用 collect2 将 crt1.o、crti.o、crtbegin.o、crtend.o、crtn.o、库文件 libc、libgcc、libgcc_s、我们自己的多个可重定位的目标文件进行链接。collect2 调动 ld，真正进行链接的是 ld。

- gcc xxx.c -lyyy -Lzzz

  除了系统提供的库文件 libc、libgcc、libgcc_s，我们还可以自己指明要和哪个库文件进行链接。

### 链接器的工作

链接器对可重定位的目标文件进行以下操作，得到可执行文件：

- 链接器把ELF文件看成是Section的集合，将可重定位的目标文件中的Section合并成几个Segment，生成可执行文件

  对链接器来说，ELF 文件中的 program header table 在链接过程中用不到，可有可无；section header table 中保存了所有Section的描述信息，通过Section Header Table可以找到每个Section在文件中的位置。

  一个或多个 section 组成一个 segment，一个 section 有单独的权限，一个 segment 也有单独的权限。

  有些Section只对链接器有意义，在运行时用不到，也不需要加载到内存，那么就不属于任何Segment，但仍然留在可执行文件中。

  还有一些 section 没有存在的必要，在可执行文件中被删除了。

  这个过程根据**链接脚本**进行。

- 链接器修改可重定位的目标文件中的信息，对地址做重定位

  链接器要修改指令，把其中的相对地址都改成加载时的内存地址，这些指令才能正确执行。

- 链接器把多个可重定位的目标文件合并成一个可执行文件，并将多个可重定位的目标文件中的符号做好**符号解析**

  如果只有一个可重定位的目标文件，也需要经过链接才能成为可执行文件。

  一个目标文件中引用了某个符号，链接器在另一个目标文件中找到这个符号的定义并确定它的地址，这个过程叫做符号解析（Symbol Resolution）。

  凡是被多次声明的变量或函数，必须有且只有一个声明是定义，如果有多个定义，或者一个定义都没有，链接器就无法完成链接。

- 链接器将可重定位的目标文件和库文件进行链接

  gcc 不加 -l 和 -L 选项时，也会调动 collect2，会用 -l 指定库文件 libc、libgcc、libgcc_s，并用 -L 指定搜索库文件的目录。

  我们也可以在调用 gcc 时自己加上 -l 和 -L 选项，使用自己创建的库文件。

  - 如何搜索共享库和静态库

    在处理 -lxxx 选项时，gcc首先到-L选项指定的目录下查找，首先看有没有共享库libxxx.so，如果有就链接它，否则再找有没有静态库libxxx.a，如果有就链接它，如果还是没有，就到默认搜索路径下按同样的步骤查找。默认路径是 `$ gcc -print-search-dirs` 结果中的 libraries。

    gcc在链接时优先考虑共享库，其次才是静态库，如果希望gcc只考虑静态库，可以指定-static选项。

  - 动态链接器

    如果指定了共享库，需要动态链接，所以用-dynamic-linker选项指定动态链接器是/lib/ld-linux.so.2。

    我们调用 gcc 时，gcc 已经在调动 collect2 的时候处理好了这个选项，我们不用自己指明动态链接器。

  - 链接共享库和静态库时做的工作

    链接共享库时，链接器只是确认可执行文件引用的某些符号在libxxx中有定义，并没有最终确定这些符号的地址，这些符号在可执行文件中仍然是未定义符号，要在运行时做动态链接。

    因为动态链接是在加载可执行文件后做的，所以链接器要在可执行文件中记录动态链接器的路径和共享库的路径。这样当可执行文件被加载后才能正常进行动态链接。

    而链接静态库时，链接器会把静态库中的目标文件取出来和可执行文件真正链接在一起。另外，和一个静态库链接与直接和这个静态库中需要的目标文件链接没有区别。静态库只是打个包而已。

    如果可执行文件中的未定义符号在动态库或静态库中没有定义，那么 collect2 会报错。

- 我的理解

  链接器先根据链接脚本创建一些 segment，并将合适的 section 分到合适的 segment，这时 segment 和 section 也都分到了虚拟地址。链接器还会根据链接脚本添加一些特殊地址定义成的符号，例如 __bss_start 和 _end。

  然后链接器需要根据重定位段对一些 section 中的指令进行重定位。链接器还需要进行符号解析。

  最后一些没用的段会被删掉。

### 链接脚本

链接过程是由一个链接脚本（Linker Script）控制的，链接脚本决定了给每个段分配什么地址，如何对齐，不同可重定位的目标文件的相同段哪个段在前，哪个段在后，哪些段合并到同一个Segment。

另外链接脚本还要把一些特殊地址定义成符号，例如__bss_start代表.bss段的起始地址，_end代表.bss段的结束地址，这些符号会出现在可执行文件的符号表中，加载器可以由这些符号得知.bss段的地址范围，以便把它清零。

如果用ld做链接时没有通过-T选项指定链接脚本，则使用ld的默认链接脚本，默认链接脚本可以用ld --verbose命令查看。

详细见[链接脚本](part-2/chapter-19.md#链接脚本)。

## 汇编代码生成的可执行文件

这节展示的可执行文件是 17.3 节的汇编代码直接生成的，如果是由 C 代码生成汇编代码生成生成可重定位的目标文件生成可执行文件，可执行文件内容可能不一样。

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

- Offset

  Offset 是 program header 第一个字节在文件中的位置。

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

原来可重定位的目标文件符号表中的Value都是相对地址，现在都改成绝对地址了。

`_start` 在可执行文件中必须存在，可执行文件就是从 `_start` 开始执行的。

链接器根据链接脚本把一些特殊地址定义成符号，例如__bss_start代表.bss段的起始地址，_end代表.bss段的结束地址，这些符号会出现在可执行文件的符号表中，加载器可以由这些符号得知.bss段的地址范围，以便把它清零。

在这里可能是 __bss_start、_edata、_end 没用处，所以地址都一样，而且 size 都是 0。_end 不知道用途是什么。

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

## C 代码生成的可执行文件

用 gcc 将 C 文件生成可执行文件时，先调动 cc1 和 as，得到可重定位的目标文件。最后会调动 collect2 将 crt1.o、crti.o、crtbegin.o、crtend.o、crtn.o 、库文件 libc、libgcc、libgcc_s、我们自己的可重定位的目标文件进行链接。collect2 调动 ld，真正进行链接的是 ld。

我们先看 crt1.o：

``` console
$ nm /usr/lib/crt1.o
00000000 R _IO_stdin_used
00000000 D __data_start
         U __libc_csu_fini
         U __libc_csu_init
         U __libc_start_main
00000000 R _fp_hw
00000000 T _start
00000000 W data_start
         U main
$ objdump -d /usr/lib/crt1.o
/usr/lib/crt1.o:     file format elf32-i386


Disassembly of section .text:

00000000 <_start>:
   0:   31 ed                   xor    %ebp,%ebp
   2:   5e                      pop    %esi
   3:   89 e1                   mov    %esp,%ecx
   5:   83 e4 f0                and    $0xfffffff0,%esp
   8:   50                      push   %eax
   9:   54                      push   %esp
   a:   52                      push   %edx
   b:   68 00 00 00 00          push   $0x0
  10:   68 00 00 00 00          push   $0x0
  15:   51                      push   %ecx
  16:   56                      push   %esi
  17:   68 00 00 00 00          push   $0x0
  1c:   e8 fc ff ff ff          call   1d <_start+0x1d>
  21:   f4                      hlt
  22:   90                      nop
  23:   90                      nop
$ readelf -a /usr/lib/crt1.o
...
Relocation section '.rel.text' at offset 0x49c contains 4 entries:
 Offset     Info    Type            Sym.Value  Sym. Name
...
00000018  00000d01 R_386_32          00000000   main
...
```

call指令前面的那条push $0x0指令其实想把main这个符号所代表的地址压栈，但不知道这个地址是多少，因为这个符号在另一个目标文件中定义，到链接时才能确定其地址，所以在指令中暂时写成0x0。

我们将 main.c 编译成可执行文件 main，main 的指令如下：

``` console
$ gcc -c main.c
$ gcc main.o -o main
$ objdump -d main
...
Disassembly of section .text:

08048300 <_start>:
 8048300:   31 ed                   xor    %ebp,%ebp
 8048302:   5e                      pop    %esi
 8048303:   89 e1                   mov    %esp,%ecx
 8048305:   83 e4 f0                and    $0xfffffff0,%esp
 8048308:   50                      push   %eax
 8048309:   54                      push   %esp
 804830a:   52                      push   %edx
 804830b:   68 10 84 04 08          push   $0x8048410
 8048310:   68 20 84 04 08          push   $0x8048420
 8048315:   51                      push   %ecx
 8048316:   56                      push   %esi
 8048317:   68 e5 83 04 08          push   $0x80483e5
 804831c:   e8 c7 ff ff ff          call   80482e8 <__libc_start_main@plt>
 8048321:   f4                      hlt
...
080483b4 <bar>:
 80483b4:   55                      push   %ebp
 80483b5:   89 e5                   mov    %esp,%ebp
 80483b7:   83 ec 10                sub    $0x10,%esp
...
080483cb <foo>:
 80483cb:   55                      push   %ebp
 80483cc:   89 e5                   mov    %esp,%ebp
 80483ce:   83 ec 08                sub    $0x8,%esp
...
080483e5 <main>:
 80483e5:   55                      push   %ebp
 80483e6:   89 e5                   mov    %esp,%ebp
 80483e8:   83 ec 08                sub    $0x8,%esp
...
```

链接完成后，crt1.o中定义的符号_start和main.o中定义的符号bar、foo、main都合并到可执行文件的.text段中。符号main的地址是0x080483e5，因此_start中的push $0x0指令被链接器改成了push $0x80483e5。这个过程叫做符号解析。

crt1.o 引用了一个未定义符号__libc_start_main，这个符号在其他几个目标文件中也没有定义，生成的可执行文件中也没有定义。但是在共享库 libc 中有定义。根据[链接器的工作](#链接器的工作)中所说，链接共享库时，链接器只是确认可执行文件引用的某些符号在 libc 中有定义，并没有最终确定这些符号的地址，这些符号在可执行文件中仍然是未定义符号，要在运行时做动态链接。

所以链接器只是在可执行文件中记录了动态链接器的路径和共享库的路径。这样当可执行文件被加载后才能正常进行动态链接。

流程总结：进入 _start 后，一系列参数被压栈，main 的地址也被压栈，然后 call __libc_start_main@plt，调用 __libc_start_main 的时候需要动态链接。__libc_start_main 做初始化工作，然后根据栈中的 main 地址调用 main 函数，调用 main 函数时传 argc 和 argv 给 main 函数，main 函数可以用也可以不用。调用大概是这样 `exit(main(argc, argv));`。

关于进程如何终止请看[终止：进程正常或异常终止](#终止：进程正常或异常终止)。

## 去除符号信息的可执行文件

用strip命令去除可执行文件中的符号信息，这样可以有效减小文件的尺寸而不影响运行：

``` console
$ strip max
$ readelf -a max
...
```

注意不要对可重定位的目标文件和共享库使用strip命令，因为链接器需要利用可重定位的目标文件和共享库中的符号信息来做链接。

## 加载：可执行文件到进程

保存在硬盘上的程序是不能被CPU直接取指令执行的，操作系统在执行程序时会把它从硬盘拷贝到内存，这样CPU才能取指令执行，这个过程称为加载（Load）。

程序加载到内存之后，成为操作系统调度执行的一个任务，就称为进程（Process）。

> .section指示把代码划分成若干个段（Section），程序被操作系统加载执行时，每个段被加载到不同的地址，操作系统对不同的页面设置不同的读、写、执行权限。

### 加载器的工作

加载器把保存在硬盘上的可执行文件（hello）加载到内存，得到进程：

- 加载器把ELF文件看成是Segment的集合，加载器（Loader）根据可执行文件中的Segment信息加载运行这个程序

  对加载器来说，ELF 文件中的 Section Header Table 在加载过程中用不到，可有可无；program header table 中保存了所有segment的描述信息，通过 program header table 可以找到每个 segment 在文件中的位置。

- 动态链接

  1.  操作系统在加载执行可执行程序时，首先查看它有没有需要动态链接的未定义符号。

  2.  如果需要做动态链接，就查看这个程序指定了哪些共享库，以及用什么动态链接器来做动态链接。我们在链接时用-lc选项指定了共享库libc，用-dynamic-linker/lib/ld-linux.so.2指定了动态链接器，这些信息都会写到可执行文件中。

  3.  动态链接器加载共享库，在其中查找这些未定义符号的定义，完成链接过程。

  这上面的第 3 步描述有点问题，完整流程见 https://stackoverflow.com/a/50790258/14067245：

  The Linux kernel loads a.out into memory. It then examines PT_INTERP segment (if any).

  If that segment is not present, the binary is statically linked and the kernel transfers control to the `Elf{32,64}Ehdr.e_entry` (usually the `_start` routine).

  If the PT_INTERP segment is present, the kernel loads it into memory, and transfers control to it's .e_entry. It is here that the dynamic linking begins.

  The dynamic loader relocates itself, then looks in a.outs PT_DYNAMIC segment for instructions on what else is necessary.

  For example, it will usually find one or more DT_NEEDED entries -- shared libraries that a.out was directly linked against. The loader loads any such libraries, initializes them, and resolves any data references between them.

  IF a.outs PT_DYNAMIC has a DT_FLAGS entry, and IF that entry contains DF_BIND_NOW flag, then function references from a.out will also be resolved. Otherwise (and assuming that LD_BIND_NOW is not set in the environment), lazy PLT resolution will be performed (resolving functions as part of first call to any given function).

  .plt 协助完成动态链接。看 19.4.2。

  - 看可执行文件需要动态链接哪些共享库

    我们可以用ldd命令查看可执行文件依赖于哪些共享库：

    ``` console
    $ ldd main
            linux-gate.so.1 =>  (0x00fe2000)
            libstack.so => not found
            libc.so.6 => /lib/tls/i686/cmov/libc.so.6 (0x0065a000)
            /lib/ld-linux.so.2 (0x001de000)
    ```

    ldd模拟运行一遍main程序，在运行过程中做动态链接，从而得知这个程序依赖于哪些共享库，这些共享库都在什么路径下。

    我们在第18.2节讲过gcc调动ld做链接时用-dynamic-linker /lib/ld-linux.so.2选项指定动态链接器的路径，动态链接器它也像其他共享库一样加载到进程的地址空间中。

    而另外一个选项-lc只说明需要链接libc库，却没有指出libc库的完整路径，-lstack也是如此，共享库的路径需要在运行时由**动态链接器/lib/ld-linux.so.2去查找**。

    在上面的例子中，动态链接器找到libc的路径是/lib/tls/i686/cmov/ libc.so.6，而libstack的路径没有找到，无法完成链接。

    linux-gate.so.1这个共享库文件其实并不存在，它是由内核虚拟出来的，所以没有对应的路径。linux-gate.so.1负责处理一些特殊的系统调用，感兴趣的读者可以参考http://www.trilithium.com/ johan/2005/08/linux-gate/。

  - 动态链接器到哪里搜索共享库

    动态链接器查的是 soname。

    1.  首先在环境变量LD_LIBRARY_PATH保存的路径中查找。

        我们可以把这个环境变量设置成我们自己共享库的目录的绝对路径。这种方法只适合在开发调试中临时用一下，设置环境变量LD_LIBRARY_PATH通常是不推荐的，理由可以参考Why LD_LIBRARY_PATH is bad（http://xahlee.org/ UnixResource_dir/_/ldpath.html）。

    2.  然后从缓存文件/etc/ld.so.cache中查找。这个缓存文件是由ldconfig命令读取配置文件/etc/ld.so.conf生成的。

        我们可以把自己的共享库所在目录的绝对路径添加到 /etc/ld.so.conf，每个路径占一行。然后运行 ldconfig：`sudo ldconfig -v`。

        ldconfig命令除了处理/etc/ld.so.conf中配置的目录之外，还处理一些默认目录，如/lib、/usr/lib等，处理的过程主要是建立索引以便快速查找，处理之后生成/etc/ld.so.cache缓存文件，动态链接器就从这个缓存文件中搜索共享库。

        hwcap是x86平台Linux特有的一种机制，系统检测到当前平台是i686而不是i586或i486，所以在运行程序时使用i686的库，这样可以更好地发挥平台的性能，所以上面ldd命令的输出结果显示动态链接器搜索到的libc库是/lib/tls/i686/cmov/libc.so.6，而不是/lib/libc.so.6。

    3.  如果上述步骤都找不到，则到默认的系统库文件目录中查找，先是/lib然后是/usr/lib。

        我们也可以把自己的共享库直接拷贝到 /usr/lib 或 /lib。

    4.  可执行文件中的 RPATH

        其实我们可以直接把自己的共享库所在目录的绝对路径在调用 gcc 的时候写到可执行文件中。

        ``` console
        gcc main.c -g -L. -lstack -o main -Wl,-rpath,/home/juhan/19/19.4
        ```

        注意选项-Wl,-rpath,/home/akaedu/testdir，-Wl表示gcc传给链接器的选项，在这个例子中传给链接器的选项是-rpath /home/akaedu/testdir。可以看到readelf输出的.dynamic段的信息中多了一条rpath记录。还可以看出，可执行文件运行时需要哪些共享库也都记录在.dynamic段中。

        当然rpath这种办法也是不推荐的，把共享库的搜索路径写到可执行文件中也是一种硬编码的做法。

### 内存布局

- 环境变量

  环境变量（Environment Variable）是进程运行时保存在内存中的一组字符串，每个字符串都是“key=value”这样的形式，key是变量名，value是变量的值。

  设置一个进程的环境变量有两种方法：

  1.  ``` console
      KEY=VALUE ./main
      ```

      这种方法表示只有当前创建的main进程才获得这个环境变量，Shell进程本身不保存这个环境变量，以后执行的其他命令也不会获得它。

  2.  ``` console
      export KEY=VALUE
      ./main
      ```

      第一条命令在当前Shell进程中设置一个环境变量LD_LIBRARY_PATH=/home/ akaedu/testdir，一旦在Shell进程中设置了环境变量，以后每次执行命令时Shell进程都会把自己的环境变量传给新创建的进程，所以第二条命令创建的进程main就会获得这个环境变量。

- 栈空间

  在执行程序时，操作系统为进程分配一块栈空间来保存函数栈帧，esp寄存器总是指向栈顶。

  在x86平台上这个栈是从高地址向低地址增长的，我们知道每次调用一个函数都要分配一个栈帧来保存参数和局部变量

  为了安全性，Linux内核为每个新进程指定的栈空间起始地址是随机的，所以每次运行这个程序得到的地址都不一样，但通常都是0xbf??????这样的一个地址。

- 内存布局

  见[进程地址空间](#part-2/chapter-19.md#进程地址空间)。

## 执行：CPU 执行可执行文件

可执行文件已经被加载到内存中了。CPU 只重复做一件事：根据程序计数器寄存器中保存的下一条指令地址，从内存中取得这条指令，执行这条指令。

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

## 终止：进程正常或异常终止

### 正常终止（退出）

- main 函数中正常使用 return

  main 函数如果正常使用 return 语句返回，那么这个返回值会被它的调用者接收到。main 函数大概像这样被调用 `exit(main(argc, argv));`。

- exit 库函数

  exit 是 libc 的库函数，它首先做一些清理工作，然后调用下面要说的 `_exit(2)` 终止进程，main 函数的返回值最终被传给 `_exit(2)` 系统调用函数，成为进程的退出状态。

- main 函数中可以直接调用 exit 函数

  我们也可以在main函数中直接调用exit函数终止进程而不返回到启动例程，例如：

  ``` console
  #include <stdio.h>
  int main(void)
  {
      exit(4);
  }
  ```

  使用时要包含头文件stdlib.h。

  在 main 函数中使用 exit(4) 和 return 4 效果一样。

- main 函数也可以用 _exit 函数退出

  在 C 程序中也可以调用 `_exit` 函数退出（需要包含头文件 unistd.h），它是 `_exit` 系统调用的简单包装。

  它可能是一个 C 函数，其中内嵌了movl $1,%eax、movl ?, %ebx和int $0x80三条指令（稍后在第18.5节介绍这种语法）。

  它也可能是纯用汇编写的，但要符合C编译器的Calling Convention，这样才能当成一个C函数来调用。

  这种对int $0x80指令简单包装的C函数通常也称为系统调用，在Man Page中系统调用位于第2个Section，例如_exit(2)，而库函数位于第3个Section，例如exit(3)。exit(3) 先做一些清理工作，然后调用_exit(2)进内核终止当前进程。

  unistd.h 中声明的函数不是 C 标准库函数，是 POSIX 标准定义的 UNIX 系统函数。在 libc 中实现。

- 正常终止

  一个进程调用 exit(3) 或 _exit(2) 终止，或者从main函数返回而终止，都属于正常终止（Normal Termination），也称为退出（Exit）。

### 退出状态

进程执行完成后，可以查看进程的退出状态：

``` console
$ ./a.out
$ echo $?
4
```

按照惯例，退出状态为 0 表示程序执行成功，退出状态非 0 表示出错。

退出状态只有 8 位，而且被 Shell 解释成无符号数，所以如果 return -1，`$ echo $?` 会得到 255。

### 异常中止：进程收到信号，内核强行终止进程

但并非所有的进程终止都是正常的，比如按Ctrl+C组合键终止一个进程，或者运行时产生段错误，或者用kill命令终止一个进程，这几种情况本质上都是进程收到一个信号然后内核把进程强行终止掉了，进程并没有执行_exit系统调用，也没有退出状态，这称为异常终止（Abnormal Termination）。关于信号请查阅参考文献[31]的第10章。
