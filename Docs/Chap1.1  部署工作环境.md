## 环境搭建
- VMware Workstation 14Pro
- Ubuntu-20.04.5-amd64
- Bochs 2.6.11
## 完整步骤
### S1：配置虚拟机环境
采用Ubuntu-20.04.5搭建基础的Linux环境，过程略
### S2：安装必要的依赖

1. 在安装好的虚拟机上安装Vim，openssh-client，openssh-server，net-tools等必要的工具
2. 安装gcc（g++），gdb，gtk相关库，cairo库
```
sudo apt-get install build-essential
sudo apt-get install xorg-dev
sudo apt-get install libgtk2.0-dev
```
### S3：下载安装Bochs
Bochs（读作box）是一个以GNU宽通用公共许可证发放的开放源代码的x86、x86-64IBM PC兼容机模拟器和调试工具，支持处理器（包括保护模式），内存，硬盘，显示器，以太网，BIOS，IBM PC兼容机的常见硬件外设的仿真。利用Bochs的方便主要体现在开发和调试仿真操作系统的时候，一旦崩溃，不会造成主机操作系统（这里面是指虚拟机中的Linux）崩溃，降低开发的调试的成本。

#### S3.1：下载解压
下载地址：[https://bochs.sourceforge.io/](https://bochs.sourceforge.io/)<br />选择需要的版本进行下载，我这里选用的是2.6.11（最新版的2.7在make过程中会有无法修复的错误，建议降版本）
```
tar -zxvf bochs-2.6.11.tar.gz
```
解压后进入目录
#### S3.2：configure
```
./configure --with-x11 --with-wx --enable-debugger --enable-disasm --enable-all-optimizations --enable-readline --enable-long-phy-address --enable-ltdl-install --enable-idle-hack --enable-plugins --enable-a20-pin --enable-x86-64 --enable-smp --enable-cpu-level=6 --enable-large-ramfile --enable-repeat-speedups --enable-fast-function-calls  --enable-handlers-chaining  --enable-trace-linking --enable-configurable-msrs --enable-show-ips  --enable-debugger-gui --enable-iodebug --enable-logging --enable-assert-checks --enable-fpu --enable-vmx=2 --enable-svm --enable-3dnow --enable-alignment-check  --enable-monitor-mwait --enable-avx  --enable-evex --enable-x86-debugger --enable-pci --enable-usb --enable-voodoo

```
删去--enable-cpp可以防止后续由于文件后缀出现的诸多问题

#### S3.3：make&make install
完成上述操作后，切换到root进行make操作<br />在Makefile的LIBS后追加 -lm 和-lpthread<br />如果未出现报错，直接make install完成Bochs的安装

#### S3.4：处理报错
错误1：
```
x86_64-pc-linux-gnu-g++ -c -I.. -I./.. -I../instrument/stubs -I./../instrument/stubs -I. -I./. -march=native -O2 -pipe -D_FILE_OFFSET_BITS=64 -D_LARGE_FILES -pthread   -I/usr/include/SDL -D_GNU_SOURCE=1 -D_REENTRANT dbg_main.cc -o dbg_main.o
In file included from dbg_main.cc:28:
dbg_main.cc: In function ‘void bx_dbg_tlb_lookup(bx_lin_address)’:
../cpu/cpu.h:383:28: error: invalid use of ‘this’ in non-member function
  383 | #  define BX_CPU_THIS_PTR  this->
      |                            ^~~~
../cpu/tlb.h:62:32: note: in expansion of macro ‘BX_CPU_THIS_PTR’
   62 | #define BX_ITLB_INDEX_OF(lpf) (BX_CPU_THIS_PTR ITLB.get_index_of(lpf))
      |                                ^~~~~~~~~~~~~~~
dbg_main.cc:1497:18: note: in expansion of macro ‘BX_ITLB_INDEX_OF’
 1497 |   Bit32u index = BX_ITLB_INDEX_OF(laddr);
      |                  ^~~~~~~~~~~~~~~~
../cpu/cpu.h:383:28: error: invalid use of ‘this’ in non-member function
  383 | #  define BX_CPU_THIS_PTR  this->
      |                            ^~~~
../cpu/tlb.h:59:37: note: in expansion of macro ‘BX_CPU_THIS_PTR’
   59 | #define BX_DTLB_INDEX_OF(lpf, len) (BX_CPU_THIS_PTR DTLB.get_index_of((lpf), (len)))
      |                                     ^~~~~~~~~~~~~~~
dbg_main.cc:1501:11: note: in expansion of macro ‘BX_DTLB_INDEX_OF’
 1501 |   index = BX_DTLB_INDEX_OF(laddr, 0);
      |           ^~~~~~~~~~~~~~~~
make[1]: *** [Makefile:67: dbg_main.o] Error 1

```
解决方案：<br />参考[https://salsa.debian.org/debian/bochs/-/tree/master](https://salsa.debian.org/debian/bochs/-/tree/master)
```
Description: Fix the build with SMP enabled
Origin: https://sourceforge.net/p/bochs/code/13778/

Index: bochs/bx_debug/dbg_main.cc
===================================================================
--- bochs/bx_debug/dbg_main.cc	(revision 13777)
+++ bochs/bx_debug/dbg_main.cc	(working copy)
@@ -1494,11 +1494,11 @@
 {
   char cpu_param_name[16];
 
-  Bit32u index = BX_ITLB_INDEX_OF(laddr);		//这一行改成下面一行
+  Bit32u index = BX_CPU(dbg_cpu)->ITLB.get_index_of(laddr);
   sprintf(cpu_param_name, "ITLB.entry%d", index);
   bx_dbg_show_param_command(cpu_param_name, 0);
 
-  index = BX_DTLB_INDEX_OF(laddr, 0);		//同理
+  index = BX_CPU(dbg_cpu)->DTLB.get_index_of(laddr);
   sprintf(cpu_param_name, "DTLB.entry%d", index);
   bx_dbg_show_param_command(cpu_param_name, 0);
 }

Description: Fix the build with SMP enabled
Origin: https://sourceforge.net/p/bochs/code/13778/

Index: bochs/bx_debug/dbg_main.cc
===================================================================
--- bochs/bx_debug/dbg_main.cc	(revision 13777)
+++ bochs/bx_debug/dbg_main.cc	(working copy)
@@ -1494,11 +1494,11 @@
 {
   char cpu_param_name[16];
 
-  Bit32u index = BX_ITLB_INDEX_OF(laddr);		//这一行改成下面一行
+  Bit32u index = BX_CPU(dbg_cpu)->ITLB.get_index_of(laddr);
   sprintf(cpu_param_name, "ITLB.entry%d", index);
   bx_dbg_show_param_command(cpu_param_name, 0);
 
-  index = BX_DTLB_INDEX_OF(laddr, 0);		//同理
+  index = BX_CPU(dbg_cpu)->DTLB.get_index_of(laddr);
   sprintf(cpu_param_name, "DTLB.entry%d", index);
   bx_dbg_show_param_command(cpu_param_name, 0);
 }

```
错误2：<br />error: 'XRRQueryExtension' was not declared in this scope; did you mean 'XQueryExtension'?<br />解决方案：<br />更改gui/x.cpp，在首行添加头文件 #include<X11/extensions/Xrandr.h>

### S4：配置Bochs
#### S4.1：本部分最麻烦的步骤
配置Bochs目的就是告诉系统这是一个什么样子的机器<br />配置文件在**安装目录下**的share/doc/bochs文件夹里有个样例

1. 把他拷贝出来然后做一些修改就行了（这里我将新的配置文件命名为bochsrc.disk）
```
cp BOCHSPATH/share/doc/bochs/bochsrc-sample.txt BOCHSPATH/bin/bochsrc.disk
# 其中BOCHSPATH代表Bochs的安装目录，一般为/usr/local
```

2. 在配置文件中改动一下部分：
```
megs : 32

romimage: file=BOCHSPATH/share/bochs/BIOS-bochs-latest
vgaromimage: file=BOCHSPATH/share/bochs/VGABIOS-lgpl-latest

boot: disk

log: bochsout.txt

mouse:enabled=0
keyboard——mapping:enabled=1,
map=file=BOCHSPATHshare/bochs/keymaps/x11-pc-us.map

  
# enable keymapping, using Us layout as default  
keyboard:keymap=/usr/local/share/bochs/keymaps/x11-pc-us.map

```
#### S4.2：可能存在的问题
##### S4.2.1：sound和speaker的配置
```
bochsrc:915: Bochs is not compiled with lowlevel sound support.
<这里915也可能是其他行>
```
解决方法：删除或注释掉关于sound和speaker的配置

##### S4.2.2：缺少依赖导致undefined reference
```
undefined reference to symbol ’XSetForeground’
```
解决方法：sudo apt install xorg-dev

##### S4.2.3：ata0-0: could not open hard drive image file '30M.sample'
解决方法：使用了从bochsrc-sample.txt拷贝过来的配置，其中有一项默认配置ata0-master，将该配置注释掉，换成ata0-master: type=none。


### S5：创建虚拟硬盘
直接在命令行（bin目录下）输入**bximage**，按照指引配置完成即可<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/32593778/1672909276570-3a93cbfa-99db-436b-9a44-8325036d2640.png#averageHue=%23310b25&clientId=udcf826a9-ce8d-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=458&id=u5d29627a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=458&originWidth=910&originalType=binary&ratio=1&rotation=0&showTitle=false&size=144008&status=done&style=none&taskId=ufb4cd818-0a77-4082-8f2b-1b7d1cdcfab&title=&width=910)<br />随后需要更改配置文件，将磁盘和磁道扇区写入配置文件区域<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/32593778/1672909330400-311ef107-5f41-4d52-b559-c646615ecc62.png#averageHue=%23eeebe8&clientId=udcf826a9-ce8d-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=39&id=u47ba219f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=39&originWidth=929&originalType=binary&ratio=1&rotation=0&showTitle=false&size=5973&status=done&style=none&taskId=ub4ccbd17-4293-43ec-9024-ba872da7c0e&title=&width=929)<br />随后启动<br />最终系统报错为**Boot failed:not a bootable device**<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/32593778/1672898709448-f99834c4-9439-41ee-94ea-75d76f24bb76.png#averageHue=%23bdcfc6&clientId=udcf826a9-ce8d-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=599&id=u2dd31511&margin=%5Bobject%20Object%5D&name=image.png&originHeight=599&originWidth=1144&originalType=binary&ratio=1&rotation=0&showTitle=false&size=104703&status=done&style=none&taskId=u857fd478-366d-4923-adee-328cb9bc4ba&title=&width=1144)<br />由于没有编写MBR主引导，所以依旧不能正常进入系统，留到下一章节实现

### 特别说明
由于Linux的选择（CentOS/Ubuntu）以及Bochs的版本不同，报错会有很多有出入的地方，需要自行搜索解决
