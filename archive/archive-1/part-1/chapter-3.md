# 3. 简单函数

## 3.1 数学函数

### 函数 3.1

``` c
#include <math.h>
#include <stdio.h>

int main(void)
{
    double pi = 3.1416;
    printf("sin(pi/2)=%f\nln1=%f\n", sin(pi/2), log(1.0));
    return 0;
}
```

编译运行这个程序，结果如下：

``` shell
$ gcc main.c -lm
$ ./a.out
sin(pi/2)=1.000000
ln1=0.000000
```

- 参数、函数、函数调用 3.1

  在 C 语言的术语中，`1.0` 是参数（Argument），`log` 是函数（Function），`log(1.0)` 是函数调用（Function Call）。

- 函数调用也是表达式，由函数调用运算符和两个操作数组成 3.1

  `sin(pi/2)` 和 `log(1.0)` 这两个函数调用在我们的`printf` 语句中处于什么位置呢？在上一章讲过，这应该是写**表达式**的位置。

  - 函数名是操作数（函数名的类型是函数类型），参数也是操作数

    因此函数调用也是一种表达式，这个表达式由**函数调用运算符**（`()`括号）和两个**操作数**组成，操作数 `log` 是一个函数名（Function Designator），它的类型是一种函数类型（Function Type），操作数 `1.0` 是 `double` 型的。

  - 函数调用表达式的值为函数返回值

    `log(1.0)` 这个表达式的值就是对数运算的结果，也是 `double` 型的，在 C 语言中函数调用表达式的值称为函数的返回值（Return Value）。

- 新学的语法规则 3.1

  ```
  表达式 → 函数名
  表达式 → 表达式(参数列表)
  参数列表 → 表达式, 表达式, ...
  ```

  所以 `printf` 也是一个函数，上例中的 `printf("sin(pi/2)=%f\nln1=%f\n", sin(pi/2), log(1.0))` 是带三个参数的函数调用，而函数调用也是一种表达式，因此 `printf` 语句也是表达式语句的一种。

- Side Effect 3.1

  但是 `printf` 感觉不像一个数学函数，为什么呢？

  因为像 `log` 这种函数，我们传进去一个参数会得到一个返回值，我们调用 `log` 函数就是为了得到它的返回值，至于 `printf`，我们并不关心它的返回值（事实上它也有返回值，表示实际打印的字符数），我们调用 `printf` 不是为了得到它的返回值，而是为了利用它所产生的副作用（Side Effect）打印。

  C 语言的函数可以有 Side Effect，这一点是它和数学函数在概念上的根本区别。

  把赋值运算符看作函数调用运算符，那么赋值表达式也是有 Side Effect 的。如果把 `a = b` 这个赋值表达式看成函数调用表达式，返回值就是所赋的值，既是 `b` 的值也是 `a` 的值，但除此之外还产生了 Side Effect，就是变量 `a` 被改变了。

  改变计算机存储单元里的数据或者做输入输出操作都算 Side Effect。

### 头文件、库函数、库文件 3.1

- 如何包含头文件

  程序第一行的 `#` 号（Pound Sign，Number Sign 或 Hash Sign）和 `include` 表示包含一个头文件（Header File），后面尖括号（Angel Bracket）中就是文件名（这些头文件通常位于 `/usr/include` 目录下）。

- 头文件声明了库函数，要用库函数必须包含头文件

  头文件中声明了我们程序中使用的**库函数**，根据先声明后使用的原则，要使用 `printf` 函数必须包含 `stdio.h`，要使用数学函数必须包含 `math.h`。

  如果什么库函数都不使用就不必包含任何头文件，例如写一个程序 `int main(void){int a;a=2;return 0;}`，不需要包含头文件就可以编译通过，当然这个程序什么也做不了。

- 库函数在库文件里

  使用 `math.h` 中声明的**库函数**还有一点特殊之处，gcc 命令行必须加 `-lm` 选项，因为数学函数位于 `libm.so` 库文件中（这些库文件通常位于 `/lib` 目录下），`-lm` 选项告诉编译器，我们程序中用到的数学函数要到这个库文件里找。

  本书用到的大部分库函数（例如 `printf`）位于 `libc.so` 库文件中，使用 `libc.so` 中的库函数在编译时不需要加 `-lc` 选项，当然加了也不算错，因为这个选项是 `gcc` 的默认选项。

  关于头文件和库函数目前理解这么多就可以了，到第 20 章「链接详解」再详细解释。

### C 标准由 C 语法和 C 标准库组成 3.1

C 标准主要由两部分组成，一部分描述 C 的语法，另一部分描述 C 标准库。

- C 标准库定义了一组标准头文件

  C 标准库定义了一组**标准头文件**，每个头文件中包含一些相关的函数、变量、类型声明和宏定义。

- 支持 C 除了实现 C 编译器还要实现 C 标准库

  要在一个平台上支持 C 语言，不仅要实现 C 编译器，还要实现 C 标准库，这样的实现才算符合 C 标准。

  不符合 C 标准的实现也是存在的，例如很多单片机的 C 语言开发工具中只有 C 编译器而没有完整的 C 标准库。

- C 函数库

  - `glibc` 函数库包括 C 标准库实现和其他系统函数

    在 Linux 平台上最广泛使用的 C 函数库是 `glibc`，其中包括 C 标准库的实现，也包括本书第三部分介绍的所有系统函数。

    几乎所有 C 程序都要调用 `glibc` 的库函数，所以 `glibc` 是 Linux 平台 C 程序运行的基础。

  - `glibc` 提供头文件和库文件

    `glibc` 提供一组头文件和一组库文件。

  - `libc.so` 库文件、`libm.so` 库文件、`libpthread.so` 库文件

    最基本、最常用的 C 标准库函数和系统函数在 `libc.so` 库文件中，几乎所有 C 程序的运行都依赖于 `libc.so`，有些做数学计算的 C 程序依赖于 `libm.so`，以后我们还会看到多线程的 C 程序依赖于 `libpthread.so`。

    以后我说 `libc` 时专指 `libc.so` 这个库文件，而说`glibc` 时指的是 `glibc` 提供的所有库文件。

  - 别的函数库

    `glibc` 并不是 Linux 平台唯一的基础 C 函数库，也有人在开发别的 C 函数库，比如适用于嵌入式系统的 `uClibc`。

## 3.2 自定义函数

我们不仅可以调用 C 标准库提供的函数，也可以定义自己的函数。

### `main` 3.2

事实上我们已经定义过 `main` 函数了。

``` c
int main(void)
{
    int hour = 11;
    int minute = 59;
    printf("%d and %d hours\n", hour, minute / 60);
    return 0;
}
```

`main` 函数的特殊之处在于执行程序时它**自动被操作系统调用**，操作系统就认准了 `main` 这个名字，除了名字特殊之外，`main` 函数和别的函数没有区别。

### 自定义函数语法规则 3.2

我们对照着 `main` 函数的定义来看语法规则：

```
函数定义 → 返回值类型 函数名(参数列表) 函数体
函数体 → { 语句列表 }
语句列表 → 语句列表项 语句列表项 ...
语句列表项 → 语句
语句列表项 → 变量声明、类型声明或非定义的函数声明
非定义的函数声明 → 返回值类型 函数名(参数列表);
```

- 空的参数列表

  由于我们定义的 `main` 函数不带任何参数，参数列表应写成 `void`。

- 函数体由语句和声明组成

  函数体可以由若干条语句和声明组成，C89 要求所有声明写在所有语句之前（本书的示例代码都遵循这一规定），而 C99 的新特性允许语句和声明按任意顺序排列，只要每个标识符都遵循先声明后使用的原则就行。

- `main` 的返回值

  `main` 函数的返回值是 `int` 型的，`return 0;` 这个语句表示返回值是 0，`main` 函数的返回值是返回给**操作系统**看的，因为 `main` 函数是**被操作系统调用**的，通常程序执行成功就返回 0，在执行过程中出错就返回一个非零值。

  比如我们将 `main` 函数中的 `return` 语句改为 `return 4;` 再执行它，执行结束后可以在 Shell 中看到它的退出状态（Exit Status）：

  ``` shell
  $ ./a.out 
  11 and 0 hours
  $ echo $?
  4
  ```

  `$?` 是 Shell 中的一个特殊变量，表示上一条命令的退出状态。

- `main` 的返回值类型和参数列表

  - 不写返回值类型就返回 `int`，不写参数列表就表示参数列表是什么都行

    K&R 书上的 `main` 函数定义写成 `main(){...}` 的形式，不写返回值类型也不写参数列表，这是 Old Style C 的风格。

    Old Style C 规定不写返回值类型就表示返回 `int` 型，不写参数列表就表示参数类型和个数没有明确指出。这种宽松的规定使编译器无法检查程序中可能存在的 Bug，增加了调试难度，不幸的是现在的 C 标准为了兼容旧的代码仍然保留了这种语法，但读者绝不应该继续使用这种语法。

  - `main` 参数列表的两种标准写法

    其实操作系统在调用 `main` 函数时是传参数的，`main` 函数最标准的形式应该是 `int main(int argc, char *argv[])`，在第 6 节「指向指针的指针与指针数组」详细介绍。

    C 标准也允许 `int main(void)` 这种写法，如果不使用系统传进来的两个参数也可以写成这种形式。但除了这两种形式之外，定义 `main` 函数的其它写法都是错误的或不可移植的。

- `void` 返回值类型

  ``` c
  void newline(void)
  {
      printf("\n");
  }
  ```

  函数返回值类型为 `void`，则函数体内不用写 `return` 语句，表示函数没有返回值。

  其实准确来说，任何函数调用表达式都有值和类型，`newline()` 这样没有返回值的函数调用表达式，语法上规定它的类型是 `void`，值未知，这样任何表达式都有值，不必考虑特殊情况，编译器的语法解析比较容易实现；然后从语义上规定 `void` 类型的表达式不能参与运算，因此`newline() + 1` 这样的表达式不能通过语义检查，从而兼顾了语法上的一致和语义上的不矛盾。

- 函数声明、函数定义、函数原型（Prototype）

  - 函数原型

    比如 `void new line(void)` 这一行，声明了一个函数的名字、参数类型和个数、返回值类型，这称为函数原型。

  - 函数声明和函数定义

    在代码中可以单独写一个函数原型，后面加 `;` 号结束，而不写函数体，例如：

    ```c
    void newline(void);
    ```

    这种写法只能叫**函数声明**而不能叫**函数定义**，只有带**函数体**的声明才叫定义。

    上一章讲过，只有分配**存储空间**的变量声明才叫变量定义，其实函数也是一样，编译器只有见到函数定义才会生成指令，而指令在程序运行时当然也要占存储空间。

  - 函数原型（在函数声明和函数定义里面）必须出现在函数调用之前

    那么没有函数体的函数声明有什么用呢？它为编译器提供了有用的信息，编译器在翻译代码的过程中，只有见到函数原型（不管带不带函数体）之后才知道这个函数的名字、参数类型和返回值，这样碰到函数调用时才知道怎么生成相应的指令，所以函数原型必须出现在函数调用之前，这也是遵循“先声明后使用”的原则。

  - 不是所有的函数声明都包含了完整的函数原型

    由于有 Old Style C语法的存在，并非所有函数声明都包含完整的函数原型，例如 `void threeline();` 这个声明并没有明确指出参数类型和个数，所以不算函数原型，这个声明提供给编译器的信息只有函数名和返回值类型。

    如果在这样的声明之后调用函数，编译器不知道参数的类型和个数，就不会做语法检查，所以很容易引入 Bug。

    读者需要了解这个知识点以便维护别人用 Old Style C 风格写的代码，但绝不应该按这种风格写新的代码。

  - 调用函数以前没有函数声明（隐式声明）

    如果在调用函数之前没有声明会怎么样呢？有的读者也许碰到过这种情况，我可以解释一下，但绝不推荐这种写法。

    ``` c
    #include <stdio.h>

    int main(void)
    {
        printf("Three lines:\n");
        threeline();
        printf("Another three lines.\n");
        threeline();
        return 0;
    }

    void newline(void)
    {
        printf("\n");
    }

    void threeline(void)
    {
        newline();
        newline();
        newline();
    }
    ```

    编译时会报警告：

    ``` shell
    $ gcc main.c
    main.c:17: warning: conflicting types for ‘threeline’
    main.c:6: warning: previous implicit declaration of ‘threeline’ was here
    ```

    但仍然能编译通过，运行结果也对。

    这里涉及到的规则称为函数的隐式声明（Implicit Declaration），在 `main` 函数中调用 `threeline` 时并没有声明它，编译器认为此处隐式声明了 `int threeline(void);`，隐式声明的函数返回值类型都是 `int`，由于我们调用这个函数时没有传任何参数，所以编译器认为这个隐式声明的参数类型是 `void`，这样函数的参数和返回值类型都确定下来了，编译器根据这些信息为函数调用生成相应的指令。

    然后编译器接着往下看，看到 `threeline` 函数的原型是 `void threeline(void)`，和先前的隐式声明的返回值类型不符，所以报警告。

    好在我们也没用到这个函数的返回值，所以执行结果仍然正确。

## 3.3 形参和实参

### 带参数的函数

下面我们定义一个带参数的函数，我们需要在**函数定义**中**指明**参数的个数和每个参数的类型，定义参数就像定义变量一样，需要为每个参数指明类型，参数的命名也要遵循标识符命名规则。

例如：

``` c
#include <stdio.h>

void print_time(int hour, int minute)
{
	  printf("%d:%d\n", hour, minute);
}

int main(void)
{
	  print_time(23, 59);
	  return 0;
}
```

- 定义参数不能把相同类型变量列在一起

  需要注意的是，定义变量时可以把相同类型的变量列在一起，而定义参数却不可以，例如下面这样的定义是错的：

  ``` c
  void print_time(int hour, minute)
  {
      printf("%d:%d\n", hour, minute);
  }
  ```

- 形参和实参

  我们调用 `print_time(23, 59)` 时，函数中的参数 `hour` 就代表 `23`，参数 `minute` 就代表`59`。

  确切地说，当我们讨论函数中的 `hour` 这个参数时，我们所说的“参数”是指形参（Parameter），当我们讨论传一个参数 `23` 给函数时，我们所说的“参数”是指实参（Argument）。

  - 形参相当于函数中定义的变量

    记住这条基本原理：形参相当于函数中定义的变量，调用函数传递参数的过程相当于定义形参变量并且用实参的值来初始化。

  - Call by Value

    C 语言的这种传递参数的方式称为 Call by Value。

    在调用函数时，每个参数都需要得到一个值，函数定义中有几个形参，在调用时就要传几个实参，不能多也不能少，每个参数的类型也必须对应上。

  - 可变参数

    肯定有读者注意到了，为什么我们每次调用 `printf` 传的实参个数都不一样呢？因为 C 语言规定了一种特殊的参数列表格式，用命令 `man 3 printf` 可以查看到 `printf` 函数的原型：

    ``` c
    int printf(const char *format, ...);
    ```

    第一个参数是 `const char *` 类型的，后面的 `...` 可以代表 0 个或任意多个参数，这些参数的类型也是不确定的，这称为可变参数（Variable Argument），第 6 节「可变参数」将会详细讨论这种格式。

  - 接口

    总之，每个函数的原型都明确规定了返回值类型以及参数的类型和个数，即使像printf这样规定为“不确定”也是一种明确的规定，调用函数时要严格遵守这些规定。

    有时候我们把函数叫做接口（Interface），调用函数就是使用这个接口，使用接口的前提是必须和接口保持一致。

### Man Page

Man Page 是 Linux 开发最常用的参考手册，由很多页面组成，每个页面描述一个主题，这些页面被组织成若干个 Section。

- FHS 规定了 Man Page 的 Section

  FHS（Filesystem Hierarchy Standard）标准规定了 Man Page 各 Section 的含义如下：

  ![man-page-section](image/man-page-section.png)

- 用户命令和系统管理命令

  注意区分用户命令和系统管理命令，用户命令通常位于 `/bin` 和 `/usr/bin` 目录，系统管理命令通常位于 `/sbin` 和 `/usr/sbin` 目录，一般用户可以执行用户命令，而执行系统管理命令经常需要 `root `权限。

- 系统调用和库函数

  系统调用和库函数的区别将在第 2 节「main 函数和启动例程」说明。

- 页面重名

  Man Page 中有些页面有重名，比如敲 `man printf` 命令看到的并不是 C 函数 `printf`，而是位于第 1 个 Section 的系统命令 `printf`。

  要查看位于第 3 个 Section 的 `printf` 函数应该敲 `man 3 printf`。

  也可以敲 `man -k printf` 命令搜索哪些页面的主题包含 `printf` 关键字。

  本书会经常出现类似 `printf(3)` 这样的写法，括号中的 3 表示 Man Page 的第 3 个 Section，或者表示“我这里想说的是 `printf` 库函数而不是 `printf` 命令”。

## 3.4 全局变量、局部变量和作用域

### 局部变量

我们把函数中定义的变量称为局部变量（Local Variable），由于**形参**相当于函数中定义的变量，所以形参也是一种局部变量。

- 局部的两层含义

  在这里「局部」有两层含义：

  - 一个函数中定义的变量不能被另一个函数使用

    一个函数中定义的变量不能被另一个函数使用。例如 `print_time` 中的 `hour` 和 `minute` 在 `main` 函数中没有定义，不能使用，同样 `main` 函数中的局部变量也不能被 `print_time` 函数使用。

    如果这样定义：

    ``` c
    void print_time(int hour, int minute)
    {
        printf("%d:%d\n", hour, minute);
    }

    int main(void)
    {
        int hour = 23, minute = 59;
        print_time(hour, minute);
        return 0;
    }
    ```

    `main` 函数中定义了局部变量 `hour`，`print_time` 函数中也有参数 `hour`，虽然它们名称相同，但仍然是两个不同的变量，代表不同的存储单元。

    `main` 函数的局部变量 `minute` 和 `print_time` 函数的参数 `minute` 也是如此。

  - 每次调用函数时局部变量都表示不同的存储空间

    每次调用函数时局部变量都表示不同的存储空间。局部变量在每次函数调用时分配存储空间，在每次函数返回时释放存储空间，例如调用 `print_time(23, 59)` 时分配 `hour` 和 `minute` 两个变量的存储空间，在里面分别存上 23 和 59，函数返回时释放它们的存储空间，下次再调用 `print_time(12, 20)` 时又分配 `hour` 和 `minute` 的存储空间，在里面分别存上 12 和 20。

    有时候调用函数的结果可能会让人觉得函数的局部变量有一直存在的固定存储空间，但这是错觉。等学到例 19.1「研究函数的调用过程」就能解释这些现象了。

### 全局变量

与局部变量的概念相对的是全局变量（Global Variable），全局变量定义在所有的函数体之外，它们在程序开始运行时分配存储空间，在程序结束时释放存储空间，在任何函数中都可以访问全局变量，例如：

``` c
#include <stdio.h>

int hour = 23, minute = 59;

void print_time(void)
{
	  printf("%d:%d in print_time\n", hour, minute);
}

int main(void)
{
	  print_time();
	  printf("%d:%d in main\n", hour, minute);
	  return 0;
}
```

- 尽量不要使用全局变量，会引入 Bug

  正因为全局变量在任何函数中都可以访问，所以在程序运行过程中全局变量被读写的顺序从源代码中是看不出来的，源代码的书写顺序并不能反映函数的调用顺序。

  程序出现了 Bug 往往就是因为在某个不起眼的地方对全局变量的读写顺序不正确，如果代码规模很大，这种错误是很难找到的。

- 局部变量对比全局变量要好得多

  而对局部变量的访问不仅局限在一个函数内部，而且局限在一次函数调用之中，从函数的源代码很容易看出访问的先后顺序是怎样的，所以比较容易找到 Bug。

  因此，虽然全局变量用起来很方便，但一定要慎用，能用函数传参代替的就不要用全局变量。

### 全局变量和局部变量重名

如果全局变量和局部变量重名了会怎么样呢？如果上面的例子改为：

``` c
#include <stdio.h>

int hour = 23, minute = 59;
int x = 10;

void print_time(void)
{
    printf("%d:%d in print_time\n", hour, minute);
}

int main(void)
{
    int hour = 0, minute = 30;
    print_time();
    printf("%d:%d in main\n", hour, minute);
    printf("x = %d\n", x);
    return 0;
}
```

- 作用域

  在 C 语言中每个标识符都有特定的作用域。

  全局变量是定义在所有函数体之外的标识符，它的作用域从定义的位置开始直到源文件结束，而 `main` 函数局部变量的作用域仅限于 `main` 函数之中。

- 标识符的值用大小纸的方法找

  如上图所示，设想整个源文件是一张大纸，也就是全局变量的作用域，而 `main` 函数是盖在这张大纸上的一张小纸，也就是 `main` 函数局部变量的作用域。

  在小纸上用到标识符 `hour` 和 `minute` 时应该参考小纸上的定义，因为大纸（全局变量的作用域）**被盖住**了，如果在小纸上用到某个标识符却没有找到它的定义，那么再去翻看下面的大纸上有没有定义，例如上图中的变量 `x`。

### 全局变量和局部变量的 Initializer

到目前为止我们在初始化一个变量时都是用常量做 Initializer，其实也可以用表达式做 Initializer，但要注意一点：局部变量可以用类型相符的任意表达式来初始化，而全局变量只能用常量表达式（Constant Expression）初始化。

- 全局变量只能用常量表达式初始化

  - 全局变量的初始值在编译时就要计算出来

    例如，全局变量 `pi` 这样初始化是合法的：

    ``` c
    double pi = 3.14 + 0.0016;
    ```

    但这样初始化是不合法的：

    ``` c
    double pi = acos(-1.0);
    ```

    然而局部变量这样初始化却是可以的。

    程序**开始运行**时要用适当的值来初始化全局变量，所以初始值必须保存在**编译生成**的可执行文件中，因此初始值在**编译时**就要计算出来，然而上面第二种 Initializer 的值必须在**程序运行时**调用 `acos` 函数才能得到，所以不能用来初始化全局变量。

    请注意区分编译时和运行时这两个概念。

  - 全局变量只能用常量表达式初始化，有些非常量表达式就算可以在编译时计算出来也不行

    为了**简化**编译器的实现，C 语言从语法上规定全局变量只能用常量表达式来初始化，因此下面这种全局变量初始化是不合法的：

    ``` c
    int minute = 360;
    int hour = minute / 60;
    ```

    虽然在编译时计算出 `hour` 的初始值是可能的，但是 `minute / 60` 不是常量表达式，不符合语法规定，所以编译器不必想办法去算这个初始值。

### 全局变量和局部变量定义时不初始化

- 全局变量初始值是 0，局部变量初始值不确定

  如果全局变量在定义时不初始化则初始值是 0，如果局部变量在定义时不初始化则初始值是不确定的。

- 局部变量在使用前一定要先赋值

  所以，局部变量在使用之前一定要先赋值，如果基于一个不确定的值做后续计算肯定会引入 Bug。

### 函数体中的函数声明和函数定义

- 函数体中的函数声明

  从第 2 节「自定义函数」介绍的语法规则可以看出，非定义的函数声明也可以写在局部作用域中，例如：

  ``` c
  int main(void)
  {
      void print_time(int, int);
      print_time(23, 59);
      return 0;
  }
  ```

  这样声明的标识符 `print_time` 具有局部作域，只在 `main` 函数中是有效的函数名，出了 `main` 函数就不存在 `print_time` 这个标识符了。

- 函数声明可以只写参数类型不写名字

  写非定义的函数声明时参数可以只写类型而不起名，例如上面代码中的 `void print_time(int, int);`，只要告诉编译器参数类型是什么，编译器就能为 `print_time(23, 59)` 函数调用生成正确的指令。

- 函数体中的函数定义

  另外注意，虽然在一个函数体中可以声明另一个函数，但不能定义另一个函数，C 语言不允许嵌套定义函数。

  但 gcc 的扩展特性允许嵌套定义函数，本书不做详细讨论。









- 类型转换 2.5

  回到先前的例子，要得到更精确的结果可以这样：

  ``` c
  printf("%d hours and %d percent of an hour\n", hour, minute * 100 / 60);
  printf("%d and %f hours\n", hour, minute / 60.0);
  ```

  在第二个 `printf` 中，表达式是 `minute / 60.0`，60.0 是 `double` 型的，`/` 运算符要求左右两边的操作数类型一致，而现在并不一致。

  C 语言规定了一套隐式类型转换规则，在这里编译器自动把左边的 `minute` 也转成 `double` 型来计算，整个表达式的值也是 `double` 型的，在格式化字符串中应该用 `%f` 转换说明与之对应。

  本来编程语言作为一种形式语言要求有简单而严格的规则，自动类型转换规则不仅很复杂，而且使 C 语言的形式看起来也不那么严格了，C 语言这么设计是为了书写程序简便而做的折衷，有些事情编译器可以自动做好，程序员就不必每次都写一堆繁琐的转换代码。然而 C 语言的类型转换规则非常难掌握，本书的前几章会尽量避免类型转换，到第 3 节「类型转换」再集中解决这个问题。
