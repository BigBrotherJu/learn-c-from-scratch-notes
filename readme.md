# 《一站式学习 C 编程》笔记

## 笔记目录

- [总结](summary-notes.md)
- [从 C 代码到进程](from-c-to-process.md)
- [书目推荐](book-recommendation.md)
- 第一部分：C 语言入门
  - 第 1 章：[程序的基本概念](part1/chapter-1.md)
  - 第 2 章：[常量、变量和表达式](part1/chapter-2.md)
  - 第 3 章：[简单函数](part1/chapter-3.md)
  - 第 4 章：[分支语句](part1/chapter-4.md)
  - 第 5 章：[深入理解函数](part1/chapter-5.md)
  - 第 6 章：[循环语句](part1/chapter-6.md)
  - 第 7 章：[结构体](part1/chapter-7.md)
  - 第 8 章：[数组的基本概念](part1/chapter-8.md)
  - 第 9 章：[编码风格](part1/chapter-9.md)
  - 第 10 章：[GDB](part1/chapter-10.md)
  - 第 11 章：[排序与查找](part1/chapter-11.md)
  - 第 12 章：[栈与队列](part1/chapter-12.md)
  - [本阶段总结](part1/summary.md)
- 第二部分：C 语言本质
  - 第 13 章：[计算机中数的表示](part-2/chapter-13.md)
  - 第 14 章：[数据类型详解](part-2/chapter-14.md)
  - 第 15 章：[运算符详解](part-2/chapter-15.md)
  - 第 16 章：[计算机体系结构基础](part-2/chapter-16.md)
  - 第 17 章：[x86 汇编程序基础](part-2/chapter-17.md)
  - 第 18 章：[汇编与 C 之间的关系](part-2/chapter-18.md)
  - 第 19 章：[链接详解](part-2/chapter-19.md)
  - 第 20 章：[预处理](part-2/chapter-20.md)
  - 第 21 章：[Makefile 基础](part-2/chapter-21.md)
  - 第 22 章：[指针](part-2/chapter-22.md)
  - 第 23 章：[函数接口](part-2/chapter-23.md)
  - 第 24 章：[C 标准库](part-2/chapter-24.md)
  - 第 25 章：[链表、二叉树和哈希表](part-2/chapter-25.md)
  - [本阶段总结](part-2/summary.md)

## 这本书的一些背景

这个仓库存放的是关于[《一站式学习 C 编程》][1]的笔记。这本书的作者是[宋劲杉][2]。

[1]: https://book.douban.com/subject/6025290/

[2]: https://github.com/seanjsong

注意，和《一站式学习 C 编程》有关的书籍一共有四种。它们分别是：作者最早发布在网络上的电子书[《Linux C 编程一站式学习》][3]，托管在 <https://github.com/akaedu/book/tree/gh-pages>；电子工业出版社的纸质书[《Linux C 编程一站式学习》][4]；电子工业出版社在前者基础上修订的纸质书[《一站式学习 C 编程》][1]；作者最新修订的[电子书][5]，托管在 <https://github.com/learning-linux-c-cpp/akabook>。

[3]: https://akaedu.github.io/book/index.html

[4]: https://book.douban.com/subject/4141733/

[5]: https://web.archive.org/web/20170924163408/http://songjinshan.com:80/akabook/zh/index.html

四者的区别在作者最新修订电子书的[《历史》][6] 这一章节有详细阐述。简单来说，两本纸质书都少了最早电子书的第三部分[《Linux 系统编程》][7]，但是两本纸质书在最早电子书的基础上进行了一些修订。而最新电子书只写到[《计算机中数的表示》][8]，剩下的章节就没写了。而且目前只有[最早电子书网站][3]还能正常访问，最新电子书网站服务器已经关闭，只能通过 [Internet Archieve][9] 访问或者在 GitHub 上查看源码。

[6]: https://github.com/learning-linux-c-cpp/akabook/blob/master/zh/history.rst

[7]: https://akaedu.github.io/book/pt03.html

[8]: https://github.com/learning-linux-c-cpp/akabook/blob/master/zh/number.rst

[9]: https://archive.org/

这本书写得非常好，在最新电子书的[目录页][10]，可以看到除了最早电子书的三部分外，还有《从 C 到 C++》和《从 C/C++ 到动态语言》这两章。可以看出作者的野心非常大，但是作者没有继续更新最新电子书这一点，让人感到非常惋惜。另外，作者还开了一个关于最新电子书的 [Google Groups][11]，里面已经很久没人说话了，感兴趣的朋友可以去看一看。

[10]: https://github.com/learning-linux-c-cpp/akabook/blob/master/zh/index.rst

[11]: https://groups.google.com/g/learning-linux-c-cpp

知乎上有一个关于这本书的[问题][12]，里面有一些资源。

[12]: https://www.zhihu.com/question/34069391

关于答案，可以在上面知乎问题的回答中找到网友编撰的[答案][13]。Google Groups 中也有作者本人提供的[答案][14]，不过好像不怎么全。

[13]: https://www.zybuluo.com/ChristopherWu/note/72463

[14]: https://groups.google.com/g/learning-linux-c-cpp/c/Lz0lglbfNuY

## 我对这本书的评价

https://book.douban.com/review/13947735/

我对这本书的总体评价是：有瑕有瑜，瑕不掩瑜。

瑜：

1 全面地介绍了 C99 语法，和 C99 标准非常贴近，书中很多内容就是 C99 标准的内容。看完本书再看 C99 标准会轻松很多。

2 介绍 C 语言语法的同时，也对计算机的组成原理以及底层的汇编、链接、加载、ELF 等做了提纲挈领的介绍。

3 介绍 C 语言语法的同时，也介绍了一些重要的编程思想、编程方法和最佳实践。

4 介绍 C 语言语法的同时，以 GNU/Linux 为平台，介绍了一些重要命令行工具的基本使用，如 GCC、GDB、make。

5 看完这本书，你会大概了解一个 C 代码文件是怎么成为一个进程的。

瑕：

1 对一些概念的讲解还存在问题，比如左值右值和 ELF 文件。左值右值我是后来自己翻了 C99 标准以及解释 C99 标准的书籍才明白什么意思，这个是可以改正的。ELF 文件，特别是最后的共享库解释的不是很清楚，我翻来覆去看了很久，但是这本书的定位是入门级别，如果事无巨细地全部解释清楚，篇幅会太长，这个无法强求。

2 课后习题不够。练习不够容易造成知识的遗忘。已有的习题也没有答案，学习体验不好。作者还特别喜欢在正文中问问题，问了又不给答案，说实话问了问题不给答案不如不问。

这本书对 C 语言的讲解绝对碾压国内大部分大学的 C 语言课程。作者在前言中叙述的衔接不同有联系的课程的教育理念其实已经被广泛应用在西方大学的课堂中，像加州大学伯克利分校的计算机组成原理就是一门课学习 C 语言、汇编语言以及组成原理的知识。

另外，这本书对国内市场上绝大部分 C 语言书籍（特别是某清华大学教授的书）就是降维打击，但是图书市场好像不买账，这本书出版几年以后就绝版了。希望国内图书市场能够出现更多这样优秀的技术类书籍。
