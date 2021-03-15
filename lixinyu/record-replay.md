#### record/replay(rr)

rr的支持主要在qemu的replay目录下，文档可以在https://wiki.qemu.org/Features/record-replay（更新慢）看到，

在qemu的docs/replay.txt也有更详细的文档。



rr的产生是为了确定性的执行，qemu在模拟系统时会遇到不确定的外部时间，主要是中断，例如时钟中断，键盘输入，网络包的到来，这些都是具有不确性的。

rr通过record将这些外部事件记录在文件中，然后replay时只从文件中读取record时记录的事件，然后在准确的时间将这些事件发送给cpu，便可得到和record时一样的结果。



rr的一个基础是icount，通过icount机制可以对执行的指令进行精确计数，然后结合外部事件，即可精确记录在第多少条指令发生了什么事件（rr时异步的io也是交给rr来处理，避免出现不一致的情形），然后replay时达到相应的指令数便会停下来，处理这些事件。在翻译时只需要关注icount的实现即可，record/replay无需关注。



对于原qemu在5.1.0上rr机制已经比较完善，可以对windows-xp进行操作，xqm上也已经移植到system-rr分支上，system-rr分支主要是为了和原生qemu进行比对才产生的，一般是用原qemu进行record，然后原qemu的xqm分别进行replay操作，然后根据输出的信息进行比对。



一般来说只需要比较执行路径(pc)或者寄存器值就行，也可以禁用本地码查找tlb，直接调用helper，然后在helper中输出内存的访问。总之粒度越细，代价越高，时间越长，但更有可能发现出错的第一现场，而粗粒度的对比一般还要经过回溯的操作，难以发现具体的出错点。



目前rr分支上的eflags计算与原qemu保持一致，如果出现不一致还需要在target/i386/cc_helper.c中找到计算方式和我们的进行比较让两者保持一致。undefined怎么简单怎么改，改qemu的也没问题。目前还有一些小问题，如访存顺序等，慢慢修改总可以和tcg做到差不多。



#### rr的使用

```sh
#record
qemu-system-i386 \
     -icount shift=7,rr=record,rrfile=replay.bin \
     -drive file=disk.qcow2,if=none,snapshot,id=img-direct \
     -drive driver=blkreplay,if=none,image=img-direct,id=img-blkreplay \
     -device ide-hd,drive=img-blkreplay \
     -netdev user,id=net1 -device rtl8139,netdev=net1 \
     -object filter-replay,id=replay,netdev=net1
```

record使用方式如上，replay时将rr=record替换为rr=replay即可，具体命令参数不同版本可能不一样，详情查看qemu/docs/replay.txt，在rr时原磁盘不会发生任何改变。如果不需要网络可以使用-net none选项。

一般在replay时采用 -d cpu\exec\int\pcall\nochain这些选项来输出信息，也可以通过gdb调试，replay可以保证整个cpu\mem\device都是和record时一样的。一般来说如果是xqm和原qemu进行replay由于设计的不一致flag、浮点还存在轻微的不一致。但是ms-dos和free-dos的正确对比，说明system-rr分支上和原qemu还是能保持基本一致的。



#### 下一步设计

完善system-rr分支，让xqm和qemu尽可能保持一致。以qemu作参考有十分有利于找出bug。

可能还存在部分基本块划分不一致，在nochain时会影响对比。

tb-link时的record/replay应该没有问题，主要可以根据icount进行bug的寻找。

快照技术，保存任意时刻机器状态，并重新开始执行，大大加快到达bug点的速度。

地址的watch_point，和其他gdb的支持。









