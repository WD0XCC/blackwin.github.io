

> windbg  是微软官方提供的一个功能强大的调试器， 可以用于用户态和内核态调试。此外不仅仅用来调试程序， 还可以分析内核、分析崩溃转储文件。在windows平台上有着先天的兼容优势。是安全研究者必须掌握的一个工具。



windbg操作手册： http://windbg.info/doc/1-common-cmds.html



# 1、Windbg基础使用



## 1.1 工作空间

工作空间用来描述调试项目的属性、参数设置等信息。windbg程序目录中的theme子目录提供了4中定制的工作空间设置。将每个主题REG文件导入注册表即可应用该主题。

## 1.2 调试符号

调试符号及其设置参考文章：

http://blog.secknow.cn/post/windows_symbols_download.html

## 1.3 调试过程



调试程序的方式: 

* 以打开方式
* 附加方式
* 分析dump文件

调试模式：

* 远程调试
* 内核调试： 内核调试模式分为5中，net、USB、1394、COM（这四种为双机调试模式）和本地调试



注意： Windbg调试程序时， 反汇编代码默认停留在ntdll.dll中的系统断点处。不会直接同流在程序入口点，可在命令行窗口输入：“：g@$exentry”转到程序入口处。

# 2、 windbg 常用命令



## 2.0 基本命令

**查看和加载模块**

windbg有三种命令： 以.开始的元命令、基本命令和以！开始的扩展命令。

```C
元命令中常用的：
    .hh:           查看指定命令的帮助手册
    .cls:          清屏
    .help:         查看帮助
    .reload:       重新加载符号文件 
	.symfix  path: 设置符号文件的下游缓存路径
    .sympath path: 设置符号文件的下游缓存路径
    .restart:      重新启动调试目标

基本命令：
    ld:            从符号文件目录或符号服务器中加载符号
    lm：           显示已加载模块列表
    lmf            显示所有模块及其路径
    lmD            显示所有模块详细信息
    uf             反汇编某个函数， 比如uf test!main
    ub             反汇编某地址之前的代码，比如ub 0x 0x410040 L20
        
扩展命令常用：
    !analyze -v         详细显示当前异常信息，常用于分析dmp文件
    !address 400000     查看指定内存地址的信息 
    !peb                进程环境块
    !teb                线程环境快
    !locks              查看进程中有些锁处于锁定状态
    !cs -l              查看处于锁定状态的关键区(临界区)
    !handle 000000c0 f  查看句柄000000c0的信息

其他命令：
    x [] Mudole!Symbol:  符号检索功能。其中符号名可以用*、？、[]、#、+等特殊字符进行模糊匹配。eg: x nt!memset  查看内核函数memset
    rM ff: 显示x86架构的所有常用寄存器
```

**单步跟踪**

```C
t:F8/F11, 追踪执行，遇到call指令时跟进
p: F10， 单步执行， 遇到call不跟进
g： F5, 运行程序
pa address: 单步到指定地址，不进入call
ta address: 进入到指定地址， 进入call
pc count: 单步执行到下一个call
tc count: 单步执行到下一个call, 并跟进
tb count: 遇到下一个分支指令，遇到call跟进执行， 适用于内核调试
pt: 单步执行到吓一跳call返回指令
tt: 单步执行到吓一跳call返回指令， 遇到call跟进
ph: 单步到下一条分支指令
th: 单步到下一条分支指令，遇到call跟进
wt: 自动追踪函数执行过程
```



## 2.1 断点命令

**软件断点： 用于设置软件断点INT3**

```
bp：设置和地址关联的断点。
    bp[ID] [options] [Address[Passes]]  ["commandstring"]
	ID：指定断点ID,可缺省，用于模式不限个数， 内核调试限制32个断点
	Options:可缺省。/1(中断后自动删除该断点)、/c(指定最大的调用深度)、/C(指定最小的调用深度)
	Address: 地址或符号
	Passes: 忽略中端次数，可缺省
	CommandString: 用与指定一组命令，当中断时会自动执行这组命令。用双引号包围。

bu: 设置和符号相关的断点， 符号一旦失效断点页会失效。
	eg： bu kernel32!GetVersion
	bu设置的断点在工作空间中， 下次启动windbg时会自动设置该断点

bm: 设置含通配符的断点。可以一次创建一个或者多个bu/bp断点。
	eg:bm msvcr80d!print*   //对模块msvcr80d中以print开头的所有函数设置断点
```



**硬件断点**

硬件断点可以实现一些软件断点无法实现的功能，如监视I/O访问等。

```
ba[ID] Access Size [Options] [Address [passes]] ["commandstring"]
	ID：指定断点ID,可缺省
	Access: 指定断点触发的访问方式：e在读取或执行时中断、r在读取时中断、w在写入数据时中断、i在执行输入/输出访问I/O时触发断点。
	Size:  访问长度, x86中用1、2、4分别代表1字节、2字节、3字节， x64中多了一个8， 代表4字节
	Address: 断点地址，按四则的值进行对齐
	Passes: 忽略中端次数，可缺省
	CommandString: 用与指定一组命令，当中断时会自动执行这组命令。用双引号包围。
```



**条件断点**

软件断点和硬件断点都支持条件断点，当断点被触发后，windbg会执行一些自定义的判断并执行。

```C
bp|bu|bm|ba  _address "j (condiction) 'OPtionalCommands'; 'gc'"

bp|bu|bm|ba  _address ".if(condiction){OPtionalCommands}.else {gc}"
```

windbgz支持更为复杂的条件断点表达式，善用条件断点可以减少不必要的中断，以提高效率。

**管理断点**

```C
bl: 列出当前断点。

bc: 删除断点 //bc  *  删除所有断点

bd: 禁止断点 //bd  1-3, 4   禁止1234号断点

be: 启用断点  
    
```



## 2.2 调用栈命令

栈是进行函数调用的基础，因此观察和分析栈式最重要的调试手段。

```c
k:  基本的查看调用栈

kb: 只用于显放在栈上的前三个参数

kp: 把参数和参数值以函数原型的形式显示出来,包括参数类型、名字、取值（前提时福海完整）

kv: 在kb命令基础上增加帧指针省略信息和调用约定的显示

kd：用于列出栈中的数据
```



## 2.3 内存相关命令

**查看内存：d[类型] [地址范围]**

```C
类型：
	da:
	db: 字节和ASCII字符串
	dc: DWORD和ASCII字符串
	dd: 4字节DWORD格式
	dD:
	df: 4字节单精度浮点数格式
	dp:
	dq: 8字节格式
	du: Unicode字符串
	dw: 双字节WORD格式C
	dW: 双字节和ASCII字符串
	dyb: 显示二进制和字节
	dyd: 显示二进制和DWROD值
	ds: 
	dS: 显示地址及相关符号

地址范围： 用L+数字标识
    dw  4012465  L5: 标识以DWORD格式显示地址 4012465处的前四个数据
```



**搜索内存**

```C
s - [类型] range patten

类型：标识搜索的数据类型。 b: BHYTE、w:WORD、d:DWORD、a:ASCII、u:Unicode,默认类型b

range：表示地址范围。用起始地址 + 终止地址； 或起始地址 + L(长度表示)，若超过256MB, 用L?length

patten: 指定要搜索的内容

s -a 0x00000000 L?0x7fffffff myest:   表示在目标地址为2GB的用户内存空间中搜索ASCII字符串mytest
```



**修改内存**

```C
e {a|u|za|zu} address "string"
  a: 以0结尾的ASCII字符串
  u: 以0结尾的Unicode字符串
  za: 不以0结尾的ASCII字符串
  zu: 不以0结尾的Unicode字符串

e {a|b|d|D|f|q|u|w}  address [values]   //a|b|d|D|f|q|u|w表示类型
```

**观察内存属性C**

```C
!adress  [内存地址]
```



## 2.4 进程线程关命令

栈是进行函数调用的基础，因此观察和分析栈式最重要的调试手段

```C
~*命令显示当前所有线程的详细信息

~* kv可以打印所有线程堆栈。

[~1s] 切换到1号线程，然后可以使用kv查看1号线程的调用栈

!runaway 显示所有线程的CPU消耗

.process 命令指定要用作进程上下文的进程

!process 0 0 获取用户空间的所有的进程的信息

.process /r /p 你需要断的应用程序的EProcess地址    通过/r /p来切换进程上下文

|. 显示当前调试进程

|* 显示当前调试中的所有进程
```



# 3、windbg高级用法

## 3.1 远程调试

## 3.2 内核调试

## 3.3 脚本

