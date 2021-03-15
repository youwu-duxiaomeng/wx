# US20190235849 translation-cache management 



二进制翻译的目的：1.向前向后兼容；2.提高处理效率 

二进制翻译可能是动态，边翻译边执行。二进制翻译通常会建立一个或者多个translation cache（T-caches）去存储翻译后的代码，传统的二进制翻译系统使用LRU的策略进行T-cache的替换。 

这篇专利提出新的T-cache的管理替换策略，使用cost-benefit来进行决策。 

<img src="picture/US20190235849_1" width="400" align=left />                                                                                                                 



计算设备100将源二进制代码翻译成目标二进制代码存入T-cache（可以在内存124中）, 计算设备分配cost-benefit metric（翻译的成本和带来的性能收益）给每一个cache中的代码片段，cache替换时替换掉cost-benefit metric低的。

 

 





<img src="picture/US20190235849_2" width="400" align=left />二级制翻译运行时202翻译原二进制码210到目标二进制码212，存放在T-cache中。代码分析器204记录或者评估翻译时间和性能收益，产生cache中的代码块的cost-benefit metric。cache管理器206决定抛弃cache中的那些代码块。Translation Cache存储翻译后的代码，cost-benefit metric，age等，这个cache可以放在内存124中，也可以是处理器120的专用cache或者存储在其他计算设备中。 



<img src="picture/US20190235849_3" width="400" align=left /> 











  图3是一个cost-benefit metric计算的流程图













<img src="picture/US20190235849_4" width="440" align=left /> 









图4是T-cache的管理流程图 



