## Linux 下如何查看端口被哪个进程占用？

Linux 查看端口占用情况可以使用 lsof 和 netstat 命令。

### lsof
lsof(list open files)是一个列出当前系统打开文件的工具。

lsof 查看端口占用语法格式：


```
lsof -i:端口号
```

实例
查看服务器 8000 端口的占用情况：


```
lsof -i:8000
COMMAND   PID USER   FD   TYPE   DEVICE SIZE/OFF NODE NAME
nodejs  26993 root   10u  IPv4 37999514      0t0  TCP *:8000 (LISTEN)
```

可以看到 8000 端口已经被轻 nodejs 服务占用。

lsof -i 需要 root 用户的权限来执行。

更多 lsof 的命令如下：


```
lsof -i:8080：查看8080端口占用
lsof abc.txt：显示开启文件abc.txt的进程
lsof -c abc：显示abc进程现在打开的文件
lsof -c -p 1234：列出进程号为1234的进程所打开的文件
lsof -g gid：显示归属gid的进程情况
lsof +d /usr/local/：显示目录下被进程开启的文件
lsof +D /usr/local/：同上，但是会搜索目录下的目录，时间较长
lsof -d 4：显示使用fd为4的进程
lsof -i -U：显示所有打开的端口和UNIX domain文件
```

### netstat
netstat -tunlp 用于显示 tcp，udp 的端口和进程等相关情况。

netstat 查看端口占用语法格式：


```
netstat -tunlp | grep 端口号
-t (tcp) 仅显示tcp相关选项
-u (udp)仅显示udp相关选项
-n 拒绝显示别名，能显示数字的全部转化为数字
-l 仅列出在Listen(监听)的服务状态
-p 显示建立相关链接的程序名
```

例如查看 8000 端口的情况，使用以下命令：

```
netstat -tunlp | grep 8000
tcp        0      0 0.0.0.0:8000            0.0.0.0:*               LISTEN      26993/nodejs
```
   
更多命令：


```
netstat -ntlp   //查看当前所有tcp端口
netstat -ntulp | grep 80   //查看所有80端口使用情况
netstat -ntulp | grep 3306   //查看所有3306端口使用情况
```
