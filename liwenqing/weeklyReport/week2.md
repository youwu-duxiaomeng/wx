### 本周进展

- 将添加了YGJK和aes加密加速器的boom烧到FPGA开发板上运行程序
- 在FPGA上测试，声明不同数组大小情况下，相同程序时间差异较大，添加硬件访存性能计数器，发现之前测试过程中数据有提前存在L2cache的情况，重新进行实验，保证所有数据最开始都在内存中。
- 重新实验，数据都在内存中，实验数据变差，添加small BOOM硬件预取，取得一些性能提升
- 阅读论文

### 下周计划
- 进行软件预取的实验
- 开始写论文
- 
