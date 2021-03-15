# Coremark

### 定义和定性叙述

Coremark是一个面向计算机中央处理器（CPU）性能的基准程序集。

### 词源

它由EEMBC（The Embedded Microprocessor Benchmark Consortium）的Shay Gal-On于2009年提出并开发，旨在成为处理器核性能方面的行业标准。

### 概念形成过程

CoreMark是一个简单而丰富的基准测试，专门用于测试处理器核，取代了过时的Dhrystone基准。 运行CoreMark会产生一个分数，从而使用户可以在处理器之间进行快速比较。任何人都可以在https://www.eembc.org/coremark/index.php上传和查看CoreMark分数，认证分数会通过EEMBC认证实验室的严格分析。

### 基本内容

代码使用C语言编写，包含以下算法的实现：列表处理（查找和排序）、矩阵处理（矩阵运算）、状态机（确定输入流是否包含有效数字）和CRC（循环冗余校验）。 它可以运行在8-bit到64-bit的微控制器和微处理器上。该代码受Apache License 2.0的约束，免费使用，但所有权由联盟保留，禁止发布以CoreMark名称命名的修改版本。

### 意义和影响

与Dhrystone一样，CoreMark体积小，便于携带，易于理解，免费，并且显示单个数字基准分数。与Dhrystone不同的是，CoreMark具有特定的运行和报告规则，旨在避免Dhrystone存在的问题。CoreMark采用的CRC算法具有双重功能，它提供了嵌入式应用程序中常见的工作负载，并确保CoreMark基准测试正确运行，实质上提供了一种自检机制。为确保编译器无法在编译时预先计算结果，避免受到编译器优化的影响，基准测试中的每个操作都会得出一个在编译时不可用的值。此外，在基准测试的计时部分使用的所有代码都是基准测试本身的一部分（无库调用）。

