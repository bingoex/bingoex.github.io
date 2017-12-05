---
layout: post
title: Linux下开发－exec
categories: Linux C/C++
description: 
keywords: 
---


fork函数是用于创建一个子进程，该子进程几乎是父进程的副本，而有时我们希望子进程去执行另外的程序，exec函数族就提供了一个在进程中启动另一个程序执行的方法。它可以根据指定的文件名或目录名找到可执行文件，并用它来取代原调用进程的数据段、代码段和堆栈段，在执行完之后，原调用进程的内容除了进程号外，其他全部被新程序的内容替换了。另外，这里的可执行文件既可以是二进制文件，也可以是Linux下任何可执行脚本文件

实际上，在Linux中并没有exec函数，而是有6个以exec开头的函数族。
```c
#include <unistd.h>
int execl(const char *path, const char *arg, ...)
int execv(const char *path, char *const argv[])
int execle(const char *path, const char *arg, ..., char *const envp[])
int execve(const char *path, char *const argv[], char *const envp[])
int execlp(const char *file, const char *arg, ...)
int execvp(const char *file, char *const argv[])
```
函数返回值
- 成功：函数不会返回
- 出错：返回-1，失败原因记录在error中

①    查找方式：上表其中前4个函数的查找方式都是完整的文件目录路径，而最后2个函数（也就是以p结尾的两个函数）可以只给出文件名，系统就会自动从环境变量“$PATH”所指出的路径中进行查找。

②    参数传递方式：exec函数族的参数传递有两种方式，一种是逐个列举的方式，而另一种则是将所有参数整体构造成指针数组进行传递。

③    环境变量：exec函数族使用了系统默认的环境变量，也可以传入指定的环境变量。这里以“e”（environment）结尾的两个函数execle、execve就可以在envp[]中指定当前进程所使用的环境变量替换掉该进程继承的所以环境变量。



在Linux中，Shell进程是所有执行码的父进程。当一个执行码执行时，Shell进程会fork子进程然后调用exec函数去执行执行码。Shell进程堆栈中存放着该用户下的所有环境变量，使用execl、execv、execlp、execvp函数使执行码重生时，Shell进程会将所有环境变量复制给生成的新进程；而使用execle、execve时新进程不继承任何Shell进程的环境变量，而由envp[]数组自行设置环境变量。


事实上，这6个函数中真正的系统调用只有execve，其他5个都是库函数，它们最终都会调用execve这个系统调用

![](/images/posts/2015-12-14-linux-c-exec.md/1.png)

```c
char *const ps_argv[] ={"ps", "-o", "pid,ppid,pgrp,session,tpgid,comm", NULL};
char *const ps_envp[] ={"PATH=/bin:/usr/bin", "TERM=console", NULL};
execl("/bin/ps", "ps", "-o", "pid,ppid,pgrp,session,tpgid,comm", NULL);
execv("/bin/ps", ps_argv);
execle("/bin/ps", "ps", "-o", "pid,ppid,pgrp,session,tpgid,comm", NULL, ps_envp);
execve("/bin/ps", ps_argv, ps_envp);
execlp("ps", "ps", "-o", "pid,ppid,pgrp,session,tpgid,comm", NULL);
execvp("ps", ps_argv);
```

在使用exec函数族时，一定要加上错误判断语句。因为exec很容易执行失败，其中最常见的原因有：
- 找不到文件或路径，此时errno被设置为ENOENT。
- 数组argv和envp忘记用NULL结束，此时errno被设置为EFAULT。
- 没有对应可执行文件的运行权限，此时errno被设置为EACCES。




