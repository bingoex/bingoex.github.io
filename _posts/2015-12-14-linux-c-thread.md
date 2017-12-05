---
layout: post
title: Linux下开发－线程详解
categories: Linux C/C++
description: 
keywords: 
---



其实在Linux中，新建的线程并不是在原先的进程中，而是系统通过一个系统调用clone()。该copy了一个和原先进程完全一样的进程。不过这个copy过程和fork不一样。copy后的进程和原先的进程共享了所有的变量，运行环境（clone的实现是可以指定新进程与老进程之间的共享关系，100%共享就表示创建了一个线程）。这样，原先进程中的变量变动在copy后的进程中便能体现出来。



# pthread_detach

如果线程是joinable状态，当线程函数自己返回退出时或pthread_exit时都不会释放线程所占用堆栈和线程描述符（总计8K多）。只有当你调用了pthread_join之后这些资源才会被释放。 

若是unjoinable状态的线程，这些资源在线程函数退出时或pthread_exit时自动会被释放。

unjoinable属性可以在pthread_create时指定，或在线程创建后在线程中pthread_detach自己,如：pthread_detach(pthread_self())，将状态改为unjoinable状态，确保资源的释放。或者将线程置为joinable,然后适时调用pthread_join。



# Thread Specific Data

表面上看起来这是一个全局变量，所有线程都可以使用它，而它的值在每一个线程中又是单独存储的。
```c
#include <malloc.h>  
#include <pthread.h>    
#include <stdio.h>  

/* The key used to associate a log file pointer with each thread. */    
static pthread_key_t thread_log_key;  

/* Write MESSAGE to the log file for the current thread. */    
void write_to_thread_log (const char* message)    {  
    FILE* thread_log = (FILE*) pthread_getspecific (thread_log_key);   
    fprintf (thread_log, “%s\n”, message);  
}  

/* Close the log file pointer THREAD_LOG. */    
void close_thread_log (void* thread_log)  {  
    fclose ((FILE*) thread_log);      
}  

void* thread_function (void* args)    {  
    char thread_log_filename[20];    
    FILE* thread_log;  

    /* Generate the filename for this thread’s log file. */    
    sprintf (thread_log_filename, “thread%d.log”, (int) pthread_self ());  

    /* Open the log file. */    
    thread_log = fopen (thread_log_filename, “w”);  

    /* Store the file pointer in thread-specific data under thread_log_key. */    
    pthread_setspecific (thread_log_key, thread_log);  

    write_to_thread_log (“Thread starting.”);  

    /* Do work here... */    
    return NULL;  
}  

int main ()    {    
    int i;    
    pthread_t threads[5];  

    /* Create a key to associate thread log file pointers in  
       thread-specific data. Use close_thread_log to clean up the file  
       pointers. */    
    pthread_key_create (&thread_log_key, close_thread_log);  

    /* Create threads to do the work. */    
    for (i = 0; i < 5; ++i)  
        pthread_create (&(threads[i]), NULL, thread_function, NULL);  

    /* Wait for all threads to finish. */    
    for (i = 0; i < 5; ++i)    
        pthread_join (threads[i], NULL);  
    return 0;    
}    
```



# 线程和exec()

当在线程中调用exec()时，该线程被完全的被替代，线程ID不确定。除了被替代的线程，其他线程都被销毁。线程的thread-specific data销毁函数和清除函数都不会被调用。所有进程的互斥量和条件变量消失。



# 线程和fork()

当在多线程进程中调用fork()，只有调用fork()的线程被复制到子进程。

虽然只有调用fork()的线程被复制到子进程，但是子进程的全局变量，互斥量，条件变量的状态都将和在父进程中的一样。也就是说，如果它在父进程中被锁住，则它在子进程中也是被锁住的。

thread-specific data的销毁函数和清除函数都不会被调用。在多线程中调用fork()可能会引起内存泄露。比如在其他线程中创建的thread-specific data，在子进程中将没有指针来存取这些数据，造成内存泄露。

因为以上这些问题，在线程中调用fork()的后，我们通常都会在子进程中调用exec()。因为exec()能让父进程中的所有互斥量，条件变量（pthread objects）在子进程中统统消失（用新数据覆盖所有的内存）。

应该避免在一个多线程的程序中使用fork。



# 线程取消选项

可取消状态属性可以是PTHREAD_CANCEL_ENABLE和PTHREAD_CANCEL_DISABLE，线程可以通过调用pthread_setcancelstate修改它的可取消状态。

```c
#include<pthread.h>  
int pthread_setcancelstate(int state, int *oldstate); //成功则返回0，否则返回错误编号。  
```
pthread_setcancelstate把当前可取消状态设置为state，把原来的可取消状态存放在oldstate指向的内存单元中，这两步是原子操作。

pthread_cancel并不等待线程终止，在默认情况下，线程在取消请求发出以后还是继续运行，直到线程到达某个取消点。

取消点是线程检查是否被取消并按照请求进行动作的一个位置。POSIX.1保证在下表中列出的任何函数，取消点都会出现。

![](/images/posts/2015-12-14-linux-c-thread.md/1.png)

下表POSIX.1定义的可选取消点：

![](/images/posts/2015-12-14-linux-c-thread.md/2.png)

线程启动是默认的可取消状态是PTHREAD_CANCEL_ENABLE。当状态设为PTHREAD_CANCEL_DISABLE时，对pthread_cancel的调用并不会杀死线程，相反，取消请求对这个线程来说处于未决状态。当取消状态再次变成PTHREAD_CANCEL_ENABLE时，线程将在下一个取消点上对所有未决的取消请求进行处理。

如果应用程序在很长一段时间内都不会调用到上面的函数，那么可以调用pthread_testcancel函数在程序中自己添加取消点。
```c
#include<pthread.h>  
void pthread_testcancel(void); 
```
调用pthread_testcancel时，如果有个取消请求正处于未决状态，而且取消并没有设置为无效，那么线程就会被取消。但是如果取消被置成无效，pthread_testcancel调用就没有任何效果。

上述述的默认取消类型也称为延迟取消，调用pthread_cancel以后，在线程到达取消点之前，并不会出现真正的取消，可以通过调用pthread_setcanceltype来修改取消类型。
```c
#include<pthread.h>  
int pthread_setcanceltype(int type, int *oldtype); //成功则返回0，否则返回错误编号。  
```
type参数可以是PTHERAD_CANCEL_DEFERRED（延迟取消）或PTHREAD_CANCEL_ASYNCHRONOUS（异步取消），

使用异步取消时，线程可以在任何时间取消，而不是非要等到遇到取消点才能被取消。



# 线程锁

<http://blog.csdn.net/b2222505/article/details/78232678>







