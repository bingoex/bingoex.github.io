---
layout: post
title: Linux锁机制
categories: Linux C/C++
description: 
keywords: 
---



# flock、lockf和fcntl

Linux的文件锁主要有两种：flock和lockf。lockf只是fcntl系统调用的一个封装。

lockf或fcntl实现了更细粒度文件锁，即记录锁，可以**对文件的部分字节上锁**，而flock只能对整个文件加锁。

应该避免在多线程场景下使用flock对文件加锁，而lockf/fcntl则没有这个问题。



# 建议锁和强制锁

**建议锁**：如果某一个进程对一个文件持有一把锁之后，其他进程仍然可以直接对文件进行操作的。只是一种编程上的约定。

**强制锁**：试图实现一套内核级的锁操作。当有进程对某个文件上锁之后，其他进程即使不在操作文件之前检查锁，也会在open、read或write等文件操作时发生错误。

flock和lockf都是建议锁。

fcntl系统调用可以支持强制锁。



# 线程锁

多线程共享地址空间在方便我们编程的同时，也带来了一些麻烦，其中就包括共享资源的争抢问题.

**互斥锁(pthread_mutex_t)**。当一个线程拿到锁的时候，其他线程运行到这里的时候会被**挂起阻塞**，等待锁释放后再唤醒运行。也可用作进程间同步，不过需要利用共享内存。
```c
#include <pthread.h>
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_trylock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);
int pthread_mutex_init(pthread_mutex_t *restrict mutex, const pthread_mutexattr_t *restrict attr);
static pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
```

**自旋锁(pthread_spinlock_t)**。没抢到锁的线程不会挂起，而是不断的检查锁是否被释放，其原理是内部保存了一个原子变量，加锁和解锁就是对原子变量的增减，被阻塞的线程不断检查这个变量的值，决定是否继续忙等。由于用户态程序无法控制线程调度，所以可能出现一个线程刚刚拿到了锁就被切换了出去，造成另一个线程在整个时间片内自旋，白白浪费cpu资源。

**读写锁(pthread_rwlock_t)**。读锁可以被多个线程持有，而同时只能有一个写线程，且不能同时存在读写操作。读多写少的场合用读写锁可以提高并发性，但读写锁的实现本身就要维护一个原子的读者数量，如果锁的粒度比较小，锁本身的消耗就不可忽略了。

读写锁加解锁的代价比普通锁大，但是依然有其使用场景：

少量操作需要写锁，多个线程需要读锁，并且持有读锁的时间比较长（显著高于加解锁的耗时）那么用读写锁能够明显提高并发度。

**条件变量（pthread_cond_t）**。与pthread_mutex_t配合，让线程睡眠，不占用CPU，直到事件发生（pthread_cond_broadcast、pthread_cond_signal）为止。也可用作进程间同步，不过需要利用共享内存。

在使用pthread_cond_wait函数之前，我们要先用一个互斥锁锁住，然后当我们调用pthread_cond_wait函数进入睡眠。该函数原子的执行两个动作：

1，给互斥锁解锁。（这就要求在调用这个函数之前要上锁）

2，把调用线程投入睡眠。

一旦这个函数被唤醒后，那么在此函数返回前重新给互斥锁上锁。这也就决定了在接下来的程序里必须有解锁的步骤。
```c
#include <pthread.h>  
#include <stdio.h>  
#include <stdlib.h>  
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;/*初始化互斥锁*/  
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;/*初始化条件变量*/  
void *thread1(void *);  
void *thread2(void *);  
int i=1;  
int main(void)  {  
    pthread_t t_a;  
    pthread_t t_b;  
    pthread_create(&t_a,NULL,thread1,(void *)NULL);/*创建进程t_a*/  
    pthread_create(&t_b,NULL,thread2,(void *)NULL); /*创建进程t_b*/  
    pthread_join(t_a, NULL);/*等待进程t_a结束*/  
    pthread_join(t_b, NULL);/*等待进程t_b结束*/  
    pthread_mutex_destroy(&mutex);  
    pthread_cond_destroy(&cond);  
    exit(0);  
}  

void *thread1(void *junk)  {  
    for(i=1;i<=6;i++){  
        pthread_mutex_lock(&mutex);/*锁住互斥量*/  
        if(i%3==0){  
            printf("thread1:signal 1  %d/n", __LINE__);  
            pthread_cond_signal(&cond);/*条件改变，发送信号，通知t_b进程*/  
            printf("thread1:signal 2  %d/n", __LINE__);  
            sleep(1);  
        }  
        pthread_mutex_unlock(&mutex);/*解锁互斥量*/  
        sleep(1);  
    }  
}  

void *thread2(void *junk)  {  
    while(i<6){  
        pthread_mutex_lock(&mutex);   
        if(i%3!=0){  
            printf("thread2: wait 1  %d/n", __LINE__);  
            pthread_cond_wait(&cond,&mutex);/*解锁mutex，并等待cond改变*/  
            printf("thread2: wait 2  %d/n", __LINE__);  
        }  
        pthread_mutex_unlock(&mutex);  
        sleep(1);  
    }  
}  
```
如果信号处理函数打断了pthread_cond_wait()，该函数要么自动重新自行（linux是这样实现的），或者返回0（这时应用要检查返回值，判断是否为假唤醒）

**信号量**:

**信号量集**：信号量机制解决了单个资源的互斥访问，信号量集解决同时需要多个资源时的互斥访问。


# 脚本

-n参数可以让flock命令以非阻塞方式探测一个文件是否已经被加锁。

脚本退出的时候锁会被释放，所以这里可以不用显式的使用flock解锁。

`*/1 * * * * /usr/bin/flock -xn/tmp/script.lock -c '/home/bash/script.sh'`




