# 7. 结构体

## 7.1 复合类型与结构体

### 基本类型和复合类型

- 基本类型

  在编程语言中，最基本的、不可再分的数据类型称为基本类型（Primitive Type），例如整型、浮点型；

- 复合类型

  根据语法规则由基本类型组合而成的类型称为复合类型（Compound Type），例如字符串是由很多字符组成的。

- 复合类型的两面性为数据抽象奠定了基础

  有些场合下要把复合类型当做一个整体来用，而另外一些场合下需要分解组成这个复合类型的各种基本类型，复合类型的这种两面性为数据抽象（Data Abstraction）奠定了基础。

### 学习编程语言时要注意的三个方面

参考文献[12]的1.1节指出，在学习一门编程语言时要特别注意以下三个方面：

1.  这门语言提供了哪些Primitive，比如基本类型，比如基本运算符、表达式和语句。

2.  这门语言提供了哪些组合规则，比如基本类型如何组成复合类型，比如简单的表达式和语句如何组成复杂的表达式和语句。

3.  这门语言提供了哪些抽象机制，包括数据抽象和过程抽象（Procedure Abstraction）。

本章以结构体为例讲解**数据类型的组合**和**数据抽象**。至于**过程抽象**，我们在第4.2节已经见过最简单的形式，就是把一组语句用一个函数名封装起来，当做一个整体使用，本章将介绍更复杂的过程抽象。

### 声明结构体和定义结构体变量

现在我们用C语言表示一个复数。从直角坐标系来看，复数由实部和虚部组成，从极坐标系来看，复数由模和辐角组成，两种坐标系可以相互转换。

- 声明 tag

  如果用实部和虚部表示一个复数，我们可以写成由两个double型组成的结构体：

  ``` c
  struct complex_struct {
      double x, y;
  };
  ```

  这一句定义了标识符complex_struct（同样遵循标识符的命名规则），这种标识符在C语言中称为Tag。

  - 声明类型不能忘记 ;

    但即使像先前的例子那样只定义了complex_struct这个Tag而不定义变量，}后面的;号也不能少。这点一定要注意，类型定义也是一种声明，声明都要以;号结尾，结构体类型定义的}后面少;号是初学者常犯的错误。

- 声明 tag 的同时定义变量

  struct complex_struct { double x, y; }整个可以看作一个类型名，就像int或double一样，只不过它是一个复合类型。

  如果用这个类型名来定义变量，可以这样写：

  ``` c
  struct complex_struct {
      double x, y;
  } z1, z2;
  ```

  这样z1和z2就是两个变量名，变量定义后面带个;号是我们早就习惯的。

- 上面两种方式声明好 tag 以后，可以用 struct tag 直接定义变量

  不管是用上面两种形式的哪一种定义了complex_struct这个Tag，以后都可以直接用struct complex_struct来代替类型名了。

  例如可以这样定义另外两个复数变量：

  ``` c
  struct complex_struct z3, z4;
  ```

- 不声明 tag 定义变量

  如果在定义结构体类型的同时定义了变量，也可以不必写Tag，例如：

  ``` c
  struct {
      double x, y;
  } z1, z2;
  ```

  但这样就没办法再次引用这个结构体类型了，因为它没有名字。

- 在全局作用域中声明 Tag

  结构体Tag也可以定义在全局作用域中，这样定义的Tag在其定义之后的各函数中都可以使用。

  例如：

  ``` c
  struct complex_struct { double x, y; };

  int main(void)
  {
      struct complex_struct z;
  }
  ```

### 初始化结构体变量

结构体变量也可以在定义时初始化，例如：

``` c
struct complex_struct z = { 3.0, 4.0 };
```

Initializer中的数据依次赋给结构体的各成员。

如果Initializer中的数据比结构体的成员多，编译器会报错，但如果只是末尾多个逗号则不算错。

如果Initializer中的数据比结构体的成员少，未指定的成员将用0来初始化，就像未初始化的全局变量一样。

例如以下几种形式的初始化都是合法的：

``` c
double x = 3.0;
struct complex_struct z1 = { x, 4.0, }; /* z1.x=3.0, z1.y=4.0 */
struct complex_struct z2 = { 3.0, }; /* z2.x=3.0, z2.y=0.0 */
struct complex_struct z3 = { 0 }; /* z3.x=0.0, z3.y=0.0 */
```

- C99：局部结构体变量可以用变量的值初始化成员，全局结构体变量只能用常量表达式初始化成员

  注意，z1必须是局部变量才能用另一个变量x的值来初始化它的成员，如果是全局变量就只能用常量表达式来初始化。这也是C99的新特性。

- C89：无论是初始化全局变量还是局部变量，只能用常量表达式

  C89只允许在{}中使用常量表达式来初始化，无论是初始化全局变量还是局部变量。

- C99 新特性：designated initializer

  Designated Initializer是C99引入的新特性，用于初始化稀疏（Sparse）结构体和稀疏数组很方便。

  有些时候结构体或数组中只有某一个或某几个成员需要初始化，其他成员都用0初始化即可，用Designated Initializer语法可以很方便地针对每个成员做初始化（Memberwise Initialization）例如：

  ``` c
  struct complex_struct z1 = { .y = 4.0 }; /* z1.x=0.0, z1.y=4.0 */
  ```

  数组的Memberwise Initialization语法将在下一章介绍。

- initializer 语法总结

  Initializer的语法总结如下：

  Initializer → 表达式

  Initializer → { 初始化列表 }

  初始化列表 → Designated-Initializer, Designated-Initializer, ...

  （最后一个 Designated-Initializer 末尾可以有一个多余的 , 号）

  Designated-Initializer → Initializer

  Designated-Initializer → .标识符 = Initializer

  Designated-Initializer → [常量表达式] = Initializer

### {} 给结构体变量赋值

- {} 语法不能用于结构体变量的赋值

  {}这种语法不能用于结构体的赋值，例如这样是错误的：

  ``` c
  struct complex_struct z1;
  z1 = { 3.0, 4.0 };
  ```

  以前我们初始化基本类型的变量所使用的Initializer都是**表达式**，表达式当然也可以用来赋值，但现在这种由{}括起来的Initializer并**不是表达式**，所以不能用来赋值。

- C99 新的表达式语法

  C99引入一种新的表达式语法Compound Literal可以用来赋值，例如z1 = (struct complex_struct){ 3.0, 4.0 };，本书不使用这种新语法。

### 两个结构体变量

- 两个结构体变量之间可以使用赋值运算符

  结构体变量之间使用赋值运算符是允许的。

- 用一个结构体变量初始化另一个结构体变量，被初始化的必须是局部变量

  用一个结构体变量初始化另一个结构体变量也是允许的。

  同样地，z2必须是**局部变量**才能用变量z1的值来初始化。

- 例子

  例如：

  ``` c
  struct complex_struct z1 = { 3.0, 4.0 };
  struct complex_struct z2 = z1;
  z1 = z2;
  ```

- 结构体变量可以当函数的参数和返回值

  既然结构体变量之间可以相互赋值和初始化，也就可以当做函数的参数和返回值来传递：

  ``` c
  struct complex_struct add_complex(struct complex_struct z1, struct complex_struct z2)
  {
      z1.x = z1.x + z2.x;
      z1.y = z1.y + z2.y;
      return z1;
  }
  ```

- . 组成的表达式能不能做左值

  由.后缀运算符组成的表达式能不能做左值取决于.后缀运算符左边的操作数能不能做左值。

  在上面的例子中，z是一个变量，可以做左值，因此表达式z.x也可以做左值，但表达式add_complex(z, z).x只能做右值而不能做左值，因为add_complex(z, z)不能做左值。

### 结构体变量的成员

- 访问结构体变量的成员

  每个复数变量都有两个成员（Member）x和y，可以用.后缀运算符（.号，Period）来访问。

- 结构体变量的成员如何存储

  这两个成员的存储空间是相邻的，合在一起组成复数变量的存储空间。

  我们在第18.4节会看到，结构体成员之间也可能有若干个填充字节。

- 访问结构体变量成员的例子

  看下面的例子：

  ``` c
  #include <stdio.h>

  int main(void)
  {
      struct complex_struct { double x, y; } z;
      double x = 3.0;
      z.x = x;
      z.y = 4.0;
      if (z.y < 0)
          printf("z=%f%fi\n", z.x, z.y);
      else
          printf("z=%f+%fi\n", z.x, z.y);
      return 0;
  }
  ```

  - x 和 z.x 不冲突

    注意上例中变量x和变量z的成员x的名字并不冲突，因为变量z的成员x只能通过表达式z.x来访问，编译器可以从语法上区分哪个x是变量x，哪个x是变量z的成员x，第18.3节会讲到这两个标识符x属于**不同的命名空间**。

### 结构体类型、算术类型、标量类型

- 结构体类型的限制

  结构体类型用在表达式中有很多限制，不像基本类型那么自由，比如+ - * /等算术运算符和&& || !等逻辑运算符都不能作用于结构体类型，if语句、while语句中的控制表达式的值也不能是结构体类型。

- 算术类型：整型、浮点型

  严格来说，可以做算术运算的类型称为算术类型（Arithmetic Type），算术类型包括整型和浮点型。

- 标量类型：算术类型和指针类型

  可以表示零和非零，可以参与逻辑与、或、非运算或者做控制表达式的类型称为标量类型（Scalar Type），标量类型包括算术类型和以后要讲的指针类型，详见图22.5。

## 7.2 数据抽象

- 抽象就是提取公因式

  其实“抽象”这个概念并没有那么抽象，简单地说就是“提取公因式”：ab+ac=a(b+c)。如果a变了，ab和ac这两项都需要改，但如果写成a(b+c)的形式就只需要改其中一个因子。

- 复数运算程序

  - 复数存储表达层

    在我们的复数运算程序中，复数有可能用直角坐标或极坐标来表示，我们把这个有可能变动的因素提取出来组成复数存储表示层：real_part、img_part、magnitude、angle、make_from_real_img、make_from_mag_ang。

    这一层看到的数据是结构体的两个成员x和y，或者r和A，如果改变了结构体的实现就要改变这一层函数的实现，但函数接口不改变，因此调用这一层函数接口的复数运算层不需要改变。

  - 复数运算层

    复数运算层看到的数据只是一个抽象的“复数”的概念，知道它有直角坐标和极坐标，可以调用复数存储表示层的函数得到这些坐标。

  - 更上层

    再往上看，其他使用复数运算的程序看到的数据是一个更为抽象的“复数”的概念，只知道它是一个数，像整数、小数一样可以加减乘除，甚至连它有直角坐标和极坐标也不需要知道。

- 抽象层

  这里的复数存储表示层和复数运算层称为抽象层（Abstraction Layer），从底层往上层来看，复数越来越抽象了，把所有这些层组合在一起就是一个完整的系统。

  - 组合和抽象

    组合使得系统可以任意复杂，而抽象使得系统的复杂性是可以控制的，任何改动都只局限在某一层，而不会波及整个系统。

  - Another level of indirection

    著名的计算机科学家Butler Lampson说过：“All problems in computer science can be solved by another level of indirection.”这里的indirection其实就是abstraction的意思。

## 7.3 数据类型标志

``` c
enum coordinate_type { RECTANGULAR, POLAR };
```

- 声明枚举类型

  enum关键字的作用和struct关键字类似，把coordinate_type这个标识符定义为一个Tag，struct complex_struct表示一个结构体类型，而enum coordinate_type表示一个枚举（Enumeration）类型。

- 枚举类型的成员

  - 成员是常量

    枚举类型的成员是常量，它们的值由编译器自动分配，例如定义了上面的枚举类型之后，RECTANGULAR就表示常量0，POLAR表示常量1。

    如果不希望从0开始分配，可以这样定义：

    ``` c
    enum coordinate_type { RECTANGULAR = 1, POLAR };
    ```

    这样，RECTANGULAR就表示常量1，而POLAR表示常量2。

  - 枚举类型成员是整型的常量，可以用在需要整型常量表达式的地方

    枚举常量也是一种整型，其值在编译时确定，因此也可以出现在常量表达式中，可以用于初始化全局变量或者作为case分支的判断条件。

  - 枚举类型的成员名和变量名在同一命名空间

    有一点需要注意，虽然结构体的成员名和变量名不在同一命名空间中，但枚举的成员名却和变量名在同一命名空间中，所以会出现命名冲突。

    例如这样是不合法的：

    ``` c
    int main(void)
    {
        enum coordinate_type { RECTANGULAR = 1, POLAR };
        int RECTANGULAR;
        printf("%d %d\n", RECTANGULAR, POLAR);
        return 0;
    }
    ```

  - <span class="thoughts">变量名覆盖枚举类型成员名</span>

    ``` c
    #include <stdio.h>

    enum coordinate_type { RECTANGULAR = 1, POLAR };

    int main(void)
    {
        int RECTANGULAR;
        printf("%d %d\n", RECTANGULAR, POLAR);
        return 0;
    }
    ```

    int 型的 RECTANGULAR 把同名的枚举类型成员名覆盖了，printf 打印的是 int 型的。

  - <span class="thoughts">枚举类型成员就是一个整数</span>

    定义了枚举类型以后，任何用到整型常量的地方都可以用枚举类型。比如初始化整型变量，或给整型变量赋值。

- <span class="thoughts">枚举类型变量</span>

  用 enum tag 就可以定义一个枚举类型变量，这个变量其实就是整型的，可以用枚举类型成员给它初始化和赋值，也可以直接用整型常量。

## 7.4 嵌套结构体

- 声明嵌套的结构体类型

  结构体也是一种递归定义：结构体的成员具有某种数据类型，而结构体本身也是一种数据类型。换句话说，结构体的成员可以是另一个结构体，即结构体可以嵌套定义。

  例如我们在复数的基础上定义复平面上的线段：

  ``` c
  struct segment {
      struct complex_struct start;
      struct complex_struct end;
  }
  ```

- 初始化嵌套结构体类型变量

  从第7.1节讲的Initializer的语法可以看出，Initializer也可以嵌套，因此嵌套结构体可以嵌套地初始化，例如：

  ``` c
  struct segment s = {{ 1.0, 2.0 }, { 4.0, 6.0 }};
  ```

  也可以平坦（Flat）地初始化。例如：

  ``` c
  struct segment s = { 1.0, 2.0, 4.0, 6.0 };
  ```

  甚至可以把两种方式混合使用（这样可读性很差，应该避免）：

  ``` c
  segment s = {{ 1.0, 2.0 }, 4.0, 6.0 };
  ```

  利用C99的新特性也可以做Memberwise Initialization，例如：

  ``` c
  struct segment s = { .start.x = 1.0, .end.x = 2.0 };
  ```

- 访问嵌套结构体变量的成员

  访问嵌套结构体的成员要用到多个.后缀运算符，例如：

  ``` c
  s.start.t = RECTANGULAR;
  s.start.a = 1.0;
  s.start.b = 2.0;
  ```
