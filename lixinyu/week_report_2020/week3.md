1、本周进展

- 在进行debug工具的开发
- 主要解决了原生qemu基本快划分不一致的问题
- 消除（修改qemu和x86-qemu-mips源码（不影响程序正确性））了一些影响程序寄存器（可能不影响程序正确性）的因素，便于对比
- 现在main之前可以做到每个基本快执行前后寄存器（包括fpu）状态完全一致
- main函数之后，第一个不一致的是浮点寄存器，由于80位浮点导致，现在准备尝试不影响程序正确性的基础下消除这部分不一致。

2、下周计划

- 

3、问题和需要的帮助

- 无