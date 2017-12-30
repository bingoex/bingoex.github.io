---
layout: post
title: Linux下开发－进程间通信
categories: Linux C/C++
description: 
keywords: 
---


# 信号

当引发信号的事件发生时，为进程产生一个信号（**信号产生**）（或向进程发送一个信号）。事件可以是硬件异常、软件条件、终端产生的信号或调用kill函数。

在产生了信号时，内核通常在进程表中设置一个某种形式的标志（**信号递送**）。当对信号采取了这种动作时，我们说向进程递送了一个信号。

在信号产生（generation）和递送（delivery）之间的时间间隔，称**信号是未决的**（pending）。

进程可以选用信号递送阻塞。如果为进程产生了一个选择为阻塞的信号，而且对该信号的动作是系统默认动作或捕捉该信号，则为该进程**将此信号保持为未决状态，直到该进程对此信号解除了阻塞**，或者将对此信号的动作更改为忽略。

内核在递送一个原来被阻塞（现在解除了阻塞）的信号给进程时，才决定对它的处理方式。于是进程在信号递送给它之前仍可改变对该信号的动作。

进程调用sigpending函数来判定哪些信号是设置为阻塞并处于未决状态的。

每个进程都有一个信号屏蔽字（signal mask），它规定了当前要阻塞递送到该进程的信号集。对于每种可能的信号，该屏蔽字中都有一位与之对应。对于某种信号，若其对应位已设置，则它当前是被阻塞的。进程可以调用sigprocmask来检测和更改其当前信号屏蔽字。

信号数量可能会超过整型所包含的二进制位数，因此POSIX.1定义了一个新数据类型sigset_t，用于保存一个信号集。

## 信号与线程的关系

一般我们谈到信号时，都是只与进程挂钩的（多进程设计模式），很少会在线程中使用信号。
如果一个没有处理的信号的**默认动作是停止SIGSTOP或终止SIGKILL，那么不管这个信号是发送给哪个线程，整个进程都会停止或终止**。

一个进程中的**所有线程对某个信号都共享相同的信号处理函数**。

**使用pthread_kill()或者pthread_sigqueue()。这些函数允许一个线程发送信号到另一个线程**（同一进程中）。其他情况都是把信号发送到整个进程（比如，kill()和sigqueue()）。**当一个信号被发送到一个多线程的进程中（注意是发送到进程）。内核会选择该进程中的任意线程来处理该信号**。这种做法是为了保持进程中信号的语意，保证不会在多线程进程中一个信号多次被执行。

## 常用API
```c
int pthread_sigmask(int how, const sigset_t *set,sigset_t *oldest)
int pthread_kill(pthread_t thread, int sig)
int pthread_sigqueue(pthread_t thread, int sig, constunion sigval value)
int sigwait(const sigset_t *set, int *sig)
```

线程可以使用pthread_sigmask()来独立的**屏蔽某些信号**。通过这种方法，程序员可以控制那些线程响应那些信号。当线程被创建时，它将继承创建它的线程的信号掩码

**信号处理函数中不要使用线程不安全的函数（如pthread）**
在处理信号之前，对所有的异步信号进行阻塞，等工作处理完毕后，再恢复阻塞的信号。

## 缺点
- 信号的花销太大。发送信号要做系统调用。内核要中断接收进程、要管理它的堆栈、要调用处理程序、要恢复被中断的进程等。
- 信号种类有限，只有31种，而且信号能传递的信息量十分有限。
- 信号没有优先级，也没有次数的概念




# 共享内存

共享内存就是允许**多个不相关的进程访问同一个逻辑内存**。是最快的工程间通信方式。

共享内存并未提供同步机制，也就是说，在第一个进程结束对共享内存的写操作之前，并无自动机制可以阻止第二个进程开始对它进行读取。所以我们通常需要用其他的机制来同步对共享内存的访问，例如前面说到的信号量。

代码详情可以看

<https://github.com/bingoex/comm/blob/master/src/cm_shm.h>

## mmap
```c
void *mmap( void *start, size_tlength, int port, int flags, int fd, off_t offset)
```
flags : 映射文件属性值，可取 MAP_ANON, MAP_PRIVATE, MAP_SHARED，分别代表**匿名映射，私有copy-on-write映射，和共享映射**。映射的初期并没有 真正分配内存，只有访问页面的时候，引发一个缺页异常，这时才真正分配内存。



# 管道

- 普通管道PIPE,。通常有两种限制,一是单工,只能单向传输。二是只能在父子或者兄弟进程间使用。
- 流管道。双工，但一样只能在父子或者兄弟进程间使用。
- 命名管道。单工，但可以多进程间使用。
<http://www.cnblogs.com/masky5310/archive/2012/08/05/2623764.html>

## 原理

在Linux中，使用两个file数据结构来实现管道。这两个file数据结构中的f_inode指针指向同一个**临时创建的VFS I节点**，而该VFS I节点本身又**指向内存中的一个物理页**，如图所示。两个file数据结构中的f_op指针指向不同的文件操作例程向量表：一个用于向管道中写，另一个用于从管道中读。这种实现方法掩盖了底层实现的差异，从进程的角度来看，读写管道的系统调用和读写普通文件的普通系统调用没什么不同。当写进程向管道中写时，字节被拷贝到了共享数据页，当读进程从管道中读时，字节被从共享页中拷贝出来。Linux必须同步对于管道的存取，必须保证管道的写和读步调一致。Linux使用锁、等待队列和信号（locks，wait queues and signals）来实现同步。

![](/images/posts/2015-12-14-linux-c-process.md/1.png)

如果共享数据页中有足够的空间能把所有的字节都写到管道中，而且管道没有被读进程锁定，则Linux就在管道上为写进程加锁，并把字节从进程的地址空间拷贝到共享数据页。如果管道被读进程锁定或者共享数据页中没有足够的空间，则当前进程被迫睡眠，它被挂在管道I节点的等待队列中等待，而后调用调度程序，让另外一个进程运行。睡眠的写进程是可以中断的（interruptible），所以它可以接收信号。当管道中有了足够的空间可以写数据，或者当锁定解除时，写进程就会被读进程唤醒。当数据写完之后，管道的VFS I 节点上的锁定解除，在管道I节点的等待队列中等待的所有读进程都会被唤醒。

## 命名管道

命名管道是一种特殊类型的文件（调用**mkfifo**会产生一个文件），因为Linux中所有事物都是文件，它在文件系统中以文件名的形式存在。程序不能是O_RDWR模式打开FIFO文件进行读写操作，这样做的后果未明确定义，**只能一读一写**。
```c
#include <stdio.h>  
#include <stdlib.h>  
#include <string.h>  
#include <fcntl.h>  
#include <limits.h>  
#include <sys/types.h>  
#include <sys/stat.h>   
#define FIFO_NAME "/tmp/Linux/my_fifo"  
#define BUFFER_SIZE PIPE_BUF  
#define TEN_MEG (1024 * 1024 * 10)  

int main() {  
    int pipe_fd;  
    int res;  
    int open_mode = O_WRONLY;  

    int bytes = 0;  
    char buffer[BUFFER_SIZE + 1];  

    if (access(FIFO_NAME, F_OK) == -1)  {  
        res = mkfifo(FIFO_NAME, 0777);  
        if (res != 0) {  
            fprintf(stderr, "Could not create fifo %s/n", FIFO_NAME);  
            exit(EXIT_FAILURE);  
        }  
    }  

    printf("Process %d opening FIFO O_WRONLY/n", getpid());  
    pipe_fd = open(FIFO_NAME, open_mode);  
    printf("Process %d result %d/n", getpid(), pipe_fd);  

    if (pipe_fd != -1) {  
        while (bytes < TEN_MEG)  {  
            res = write(pipe_fd, buffer, BUFFER_SIZE);  
            if (res == -1)  
            {  
                fprintf(stderr, "Write error on pipe/n");  
                exit(EXIT_FAILURE);  
            }  
            bytes += res;  
        }  
        close(pipe_fd);  
    }  
    else  
    {  
        exit(EXIT_FAILURE);  
    }  

    printf("Process %d finish/n", getpid());  
    exit(EXIT_SUCCESS);  
}  
```
```c
#include <stdio.h>  
#include <stdlib.h>  
#include <string.h>  
#include <fcntl.h>  
#include <limits.h>  
#include <sys/types.h>  
#include <sys/stat.h>    
#define FIFO_NAME "/tmp/Linux/my_fifo"  
#define BUFFER_SIZE PIPE_BUF    
int main() {  
    int pipe_fd;  
    int res;  

    int open_mode = O_RDONLY;  
    char buffer[BUFFER_SIZE + 1];  
    int bytes = 0;  

    memset(buffer, '/0', sizeof(buffer));  

    printf("Process %d opeining FIFO O_RDONLY/n", getpid());  
    pipe_fd = open(FIFO_NAME, open_mode);  
    printf("Process %d result %d/n", getpid(), pipe_fd);  

    if (pipe_fd != -1){  
        do{  
            res = read(pipe_fd, buffer, BUFFER_SIZE);  
            bytes += res;  
        }while(res > 0);  
        close(pipe_fd);  
    }  
    else{  
        exit(EXIT_FAILURE);  
    }  

    printf("Process %d finished, %d bytes read/n", getpid(), bytes);  
    exit(EXIT_SUCCESS);  
}  
```

## 协同进程

当一个程序产生某个过滤程序的输入，同时又读取该过滤程序的输出时，则该过滤程序就成为**协同进程(coprocess)**。

![](/images/posts/2015-12-14-linux-c-process.md/2.png)

```c
//filter.c
#include <errno.h>
#include <fcntl.h>
#include <sys/wait.h>
#include <sys/stat.h>
#include <unistd.h>
#include <stdio.h>
#define MAXLINE 128
int main(void){
    int     n,  int1, int2;
    char    line[MAXLINE];
    while ((n = read(STDIN_FILENO, line, MAXLINE)) > 0) {
        line[n] = 0;        /* null terminate */
        if (sscanf(line, "%d%d", &int1, &int2) == 2) {
            sprintf(line, "%d/n", int1 + int2);
            n = strlen(line);
            if (write(STDOUT_FILENO, line, n) != n)
                printf("write error/n");
        } else {
            if (write(STDOUT_FILENO, "invalid args/n", 13) != 13)
                printf("write error/n");
        }
    }

    exit(0);
}

运行：
[root@localhost yuan]# gcc -o filterd filterd.c
[root@localhost yuan]# ./filterd
12 34
46
```

示例2：驱动上述过滤程序filter.c的程序
```c
//filterd.c
#include <errno.h>
#include <fcntl.h>
#include <sys/wait.h>
#include <sys/stat.h>
#include <unistd.h>
#include <stdio.h>
#include <signal.h>
#define MAXLINE 128
static void sig_pipe(int);     /* our signal handler */
int main(void){
    int     n, fd1[2], fd2[2];
    pid_t   pid;
    char   line[MAXLINE]; 
    if (signal(SIGPIPE, sig_pipe) == SIG_ERR)
        printf("signal error/n");
    if (pipe(fd1) < 0 || pipe(fd2) < 0)
        printf("pipe error/n");
    if ((pid = fork()) < 0) {
        printf("fork error/n");
    } else if (pid > 0) {
        printf("parent process/n");              /* parent */
        close(fd1[0]);
        close(fd2[1]);
        while (fgets(line, MAXLINE, stdin) != NULL) {
            n = strlen(line);
            if (write(fd1[1], line, n) != n)
                printf("write error to pipe/n");
            if ((n = read(fd2[0], line, MAXLINE)) < 0)
                printf("read error from pipe/n");
            if (n == 0) {
                printf("child closed pipe");
                break;
            }
            line[n] = 0;    /* null terminate */
            if (fputs(line, stdout) == EOF)
                printf("fputs error/n");
        }

        if (ferror(stdin))
            printf("fgets error on stdin/n");

        exit(0);
    } else {
        printf("child process/n");                /* child */
        close(fd1[1]);
        close(fd2[0]);
        if (fd1[0] != STDIN_FILENO) {
            if (dup2(fd1[0], STDIN_FILENO) != STDIN_FILENO)
                printf("dup2 error to stdin/n");
            close(fd1[0]);
        }

        if (fd2[1] != STDOUT_FILENO) {
            if (dup2(fd2[1], STDOUT_FILENO) != STDOUT_FILENO)
                printf("dup2 error to stdout/n");
            close(fd2[1]);
        }

        if (execl("./filter", "filter", (char *)0) < 0)
            printf("execl error/n");
    }

    exit(0);
}

static void sig_pipe(int signo)
{
    printf("SIGPIPE caught/n");
    exit(1);
}

运行：filter存在
[root@localhost yuan]# gcc -o filterd filterd.c
[root@localhost yuan]# ./filterd
child process
parent process
12 34
46
```

## 缺点

- 因为读数据的同时也将数据从管道移去，因此，管道不能用来对多个接收者广播数据。
- 管道中的数据被当作字节流，因此无法识别信息的边界。
- 如果一个管道有多个读进程，那么写进程不能发送数据到指定的读进程。同样，如果有多个写进程，那么没有办法判断是它们中那一个发送的数据



# 消息队列

消息的一个链表，它允许一个或多个进程向它写消息，一个或多个进程从中读消息。

## 原理

向消息队列中写的进程（wwait）和等待从消息队列中读的进程(rwait)。如果某进程向一个消息队列发送消息而发现该队列已满，则进程挂在wwait队列中等待。从该消息队列中读取消息的进程将从队列中删除消息，从而腾出空间，再唤醒wwait队列中等待的进程。如果某进程从一个消息队列中读消息而发现该队列已空，则进程挂在rwait队列中等待。向该消息队列中发送消息的进程将消息加入队列，再唤醒rwait队列中等待的进程。
```c
int sys_msgget (key_t key, int msgflg)
int sys_msgsnd (int msqid, struct msgbuf*msgp, size_t msgsz, int msgflg)
ssize_t msgrcv(int msqid, void *msgp,size_t msgsz, long msgtyp, int msgflg);
int msgctl ( int msgqid, int cmd, structmsqid_ds *buf );
```
```c
#define MSGKEY 1024   
struct msgstru  
{  
    long msgtype;  
    char msgtext[2048];  
};  


//生产消息
msqid=msgget(MSGKEY,IPC_EXCL);  /*检查消息队列是否存在*/  
if(msqid < 0){  
    msqid = msgget(MSGKEY,IPC_CREAT|0666);/*创建消息队列*/  
    if(msqid <0){  
        printf("failed to create msq | errno=%d [%s]\n",errno,strerror(errno));  
        exit(-1);  
    }  
}   
while (1){  
    printf("input message type(end:0):");  
    scanf("%d",&msg_type);  
    if (msg_type == 0)  
        break;  
    printf("input message to be sent:");  
    scanf ("%s",str);  
    msgs.msgtype = msg_type;  
    strcpy(msgs.msgtext, str);  
    /* 发送消息队列 */  
    ret_value = msgsnd(msqid,&msgs,sizeof(struct msgstru),IPC_NOWAIT);  
    if ( ret_value < 0 ) {  
        printf("msgsnd() write msg failed,errno=%d[%s]\n",errno,strerror(errno));  
        exit(-1);  
    }  
}  
msgctl(msqid,IPC_RMID,0); //删除消息队列 



//获取消息
/*子进程，监听消息队列*/  
void childproc(){  
    struct msgstru msgs;  
    int msgid,ret_value;  
    char str[512];  

    while(1){  
        msgid = msgget(MSGKEY,IPC_EXCL );/*检查消息队列是否存在 */  
        if(msgid < 0){  
            printf("msq not existed! errno=%d [%s]\n",errno,strerror(errno));  
            sleep(2);  
            continue;  
        }  
        /*接收消息队列*/  
        ret_value = msgrcv(msgid,&msgs,sizeof(struct msgstru),0,0);  
        printf("text=[%s] pid=[%d]\n",msgs.msgtext,getpid());  
    }  
    return;  
}  

void main()  
{  
    int i,cpid;  

    /* create 5 child process */  
    for (i=0;i<5;i++){  
        cpid = fork();  
        if (cpid < 0)  
            printf("fork failed\n");  
        else if (cpid ==0) /*child process*/  
            childproc();  
    }  
}
```

## 优点

消息队列和管道提供相似的服务，但消息队列要更加强大并解决了管道中所存在的一些问题。消息队列传递的消息是不连续的、有格式的信息，给对它们的处理带来了很大的灵活性。

## 缺点

小消息的传送效率很高，但大消息的传送性能则较差。因为消息传送的过程中要经过从用户空间到内核空间，再从内核空间到用户空间的拷贝，所以，大消息的传送其性能较差。另外，消息队列不支持广播，而且内核不知道消息的接收者






