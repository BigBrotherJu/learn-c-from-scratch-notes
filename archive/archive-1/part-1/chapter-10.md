# 10. gdb

## 10.1 单步执行和跟踪函数调用

- 编译时加上 -g 选项，生成的可执行文件才能用 gdb 调试

  在编译时要加上 -g 选项，生成的可执行文件才能用 gdb 进行源码级调试。

- -g 只是把可执行文件的指令和源代码的语句对应起来，不是把源文件嵌入可执行文件中，所以在调试的时候源文件必须存在

  -g 选项的作用是在可执行文件中加入源代码的信息，比如可执行文件中第几条机器指令对应源代码的第几行，但并不是把整个源文件嵌入到可执行文件中，所以在调试时必须保证 gdb 能找到源文件。

- 直接敲回车

  gdb 提供了一个很方便的功能，在提示符下直接敲回车表示重复上一条命令。

- gdb 常用命令有简写形式

  gdb 的很多常用命令有简写形式，例如 list 命令可以写成 l。

- 退出 gdb

  现在退出 gdb 的环境：

  (gdb) quit

- 列出的语句是即将执行的下一条

- list/l

- list/l 行号

- list/l 函数名

- start

- next/n

- step/s

- backtrace/bt

- info/i locals

- frame/f

- frame/f 帧编号

- printf/p 表达式

  print 命令后面跟的是表达式，而我们知道赋值和函数调用也都是表达式，所以也可以用 print 命令修改变量的值或者调用函数

- $1

  `$1` 表示 gdb 保存着这些中间结果，$ 后面的编号会自动增长，在命令中可以用 $1、$2、$3 等编号代替相应的值。

- finish

- set var x = y

## 10.2 断点

- display 变量名

- undisplay 跟踪显示号

- break/b 行号

- break/b 函数名

- break/b 行号/函数名 if ...

- continue/c

- info/i breakpoints

- delete breakpoints 断点号

- delete breakpoints

- disable breakpoints 断点号

- enable 断点号

- run/r

## 10.3 观察点

- x/FMT 变量名

- watch 变量名

  我们知道断点是当程序执行到某一代码行时中断，而观察点是当程序访问某个存储单元时中断，如果我们不知道某个存储单元是在哪里被改动的，这时候观察点尤其有用。

- info/i watchpoints

## 10.4 段错误

如果某个函数的局部变量发生访问越界，有可能并不立即产生段错误，而是在函数返回时产生段错误。
