---
layout: post
title: Linux下开发－wait和waitpid
categories: Linux C/C++
description: 
keywords: 
---


# wait
```c
#include<sys/types.h> /* 提供类型pid_t的定义 */
#include<sys/wait.h>
pid_t wait(int *status);
```

进程一旦调用了wait就立即阻塞自己，由wait自动分析是否当前进程的某个子进程已经退出，如果让它找到了这样一个已经变成僵尸的子进程，wait就会收集这个子进程的信息，并把它彻底销毁后返回。如果没有找到这样一个子进程，wait就会一直阻塞在这里，直到有一个出现为止。

参数status用来保存被收集进程退出时的一些状态，它是一个指向int类型的指针。但如果我们对这个子进程是如何死掉毫不在意，只想把这个僵尸进程消灭掉，我们就可以设定这个参数为NULL（pid = wait(NULL);

如果成功，wait会返回被收集的子进程的进程ID，如果调用进程没有子进程，调用失败，此时wait返回-1，同时errno被置为ECHILD。



# waitpid

```c
#include<sys/types.h> /* 提供类型pid_t的定义 */
#include<sys/wait.h>
pid_t waitpid(pid_t pid,int *status,int options)
```

从本质上讲，系统调用waitpid和wait的作用是完全相同的
```c
static inline pid_t wait(int * wait_stat){
    return waitpid(-1, wait_stat, 0);
}
```

pid：从参数的名字pid和类型pid_t中就可以看出，这里需要的是一个进程ID。

pid>0时，只等待进程ID等于pid的子进程，不管其它已经有多少子进程运行结束退出了，只要指定的子进程还没有结束，waitpid就会一直等下去。

pid=-1时，等待任何一个子进程退出，没有任何限制，此时waitpid和wait的作用一模一样。

pid=0时，等待同一个进程组中的任何子进程，如果子进程已经加入了别的进程组，waitpid不会对它做任何理睬。

pid<-1时，等待一个指定进程组中的任何子进程，这个进程组的ID等于pid的绝对值。

options：options提供了一些额外的选项来控制waitpid，目前在Linux中只支持WNOHANG和WUNTRACED两个

如果使用了WNOHANG参数调用waitpid，即使没有子进程退出，它也会立即返回，不会像wait那样永远等下去。而WUNTRACED参数，涉及到一些跟踪调试方面的知识



