# Frontend设计



整体先将inst queue放在外面方便调试，等最后再放进整体前端模块中。

## ITLB

<font color=#FF0000>在接口加入了握手信号，增加了miss信号。内部逻辑还没有改。</font>

<font color=#FF0000>预计改为：</font>

<font color=#FF0000>发生miss，发出miss请求，ITLB等待miss请求响应回填，期间暂停流水线。</font>



## ICache

<font color=#FF0000>将iTLB的物理地址送到ICache中，进行tag比较，然后将4路的命中信息、指令、物理地址送往IR模块。icache读出的指令valid当hit且指令有效时为1，比较逻辑还未实现。输出的tag改为hit信息。itlb相关信息一起从icache送出，不使用握手信号</font>

<font color=#FF0000>没有加uncache的处理</font>

接收PC模块发来的pc值，不进行握手

## IR级

<font color=#FF0000>ICache送来的指令进行选择，判断是否有uncache、cache miss，如果有则送回icache处理miss，堵流水线。</font>



## BHT

同步RAM，当拍给pc下一拍拿到数据。

pc valid 时进行查询

将pc valid存入寄存器，查询结果存入寄存器。当锁存的pc valid寄存器中为1，则从查询结果的线上取值，如果寄存器中的pc valid为0，表示上一拍没有新的pc进入，可能流水线阻塞，查询结果从寄存器中取。

GHR全局历史寄存器21位，来自IR级。

PHT表1024项，每行16个2位饱和计数器。

每次pc读出一行。

ghr和pc index进行异或作为查询PHT的index。ghr更新来自ir级分支预测后的结果左移，或是例外分支错后从brbus完全拷贝。

 



## BTB

根据PC查询是否是跳转指令，如果命中，预测跳转目标。

当拍返回结果，使用寄存器存储表项，entry 64，PC存2-31位，对应target30位，两位饱和计数器表示是否跳转。



## JTB

Jump类的跳转指令。

cam表，表项32项，每项1位有效位V，pc的3到31位，30位target。Reg存储。

pc传进来进行匹配，匹配结果存入reg，下一拍进行读操作传入下一级。

