GCC技术参考大全
==============

<font face=楷体>
GCC: The Complete Reference				
Arthur Griffith 著  2002        


</font>


## Gcc命令

|  command  | description |
|-----------|-------------|
| -Wall     | W 能够对源代码进行检测并显示可能存在的错误,即使能够编译通过|
| -I*dir*     |  -I dir 寻找头文件的路径 |
| -L*dir*     | -L dir 搜索库文件的路径  |
| -l*NAME*	| 链接库文件。可直接使用路径；或-lNAME (name来自libNAME) |
| -v        |  |
| -D*NAME* 或者 -D*NAME=VALUE*		| 通过命令行选项，定义c文件中的某个宏变量 |
| $cpp -dM /dev/null | 查看系统定义的宏(以双下划线开头),通过预处理器cpp |






第一部分 自由软件编译程序  
=================


第一章 GCC简介
-------------------
略

第二章 查询与安装编译程序
------------------------------







第二部分 使用编译程序集合
=================


第三章 预处理程序
---------------------


第4章 编译C程序
------------------


第5章 编译C++程序
------------------------


第8章 编译 Java 
-------------------


第10章 混合语言
-------------------


第11章 国际化  
----------------




第3部分  外设和内设
==============



第12章 连接和库
-------------------



第13章 使用GNU调试器
---------------------------


第14章 make 和 Autoconf
---------------------------------



第15章 GUN汇编器
-----------------------




第16章 交叉编译及窗体端口
--------------------------------





第17章 嵌入式系统
----------------------



第18章 编译程序输出
-------------------------



第19章 实现一种语言
-------------------------



第20章 寄存器传送语言
---------------------------




第21章 机器相关的编译程序选项
--------------------------------------

