# 6. 循环语句

## 6.1 while 语句

- 迭代：每次都有一点区别的重复工作

  在第 3 节「递归」中，我们介绍了用递归求 n! 的方法，其实每次递归调用都在重复做同样一件事，就是把 n 乘到 (n-1)! 上然后把结果返回。

  虽说是重复，但每次做都稍微有一点区别（n 的值不一样），这种**每次都有一点区别**的重复工作称为迭代（Iteration）。

  我们使用计算机的主要目的之一就是让它做重复迭代的工作，因为把一件工作重复做成千上万次而不出错正是计算机最擅长的，也是人类最不擅长的。

- 迭代可以用递归来做，也可以用循环来做

  虽然迭代用递归来做就够了，但 C 语言提供了循环语句使迭代程序写起来更方便。

- while 语句

  和 if 语句类似，while 语句由一个控制表达式和一个子语句组成，子语句可以是由若干条语句组成的语句块。

  ```
  语句 → while (控制表达式) 语句
  ```

  如果控制表达式的值为真，子语句就被执行，然后再次测试控制表达式的值，如果还是真，就把子语句再执行一遍，再测试控制表达式的值……

  这种控制流程称为循环（Loop），子语句称为循环体。

  如果某次测试控制表达式的值为假，就跳出循环执行后面的语句，如果第一次测试控制表达式的值就是假，那么直接跳到循环后面的语句，循环体一次都不执行。

  - 累加器和循环变量

    在 while 语句中有两类常见的变量：累加器和循环变量。

    在 while 语句中累加器（Accumulator）的作用是把每次循环的中间结果累积起来，循环结束后得到的累积值就是最终结果。

    循环变量（Loop Variable）在每次循环中都要被改变值，在控制表达式中要测试它的值，这两点合起来起到控制循环次数的作用。

    有些循环采用递减的循环变量，也有些循环采用递增的循环变量。

    累加器和循环变量这两种模式在循环中都很常见。

- 递归和循环的区别

  可见，递归能解决的问题用循环也能解决，但解决问题的思路不一样。

  用递归解决这个问题靠的是递推关系 n!=n·(n-1)!，用循环解决这个问题则更像是把这个公式展开了：n!=n·(n-1)·(n-2)·…·3·2·1。

  - 有些时候循环程序比递归程序更容易理解

    把公式展开了理解会更直观一些，所以有些时候循环程序比递归程序更容易理解。

  - 有些时候递归程序比循环程序更容易理解

    但也有一些公式要展开是非常复杂的甚至是不可能的，反倒是递推关系更直观一些，这种情况下递归程序比循环程序更容易理解。

- 函数式编程、命令式编程

  此外还有一点不同：看图 5.2「factorial(3) 的调用过程」，在整个递归调用过程中，虽然分配和释放了很多变量，但所有变量都只在初始化时赋值，没有任何变量的值发生过改变。

  而上面的循环程序则通过对 `n` 和 `result` 这两个变量多次赋值来达到同样的目的。

  前一种思路称为函数式编程（Functional Programming），而后一种思路称为命令式编程（Imperative Programming），这个区别类似于第 1 节「程序和编程语言」讲的 Declarative 和 Imperative 的区别。

  - 函数式编程的函数没有 Side Effect

    函数式编程的「函数」类似于数学函数的概念，回顾一下第 1 节「数学函数」所讲的，数学函数是没有 Side Effect的，而 C 语言的函数可以有 Side Effect，比如在一个函数中修改某个全局变量的值就是一种 Side Effect。

    第 4 节「全局变量、局部变量和作用域」指出，全局变量被多次赋值会给调试带来麻烦，如果一个函数体很长，控制流程很复杂，那么局部变量被多次赋值也会有同样的问题。因此，不要以为「变量可以多次赋值」是天经地义的。

  - 有很多编程语言完全采用函数式编程模式

    有很多编程语言可以完全采用函数式编程的模式，避免 Side Effect，例如 LISP、Haskell、Erlang 等。

  - C 主要还是采用 Imperative 模式

    用 C 语言编程主要还是采用 Imperative 的模式，但要记住，给变量多次赋值时要格外小心，在代码中多次读写同一变量应该以一种一致的方式进行。

    所谓「一致的方式」是说应该有一套统一的规则，规定在一段代码中哪里会对某个变量赋值、哪里会读取它的值，比如在第 2.4 节「errno 与 perror函数」会讲到访问 errno 的规则。

- 无穷循环

  递归函数如果写得不小心就会变成无穷递归，同样道理，循环如果写得不小心就会变成无限循环（Infinite Loop）或者叫死循环。

  如果 while 语句的控制表达式永远为真就成了一个死循环，例如 while (1) {...}。

  在写循环时要小心检查你写的控制表达式有没有可能取值为假，除非你故意写死循环（有的时候这是必要的）。在上面的例子中，不管 n 一开始是几，每次循环都会把 n 减掉 1，n越来越小最后必然等于 0，所以控制表达式最后必然取值为假，但如果把 n = n - 1; 这句漏掉就成了死循环。

## 6.2 do/while 语句

do/while 语句的语法是：

```
语句 → do 语句 while (控制表达式);
```

while 语句先测试控制表达式的值再执行循环体，而 do/while 语句先执行循环体再测试控制表达式的值。

如果控制表达式的值一开始就是假，while 语句的循环体一次都不执行，而 do/while 语句的循环体仍然要执行一次再跳出循环。

其实只要有 while 循环就足够了，do/while 循环和后面要讲的 for 循环都可以改写成 while 循环，只不过有些情况下用 do/while 或 for 循环写起来更简便，代码更易读。

## 6.3 for 语句

### for 语句相关

- 语法

  for语句的语法是：

  ```
  for (表达式1; 控制表达式2; 表达式3) 语句
  ```

- 和 for 循环等价的 while 循环

  如果不考虑循环体中包含 continue 语句的情况（稍后介绍 continue 语句），这个 for 循环等价于下面的 while 循环：

  ```
  表达式1;
  while (控制表达式2) {
      语句
      表达式3;
  }
  ```

- 控制表达式 2 不能空

  从这种等价形式来看，表达式 1 和 3 都可以为空，但控制表达式 2 是必不可少的，例如 for (;1;) {...} 等价于 while (1) {...} 死循环。

- 控制表达式 2 如果空，则认为它的值为真

  C 语言规定，如果控制表达式 2 为空，则认为控制表达式 2 的值为真，因此死循环也可以写成 for (;;) {...}。

### 自增自减运算符

- 前缀

  ++ 称为前缀自增运算符（Prefix Increment Operator），类似地，-- 称为前缀自减运算符（Prefix Decrement Operator）。

  ++i 相当于i = i + 1。

  --i 相当于 i = i - 1。

  如果把 ++i 这个表达式看作一个函数调用，除了传入一个参数返回一个值（等于参数值加 1）之外，还产生一个 Side Effect，就是把变量 i 的值增加了 1。

- 后缀

  ++ 和 -- 运算符也可以用在变量后面，例如 i++ 和 i--，为了和前缀运算符区别，这两个运算符称为后缀自增运算符（Postfix Increment Operator）和后缀自减运算符（Postfix Decrement Operator）。

  如果把 i++ 这个表达式看作一个函数调用，传入一个参数返回一个值，返回值就等于参数值（而不是参数值加 1），此外也产生一个 Side Effect，就是把变量 i 的值增加了 1。它和 ++i 的区别就在于返回值不同。

  同理，--i 返回减 1 之后的值，而 i-- 返回减 1 之前的值，但这两个表达式都产生同样的 Side Effect，就是把变量 i 的值减了 1。

- `a+++++b` 这个表达式如何理解

  应该理解成 `a++ ++ +b` 还是 `a++ + ++b`，还是 `a + ++ ++b` 呢？应该按第一种方式理解。

  编译的过程分为词法解析和语法解析两个阶段，在词法解析阶段，编译器总是从前到后找**最长的合法** Token。把这个表达式从前到后解析，变量名 a 是一个 Token，a 后面有两个以上的 + 号，在 C 语言中一个 + 号是合法的 Token（可以是加法运算符或正号），两个 + 号也是合法的 Token（可以是自增运算符），根据**最长匹配**原则，编译器绝不会止步于一个 + 号，而一定会把两个 + 号当作一个 Token。再往后解析仍然有两个以上的 + 号，所以又是一个 ++ 运算符。再往后解析只剩一个 + 号了，是加法运算符。再往后解析是变量名 b。

  词法解析之后进入下一阶段语法解析，a 是一个表达式，表达式 ++ 还是表达式，表达式再 ++ 还是表达式，表达式再 +b 还是表达式，语法上没有问题。

  最后编译器会做一些基本的语义分析，这时就有问题了，++ 运算符要求操作数能做左值，a 能做左值所以 a++ 没问题，但表达式 a++ 的值只能做右值，不能再 ++ 了，所以最终编译器会报错。

### C99 新的 for 循环语法

C99 规定了一种新的 for 循环语法，在表达式 1 的位置可以有变量定义。

例如上例的循环变量 i 可以只在 for循环中定义：

``` c
int factorial(int n)
{
    int result = 1;
    for(int i = 1; i <= n; i++)
       result = result * i;
    return result;
}
```

如果这样定义，那么变量 i 只是 for 循环中的局部变量而不是整个函数的局部变量，相当于第 1 节「if 语句」讲过的语句块中的局部变量，在循环结束后就不能再使用 i 这个变量了。

这个程序用 gcc 编译要加上选项 -std=c99。这种语法也是从 C++ 借鉴的，考虑到兼容性不建议使用这种写法。

## 6.4 break 和 continue 语句

- break 语句

  在第 4 节「switch 语句」中我们见到了 break 语句的一种用法，用来跳出 switch 语句块，这个语句也可以用来跳出循环体。

- continue 语句

  continue 语句也会终止当前循环，和 break 语句不同的是，continue 语句终止当前循环后又回到循环体的开头准备执行下一次循环。

  对于 while 循环和 do/while 循环，执行 continue 语句之后测试控制表达式，如果值为真则继续执行下一次循环；对于 for 循环，执行 continue 语句之后首先计算表达式 3，然后测试表达式 2，如果值为真则继续执行下一次循环。

## 6.5 嵌套循环

- break 和 continue 只对最内层的循环或 switch 有作用

  在有多层循环或 switch 嵌套的情况下，break 只能跳出最内层的循环或 switch，continue 也只能终止最内层循环并回到该循环的开头。

- 制表符

  表格很不整齐，如果把打印语句改为 `printf("%d\t", i*j);` 就整齐了，所以 Tab 字符称为制表符。

## 6.6 goto 语句和标号

- goto 语句实现无条件跳转

  分支、循环都讲完了，现在只剩下最后一种影响控制流程的语句了，就是 goto 语句，实现无条件跳转。

- goto 语句可以跳到最外层循环

  我们知道 break 只能跳出最内层的循环，如果在一个嵌套循环中遇到某个错误条件需要立即跳出最外层循环做出错处理，就可以用 goto 语句，例如：

  ```
  for (...)
      for (...) {
          ...
          if (出现错误条件)
              goto error;
      }
  error:
      出错处理;
  ```

- 标号

  这里的 `error:` 叫做标号（Label），任何语句前面都可以加若干个标号，每个标号的命名也要遵循标识符的命名规则。

- goto 语句过于强大

  goto 语句过于强大了，从程序中的任何地方都可以无条件跳转到任何其它地方，只要在那个地方定义一个标号就行。

  - goto 只能跳转到同一个函数的某个标号处

    唯一的限制是 goto 只能跳转到同一个函数中的某个标号处，而不能跳到别的函数中。

  - 用函数可以实现从被调用函数跳回它的直接或间接调用者

    C 标准库函数 setjmp 和 longjmp 配合起来可以实现函数间的跳转，但只能从被调用的函数跳回到它的直接或间接调用者（同时从栈空间弹出一个或多个栈帧），而不能从一个函数跳转到另一个和它毫不相干的函数中。

    setjmp/longjmp 函数主要也是用于出错处理，比如函数 A 调用函数 B，函数 B 调用函数 C，如果在 C 中出现某个错误条件，使得函数 B 和 C 继续执行下去都没有意义了，可以利用 setjmp/longjmp 机制快速返回到函数 A 做出错处理，本书不详细介绍这种机制，

  - goto 的危害

    滥用 goto 语句会使程序的控制流程非常复杂，可读性很差。

    著名的计算机科学家 Edsger W. Dijkstra 最早指出编程语言中 goto 语句的危害，提倡取消 goto 语句。

  - goto 语句可以用别的方法替代

    goto 语句不是必须存在的，显然可以用别的办法替代，比如上面的代码段可以改写为：

    ``` c
    int cond = 0; /* bool variable indicating error condition */
    for (...) {
        for (...) {
            ...
            if (出现错误条件) {
                cond = 1;
                break;
            }
        }
        if (cond)
            break;
    }
    if (cond)
        出错处理;
    ```

- goto 适用场合

  通常 goto 语句只用于这种场合，一个函数中任何地方出现了错误条件都可以立即跳转到函数末尾做出错处理（例如释放先前分配的资源、恢复先前改动过的全局变量等），处理完之后函数返回。

  比较用 goto 和不用 goto 的两种写法，用 goto 语句还是方便很多。

  但是除此之外，在任何其它场合都不要轻易考虑使用 goto 语句。

- 异常可以代替这种无条件跳转

  有些编程语言（如 C++）中有异常（Exception）处理的语法，可以代替 goto 和 setjmp/longjmp 的这种用法。

- case 和 default 也是标号

  回想一下，我们在第 4 节「switch 语句」学过 case 和 default 后面也要跟冒号（:号，Colon），事实上它们是两种特殊的标号。

- 和标号有关的语法规则

  和标号有关的语法规则如下：

  ```
  语句 → 标识符: 语句
  语句 → case 常量表达式: 语句
  语句 → default: 语句
  ```

  反复应用这些语法规则进行组合可以在一条语句前面添加多个标号，例如有些语句前面有多个 case 标号。

- 重新审视 switch 语句

  现在我们再看 switch 语句的格式：

  ```
  switch (控制表达式) {
  case 常量表达式： 语句列表
  case 常量表达式： 语句列表
  ...
  default： 语句列表
  }
  ```

  {} 里面是一组语句列表，其中每个分支的第一条语句带有 case 或 default 标号，从语法上来说，switch 的语句块和其它分支、循环结构的语句块没有本质区别：

  ```
  语句 → switch (控制表达式) 语句
  语句 → { 语句列表 }
  ```