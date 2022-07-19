# WIN(CMD/Power Shell)

## 查看端口占用，释放端口

```shell
netstat -ano
```

### 过滤指定端口

```shell
netstat -ano|findstr "8081"
```

```
C:\Users\13649>netstat -ano|findstr "8081"
  TCP    0.0.0.0:8081           0.0.0.0:0              LISTENING       5712
  TCP    [::]:8081              [::]:0                 LISTENING       5712
  TCP    [::1]:8081             [::1]:58471            ESTABLISHED     5712
  TCP    [::1]:58471            [::1]:8081             ESTABLISHED     16372
```

### 查看指定pid的进程

```shell
tasklist|findstr "5721"
```

```
___go_build_go_code_gp_gi     5712 Console                    1     10,644 K
```

### 结束进程

```shell
taskkill /T /F /PID 9088
```

```
C:\Users\13649>taskkill /T /F /PID 5712
成功: 已终止 PID 13820 (属于 PID 5712 子进程)的进程。
成功: 已终止 PID 5712 (属于 PID 16212 子进程)的进程。
```



## 快捷键

1、ESC：清除当前命令行；

2、F7：显示命令历史记录，以图形列表窗的形式给出所有曾经输入的命令，并可用上下箭头键选择再次执行该命令。

3、F8：搜索命令的历史记录，循环显示所有曾经输入的命令，直到按下回车键为止；

4、F9：按编号选择命令，以图形对话框方式要求您输入命令所对应的编号（从0开始），并将该命令显示在屏幕上

5、Ctrl+H：删除光标左边的一个字符；

6、Ctrl+C Ctrl+Break，强行中止命令执行

7、Ctrl+M：表示回车确认键；

8、Alt+F7：清除所有曾经输入的命令历史记录

9、Alt+PrintScreen：截取屏幕上当前命令窗里的内容。

# Linux

## 常用快捷键

### ctrl+r

```
用途：反向搜索执行过的命令。(reverse-i-search)

若对于现有history
611 ruby foo.rb
612 ruby bar.rb
613 ruby fo.rb
614 ruby ba.rb
615 ...
...
700 ...

在不知道序号的情况下，若要运行ruby foo.rb。
1、ctrl+r
2、foo
或
1、ctrl+r
2、fo
3、ctrl+r (继续反向搜索)
```



## 环境变量

### 修改环境变量文件

```
vim /etc/profile
或者
vim ~/.bashrc
```

### 修改完后source

```
source /etc/profile
或者
source ~/.bashrc
```

### 重启后/etc/profile不生效

在~/.zshrc最后添加source /etc/profile即可

![image-20220308015702306](http://myimg.go2flare.xyz/img/image-20220308015702306.png)

## lsof

下面的一些其它东西需要牢记：

- 默认 : 没有选项，lsof列出活跃进程的所有打开文件
- 组合 : 可以将选项组合到一起，如-abc，但要当心哪些选项需要参数
- -a : 结果进行“与”运算（而不是“或”）
- -l : 在输出显示用户ID而不是用户名
- -h : 获得帮助
- -t : 仅获取进程ID
- -U : 获取UNIX套接口地址
- -F : 格式化输出结果，用于其它命令。可以通过多种方式格式化，如-F pcfn（用于进程id、命令名、文件描述符、文件名，并以空终止）

### 获取网络信息

正如我所说的，我主要将lsof用于获取关于系统怎么和网络交互的信息。这里提供了关于此信息的一些主题：

#### 使用-i显示所有连接

有些人喜欢用netstat来获取网络连接，但是我更喜欢使用lsof来进行此项工作。结果以对我来说很直观的方式呈现，我仅仅只需改变我的语法，就可以通过同样的命令来获取更多信息。

语法: lsof -i[46] [protocol][@hostname|hostaddr][:service|port]



```css
1.  #  lsof  -i

3.  COMMAND  PID USER   FD   TYPE DEVICE SIZE NODE NAME
4.  dhcpcd 6061 root 4u  IPv4  4510 UDP *:bootpc
5.  sshd  7703 root 3u  IPv6  6499 TCP *:ssh  (LISTEN)
6.  sshd  7892 root 3u  IPv6  6757 TCP 10.10.1.5:ssh->192.168.1.5:49901  (ESTABLISHED)
```

#### 使用-i 6仅获取IPv6流量



```bash
1.  #  lsof  -i 6
```

#### 仅显示TCP连接（同理可获得UDP连接）

你也可以通过在-i后提供对应的协议来仅仅显示TCP或者UDP连接信息。



```css
1.  #  lsof  -iTCP

3.  COMMAND  PID USER   FD   TYPE DEVICE SIZE NODE NAME
4.  sshd  7703 root 3u  IPv6  6499 TCP *:ssh  (LISTEN)
5.  sshd  7892 root 3u  IPv6  6757 TCP 10.10.1.5:ssh->192.168.1.5:49901  (ESTABLISHED)
```

#### 使用-i:port来显示与指定端口相关的网络信息

或者，你也可以通过端口搜索，这对于要找出什么阻止了另外一个应用绑定到指定端口实在是太棒了。



```css
1.  #  lsof  -i :22

3.  COMMAND  PID USER   FD   TYPE DEVICE SIZE NODE NAME
4.  sshd  7703 root 3u  IPv6  6499 TCP *:ssh  (LISTEN)
5.  sshd  7892 root 3u  IPv6  6757 TCP 10.10.1.5:ssh->192.168.1.5:49901  (ESTABLISHED)
```

#### 使用@host来显示指定到指定主机的连接

这对于你在检查是否开放连接到网络中或互联网上某个指定主机的连接时十分有用。



```css
1.  #  lsof  -i@172.16.12.5

3.  sshd  7892 root 3u  IPv6  6757 TCP 10.10.1.5:ssh->172.16.12.5:49901  (ESTABLISHED)
```

#### 使用@host:port显示基于主机与端口的连接

你也可以组合主机与端口的显示信息。



```ruby
1.  #  lsof  -i@172.16.12.5:22

3.  sshd  7892 root 3u  IPv6  6757 TCP 10.10.1.5:ssh->172.16.12.5:49901  (ESTABLISHED)
```

#### 找出监听端口

找出正等候连接的端口。



```bash
1.  #  lsof  -i -sTCP:LISTEN
```

你也可以grep “LISTEN”来完成该任务。



```ruby
1.  #  lsof  -i |  grep  -i LISTEN

3.  iTunes 400 daniel 16u  IPv4  0x4575228  0t0 TCP *:daap (LISTEN)
```

#### 找出已建立的连接

你也可以显示任何已经连接的连接。



```bash
1.  #  lsof  -i -sTCP:ESTABLISHED
```

你也可以通过grep搜索“ESTABLISHED”来完成该任务。



```ruby
1.  #  lsof  -i |  grep  -i ESTABLISHED

3.  firefox-b 169 daniel 49u  IPv4  0t0 TCP 1.2.3.3:1863->1.2.3.4:http (ESTABLISHED)
```

### 用户信息

你也可以获取各种用户的信息，以及它们在系统上正干着的事情，包括它们的网络活动、对文件的操作等。

#### 使用-u显示指定用户打开了什么



```bash
1.  #  lsof  -u daniel

3.  -- snipped --
4.  Dock  155 daniel  txt REG 14,2  2798436  823208  /usr/lib/libicucore.A.dylib
5.  Dock  155 daniel  txt REG 14,2  1580212  823126  /usr/lib/libobjc.A.dylib
6.  Dock  155 daniel  txt REG 14,2  2934184  823498  /usr/lib/libstdc++.6.0.4.dylib
7.  Dock  155 daniel  txt REG 14,2  132008  823505  /usr/lib/libgcc_s.1.dylib
8.  Dock  155 daniel  txt REG 14,2  212160  823214  /usr/lib/libauto.dylib
9.  -- snipped --
```

#### 使用-u user来显示除指定用户以外的其它所有用户所做的事情



```bash
1.  #  lsof  -u ^daniel

3.  -- snipped --
4.  Dock  155 jim  txt REG 14,2  2798436  823208  /usr/lib/libicucore.A.dylib
5.  Dock  155 jim  txt REG 14,2  1580212  823126  /usr/lib/libobjc.A.dylib
6.  Dock  155 jim  txt REG 14,2  2934184  823498  /usr/lib/libstdc++.6.0.4.dylib
7.  Dock  155 jim  txt REG 14,2  132008  823505  /usr/lib/libgcc_s.1.dylib
8.  Dock  155 jim  txt REG 14,2  212160  823214  /usr/lib/libauto.dylib
9.  -- snipped --
```

#### 杀死指定用户所做的一切事情

可以消灭指定用户运行的所有东西，这真不错。

```bash
1.  #  kill  -9  `lsof -t -u daniel`
```

### 命令和进程

可以查看指定程序或进程由什么启动，这通常会很有用，而你可以使用lsof通过名称或进程ID过滤来完成这个任务。下面列出了一些选项：

#### 使用-c查看指定的命令正在使用的文件和网络连接



```bash
1.  #  lsof  -c syslog-ng

3.  COMMAND    PID USER   FD   TYPE     DEVICE    SIZE       NODE NAME
4.  syslog-ng 7547 root  cwd    DIR 3,3  4096  2  /
5.  syslog-ng 7547 root  rtd    DIR 3,3  4096  2  /
6.  syslog-ng 7547 root  txt    REG 3,3  113524  1064970  /usr/sbin/syslog-ng
7.  -- snipped --
```

#### 使用-p查看指定进程ID已打开的内容

```bash
1.  #  lsof  -p 10075

3.  -- snipped --
4.  sshd  10068 root  mem    REG 3,3  34808  850407  /lib/libnss_files-2.4.so
5.  sshd  10068 root  mem    REG 3,3  34924  850409  /lib/libnss_nis-2.4.so
6.  sshd  10068 root  mem    REG 3,3  26596  850405  /lib/libnss_compat-2.4.so
7.  sshd  10068 root  mem    REG 3,3  200152  509940  /usr/lib/libssl.so.0.9.7
8.  sshd  10068 root  mem    REG 3,3  46216  510014  /usr/lib/liblber-2.3
9.  sshd  10068 root  mem    REG 3,3  59868  850413  /lib/libresolv-2.4.so
10.  sshd  10068 root  mem    REG 3,3  1197180  850396  /lib/libc-2.4.so
11.  sshd  10068 root  mem    REG 3,3  22168  850398  /lib/libcrypt-2.4.so
12.  sshd  10068 root  mem    REG 3,3  72784  850404  /lib/libnsl-2.4.so
13.  sshd  10068 root  mem    REG 3,3  70632  850417  /lib/libz.so.1.2.3
14.  sshd  10068 root  mem    REG 3,3  9992  850416  /lib/libutil-2.4.so
15.  -- snipped --
```

#### -t选项只返回PID



```bash
1.  #  lsof  -t -c Mail

3.  350
```

### 文件和目录

通过查看指定文件或目录，你可以看到系统上所有正与其交互的资源——包括用户、进程等。

#### 显示与指定目录交互的所有一切



```bash
1.  #  lsof  /var/log/messages/

3.  COMMAND    PID USER   FD   TYPE DEVICE   SIZE   NODE NAME
4.  syslog-ng 7547 root 4w REG 3,3  217309  834024  /var/log/messages
```

### 显示与指定文件交互的所有一切



```bash
1.  #  lsof  /home/daniel/firewall_whitelist.txt
```

### 高级用法

与[tcpdump](http://danielmiessler.com/study/tcpdump/)类似，当你开始组合查询时，它就显示了它强大的功能。

#### 显示daniel连接到1.1.1.1所做的一切



```css
1.  #  lsof  -u daniel -i @1.1.1.1

3.  bkdr 1893 daniel 3u  IPv6  3456 TCP 10.10.1.10:1234->1.1.1.1:31337  (ESTABLISHED)
```

#### 同时使用-t和-c选项以给进程发送 HUP 信号



```bash
1.  #  kill  -HUP `lsof -t -c sshd`
```

#### lsof +L1显示所有打开的链接数小于1的文件

这通常（当不总是）表示某个攻击者正尝试通过删除文件入口来隐藏文件内容。



```bash
1.  #  lsof  +L1

3.  (hopefully nothing)
```

#### 显示某个端口范围的打开的连接



```ruby
1.  #  lsof  -i @fw.google.com:2150=2180
```

### 结尾

本入门教程只是管窥了lsof功能的一斑，要查看完整参考，运行man lsof命令或查看[在线版本](http://www.netadmintools.com/html/lsof.man.html)。希望本文对你有所助益，也随时[欢迎你的评论和指正](http://danielmiessler.com/connect/)。

### 资源

- lsof手册页：http://www.netadmintools.com/html/lsof.man.html

本文由 Daniel Miessler撰写，首次在他[博客](http://danielmiessler.com/study/lsof/)上贴出

一般root用户才能执行lsof命令，普通用户可以看见/usr/sbin/lsof命令，
 但是普通用户执行会显示“permission denied”

## 我总结一下lsof指令的用法：

lsof abc.txt 显示开启文件abc.txt的进程

lsof -i :22 知道22端口现在运行什么程序

lsof -c abc 显示abc进程现在打开的文件

lsof -g gid 显示归属gid的进程情况

lsof +d /usr/local/ 显示目录下被进程开启的文件

lsof +D /usr/local/ 同上，但是会搜索目录下的目录，时间较长

lsof -d 4 显示使用fd为4的进程  [www.2cto.com](http://www.2cto.com)

lsof -i 用以显示符合条件的进程情况

语法: lsof -i[46] [protocol][@hostname|hostaddr][:service|port]

46 --> IPv4 or IPv6

protocol --> TCP or UDP

hostname --> Internet host name

hostaddr --> IPv4位置

service --> /etc/service中的 service name (可以不只一个)

port --> 端口号 (可以不只一个)

例子: TCP:25 - TCP and port 25

@1.2.3.4 - Internet IPv4 host address 1.2.3.4

[tcp@ohaha.ks.edu.tw](mailto:tcp@ohaha.ks.edu.tw):ftp - TCP protocol [hosthaha.ks.edu.tw](http://hosthaha.ks.edu.tw) service name:ftp

lsof -n 不将IP转换为hostname，缺省是不加上-n参数

例子: lsof -i [tcp@ohaha.ks.edu.tw](mailto:tcp@ohaha.ks.edu.tw):ftp -n

lsof -p 12 看进程号为12的进程打开了哪些文件

lsof +|-r [t] 控制lsof不断重复执行，缺省是15s刷新

-r，lsof会永远不断的执行，直到收到中断信号

+r，lsof会一直执行，直到没有档案被显示

例子：不断查看目前ftp连接的情况：lsof -i [tcp@ohaha.ks.edu.tw](mailto:tcp@ohaha.ks.edu.tw):ftp -r

lsof -s 列出打开文件的大小，如果没有大小，则留下空白

lsof -u username 以UID，列出打开的文件  [www.2cto.com](http://www.2cto.com)

关注：
 进程调试命令:truss、strace和ltrace
 进程无法启动，软件运行速度突然变慢，程序的"SegmentFault"等等都是让每个Unix系统用户头痛的问题，而这些问题都可以通过使用truss、strace和ltrace这三个常用的调试工具来快速诊断软件的"疑难杂症"。



在使用linux时，经常需要进行文件查找。其中查找的命令主要有find和grep。两个命令是有区的。

　　区别：(1)find命令是根据文件的属性进行查找，如文件名，文件大小，所有者，所属组，是否为空，访问时间，修改时间等。 

                  (2)grep是根据文件的内容进行查找，会对文件的每一行按照给定的模式(patter)进行匹配查找。
    
                  (3)which       查看可执行文件的位置 ，只有设置了环境变量的程序才可以用
    
                  (4)whereis    寻找特定文件，只能用于查找二进制文件、源代码文件和man手册页
    
                  (5)locate       配合数据库查看文件位置 ,详情：locate -h查看帮助信息

​      

# 端口请求

1)统计80端口连接数
netstat -nat|grep -i "80"|wc -l

2）统计httpd协议连接数
ps -ef|grep httpd|wc -l

3）、统计已连接上的，状态为“established
netstat -na|grep ESTABLISHED|wc -l

4)、查出哪个IP地址连接最多,将其封了.
netstat -na|grep ESTABLISHED|awk {print $5}|awk -F: {print $1}|sort|uniq -c|sort -r +0n

netstat -na|grep SYN|awk {print $5}|awk -F: {print $1}|sort|uniq -c|sort -r +0n

---------------------------------------------------------------------------------------------

1、查看apache当前并发访问数：
netstat -an | grep ESTABLISHED | wc -l

对比httpd.conf中MaxClients的数字差距多少。

2、查看有多少个进程数：
ps aux|grep httpd|wc -l

3、可以使用如下参数查看数据
server-status?auto

#ps -ef|grep httpd|wc -l
1388
统计httpd进程数，连个请求会启动一个进程，使用于Apache服务器。
表示Apache能够处理1388个并发请求，这个值Apache可根据负载情况自动调整。

#netstat -nat|grep -i "80"|wc -l
4341
netstat -an会打印系统当前网络链接状态，而grep -i "80"是用来提取与80端口有关的连接的，wc -l进行连接数统计。
最终返回的数字就是当前所有80端口的请求总数。

#netstat -na|grep ESTABLISHED|wc -l
376
netstat -an会打印系统当前网络链接状态，而grep ESTABLISHED 提取出已建立连接的信息。 然后wc -l统计。
最终返回的数字就是当前所有80端口的已建立连接的总数。

netstat -nat||grep ESTABLISHED|wc - 可查看所有建立连接的详细记录

查看Apache的并发请求数及其TCP连接状态：
　　Linux命令：
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'

（这条语句是从 新浪互动社区事业部 新浪互动社区事业部技术总监王老大那儿获得的，非常不错）返回结果示例：
　　LAST_ACK 5
　　SYN_RECV 30
　　ESTABLISHED 1597
　　FIN_WAIT1 51
　　FIN_WAIT2 504
　　TIME_WAIT 1057
　　其中的
SYN_RECV表示正在等待处理的请求数；
ESTABLISHED表示正常数据传输状态；
TIME_WAIT表示处理完毕，等待超时结束的请求数。

---------------------------------------------------------------------------------------------

查看Apache并发请求数及其TCP连接状态

查看httpd进程数（即prefork模式下Apache能够处理的并发请求数）：
　　Linux命令：

ps -ef | grep httpd | wc -l

　　返回结果示例：
　　1388
　　表示Apache能够处理1388个并发请求，这个值Apache可根据负载情况自动调整，我这组服务器中每台的峰值曾达到过2002。

查看Apache的并发请求数及其TCP连接状态：
　　Linux命令：

netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
返回结果示例：
　　LAST_ACK 5
　　SYN_RECV 30
　　ESTABLISHED 1597
　　FIN_WAIT1 51
　　FIN_WAIT2 504
　　TIME_WAIT 1057
　　其中的SYN_RECV表示正在等待处理的请求数；ESTABLISHED表示正常数据传输状态；TIME_WAIT表示处理完毕，等待超时结束的请求数。
　　状态：描述

　　CLOSED：无连接是活动 的或正在进行

　　LISTEN：服务器在等待进入呼叫

　　SYN_RECV：一个连接请求已经到达，等待确认

　　SYN_SENT：应用已经开始，打开一个连接

　　ESTABLISHED：正常数据传输状态

　　FIN_WAIT1：应用说它已经完成

　　FIN_WAIT2：另一边已同意释放

　　ITMED_WAIT：等待所有分组死掉

　　CLOSING：两边同时尝试关闭

　　TIME_WAIT：另一边已初始化一个释放

　　LAST_ACK：等待所有分组死掉




如发现系统存在大量TIME_WAIT状态的连接，通过调整内核参数解决，
vim /etc/sysctl.conf
编辑文件，加入以下内容：
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_fin_timeout = 30
然后执行 /sbin/sysctl -p 让参数生效。

net.ipv4.tcp_syncookies = 1 表示开启SYN cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭；
net.ipv4.tcp_tw_reuse = 1 表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；
net.ipv4.tcp_tw_recycle = 1 表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭。
net.ipv4.tcp_fin_timeout 修改系統默认的 TIMEOUT 时间

下面附上TIME_WAIT状态的意义：

客户端与服务器端建立TCP/IP连接后关闭SOCKET后，服务器端连接的端口
状态为TIME_WAIT

是不是所有执行主动关闭的socket都会进入TIME_WAIT状态呢？
有没有什么情况使主动关闭的socket直接进入CLOSED状态呢？

主动关闭的一方在发送最后一个 ack 后
就会进入 TIME_WAIT 状态 停留2MSL（max segment lifetime）时间
这个是TCP/IP必不可少的，也就是“解决”不了的。

也就是TCP/IP设计者本来是这么设计的
主要有两个原因
1。防止上一次连接中的包，迷路后重新出现，影响新连接
（经过2MSL，上一次连接中所有的重复包都会消失）
2。可靠的关闭TCP连接
在主动关闭方发送的最后一个 ack(fin) ，有可能丢失，这时被动方会重新发
fin, 如果这时主动方处于 CLOSED 状态 ，就会响应 rst 而不是 ack。所以
主动方要处于 TIME_WAIT 状态，而不能是 CLOSED 。

TIME_WAIT 并不会占用很大资源的，除非受到攻击。

还有，如果一方 send 或 recv 超时，就会直接进入 CLOSED 状态


如何合理设置apache httpd的最大连接数？

手头有一个网站在线人数增多，访问时很慢。初步认为是服务器资源不足了，但经反复测试，一旦连接上，不断点击同一个页面上不同的链接，都能迅速打开，这种现象就是说明apache最大连接数已经满了，新的访客只能排队等待有空闲的链接，而如果一旦连接上，在keeyalive 的存活时间内（KeepAliveTimeout，默认5秒）都不用重新打开连接，因此解决的方法就是加大apache的最大连接数。

1.在哪里设置？
apache 2.24，使用默认配置（FreeBSD 默认不加载自定义MPM配置），默认最大连接数是250

在/usr/local/etc/apache22/httpd.conf中加载MPM配置（去掉前面的注释）：
# Server-pool management (MPM specific)
```
Include etc/apache22/extra/httpd-mpm.conf
```

可见的MPM配置在/usr/local/etc/apache22/extra/httpd-mpm.conf，但里面根据httpd的工作模式分了很多块，哪一部才是当前httpd的工作模式呢？可通过执行 apachectl -l 来查看：

```
Compiled in modules:
              core.c
              prefork.c
              http_core.c
              mod_so.c
```

看到prefork 字眼，因此可见当前httpd应该是工作在prefork模式，prefork模式的默认配置是：

```
<IfModule mpm_prefork_module>
                StartServers                      5
                MinSpareServers                   5
                MaxSpareServers                  10
                MaxClients                      150
                MaxRequestsPerChild               0
```

</IfModule>

2.要加到多少？

连接数理论上当然是支持越大越好，但要在服务器的能力范围内，这跟服务器的CPU、内存、带宽等都有关系。

查看当前的连接数可以用：

```
ps aux | grep httpd | wc -l
```

或：

```
pgrep httpd|wc -l
```

计算httpd占用内存的平均数:

```
ps aux|grep -v grep|awk '/httpd/{sum+=$6;n++};END{print sum/n}'
```

由于基本都是静态页面，CPU消耗很低，每进程占用内存也不算多，大约200K。

服务器内存有2G，除去常规启动的服务大约需要500M（保守估计），还剩1.5G可用，那么理论上可以支持1.5*1024*1024*1024/200000 = 8053.06368

约8K个进程，支持2W人同时访问应该是没有问题的（能保证其中8K的人访问很快，其他的可能需要等待1、2秒才能连上，而一旦连上就会很流畅）

控制最大连接数的MaxClients ，因此可以尝试配置为：

```
<IfModule mpm_prefork_module>
                StartServers                      5
                MinSpareServers                   5
                MaxSpareServers                  10
                ServerLimit                    5500
                MaxClients                     5000
                MaxRequestsPerChild               100
```

</IfModule>

注意，MaxClients默认最大为250，若要超过这个值就要显式设置ServerLimit，且ServerLimit要放在MaxClients之前，值要不小于MaxClients，不然重启httpd时会有提示。

重启httpd后，通过反复执行pgrep httpd|wc -l 来观察连接数，可以看到连接数在达到MaxClients的设值后不再增加，但此时访问网站也很流畅，那就不用贪心再设置更高的值了，不然以后如果网站访问突增不小心就会耗光服务器内存，可根据以后访问压力趋势及内存的占用变化再逐渐调整，直到找到一个最优的设置值。

(MaxRequestsPerChild不能设置为0，可能会因内存泄露导致服务器崩溃）

更佳最大值计算的公式：

```
apache_max_process_with_good_perfermance < (total_hardware_memory / apache_memory_per_process ) * 2
apache_max_process = apache_max_process_with_good_perfermance * 1.5
```

附：

实时检测HTTPD连接数：

```
watch -n 1 -d "pgrep httpd|wc -l"
```





查看tomcat进程启动了多少个线程

获取tomcat进程pid 

```
ps -ef|grep tomcat
```

统计该tomcat进程内的线程个数 

```
ps -Lf 29295|wc -l
```



# find命令

　　　　基本格式：find  path expression

　　　　1.按照文件名查找

　　　　(1)find / -name httpd.conf　　#在根目录下查找文件httpd.conf，表示在整个硬盘查找
　　　　(2)find /etc -name httpd.conf　　#在/etc目录下文件httpd.conf
　　　　(3)**find /etc -name '*srm*'　　#使用通配符*(0或者任意多个)。表示在/etc目录下查找文件名中含有字符串‘srm’的文件**
　　　　(4)find . -name 'srm*' 　　#表示当前目录下查找文件名开头是字符串‘srm’的文件

　　　　2.按照文件特征查找 　　　　

　　　　(1)find / -amin -10 　　# 查找在系统中最后10分钟访问的文件(access time)
　　　　(2)find / -atime -2　　 # 查找在系统中最后48小时访问的文件
　　　　(3)find / -empty 　　# 查找在系统中为空的文件或者文件夹
　　　　(4)find / -group cat 　　# 查找在系统中属于 group为cat的文件
　　　　(5)find / -mmin -5 　　# 查找在系统中最后5分钟里修改过的文件(modify time)
　　　　(6)find / -mtime -1 　　#查找在系统中最后24小时里修改过的文件
　　　　(7)find / -user fred 　　#查找在系统中属于fred这个用户的文件
　　　　(8)find / -size +10000c　　#查找出大于10000000字节的文件(c:字节，w:双字，k:KB，M:MB，G:GB)
　　　　(9)find / -size -1000k 　　#查找出小于1000KB的文件

　　　　3.使用混合查找方式查找文件

　　　　参数有： ！，-and(-a)，-or(-o)。

　　　　(1)find /tmp -size +10000c -and -mtime +2 　　#在/tmp目录下查找大于10000字节并在最后2分钟内修改的文件
   　　    (2)find / -user fred -or -user george 　　#在/目录下查找用户是fred或者george的文件文件
   　　    (3)find /tmp ! -user panda　　#在/tmp目录中查找所有不属于panda用户的文件
    　　  

　　二、grep命令

　　　  基本格式：find  expression

 　　　 1.主要参数

　　　　[options]主要参数：
　　　　－c：只输出匹配行的计数。
　　　　－i：不区分大小写
　　　　－h：查询多文件时不显示文件名。
　　　　－l：查询多文件时只输出包含匹配字符的文件名。
　　　　－n：显示匹配行及行号。
　　　　－s：不显示不存在或无匹配文本的错误信息。
　　　　－v：显示不包含匹配文本的所有行。

　　　　pattern正则表达式主要参数：
　　　　\： 忽略正则表达式中特殊字符的原有含义。
　　　　^：匹配正则表达式的开始行。
　　　　$: 匹配正则表达式的结束行。
　　　　\<：从匹配正则表达 式的行开始。
　　　　\>：到匹配正则表达式的行结束。
　　　　[ ]：单个字符，如[A]即A符合要求 。
　　　　[ - ]：范围，如[A-Z]，即A、B、C一直到Z都符合要求 。
　　　　.：所有的单个字符。
　　　　* ：有字符，长度可以为0。

　　　　2.实例　 

        grep -r "字符串"  很方便

　　(1)grep 'test' d*　　#显示所有以d开头的文件中包含 test的行
　　(2)grep ‘test’ aa bb cc 　　 #显示在aa，bb，cc文件中包含test的行
　　(3)grep ‘[a-z]\{5\}’ aa 　　#显示所有包含每行字符串至少有5个连续小写字符的字符串的行
　　(4)grep magic /usr/src　　#显示/usr/src目录下的文件(不含子目录)包含magic的行
　　(5)grep -r magic /usr/src　　#显示/usr/src目录下的文件(包含子目录)包含magic的行

　　(6)grep -w pattern files ：只匹配整个单词，而不是字符串的一部分(如匹配’magic’，而不是’magical’)，

# 文件输入相关

## CAT

### cat创建文件

```
cat >文件名
# ctrl+c退出即可创建
cat >文件名 <<b 
# >是创建并重写文件内容（如果文件已存在，内容会被全部覆盖）
```

### cat追加文件

```
cat >>文件名
# ctrl+c退出文件
cat >>文件名 <<结束标记
# >>是追加文件内容（如果文件已存在，仍会保存）
```

### cat文件清空

```
cat >文件名 
# ctrl+c退出文件，会将空文件覆盖原文件
cat >文件名 <<结束标记
# 输入结束标记即可以空文件覆盖原文件
```

```
cat文件操作指令集合：
cat -n file 输出file文件中的全部内容,并且对file文件进行从 1 开始的编号
cat -b file 输出file文件中的全部内容,并且对fiel文件中除 空行外 进行从 1 开始编号
cat file1 > file2 将文件 file1 写入到 file2 中,如果 file2 存在,则覆盖file2文件,如果file2 不存在,则创建 file2
cat -n file1 > file2 对文件 file1 进行编号,并写入到 file2 中
cat > file1 直接向file1 中写入内容, 内容来自键盘在控制台输入, 按下键盘Ctrl+D结束输入,之前输入的文件保存在file1中
cat file1 file2 > file3 将文件 file1 和 file2 整合到 file3 中
cat file1 file2 >> file3 将文件 file1 和 file2 追加到 file3中 文件不存在时,创建文件
cat >> file1 直接向file1 文件追加内容,内容来自键盘输入 按下Ctrl + D 结束输入,之前输入的内容醉驾在 file1 后
加入 file1 是一个空文件
cat file1 > file2 可以用来清空文件file2
```



# 用户相关

## 添加用户，组

https://blog.csdn.net/qq_16619037/article/details/50698119

### 添加权限

```
chown -R hadoop：hadoop /usr/hadoop/

让普通用户拥有root的权限

1.root登录
2.adduser 用户名
3.passwd 用户名
  确定密码
4.修改/etc/passwd即可，把用户名的ID和ID组修改成0。
```

### 查看文件用户

1、用户列表文件：/etc/passwd/

2、用户组列表文件：/etc/group

3、查看系统中有哪些用户：

```
cut -d : -f 1 /etc/passwd
```

4、查看可以登录系统的用户：

cat /etc/passwd | grep -v /sbin/nologin | cut -d : -f 1

![img](https://images2018.cnblogs.com/blog/1460541/201808/1460541-20180821121734744-1469033012.png)

5、查看用户操作：w命令(需要root权限)

6、查看某一用户：w 用户名

7、查看登录用户：who

8、查看用户登录历史记录：last

9、修改root用户密码：

```
whoami
passwd
#先输入一次原密码
#再输入两次新密码
```

10、root用户修改其他用户密码：

```
passwd <user_name>
```

![img](https://images2018.cnblogs.com/blog/1460541/201808/1460541-20180821121648830-2084401524.png) 



### 管理用户

 在centos中增加用户使用adduser命令而创建用户组使用groupadd命令，这个是不是非常的方便呀，其实复杂点的就是用户的组与组权限的命令了，下面来给各位介绍一下吧。

1、建用户：

adduser phpq                         //新建phpq用户
passwd phpq                          //给phpq用户设置密码

2、建工作组
groupadd test                        //新建test工作组

3、新建用户同时增加工作组
useradd -g test phpq               //新建phpq用户并增加到test工作组

注：：-g 所属组 -d 家目录 -s 所用的SHELL

4、给已有的用户增加工作组

usermod -G groupname username

或者：gpasswd -a username groupname 

 

（注意：添加用户到某一个组 可以使用usermod -G groupname username这个命令可以添加一个用户到指定的组，但是以前添加的组就会清空掉。

所以想要添加一个用户到一个组，同时保留以前添加的组时，请使用gpasswd这个命令来添加操作用户）

5、临时关闭

在/etc/shadow文件中属于该用户的行的第二个字段（密码）前面加上*就可以了。想恢复该用户，去掉*即可。

或者使用如下命令关闭用户账号：

passwd peter –l

重新释放：

passwd peter –u

6、永久性删除用户账号

userdel peter

groupdel peter

usermod –G peter peter   （强制删除该用户的主目录和主目录下的所有文件和子目录）

7、从组中删除用户

编辑/etc/group 找到GROUP1那一行，删除 A 或者用命令 gpasswd -d A GROUP

8、显示用户信息

id user
cat /etc/passwd

补充:查看用户和用户组的方法

用户列表文件：/etc/passwd
用户组列表文件：/etc/group

查看系统中有哪些用户：cut -d : -f 1 /etc/passwd
查看可以登录系统的用户：cat /etc/passwd | grep -v /sbin/nologin | cut -d : -f 1
查看某一用户：w 用户名
查看登录用户：who
查看用户登录历史记录：last

## 解压相关

### unzip

1、把文件解压到当前目录下

```
unzip test.zip
```

2、如果要把文件解压到指定的目录下，需要用到-d[参数](https://so.csdn.net/so/search?q=参数&spm=1001.2101.3001.7020)。

```
unzip -d /temp test.zip
```

3、解压的时候，有时候不想覆盖已经存在的文件，那么可以加上-n参数

```
unzip -n test.zip
unzip -n -d /temp test.zip
```

4、只看一下zip压缩包中包含哪些文件，不进行解压缩

```
unzip -l test.zip
```

5、查看显示的文件列表还包含压缩比率

```
unzip -v test.zip
```

6、检查zip文件是否损坏

```
unzip -t test.zip
```

7、将压缩文件test.zip在指定目录tmp下解压缩，如果已有相同的文件存在，要求unzip命令覆盖原先的文件

```
unzip -o test.zip -d /tmp/
```

### tar

以下是对`tar`命令的一些总结：

```
1 # tar -cvf test.tar test 仅打包，不压缩 
2 # tar -zcvf test.tar.gz test 打包后，以gzip压缩 在参数f后面的压缩文件名是自己取的，习惯上用tar来做，如果加z参数，
3 则以tar.gz 或tgz来代表gzip压缩过的tar file文件
```

解压操作:

```
1 #tar -zxvf /usr/local/test.tar.gz
```

tar 解压缩命令详解

```
1 -c: 建立压缩档案
2 -x：解压
3 -t：查看内容
4 -r：向压缩归档文件末尾追加文件
5 -u：更新原压缩包中的文件
```

这五个是独立的命令，压缩解压都要用到其中一个，可以和别的命令连用但只能用其中一个。下面的参数是根据需要在压缩或解压档案时可选的。



```
1 -z：有gzip属性的
2 -j：有bz2属性的
3 -J：具有xz属性的（注3）
4 -Z：有compress属性的
5 -v：显示所有过程
6 -O：将文件解开到标准输出
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif](javascript:void(0);)

下面的参数-f是必须的 
-f: 使用档案名字，切记，这个参数是最后一个参数，后面只能接档案名。

```
1 # tar -cf all.tar *.jpg 
2 
3 # tar -rf all.tar *.gif 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif](javascript:void(0);)

```
1 这条命令是将所有.gif的文件增加到all.tar的包里面去。-r是表示增加文件的意思。
2 
3 # tar -uf all.tar logo.gif 
4 
5 
6 
7 这条命令是更新原来tar包all.tar中logo.gif文件，-u是表示更新文件的意思。
8 
9 # tar -tf all.tar 
```

javascript:void(0);)

```
 1 这条命令是列出all.tar包中所有文件，-t是列出文件的意思
 2 
 3 # tar -xf all.tar 
 4 这条命令是解出all.tar包中所有文件，-x是解开的意思
 5 
 6 压缩
 7 
 8 tar –cvf jpg.tar *.jpg //将目录里所有jpg文件打包成tar.jpg
 9 tar –czf jpg.tar.gz *.jpg //将目录里所有jpg文件打包成jpg.tar后，并且将其用gzip压缩，生成一个gzip压缩过的包，命名为jpg.tar.gz
10 tar –cjf jpg.tar.bz2 *.jpg //将目录里所有jpg文件打包成jpg.tar后，并且将其用bzip2压缩，生成一个bzip2压缩过的包，命名为jpg.tar.bz2
11 tar –cZf jpg.tar.Z *.jpg //将目录里所有jpg文件打包成jpg.tar后，并且将其用compress压缩，生成一个umcompress压缩过的包，命名为jpg.tar.Z
12 rar a jpg.rar *.jpg //rar格式的压缩，需要先下载rar for linux
13 zip jpg.zip *.jpg //zip格式的压缩，需要先下载zip for linux
14 
15 
16 解压
17 
18 tar –xvf file.tar //解压 tar包
19 tar -xzvf file.tar.gz //解压tar.gz
20 tar -xjvf file.tar.bz2   //解压 tar.bz2
21 tar –xZvf file.tar.Z   //解压tar.Z
22 unrar e file.rar //解压rar
23 unzip file.zip //解压zip
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif](javascript:void(0);)

```
总结
(1) *.tar 用 tar –xvf 解压
(2) *.gz 用 gzip -d或者gunzip 解压
(3) *.tar.gz和*.tgz 用 tar –xzf 解压
(4) *.bz2 用 bzip2 -d或者用bunzip2 解压
(5) *.tar.bz2用tar –xjf 解压
(6) *.Z 用 uncompress 解压
(7) *.tar.Z 用tar –xZf 解压
(8) *.rar 用 unrar e解压
(9) *.zip 用 unzip 解压
(10) *.xz 用 xz -d 解压
(11) *.tar.xz 用 tar -zJf 解压
```



## 网络相关

### 访问某地址

```
curl ip:port/url
telnet ip:port
```



### win刷新DNS缓存

```
ipconfig /flushdns
```



### netstat

查看端口的连接，+pid

```
netstat -anp | grep xxx
```

查看应用的建立连接的ip

```
netstat -an | grep xxx
```

查看建立的有服务的连接

```
netstat -ntlp
```



# [linux查看并发连接数](https://www.cnblogs.com/it-davidchen/p/11015495.html)

#### 1、查看TCP的并发请求数及其TCP连接状态：

```
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
netstat -n|grep  ^tcp|awk '{print $NF}'|sort -nr|uniq -c
```

或者

```
netstat -n | awk '/^tcp/ {++state[$NF]} END {for(key in state) print key,"t",state[key]}'
```

返回结果一般如下：



```
LAST_ACK 5 （正在等待处理的请求数）
SYN_RECV 30
ESTABLISHED 1597 （正常数据传输状态）
FIN_WAIT1 51
FIN_WAIT2 504
TIME_WAIT 1057 （处理完毕，等待超时结束的请求数）
```

参数描述：

```
CLOSED：无连接是活动的或正在进行
LISTEN：服务器在等待进入呼叫
SYN_RECV：一个连接请求已经到达，等待确认
SYN_SENT：应用已经开始，打开一个连接
ESTABLISHED：正常数据传输状态
FIN_WAIT1：应用说它已经完成
FIN_WAIT2：另一边已同意释放
ITMED_WAIT：等待所有分组死掉
CLOSING：两边同时尝试关闭
TIME_WAIT：另一边已初始化一个释放
LAST_ACK：等待所有分组死掉
```

#### 2、查看Nginx运行进程数

```
ps -ef | grep nginx | wc -l
```

#### 3、查看APACHE运行进程数

```
ps -ef | grep httpd | wc -l
```

#### 4、查看Web服务器进程连接数：

```
netstat -antp | grep 80 | grep ESTABLISHED -c
```

#### 5、查看MySQL进程连接数：

```
ps -axef | grep mysqld -c
```

## 文件传输

### scp

```
scp deployment.yaml ${DEPLOY_SERVER}:${DEPLOYMENT_YAML_PATH}
```

## 文件权限

### 文件属性

```
Linux lsattr命令用于显示文件属性。

用chattr执行改变文件或目录的属性，可执行lsattr指令查询其属性。
```

- lsattr

```
lsattr [-adlRvV][文件或目录...]
```

- chattr

```
chattr [-RV][-v<版本编号>][+/-/=<属性>][文件或目录...]

用chattr命令防止系统中某个关键文件被修改：

chattr +i /etc/resolv.conf
lsattr /etc/resolv.conf
会显示如下属性

----i-------- /etc/resolv.conf
让某个文件只能往里面追加数据，但不能删除，适用于各种日志文件：

chattr +a /var/log/messages
```



# Linux常见配置文件

## 常见永久配置文件

```
/etc/profile
/etc/bashrc
~/.bashrc
```

