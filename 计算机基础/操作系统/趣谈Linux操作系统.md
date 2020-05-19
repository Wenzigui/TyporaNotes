操作系统其实就像一个软件外包公司，其内核就相当于这家外包公司的老板。所以接下来的整个课程中，请你将自己的角色切换成这家软件外包公司的老板，设身处地地去理解操作系统是如何协调各种资源，帮客户做成事情的。



# 黑话 - 常用命令



## 用户与密码

-   passwd 修改账户密码

-   useradd 添加用户

```shell
useradd #{username}
# 添加用户后，要执行passwd命令设置密码，然后才能登陆
passwd
# /etc/passwd文件存放用户信息
# /etc/group文件存放用户组信息
```



## 浏览文件

-   cd 切换目录（change directory）
-   ls 列出当前目录下的文件
-   chown 改变文件所属用户
-   chgrp 改变文件所属用户组



![](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200518210124525.png)



第一个字段第一个字符表示文件类型：

-   “-”表示普通文件
-   “d”表示目录

第一个字段剩余九个字符表示权限位：

-   文件所属用户的可读(r)可写(w)可执行(x)权限
-   文件所属组的可读(r)可写(w)可执行(x)权限
-   其他用户的可读(r)可写(w)可执行(x)权限

第二个字段表示硬链接数目

第三个字段表示所属用户

第四个字段表示所属用户组

第五个字段表示文件大小

第六个字段表示文件被修改的日期



## 安装软件



-   rpm -i 安装软件
-   rpm -qa/-l 查看安装的软件列表
-   rpm -e/-r 删除软件



>   Linux 也有自己的软件管家，CentOS 下面是 yum，Ubuntu 下面是 apt-get



## 运行程序



1.  常用方式，通过shell在交互命令行里面运行`./#{filename}`
2.  后台运行，使用`nohup`，意思是不挂起，当交互命令行退出的时候，程序还要在
3.  以服务的方式运行，`systemd`，[Systemd 入门教程](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)



## 脑图总结

![](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200518213257672.png)



# 系统调用



## 创建进程



在Linux里，创建一个新进程时，需要从老进程中调用fork来实现。其中，老进程叫做“父进程”，新进程叫做“子进程”。



![](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200518215514514.png)



操作系统会在启动的时候，创建一个所有用户进程的“祖先进程”。

>   可以通过`waitpid`系统调用，将子进程的进程号作为参数，查询子进程是否运行完



## 进程内存空间管理



![](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200518220342833.png)



一个进程的内存空间不是预先分配的，是在使用到的时候才向内核申请的。



堆里面进行内存分配的系统调用：

-   brk 分配小内存
-   mmap 分配大内存



## 文件管理



-   open 打开文件
-   close 打开文件
-   creat 创建文件
-   lseek 跳到文件的某个位置
-   read 读文件
-   write 写文件



在Linux中，一切皆文件！



![](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200518221345942.png)



每个文件都有一个文件描述符（整数），通过文件描述符可以使用系统调用。



## 异常与信号处理



当项目遇到异常时，需要发送一个信号给项目组。



常见信号有以下几种：

-   在键盘输入“CTRL+C”，这就是中断的信号，正在执行的命令就会中止退出；
-   非法访问内存
-   硬件故障
-   使用kill函数



通过`sigaction`系统调用，注册信号处理函数。



## 进程间通信



-   消息队列
    -   通过系统调用`msgget`创建新队列
    -   通过`msgsnd`将消息发送到消息队列
    -   通过`msgrcv`从队列中获取消息
-   共享内存
    -   通过`shmget`创建共享内存块
    -   通过`shmat`将共享内存映射到自己的内存空间，然后进行读写
-   信号量机制，解决共享内存竞争问题
    -   `sem_wait`占用信号量（加锁）
    -   `sem_post`释放信号量（释放锁）
-   socket通信



## Glibc



Glibc 为程序员提供丰富的 API，除了例如字符串处理、数学运算等用户态服务之外，最重要的是封装了操作系统提供的系统服务，即系统调用的封装。

>   相当于一个中介，有事情找它就行，它会转化成系统调用



## 脑图总结



![](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200518223220401.png)



# x86架构



## 计算机的工作模式



![](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519075211401.png)



-   最核心的是CPU
-   CPU和其他设备连接需要总线，总线实际上就是主板上密密麻麻的集成电路
-   其他设备中最重要的是内存，用于保存计算中的中间结果



CPU包含三部分：

-   运算单元：只管计算
-   数据单元：包括CPU内部缓存和寄存器组，速度贼快，比内存还要快
-   控制单元：指挥运算单元和数据单元



![](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519080015961.png)



## 8086处理器原理



![image-20200519154212749](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519154212749.png)



数据单元：

-   处理器内置8个16为通用的寄存器，用于存储计算过程中的暂存数据



控制单元：

-   IP寄存器（指令指针寄存器）指向代码段中下一条指令的位置
    CPU会根据它不停地从内存中的代码段中加载指令到CPU的指令队列，然后交给运算单元执行
-   代码段寄存器（CS）用于找到代码在内存中的位置
-   数据段寄存器（DS）用于找到数据在内存中的位置
-   栈寄存器（SS）存储函数调用相关的操作



## 总结



![image-20200519155204767](https://raw.githubusercontent.com/RoddeHope/Figurebed/master/img/image-20200519155204767.png)