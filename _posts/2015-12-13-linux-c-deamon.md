---
layout: post
title: Linux下开发－守护进程（daemon）
categories:  C/C++
description: 
keywords: 
---


```c
int daemon(void){
    pid_t pid = fork();
    if( pid != 0 ) exit(0);//parent

    //first children
    if(setsid() == -1){
        printf("setsid failed\n");
        assert(0);
        exit(-1);
    }

    umask(0);

    pid = fork();
    if( pid != 0) exit(0);

    //second children 
    chdir ("/");

    for (int i = 0; i < 3; i++){
        close (i);
    }

    int stdfd = open ("/dev/null", O_RDWR);
    dup2(stdfd, STDOUT_FILENO);
    dup2(stdfd, STDERR_FILENO);

    return 0;
}
```

**第一次fork的作用是让shell 认为本条命令已经终止，不用挂在终端输入上**。还有一个作用是**为后面setsid服务。setsid的调用者不能是进程组组长**(group leader).** 此时父进程是进程组组长**。

**setsid**() 是本函数最重要的一个调用。它完成了daemon函数想要做的大部分事情。调用完整个函数。**子进程是会话组长**(sid==pid)，**也是进程组组长**(pgid == pid)，并且**脱离了原来控制终端**。到了这一步，**基本上不管控制终端如何怎么样。新的进程都不会收到那些信号**。

经过前面2个步骤，基本想要做的都做了。第2次fork不是必须的。也看到很多开源服务没有**fork第二次**。fork第二次主要目的是。**防止进程再次打开一个控制终端。因为打开一个控制终端的前台条件是该进程必须是会话组长。再fork一次，子进程ID != sid**（父进程的sid）。所以也无法打开新的控制终端。



# 一次fort、孤儿进程

![](/images/posts/2015-12-13-linux-c-deamon.md/1.png)





# 僵尸进程

![](/images/posts/2015-12-13-linux-c-deamon.md/2.png)




# 两次fork

![](/images/posts/2015-12-13-linux-c-deamon.md/3.png)




# 守护进程与用&结尾的后台运行程序的区别

1)守护进程已经完全脱离终端控制台了，而后台程序并未完全脱离终端，在终端未关闭前还是会往终端输出结果

2)守护进程在关闭终端控制台时不会受影响，而后台程序会随用户退出而停止，需要在以nohug xxx & 格式运行才能避免影响

3)守护进程的会话组和当前目录，文件描述符都是独立的。后台运行只是终端进行了一次fork，让程序在后台执行，这些都没改变。




# 终端信号和进程

```c
//后台进程读取/写入终端输入产生下面两个信号，或者控制终端不存在情况读取和写入会产生
signal(SIGTTOU, SIG_IGN);
signal(SIGTTIN, SIG_IGN);

//按CTRL-C ,CTRL-\ CTRL-Z会向前台进程组发送下面这些信号
signal(SIGINT,  SIG_IGN );
signal(SIGQUIT, SIG_IGN );
signal(SIGTSTP, SIG_IGN );

//终端断开，会给会话组长或孤儿进程组所有成员发送下面信号
signal(SIGHUP,  SIG_IGN );

//还有有些信号也可以由终端shell产生，需要关注
signal(SIGCONT, SIG_IGN );
signal(SIGSTOP, SIG_IGN );
```







