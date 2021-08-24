# 7. 结构体

## 7.1 复合类型与结构体

### 基本类型和复合类型

- 基本类型

  在编程语言中，最基本的、不可再分的数据类型称为基本类型（Primitive Type），例如整型、浮点型；

- 复合类型：数据抽象

  根据语法规则由基本类型组合而成的类型称为复合类型（Compound Type），例如字符串是由很多字符组成的。

  有些场合下要把复合类型当作一个整体来用，而另外一些场合下需要分解组成这个复合类型的各种基本类型，复合类型的这种两面性为数据抽象（Data Abstraction）奠定了基础。

### Primitive, Composition, Abstraction

SICP 指出，在学习一门编程语言时要特别注意以下三个方面：

- Primitive

  这门语言提供了哪些 Primitive，比如基本类型，比如基本运算符、表达式和语句。

- Composition

  这门语言提供了哪些组合规则，比如基本类型如何组成复合类型，比如简单的表达式和语句如何组成复杂的表达式和语句。

- Abstraction

  这门语言提供了哪些抽象机制，包括数据抽象和过程抽象（Procedure Abstraction）。

- 类型组合和数据抽象

  本章以结构体为例讲解数据类型的组合和数据抽象。

- 过程抽象

  至于过程抽象，我们在第 2 节「if/else 语句」已经见过最简单的形式，就是把一组语句用一个函数名封装起来，当作一个整体使用，本章将介绍更复杂的过程抽象。

### 结构体语法

- 声明新的 Tag

  如果用实部和虚部表示一个复数，我们可以写成由两个 double 型组成的结构体：

  ``` c
  struct complex_struct {
      double x, y;
  };
  ```

  这一句声明了标识符 complex_struct（同样遵循标识符的命名规则），这种标识符在 C 语言中称为 Tag。

  类型声明也是一种声明，声明都要以 ; 号结尾，结构体类型声明的 } 后面少 ; 号是初学者常犯的错误。

- 声明新的 Tag 的同时直接定义变量

  struct complex_struct { double x, y; } 整个可以看作一个类型名，就像 int 或 double 一样，只不过它是一个复合类型，如果用这个类型名来定义变量，可以这样写：

  ``` c
  struct complex_struct {
     double x, y;
  } z1, z2;
  ```

  这样 z1 和 z2 就是两个变量名，变量定义后面带个 `;` 号是我们早就习惯的。

- 声明了 Tag 以后就可以用 `struct Tag` 当作类型名来定义变量了

  不管是用上面两种形式的哪一种声明了 complex_struct 这个 Tag，以后都可以直接用 struct complex_struct 来代替类型名了。

  例如可以这样定义另外两个复数变量：

  ``` c
  struct complex_struct z3, z4;
  ```

- 可以不声明 Tag 直接定义变量

  如果在声明结构体类型的同时定义了变量，也可以不必写 Tag，例如：

  ``` c
  struct {
      double x, y;
  } z1, z2;
  ```

  但这样就没办法再次引用这个结构体类型了，因为它没有名字。

- 访问成员变量

  每个复数变量都有两个成员（Member）x 和 y，可以用 . 运算符（. 号，Period）来访问，这两个成员的存储空间是相邻的，合在一起组成复数变量的存储空间。

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

- 成员变量名和同名变量不冲突：命名空间不同

  注意上例中变量 x 和变量 z 的成员 x 的名字并不冲突，因为变量 z 的成员 x 只能通过表达式 z.x 来访问，编译器可以从语法上区分哪个 x 是变量 x，哪个 x 是变量 z 的成员 x，第 3 节「变量的存储布局」会讲到这两个标识符 x 属于不同的命名空间。

- Tag 可以声明在全局作用域中

  结构体 Tag 也可以定义在全局作用域中，这样定义的 Tag 在其定义之后的各函数中都可以使用。

- 结构体变量初始化

  结构体变量也可以在定义时初始化，例如：

  ``` c
  struct complex_struct z = { 3.0, 4.0 };
  ```

  Initializer 中的数据依次赋给结构体的各成员。如果Initializer 中的数据比结构体的成员多，编译器会报错，但如果只是末尾多个逗号则不算错。

  如果 Initializer 中的数据比结构体的成员少，未指定的成员将用 0 来初始化，就像未初始化的全局变量一样。

  例如以下几种形式的初始化都是合法的：

  ```
  double x = 3.0;
  struct complex_struct z1 = { x, 4.0, }; /* z1.x=3.0, z1.y=4.0 */
  struct complex_struct z2 = { 3.0, }; /* z2.x=3.0, z2.y=0.0 */
  struct complex_struct z3 = { 0 }; /* z3.x=0.0, z3.y=0.0 */
  ```

  - C99 新特性

    z1 必须是局部变量才能用另一个变量 x 的值来初始化它的成员，如果是全局变量就只能用常量表达式来初始化。这也是 C99 的新特性。

  - C89 只允许用常量表达式初始化

    C89 只允许在 {} 中使用常量表达式来初始化，无论是初始化全局变量还是局部变量。

- 结构体变量的赋值

  {} 这种语法不能用于结构体的赋值，例如这样是错误的：

  ``` c
  struct complex_struct z1;
  z1 = { 3.0, 4.0 };
  ```

  以前我们初始化基本类型的变量所使用的 Initializer 都是表达式，表达式当然也可以用来赋值，但现在这种由 {} 括起来的 Initializer 并不是表达式，所以不能用来赋值。

  C99 引入一种新的表达式语法 Compound Literal 可以用来赋值，例如 `z1 = (struct complex_struct){ 3.0, 4.0 };`，本书不使用这种新语法。

### 结构体变量初始化还可以用 Designated-Initializer 语法

Initializer 的语法总结如下：

```
Initializer → 表达式
Initializer → { 初始化列表 }
初始化列表 → Designated-Initializer, Designated-Initializer, ...
（最后一个Designated-Initializer末尾可以有一个多余的,号）
Designated-Initializer → Initializer
Designated-Initializer → .标识符 = Initializer
Designated-Initializer → [常量表达式] = Initializer
```

- Designated Initializer

  Designated Initializer 是 C99 引入的新特性，用于初始化稀疏（Sparse）结构体和稀疏数组很方便。

  有些时候结构体或数组中只有某一个或某几个成员需要初始化，其它成员都用 0 初始化即可，用 Designated Initializer 语法可以针对每个成员做初始化（Memberwise Initialization），很方便。

  例如：

  ``` c
  struct complex_struct z1 = { .y = 4.0 }; /* z1.x=0.0, z1.y=4.0 */
  ```

  数组的 Memberwise Initialization 语法将在下一章介绍。

### 结构体类型的限制

结构体类型用在表达式中有很多限制，不像基本类型那么自由，比如 + - * / 等算术运算符和 && || ! 等逻辑运算符都不能作用于结构体类型，if 语句、while 语句中的控制表达式的值也不能是结构体类型。

- 算数类型

  严格来说，可以做算术运算的类型称为算术类型（Arithmetic Type），算术类型包括整型和浮点型。

- 标量类型

  可以表示零和非零，可以参与逻辑与、或、非运算或者做控制表达式的类型称为标量类型（Scalar Type），标量类型包括算术类型和以后要讲的指针类型，详见图 23.5「C 语言类型总结」。

### 两个结构体变量

结构体变量之间使用赋值运算符是允许的，用一个结构体变量初始化另一个结构体变量也是允许的，例如：

``` c
struct complex_struct z1 = { 3.0, 4.0 };
struct complex_struct z2 = z1;
z1 = z2;
```

同样地，z2 必须是局部变量才能用变量 z1 的值来初始化。

既然结构体变量之间可以相互赋值和初始化，也就可以当作函数的参数和返回值来传递。

### `x.y` 能不能做左值

由 . 运算符组成的表达式能不能做左值取决于 . 运算符左边的表达式能不能做左值。

在上面的例子中，z 是一个变量，可以做左值，因此表达式 z.x 也可以做左值，但表达式 add_complex(z, z).x 只能做右值而不能做左值，因为表达式 add_complex(z, z) 不能做左值。

## 7.2 数据抽象

- 分层

  在我们的复数运算程序中，复数有可能用直角座标或极座标来表示，我们把这个有可能变动的因素提取出来组成**复数存储表示层**：real_part、img_part、magnitude、angle、make_from_real_img、make_from_mag_ang。

  这一层看到的数据是结构体的两个成员 x 和 y，或者 r 和 A，如果改变了结构体的实现就要改变这一层函数的实现，但函数接口不改变，因此调用这一层函数接口的**复数运算层**也不需要改变。

  复数运算层看到的数据只是一个抽象的「复数」的概念，知道它有直角座标和极座标，可以调用复数存储表示层的函数得到这些座标。

  再往上看，其它使用复数运算的程序看到的数据是一个更为抽象的「复数」的概念，只知道它是一个数，像整数、小数一样可以加减乘除，甚至连它有直角座标和极座标也不需要知道。

- 抽象层

  这里的复数存储表示层和复数运算层称为抽象层（Abstraction Layer），从底层往上层来看，复数越来越抽象了，把所有这些层组合在一起就是一个完整的系统。

- 抽象与组合

  组合使得系统可以任意复杂，而抽象使得系统的复杂性是可以控制的，任何改动都只局限在某一层，而不会波及整个系统。

  著名的计算机科学家 Butler Lampson 说过：「All problems in computer science can be solved by another level of indirection.」这里的 indirection 其实就是 abstraction 的意思。

## 7.3 数据类型标志

### 枚举类型

- 枚举类型声明

  ```
  enum coordinate_type { RECTANGULAR, POLAR };
  ```

  `enum coordinate_type` 表示一个枚举（Enumeration）类型。

- 成员

  - 成员的值

    枚举类型的成员是常量，它们的值由编译器自动分配，例如定义了上面的枚举类型之后，RECTANGULAR 就表示常量 0，POLAR 表示常量 1。

    如果不希望从 0 开始分配，可以这样定义：

    ```
    enum coordinate_type { RECTANGULAR = 1, POLAR };
    ```

    这样，RECTANGULAR 就表示常量 1，而 POLAR 表示常量 2。

  - 枚举常量（成员）可以用在常量表达式中

    枚举常量也是一种整型，其值在编译时确定，因此也可以出现在常量表达式中，可以用于初始化全局变量或者作为 case 分支的判断条件。

  - 成员名和变量名在同一命名空间

    枚举的成员名却和变量名在同一命名空间中，所以会出现命名冲突。

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

  - 把成员看作数字常量

    <span class = "thought">可以把成员看作数字常量。赋值语句不能给成员赋值，因为成员做不了左值，所以值变不了。</span>

- 枚举变量

  <span class = "thought">枚举类型声明好了以后用 `enum Tag` 当作类型定义变量就可以了。</span>

  <span class = "thought">枚举变量其实就是 `int` 类型变量，初始化和赋值的时候可以用枚举常量也可以直接用整数（负数）也可以。</span>

## 7.4 嵌套结构体

### 声明嵌套结构体类型

结构体也是一种递归定义：结构体的成员具有某种数据类型，而结构体本身也是一种数据类型。

换句话说，结构体的成员可以是另一个结构体，即结构体可以嵌套定义。

例如我们在复数的基础上定义复平面上的线段：

``` c
struct segment {
	struct complex_struct start;
	struct complex_struct end;
};
```

### 初始化嵌套结构体类型变量

从第 1 节「复合类型与结构体」讲的 Initializer 的语法可以看出，Initializer 也可以嵌套，因此嵌套结构体可以嵌套地初始化，例如：

struct segment s = {{ 1.0, 2.0 }, { 4.0, 6.0 }};

也可以平坦（Flat）地初始化。例如：

struct segment s = { 1.0, 2.0, 4.0, 6.0 };

甚至可以把两种方式混合使用（这样可读性很差，应该避免）：

struct segment s = {{ 1.0, 2.0 }, 4.0, 6.0 };

利用 C99 的新特性也可以做 Memberwise Initialization，例如：

struct segment s = { .start.x = 1.0, .end.x = 2.0 };

### 访问嵌套结构体的成员

访问嵌套结构体的成员要用到多个 . 运算符，例如：

s.start.t = RECTANGULAR;
s.start.a = 1.0;
s.start.b = 2.0;
