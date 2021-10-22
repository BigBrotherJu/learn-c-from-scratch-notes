# 23. 函数接口

<!-- no highlight -->

- 函数接口
- 文档（Man Page）

## 23.1 本章的预备知识

### strcpy 与 strncpy

- strcpy

  - 重复复制 src 指向的字符到 dest，直到遇到 \0

    在文档中强调了strcpy在拷贝字符串时会把结尾的'\0'也拷到dest中，因此保证了dest是以Null结尾的字符串。

  - 写越界和读越界

    但这样做存在一个问题，strcpy只知道src字符串的首地址，不知道长度，它会一直拷贝到'\0'为止，因此dest所指向的内存空间要足够大，否则有可能写越界，例如：

    ``` c
    char buf[10];
    strcpy(buf, "hello world");
    ```

    如果不保证src所指向的内存空间以'\0'结尾，也有可能读越界，例如：

    ``` c
    char buf[10] = "abcdefghij", str[4] = "hell";
    strcpy(buf, str);
    ```

    因为strcpy函数的实现者通过函数接口无法得知src字符串的长度和dest内存空间的大小，所以“确保不会写越界”应该是调用者的责任，调用者提供的dest参数应该指向足够大的内存空间，“确保不会读越界”也是调用者的责任，调用者提供的src参数指向的内存空间应该确保以'\0'结尾。

  - 不能重叠

    此外，文档中还强调了src和dest所指向的内存空间不能有重叠，例如这样调用是不允许的：

    ``` c
    char buf[10] = "hello";
    strcpy(buf, buf + 1);
    ```

    凡是有指针参数的C标准库函数基本上都有这条要求，也有个别函数例外，下一章会讲到这样的函数。

- strncpy

  strncpy的参数n指定最多从src中拷贝n个字节到dest中，换句话说，如果拷贝到'\0'就结束，如果拷贝到n个字节还没有碰到'\0'，那么也结束。

  - 规定适当的 n 使读写不越界

    如果调用者不能确定src字符串的长度，可以规定一个适当的n值，以确保读写不会越界，通常让n的值等于dest所指向的内存空间的大小：

    ``` c
    char buf[10];
    strncpy(buf, "hello world", sizeof(buf));
    ```

  - dest 可能不以 \0 结尾

    这样做也存在一个问题，文档中特别用Warning指出，这意味着dest有可能不是以'\0'结尾的。例如上面的调用，虽然把"hello world"截断为10个字符拷贝到buf中，但buf不是以'\0'结尾的，如果再printf(buf)就会读越界。如果你需要确保dest以'\0'结尾，可以这么调用：

    ``` c
    char buf[10];
    strncpy(buf, "hello world", sizeof(buf));
    buf[sizeof(buf)-1] = '\0';
    ```

  - src 长度小于 n

    strncpy还有一个特性，如果src字符串全部拷完了不足n个字节，那么还差多少个字节就补多少个'\0'，但是正如上面所述，这并不保证dest一定以'\0'结尾，当src字符串的长度大于n时，不但不补多余的'\0'，连字符串结尾的'\0'也没有。

- buffer 和 buffer overflow（写越界）

  BUGS部分说明了使用这些函数可能引起的Bug，这部分一定要仔细看。用strcpy比用strncpy更加不安全，如果在调用strcpy之前不仔细检查src字符串的长度就有可能写越界，这是一个很常见的错误，例如：

  ``` c
  void foo(char *str)
  {
      char buf[10];
      strcpy(buf, str);
      ...
  }
  ```

  str所指向的字符串有可能超过10个字符而导致写越界，在第10.4节我们看到过，这种写越界有可能当时不出错，而在函数返回时出现段错误，原因是写越界覆盖了保存在栈帧上的返回地址，函数返回时跳转到非法地址，因而出错。

  像buf这种由调用者分配并传给strcpy函数访问的一段内存通常称为缓冲区（Buffer），缓冲区写越界的错误称为缓冲区溢出（Buffer Overflow）。

  如果只是出现段错误那还不算严重，更严重的是缓冲区溢出Bug经常被恶意用户利用，使函数返回时跳转到一个事先设计好的地址，执行一段事先设计好的指令，如果设计得巧妙甚至可以启动一个Shell，然后随心所欲执行任何命令，可想而知，如果一个用root权限执行的程序存在这样的Bug，被攻陷了，后果将很严重。

  至于怎样巧妙设计和攻陷一个存在缓冲区溢出问题的程序，感兴趣的读者可以查阅参考文献[28]。

### malloc 与 free

``` c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    int number;
    char *msg;
} unit_t;

int main(void)
{
    unit_t *p = malloc(sizeof(unit_t));

    if (p == NULL) {
        printf("out of memory\n");
        exit(1);
    }
    p->number = 3;
    p->msg = malloc(20);
    strcpy(p->msg, "Hello world!");
    printf("number: %d\nmsg:%s\n", p->number, p->msg);
    free(p->msg);
    free(p);
    p = NULL;

    return 0;
}
```

- 判断成功

  虽然内存耗尽是很不常见的错误，但写程序要规范，malloc之后应该判断是否成功。

  以后要学习的大部分系统函数都有成功的返回值和失败的返回值，每次调用系统函数之后都应该判断是否成功。

- 避免出现野指针，free 一个指针后赋值 NULL

  free(p);之后，p所指的内存空间是归还了，但是p的值并没有变，因为从free的函数接口来看根本没法改变p的值，p现在指向的内存空间已经不属于用户程序，换句话说，p成了野指针，为避免出现野指针，我们应该在free(p);之后手动置p = NULL;。

- free 的顺序

  应该先free(p->msg)，再free(p)，顺序不能颠倒。

  如果先free(p)，p成了野指针，就不能再访问p->msg了。

- malloc 和 free 要配对，避免出现内存泄漏

  上面的例子只有一个简单的顺序控制流程，分配内存，赋值，打印，释放内存，退出程序。这种情况下即使不用free释放内存也可以，因为进程退出时它占用的所有内存都会释放，也就是归还给了操作系统。

  但如果一个程序长年累月运行（例如网络服务器程序），并且在循环或递归中调用malloc分配内存，则必须有free与之配对，分配一次就要释放一次，否则每次循环都分配内存，分配完了又不释放，就会慢慢耗尽系统内存，这种错误称为内存泄漏（Memory Leak）。

- malloc 返回的指针保存好，不然也会出现内存泄漏

  另外，malloc返回的指针一定要保存好，只有把它传给free才能释放这块内存，如果这个指针丢失了，就没有办法free这块内存了，也会造成内存泄漏。例如：

  ``` c
  void foo(void)
  {
      char *p = malloc(10);
      ...
  }
  ```

  foo函数返回时要释放局部变量p的内存空间，它所指向的内存地址就丢失了，这10个字节也就没法释放了。

- 内存泄漏的影响

  内存泄漏的Bug很难找到，因为它并不像访问越界一样导致程序运行错误，少量内存泄漏并不影响程序的正确运行，大量的内存泄漏会导致物理内存紧缺，换页频繁，不仅影响当前进程，而且会把整个系统都拖得很慢。

- 特殊情况

  关于malloc和free还有一些特殊情况。

  malloc(0)这种调用也是合法的，也会返回一个非NULL的指针，这个指针也可以传给free释放，但是不能通过这个指针访问内存。

  free(NULL)也是合法的，不做任何事情，但是free一个野指针是不合法的，例如先调用malloc返回一个指针p，然后连着调用两次free(p);，则后一次调用会产生运行时错误。

- malloc 和 free 的实现和原理

## 23.2 传入参数与传出参数

- 传入参数

  如果函数接口有指针参数，既可以把指针所指向的数据传给函数使用（称为传入参数，如表23.1所示），

- 传出参数

  也可以由函数填充指针所指的内存空间，传回给调用者使用（称为传出参数，如表23.2所示），例如strcpy的src参数是传入参数，dest参数是传出参数。

- value-result 参数

  有些函数的指针参数同时充当这两种角色，如select(2)的fd_set*参数，既是传入参数又是传出参数，这称为Value-result参数，如表23.3所示。

- 参数为 NULL

  很多系统函数对于指针参数是NULL的情况有特殊规定：

  如果传入参数是NULL表示取缺省值，例如pthread_create(3)的pthread_attr_t *参数，也可能表示不做特别处理，例如free(3)的参数；

  如果传出参数是NULL表示调用者不需要传出值，例如time(2)的参数。这些特殊规定应该在文档中写清楚。

## 23.3 两层指针的参数

两层指针也是指针，同样可以表示传入参数、传出参数或者Value-result参数，只不过该参数所指的内存空间应该解释成指针变量。用两层指针做传出参数的系统函数也很常见，比如pthread_join(3)的void **参数。

两层指针作为传出参数还有一种特别的用法，可以在函数中分配内存，调用者通过传出参数取得指向该内存的指针，比如getaddrinfo(3)的struct addrinfo **参数。一般来说，实现一个分配内存的函数就要实现一个释放内存的函数，所以getaddrinfo(3)有一个对应的freeaddrinfo(3)函数。

总结一下，两层指针参数如果是传出的，可以有两种情况：第一种情况，传出的指针指向静态内存（比如上面的例子），或者指向已分配的动态内存（比如指向某个链表的节点）；第二种情况是在函数中动态分配内存，然后传出的指针指向这块内存空间，这种情况下调用者应该在使用内存之后调用释放内存的函数，调用者的责任是请求分配和释放内存，实现者的责任是完成分配和释放内存的操作。由于这两种情况的函数接口相同，应该在函数的文档中说明是哪一种情况。

## 23.4 返回值是指针的情况

返回值显然是传出的而不是传入的，如果返回值传出的是指针，和上一节通过参数传出指针类似，也分为两种情况：第一种是传出指向静态内存或已分配的动态内存的指针，例如localtime(3)和inet_ntoa(3)，第二种是在函数中动态分配内存并传出指向这块内存的指针，例如malloc(3)和strdup(3)，这种情况通常还要实现一个释放内存的函数，所以有和malloc(3)对应的free(3)函数。由于这两种情况的函数接口相同，应该在函数的文档中说明是哪一种情况。

## 23.5 回调函数

- 回调函数

  如果参数是一个函数指针，调用者可以传递一个函数的地址给实现者，即调用者提供一个函数但自己不去调用它，而是让实现者去调用它，这称为回调函数（Callback Function）。

  回调函数的一个典型应用就是实现类似C++的泛型算法（Generics Algorithm）。下面实现的max函数可以在任意一组对象中找出最大值，可以是一组int、一组char或一组结构体，但是实现者并不知道怎样去比较两个对象的大小，需要调用者再提供一个做比较操作的回调函数。

- 同步调用和异步调用

  以上举例的回调函数都是被同步调用的，调用者调用max函数，max函数则调用cmp函数，相当于调用者间接调用了自己提供的回调函数。

  除此之外，异步调用也是回调函数的一种典型用法，调用者首先将回调函数传给实现者，实现者记住这个函数，这称为注册一个回调函数，然后当某个事件发生时实现者再调用先前注册过的函数，比如sigaction(2)注册一个信号处理函数，当信号产生时由操作系统调用该函数进行处理，再比如pthread_create(3)注册一个线程函数，当发生调度时操作系统切换到新注册的线程函数中运行，在GUI编程中异步回调函数更是有普遍的应用，例如为某个按钮注册一个回调函数，当用户单击按钮时调用它。

- 返回函数指针

  既然参数可以是函数指针，返回值同样也可以是函数指针，因此可以有func()()这样的调用。

  返回函数指针的函数在C语言中很少见，而在一些函数式编程语言（例如LISP）中返回函数的函数很常见，基本思想是把函数也当做一种数据来操作，可以输入、输出和参与运算，操作函数的函数称为高阶函数（High-order Function）。

## 23.6 可变参数

还有一种办法可以确定可变参数的个数，就是在参数列表的末尾传一个Sentinel，例如NULL。execl(3)就采用这种方法确定参数的个数。下面实现一个printlist函数，可以打印若干个传入的字符串。

其实printlist函数的第一个参数begin并没有用到，但是这个参数必须写，因为C语言规定可变参数列表的...前面至少要定义一个有名字的参数，要把参数列表中最后一个有名字的参数提供给va_start，这样va_start才能找到可变参数在栈上的位置。实现者应该在文档中说明参数列表必须以NULL结尾，如果调用者不遵守这个约定，函数就会出错。