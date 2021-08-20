# 9. 编码风格

- 好的代码风格对程序员来说很重要

  代码风格好不好就像字写得好不好看一样，如果一个公司招聘秘书，肯定不要字写得难看的。

  同理，代码风格糟糕的程序员肯定也是不称职的。虽然编译器不会挑剔难看的代码，照样能编译通过，但是和你一个Team的其他程序员肯定受不了，你自己也受不了，写完代码几天之后再来看，自己都不知道自己写的是什么。

- 代码是写给人看的

  参考文献[12]的第一版前言里有句话说得好：“Thus, programs must be written for people to read, and only incidentally for machines to execute.”代码主要是为了写给人看的，而不是写给机器看的，只是顺便也能用机器执行而已，如果是为了写给机器看那**直接写机器指令**就好了，没必要用高级语言了。

- 代码要写得清楚整洁

  代码和语言文字一样是为了表达思想、记载信息，所以一定要写得清楚整洁才能有效地表达。

- 软件项目中代码风格被规定死了

  正因为如此，在一个软件项目中，代码风格一般都用文档规定死了，所有参与项目的人不管他自己原来是什么风格，都要遵守统一的风格，例如Linux内核的参考文献[14]就是这样一个文档。

- Linux 内核的代码风格

  本章我们以内核的代码风格为基础来讲解好的编码风格都有哪些规定，这些规定的Rationale是什么。

  我只是以Linux内核为例来讲解编码风格的概念，并没有说内核编码风格就一定是最好的编码风格，但Linux内核项目如此成功，就足以说明它的编码风格是最好的C语言编码风格之一了。

- UNIX 标准字符终端长宽

  UNIX系统标准的字符终端是24行80列的。