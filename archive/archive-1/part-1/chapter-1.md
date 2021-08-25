# 1. 程序的基本概念

## 1.1 程序和编程语言

### 组成程序的基本操作

程序由一系列基本操作组成，基本操作有以下几类：

- 输入（Input）

  从键盘、文件或者其他设备获取数据。

- 输出（Output）

  把数据显示到屏幕，或者存入一个文件，或者发送到其他设备。

- 基本运算

  最基本的数据访问和数学运算（加减乘除）。

- 测试和分支

  测试某个条件，然后根据不同的测试结果执行不同的后续操作。

- 循环

  重复执行一系列操作。

### 指令与语句

- 指令

  机器语言（Machine Language）和汇编语言（Assembly Language）属于低级语言，直接用计算机指令（Instruction）编写程序。

  <span class="thoughts">和组成程序的五种基本操作对应，基本指令也分输入、输出、基本运算、测试分支和循环。</span>

  不同的计算机体系结构有不同的指令集（Instruction Set），可以识别的机器指令格式是不同的。

- 语句

  C、C++、Java、Python 等属于高级语言，用语句（Statement）编写程序。

  语句也分为输入、输出、基本运算、测试分支和循环等几种，和指令有直接的对应关系。

- 指令和语句的关系

  语句是计算机指令的抽象表示。

  <span class="thoughts">计算机只能执行低级语言中的指令（汇编语言的指令要先转成机器码才能执行），高级语言要执行就必须先翻译成低级语言，翻译的方法有两种——编译和解释。</span>

  - 编译：编译器把源代码的语句编译为指令，存放在可执行文件中，可执行文件再被操作系统加载执行

    C 语言的语句和低级语言的指令之间不是简单的一一对应关系，一条 `a=b+1;` 语句要翻译成三条汇编或机器指令，这个过程称为编译（Compile），由编译器（Compiler）来完成。

    - 编写、编译和执行一个 C 程序的步骤

      1.  用文本编辑器写一个 C 程序，然后保存成一个文件，例如 `program.c`（通常 C 程序的文件名后缀是 `.c`），这称为源代码（Source Code）或源文件。
      2.  运行编译器对它进行编译，编译的过程并不执行程序，而是把源代码全部翻译成机器指令，再加上一些**描述信息**，生成一个新的文件，例如 `a.out`，这个文件称为可执行文件（Executable）。
      3.  可执行文件可以被操作系统（Operating System）加载（Load）执行，计算机执行该文件中由编译器生成的指令。

  - 解释：操作系统加载执行解释器这个可执行文件，对脚本逐行进行解释执行

    有些高级语言写的程序不需要经过编译这个步骤，而是以解释的方式执行，解释执行（Interpret）的程序通常又叫做**脚本**（Script），解释执行的过程和 C 语言的编译执行过程很不一样。

    例如编写一个 Shell 脚本 `script.sh`，内容如下：

    ``` shell-script
    #! /bin/sh
    VAR=1
    VAR=$(($VAR+1))
    echo $VAR
    ```

    这里的 `/bin/sh` 称为解释器（Interpreter），解释器本身是一个可执行文件，而我们写的脚本 `script.sh` 却不是一个真正的可执行文件。

    解释器 `/bin/sh` 也是由 C 程序经过编译得到的包含机器指令的可执行文件，它被操作系统加载执行时，它所包含的机器指令指示它做这样的事情：把我们写的脚本 `script.sh` 当成数据文件读取，理解我们所写的每一行程序的意思，并一行一行地执行相应的操作。

  - 编译型语言和解释性语言的比较

    用解释型语言写的程序执行起来一定比编译型语言慢，因为用解释型语言写的程序每次执行时解释器都要把源代码分析一遍，理解程序员写这些代码是想要做什么，再去执行相应的操作，而对于编译型语言来说，这个步骤只需要做一次，就是编译器把源代码分析一遍生成可执行文件，而之后可执行文件在每次执行时就不需要再分析源代码了。

    用解释型语言写的程序也有它的优点：换个平台就可以直接执行，而不需要先编译一遍，此外，解释型语言写的程序调试起来比编译型语言方便得多。

  - 编译和解释结合起来

    既然解释型语言和编译型语言各有各的优点，有一些高级语言就把两者的优点结合起来，采用编译和解释相结合的方式执行。Java、Python、Perl 等编程语言都采用这种方式。

    以 Python 为例，程序员写的源代码文件（扩展名为 `.py`）在首次执行时被编译成字节码（Byte Code）文件（扩展名为 `.pyc`），以后每次执行该程序时 Python 解释器直接解释执行字节码文件，而不再编译源代码。

    字节码文件中也包含指令，但并非机器指令，而是 Python 语言定义的一种虚拟机（Virtual Machine）的指令。Python 语言在各种平台上都实现这种虚拟机，因此字节码文件从一种平台拷到另一种平台上仍然能被该平台的 Python 解释器解释执行。

  <span class="thoughts">总的来说，只有在谈论底层才会用指令，比如机器语言指令、汇编语言指令、指令集。而 C 这种高级语言的源代码是由许多条语句组成的。而这些语句可以通过编译器编译成底层的指令。</span>

## 1.2 自然语言和形式语言

### 自然语言和形式语言

自然语言（Natural Language）就是人类讲的语言，比如汉语、英语和法语。这类语言不是人为设计（虽然有人试图强加一些规则）而是自然进化的。

形式语言（Formal Language）是为了特定应用而人为设计的语言。例如数学家用的数字和运算符号、化学家用的分子式等。编程语言也是一种形式语言，是专门设计用来表达计算过程的形式语言。

### Syntax: Lexical and Grammar, Token and Structure

- 形式语言有 Syntax

  形式语言有严格的语法（Syntax）规则，例如，3+3=6 是一个语法正确的数学等式，而 3=+6$ 则不是，H<sub>2</sub>O 是一个正确的分子式，而 <sub>2</sub>Zz 则不是。

- Syntax 由 Token 的规则和 Structure 的规则组成

  语法规则是由符号（Token）和结构（Structure）的规则所组成的。

  Token 的概念相当于自然语言中的单词和标点、数学式中的数和运算符、化学分子式中的元素名和数字，例如 3=+6$ 的问题之一在于 $ 不是一个合法的数也不是一个事先定义好的运算符，而 <sub>2</sub>Zz 的问题之一在于没有一种元素的缩写是 Zz。

  结构是指 Token 的排列方式，3=+6$ 还有一个结构上的错误，虽然加号和等号都是合法的运算符，但是不能在等号之后紧跟加号，而 <sub>2</sub>Zz 的另一个问题在于分子式中必须把下标写在化学元素名称之后而不是前面。

- Token 的规则称为 Lexical rules，Structure 的规则称为 Grammar rules

  关于 Token 的规则称为词法（Lexical）规则，而关于结构的规则称为语法（Grammar）规则。

- Syntax 和 Grammar 都翻译成「语法」

  很不幸，Syntax 和 Grammar 通常都翻译成「语法」，这让初学者非常混乱，Syntax 的含义其实包含了 Lexical 和 Grammar 的规则，还包含一部分语义的规则，例如在 C 程序中变量应先声明后使用。

  即使在英文的文献中 Syntax 和 Grammar 也常混用，在有些文献中 Syntax 的含义不包括 Lexical 规则，只要注意上下文就不会误解。

### Parsing and Semantics

- Parsing 就是识别 Token 和分解 Structure 的过程

  当阅读一个自然语言的句子或者一种形式语言的语句时，你不仅要搞清楚每个词（Token）是什么意思，而且必须搞清楚整个句子的结构（Structure）是什么样的（在自然语言中你只是没有意识到，但确实这样做了，尤其是在读外语时你肯定也意识到了）。

  这个分析句子结构的过程称为解析（Parse）。

  例如，当你听到「The other shoe fell.」这个句子时，你理解 the other shoe 是主语而 fell 是谓语动词，一旦解析完成，你就搞懂了句子的意思。

- Semantics

  如果知道 shoe 是什么东西，fall 意味着什么，这句话是在什么上下文（Context）中说的，你还能理解这个句子主要暗示的内容——这属于语义（Semantic）的范畴。

## 1.3 程序的调试

- 编译时错误

  编译器只能翻译语法正确的程序，否则将导致编译失败，无法生成可执行文件。

  对于自然语言来说，一点**语法错误**不是很严重的问题，因为我们仍然可以读懂句子。而编译器就没那么宽容了，只要有哪怕一个很小的语法错误，编译器就会输出一条错误提示信息然后罢工，你就得不到你想要的结果。

  虽然大部分情况下编译器给出的错误提示信息能够指出错误代码的位置，但也有个别时候编译器给出的错误提示信息帮助不大，甚至会误导你。

  在开始学习编程的前几个星期，你可能会花大量的时间来纠正语法错误。等到有了一些经验之后，还是会犯这样的错误，不过会少得多，而且你能更快地发现错误原因。等到经验更丰富之后你就会觉得，语法错误是最简单最低级的错误，编译器的错误提示也就那么几种，即使错误提示是有误导的也能够立刻找出真正的错误原因是什么。相比下面两种错误，语法错误解决起来要容易得多。

- 运行时错误

  编译器检查不出这类错误，仍然可以生成可执行文件，但在运行时会出错而导致程序崩溃。

  对于我们接下来的几章将编写的简单程序来说，运行时错误很少见，到了后面的章节你会遇到越来越多的运行时错误。

  读者在以后的学习中要时刻注意区分**编译时**和**运行时**（Run-time）这两个概念，不仅在调试时需要区分这两个概念，在学习 C 语言的很多语法和规则时都需要区分这两个概念，有些事情在编译时做，有些事情则在运行时做。

- 逻辑错误和语义错误

  第三类错误是逻辑错误和语义错误。

  如果程序里有逻辑错误，编译和运行都会很顺利，看上去也不产生任何错误信息，但是程序没有干它该干的事情，而是干了别的事情。当然不管怎么样，计算机只会按你写的程序去做，问题在于你写的程序不是你真正想要的，这意味着**程序的意思**（即语义）是错的。

  找到逻辑错误在哪需要十分清醒的头脑，要通过观察程序的输出回过头来判断它到底在做什么。

## 1.4 第一个程序

### GCC 使用

- `-o` 1.4

  如果不想把文件名叫 `a.out`，可以用 gcc 的 `-o` 参数自己指定文件名。

  ``` shell
  $ gcc main.c -o main
  $ ./main
  Hello, world.
  ```

- `-Wall` 1.4

  一个好的习惯是打开 gcc 的 `-Wall` 选项，也就是让 gcc 提示所有的警告信息，不管是严重的还是不严重的，然后把这些问题从代码中全部消灭。

  ``` shell
  $ gcc -Wall main.c
  ```