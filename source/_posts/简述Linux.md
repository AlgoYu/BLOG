---
title: 简述Linux
id: BasicLinux
directory: Guide
tags:
  - Linux
  - 操作系统
  - 服务器
cover: 'https://www.linuxidc.com/upload/2018_08/180818100711451.png'
date: 2018-12-12 17:03:13
---
## **Linux** ##
>Linux是一套免费使用和自由传播的类Unix操作系统，是一个基于POSIX和UNIX的多用户、多任务、支持多线程和多CPU的操作系统。它能运行主要的UNIX工具软件、应用程序和网络协议。它支持32位和64位硬件。Linux继承了Unix以网络为核心的设计思想，是一个性能稳定的多用户网络操作系统。

如何安装我就不再写了，网上实在太多教程了，直接上干货，迅速理解Linux。
## Linux目录结构 ##
Linux不同于Windows一样区分C、D、E……盘，Linux只有一个根目录/，其他的所有目录都是从根目录衍生出来的。
![](http://www.runoob.com/wp-content/uploads/2014/06/003vPl7Rty6E8kZRlAEdc690.jpg)
在Linux的世界里，一切皆是文件，包括你的所有硬件设备，在Linux中都会映射成一个相应文件。

> 
**bin（binary）：** 该目录存放着用户经常使用的命令。
**dev（device）:** 该目录下存放的是Linux的外部设备驱动，在Linux中访问设备的方式和访问文件的方式是相同的。
**home:** 该目录是用户目录，在Linux中，每个用户都会在这个这个目录下创建一个子目录，一般子目录名是以用户的账号命名的。
**lib（library）:** 该目录里存放着系统最基本的动态连接共享库，其作用类似于Windows里的DLL文件。几乎所有的应用程序都需要用到这些共享库。
**mnt（mount）:** 该目录是为了让用户临时挂载别的文件系统的，我们可以将光驱挂载在/mnt/上，然后进入该目录就可以查看光驱里的内容了。
**proc（process）**：该目录是一个虚拟的目录，它是系统内存的映射，我们可以通过直接访问这个目录来获取或修改系统信息。
**run**：是一个临时文件系统，存储系统启动以来的信息，重启清空。
**srv（service）**：该目录存放一些服务启动之后需要提取的数据。
**tmp（temporary）:** 该目录是用来存放一些临时文件的。
**var （variable）:** 该目录中存放着在不断扩充着的东西，一般用来放日志文件。
**boot:** 该目录是存放启动Linux时使用的一些核心文件，包括一些连接文件以及镜像文件。
**etc（etcetera）:** 该目录用来存放所有的系统管理所需要的配置文件和子目录。
**media:** 该目录是linux系统自动识的设备挂载目录，例如U盘、光驱等等，当识别后，linux会把识别的设备挂载到这个目录下。
**opt（option）:** 该目录是给用户额外安装软件目录，比如你安装一个ORACLE数据库则就可以放到这个目录下，默认是空的。
**root :** 该目录为系统管理员，也称作超级权限者的用户主目录。
**sbin（superuser binary）:** 该目录存放的是系统管理员使用的命令。
**sys（sysfs）:** 该目录是linux2.6内核的一个很大的变化。该目录下安装了2.6内核中新出现的一个文件系统 sysfs 。
sysfs文件系统集成了下面3种文件系统的信息：针对进程信息的proc文件系统、针对设备的devfs文件系统以及针对伪终端的devpts文件系统。
该文件系统是内核设备树的一个直观反映。
当一个内核对象被创建的时候，对应的文件和目录也在内核对象子系统中被创建。
**usr（user）:** 该目录是用户应用程序和文件默认安装的目录下。

为了让后面的知识能够正常学习，现在先介绍两个个命令：`cd(Change Directory)`、`ls(List)`。
`列出当前目录下的文件`
```bash
[root@localhost /]# ls
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
```
`使用绝对路径改变当前目录`
```bash
[root@localhost /]# cd /root/Desktop/
```
绝对路径就是从`/`根目录开始寻找的路径，有时候你会看到啊`~/`，它代表了用户的home目录，`cd ~/`可以进入用户的home。
`使用相对路径改变当前目录`
```bash
[root@localhost Desktop]# cd ../Documents/
```
相对路径就是相对与当前目录寻找的路径,可以用符号`./`表示当前目录、`../`表示上一级目录。
上面的案例当前所在的目录是在`/root/Desktop/`下，用`../`表示上一级目录，所以`../Documents/`表示上一级目录中的Documents目录。
>**注意：**
在目录中，上一级目录是也计算成文件夹，只不过是隐藏了起来，可以使用`ls -a`查看。
```bash
[root@localhost Documents]# ls -a
.  ..
```
## 文件管理 ##
以下案例列出了一些基本文件操作。
`创建文件`
```bash
[root@localhost Desktop]# touch QQ794763733.txt
```
`删除文件`
```bash
[root@localhost Desktop]# rm QQ794763733.txt 
rm: remove regular empty file ‘QQ794763733.txt’? yes
```
可以在前面加`-f`忽略警告。
`拷贝文件`
```bash
[root@localhost Desktop]# cp QQ794763733.txt ./123.txt
```
可以直接在目录后面加上拷贝后的文件名。
`移动文件`
```bash
[root@localhost Desktop]# mv QQ794763733.txt /opt/
```
`重命名文件`
```bash
[root@localhost Desktop]# mv QQ794763733.txt ./尊重原著，请勿抄袭.txt
```
把当前目录下的`QQ794763733.txt`文件移动到当前目录下，指定了名字叫`尊重原著，请勿抄袭.txt`，所以就是重命名了。
`创建目录`
```bash
[root@localhost Desktop]# mkdir QQ794763733
```
除了创建方式不一样外，目录的其他操作基本与文件时一致的，有一个注意点，使用`rm`删除文件夹的话，需要加参数`-r`，递归删除目录里的内容，如果是空目录的话，则不用加。
## Vi/Vim编辑器 ##
>Vi是Linux自带的文本编辑器，而Vim则可以看做是Vi的增强版本，增加语法颜色、代码补全，错误跳转……等功能，在程序员中被广泛使用。

Vim有3种模式
> - 正常模式
- 插入模式
- 命令行模式

桌面上有一个文本 `test.txt`，我们使用Vim打开它。
```bash
[root@localhost Desktop]# vim test.txt
```
打开文本后就进入了正常模式，在这个模式下可以【上下左右】按键来移动光标，也可以复制粘贴，删除字符。
`正常模式`
```vim
QQ794763733
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
"test.txt" 1L, 12C                                            1,1           All
```
在正常模式下，我们按`I`就可以进入插入模式，你可以输入任何你想输入的内容，输入完毕后可以按ESC退出编辑模式。
`插入模式`
```vim
QQ794763733
现在已经可以输入内容了
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
-- INSERT --                                                  2,34-23       All
```
现在可以按 `:` 来进入命令行模式，输入 `w` 代表Write写入（保存），输入 `q` 代表退出，输入 `!` 代表强制操作。回车即可执行相关的操作了。
```vim
QQ794763733
现在已经可以输入内容了
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
~                                                                               
:wq

```
现在桌面上的文本 `test.txt` 已经保存了刚才输入的内容，并且退回到了终端。Vim还有许多用法和其他的快捷键操作，这里就大概说一下这样的基本使用。
## 远程登录，注销 ##
在实际的工作场景中，虚拟机界面或者物理服务器本地的终端都是很少接触的，因为服务器装完系统之后，都要放到IDC机房托管，如果是购买的云主机，那更碰不到服务器本体了，只能通过远程连接的方式管理自己的Linux系统。可以通过SSH远程登录软件来连接，可以自行从网上获取。
`注销操作`
```bash
[root@localhost ~]# logout
```
`关机操作`
```bash
[root@localhost ~]# shutdown -h now
```
`重启操作`
```bash
[root@localhost ~]# reboot
```
请注意，如果无法连接Linux，请查看Linux是否开启SSHD服务，或者防火墙是否屏蔽22号端口。
## 用户管理 ##
Linux系统是一个多用户多任务的分时操作系统，任何一个要使用系统资源的用户，都必须首先向系统管理员申请一个用户账号，然后以这个用户账号的身份进入系统。
`增加用户`
```bash
[root@localhost ~]# useradd QQ794763733
```
用户创建的时候会在`/home`目录下创建一个相应的文件夹作为当前用户home目录，文件夹默认为用户名，你也可以通过创建时加`-d`参数来指定文件夹。
`删除用户`
```bash
[root@localhost ~]# userdel QQ794763733
```
删除用户的时候默认不会删除这个用户的home目录，如果要删除该用户的home目录，请在用户名前面加上`-r`参数。
`修改密码`
```bash
[root@localhost home]# passwd QQ794763733
```
root用户可以直接修改指定用户的密码，普通用户修改自己的密码前需要先输入旧密码。
`查询用户`
```bash
[root@localhost home]# id QQ794763733
uid=1001(QQ794763733) gid=1001(QQ794763733) groups=1001(QQ794763733)
```
`uid`代表了用户ID，`gid`代表了用户组ID，`groups`代表了用户组信息
`查询当前用户`
```bash
[root@localhost home]# who am i
root     pts/0        2018-12-13 10:40 (:0)
```
`切换用户`
```bash
[root@localhost home]# su - QQ794763733 
Last login: Thu Dec 13 11:01:12 CST 2018 on pts/0
[QQ794763733@localhost ~]$ exit
logout
[root@localhost home]# 
```
可以直接指定用户名切换用户，权限高的用户切换到权限低的用户不需要密码，反之需要输入密码，返回到原来的用户使用 `exit`。
## 用户组管理 ##
在Linux中，每个用户都属于至少一个用户组（可以多个），用户创建的时候如果没有指定用户组，则会为该账户创建一个新的用户组，默认为该账户的用户名。
`增加用户组`
```bash
[root@localhost home]# groupadd Developer
```
`删除用户组`
```bash
[root@localhost home]# groupdel Developer
```
`创建用户时指定用户组`
```bash
[root@localhost home]# useradd -g Developer QQ794763733
[root@localhost home]# id QQ794763733
uid=1001(QQ794763733) gid=1001(Developer) groups=1001(Developer)
```
`修改用户所属用户组`
```bash
[root@localhost home]# usermod -g Hacker QQ794763733 
[root@localhost home]# id QQ794763733
uid=1001(QQ794763733) gid=1003(Hacker) groups=1003(Hacker)
```
`增加用户所属组`
```bash
[root@localhost home]# usermod -G Developer QQ794763733 
[root@localhost home]# id QQ794763733
uid=1001(QQ794763733) gid=1003(Hacker) groups=1003(Hacker),1004(Developer)
```
**用户信息文件：**
`/etc/passwd`
```bash
QQ794763733:x:1001:1004::/home/QQ794763733:/bin/bash
```
>用户名：用户口令（不可见）：用户ID：用户组ID：注释性描述：用户home目录：登录Shell

**用户口令信息文件：**
`/etc/shadow`
```bash
QQ794763733:!!:17878:0:99999:7:::
```
>用户名：加密口令（不可见）：最后一次修改时间：最小时间间隔：最大时间间隔：警告时间：不活动时间：失效时间：标志

**用户组信息文件：**
`/etc/group`
```bash
Developer:x:1004:QQ794763733
```
>用户组名：口令：用户组ID：用户组用户

## 权限管理 ##
Linux的权限管理和我们的用户与用户组管理紧紧相连，每个文件/目录都有3种权限。

> - **r:**读权限
- **w:**写权限
- **x:**执行权限

每个文件/目录都有一个拥有者，拥有组。默认是创建它的用户和用户所属组。我们可以设置文件的权限，对拥有这个文件/目录的用户、用户组、和其他用户组不同程度的开放。
使用`ls -l`查看桌面下的`test.txt`文件。
```bash
[root@localhost Desktop]# ls -l
total 4
drwxr-xr-x. 2 root root  6 Dec 13 15:00 QQ794763733
-rw-r--r--. 1 root root 46 Dec 13 10:00 test.txt
```
>第一个字母代表文件的类型：（`-`文件）、（`d`目录）、(`l`链接)。
第一个字母后的每3个字母，分别代表文件/目录：（所属用户权限）、（所属用户组权限）、（其他用户组权限）。
权限信息后的数字表示文件个数，如果是文件就是1，如果是目录，那就代表目录下的文件/目录个数。
第一个root代表所属用户，
第二个root代表所属用户组。
文件所属信息后的数字代表文件大小（字节）
然后是日期信息，
最后是文件名。

`QQ794763733`第一个字母`d`代表了这是一个目录，第一组权限`rwx`代表文件所有者拥有读、写、执行的权限,第二组`r-x`代表所属用户组的用户拥有读、执行的权限，`r-x`代表其他用户组拥有读、执行的权限。`2`这个目录下的文件/目录个数，`root`所属用户，`root`所属用户组，`6`目录所占大小字节（Byte），`Dec 13 15:00`日期信息，`QQ794763733`文件名。

`修改文件所属用户`
```bash
[root@localhost Desktop]# chown root QQ794763733
```
`修改文件所属用户组`
```bash
[root@localhost Desktop]# chgrp Developer QQ794763733
```
`修改文件权限`
```bash
[root@localhost Desktop]# chmod u=rwx,g=,o= QQ794763733
```
`u`是文件所属用户的权限，`g`是文件所属用户组的权限，`o`是其他用户组的权限
`运算符修改文件权限`
```bash
[root@localhost Desktop]# chmod o+r QQ794763733
```
`数字修改文件权限`
```bash
[root@localhost Desktop]# chmod 740 QQ794763733
```
`r`=4，`w`=2，`x`=1，计算好权限值可以直接修改。
>**注意：**
想要删除文件，必须拥有当前目录的w权限。
## 运行级别 ##
|级别|含义|
| :------: | :------: |
|0|关机|
|1|单用户模式|
|2|多用户模式|
|3|多用户联网模式|
|4|保留|
|5|图形化界面模式|
|6|重启|
运行级别`3`和`5`使我们最常用的远程或者个人电脑使用，`1`单用户模式可以用来找回root密码，因为进入单用户模式不需要输入密码。
`切换运行级别-多用户联网模式`
```bash
[root@localhost /]# init 3
```
可以直接使用`init`命令来切换运行级别，也可以可以修改`/etc/inittab`这个文件来指定默认的运行级别。切记不能把默认的运行级别设置为0和6，这样会无法正常启动。
## 网路配置 ##
网络配置可以直接修改`/etc/sysconfig/network-scripts/ifcfg-ens33`这个文件内容来进行配置，`ifcfg-ens33`的名字可能有所不同，可以根据自己的环境查看。
```vim
TYPE=Ethernet       //尊重原著QQ794763733
PROXY_METHOD=none 
BROWSER_ONLY=no 
BOOTPROTO=static    //这里修改为static为静态IP、DHCP为动态获取
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no 
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=9e053616-b273-4db9-ae5b-0e3cd34bb041
DEVICE=ens33
ONBOOT=yes              //这里要修改为yes
IPADDR=192.168.179.3    //ip地址  
NETMASK=255.255.255.0   //子网掩码
GATEWAY=192.168.179.2   //网关 
DNS1=202.98.198.167     //DNS首选地址
DNS2=202.98.192.67      //DNS备选地址
~                                                                               
~                                                                               
~                                                                               
"/etc/sysconfig/network-scripts/ifcfg-ens33" 20L, 415C        18,1          All
```
然后重启一下网络服务
```bash
[root@localhost /]# service network restart 
```
## 进程管理 ##
在Linux每个进程都会有一个ID号，每个进程都会对应一个父进程，这些进程可以再后台运行，或者在前台运行。
`查看进程`
```bash
[root@localhost /]# ps -aux
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root          1  0.0  0.3 193948  6968 ?        Ss   10:34   0:22 /usr/lib/syste
root          2  0.0  0.0      0     0 ?        S    10:34   0:00 [kthreadd]
root          3  0.0  0.0      0     0 ?        S    10:34   0:00 [ksoftirqd/0]
root          5  0.0  0.0      0     0 ?        S<   10:34   0:00 [kworker/0:0H]
root          6  0.0  0.0      0     0 ?        S    10:34   0:05 [kworker/u256:
root          7  0.0  0.0      0     0 ?        S    10:34   0:00 [migration/0]
root          8  0.0  0.0      0     0 ?        S    10:34   0:00 [rcu_bh]
root          9  0.0  0.0      0     0 ?        S    10:34   0:08 [rcu_sched]
root         10  0.0  0.0      0     0 ?        S<   10:34   0:00 [lru-add-drain
root         11  0.0  0.0      0     0 ?        S    10:34   0:00 [watchdog/0]
root         12  0.0  0.0      0     0 ?        S    10:34   0:00 [watchdog/1]
root         13  0.0  0.0      0     0 ?        S    10:34   0:00 [migration/1]
```
`ps`命令：`-a`查看所有进程，`-u`以用户的格式显示所有信息，`-x`显示后台进程运行参数

>- USER：用户名
- PID：进程ID
- %CPU：CPU占用率
- %MEM：内存占用率
- VSZ：进程所占用的虚拟内存大小（单位：KB）
- RSS：进程所占用的物理内存大小（单位：KB）
- TTY：终端名称
- STAT：进程状态，S-睡眠，s-表示该进程是绘画的先导进程，N-表示进程拥有比普通优先级更低的优先级，R-正在运行，D-短期等待，Z-僵尸进程，T-被跟踪或者被停止等等。
- START：进程的启动时间
- PID：进程ID
- TIME：CPU时间，进程使用CPU的总时间。
- COMMAND：启用进程所用的命令和参数，如果过长会被截断显示。

今天写到这里，未完待续。。
<b><font color="FF0000">文章内容均为本人手写；<br>禁止转载，请尊重原著；<br>本人QQ：794763733。</font></b>