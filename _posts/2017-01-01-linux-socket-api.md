---
layout: post
title: 【精品】linux网络API细节
categories:  C/C++
description: 
keywords: 
---



# 原理及重要细节

connect函数，是收到syn+ack，发送ack之后返回；

accept函数跟三次握手没有关系，accept是从accept队列里面取一条已建立好的连接；

bind函数只是进程占用ip+port；声明：该ip+port被这个进程占用了；

backlog是listen函数传入的第二个参数



客户端调用connect函数建立连接，内部是发送了一个SYN包到服务端.服务端如果端口没有在监听该端口，服务端回复RST包。client会报ECONNREFUSED错误。




**半连接队列**：syn队列。长度由max（tcp_max_syn_backlog，64）决定

服务端如果端口存活，该socket处于sync_recv状态。这时如果半连接队列（syn队列）未满，则将该socket存到半连接队列中，返回syn + ack

服务端发送syn+ack包后，等待客户端的回复，如果客户端一直没有回复，则服务端重发syn+ack包，重试次数由tcp_synack_retries决定

如果半连接队列已满，则抛弃该syn包。客户端没有收到回复隔5s、24s、75s重试，如果最终也没有收到回复，则client报ETimeout错误

SynFlood攻击：大量SYNC_RECV的TCP连接会导致半连接队列溢出，这样后续的连接建立请求会被内核直接丢弃。

[SYN Cookie](https://bingoex.github.io/2015/11/10/sync-cookie/)。在TCP服务器收到SYN包并返回SYN+ACK包时，不分配一个专门的数据区，而是根据这个SYN包计算出一个cookie值。在收到ACK包时，TCP服务器在根据那个cookie值检查这个ACK包的合法性。如果合法，再分配专门的数据区进行处理未来的TCP连接



**全连接队列**：accept队列。长度=min（tcp_max_syn_backlog，somaxconn）

服务端收到客户端回复的ack包后，把socket从半连接队列中移除，移到全连接队列中，socket状态变为established。服务端调用accept函数后，从全连接队列中移除该socket，但是状态还是established

全连接队列满时如果收到客户端回复的ack包，则重新向客户端发送syn+ack包或者忽略ack包



# “connection reset by peer”和”broken pipe”
1）往一个对端已经close的通道写数据的时候，对方的tcp会收到这个报文，并且反馈一个reset报文。当收到reset报文的时候，继续做读数据的时候就会抛出Connect reset by peer的异常。

2）当第一次往一个对端已经close的通道写数据的时候会和上面的情况一样，会收到reset报文。当再次往这个socket写数据的时候，就会抛出Broken pipe了。

根据tcp的约定，当收到reset包的时候，上层必须要做出处理，调用将socket文件描述符进行关闭，其实也意味着pipe会关闭，因此会抛出这个顾名思义的异常。



UDP的send( )/sendto( )只是将UDP包的数据拷贝到发送缓冲区就立即返回了，等到第2次send( )的时候错误才被捕获，这种现象为异步错误

udpocket调用connect( )之后，其只是在TCP/IP协议栈内绑定一个（协议/源IP/源端口/目的IP/目的端口）的五元组，一直维护到连接结束

无连接UDP调用1次sendto( )发送UDP包，系统要做3件事：连接=>发送=>断开连接。而有连接UDP的send( )由于已经连接好了，只需完成"发送"这一步，故有连接UDP在性能上要优于无连接UDP




# 发送接收API

下面内容较为无聊和简单，可选择性查看
```c
#include <sys/socket.h>
size_t recv(int sockfd, void * buf, size_t nbytes, int flags);
size_t recvfrom(int sockfd,  //套接字
        void * buf,  //接收数据缓冲区
        size_t len,  //接收数据长度
        int flags,   //标志
        struct sockaddr * addr, //数据发送者地址，函数调用后该地址结构被填充
        socklen_t * addrlen  //地址长度指针(注意这里是个指针)
        );
size_t recvmsg(int sockfd, struct msghdr * msg, int flag);

struct msghdr {
    void *msg_name; /* 消息的协议地址 */
    socklen_t msg_namelen; /* 地址的长度 */
    struct iovec *msg_iov; /* 多io缓冲区的地址 */
    int msg_iovlen; /* 缓冲区的个数 */
    void *msg_control; /* 辅助数据的地址 */
    socklen_t msg_controllen; /* 辅助数据的长度 */
    int msg_flags; /* 接收消息的标识 */
};

struct iovec {
    void *io_base; /* buffer空间的基地址 */
    size_t iov_len; /* 该buffer空间的长度 */
};

struct cmsghdr {//msg_control指向的内容，可以有多个，通过下面的宏操作
    socklen_t cmsg_len; /* 包含该头部的数据长度 */
    int cmsg_level; /* 具体的协议标识 IPPROTO_IP(ipv), IPPROTO_IPV6(ipv6), SOL_SOCKET(unix domain).*/
    int cmsg_type; /* 协议中的类型 如SOL_SOCKET中主要包含:SCM_RIGHTS(发送接收描述字)， SCM_CREDS(发送接收用户凭证)*/
};
CMSG_FIRSTHDR()
CMSG_NXTHDR()
CMSG_DATA(）
CMSG_LEN()
CMSG_SPACE()
```
返回值：已读字节计数的消息长度，若对方已经按序结束则返回0，出错返回-1（包括暂时无数据发送）。

recvfrom可以得到数据发送者的源地址。

recvfrom通常用于无连接套接字。否则recvfrom等同于recv。
对于SOCK_STREAM套接字，接收的数据可以比请求的少（[流式](https://bingoex.github.io/2015/11/07/net-tcp/)），标志MSG_WAITALL可以阻止这种行为，除非所需数据全部收到，recv函数才返回。对于SOCK_DGRAM和SOCK_SEQPACKET套接字，MSG_WAITALL标志没有什么影响，因为这些基于报文的套接字类型一次读取就返回整个报文。

recvmsg将接收到的数据送入多个缓冲区，或者想接收辅助数据（如fd在进程间传递）。

```c
void Recv(){
    struct sockaddr_in serv_addr;
    int sock_fd;
    char line[10];
    int size =10;
    serv_addr.sin_family= AF_INET;
    serv_addr.sin_addr.s_addr= htonl(INADDR_LOOPBACK);
    serv_addr.sin_port= htons(5000);
    sock_fd=socket(AF_INET,SOCK_STREAM,IPPROTO_TCP);
    connect(sock_fd,(struct sockaddr*)&serv_addr,sizeof(serv_addr));
    recv(sock_fd, line, size, 0);
    close(sock_fd);
}

void RecvFrom(){
    struct sockaddr_in sender_addr;
    int sock_fd;
    char line[15]="Hello World!";
    unsignedint size =sizeof(sender_addr);
    sock_fd =socket(AF_INET,SOCK_DGRAM,IPPROTO_UDP);
    sender_addr.sin_family= AF_INET;
    sender_addr.sin_addr.s_addr= htonl(INADDR_LOOPBACK);
    sender_addr.sin_port= htons(5000);
    recvfrom(sock_fd,line,13,0,(struct sockaddr*)&sender_addr,&size);
    close(sock_fd);
}

void RecvMsg(){
    int sock_fd;
    unsignedint sender_len;
    struct msghdr msg;
    struct iovec iov;
    struct sockaddr_in receiver_addr,sender_addr;
    char line[10];
    sock_fd =socket(AF_INET,SOCK_DGRAM,IPPROTO_UDP);
    receiver_addr.sin_family= AF_INET;
    receiver_addr.sin_addr.s_addr= htonl(INADDR_ANY);
    receiver_addr.sin_port= htons(5000);
    bind(sock_fd,(struct sockaddr*)&receiver_addr,sizeof(receiver_addr));
    sender_len =sizeof(sender_addr);
    msg.msg_name=&sender_addr;
    msg.msg_namelen= sender_len;
    msg.msg_iov=&iov;
    msg.msg_iovlen=1;
    msg.msg_iov->iov_base = line;
    msg.msg_iov->iov_len =10;
    msg.msg_control=0;
    msg.msg_controllen=0;
    msg.msg_flags=0;
    recvmsg(sock_fd,&msg,0);
    close(sock_fd);
}
```

```c
size_t send(int sockfd, const void * buf, size_t nbytes, int flags);
size_t sendto(//一般用于udp
        int sockfd,       //套接字
        const void * buf, //带发送数据存储缓冲区
        size_t nbytes,    //要发送数据的字节数
        int flags,        //可选标志
        const struct sockaddr_in * destaddr, //(目标地址)数据接收方 
        socklen_t destlen //目标地址结构长度
        );
size_t sendmsg(int sockfd, const struct msghdr * msg, int flag);
```




# SocketAPI

<http://blog.csdn.net/hguisu/article/details/7445768/>








# fd在进程间传送

fd的传递，就是将一个进程中的描述字传递到另一个进程中，使得该描述字依然有效，也就是使得在一个进程中的描述字传递到另一个描述字依然有效。

## 原理

在两个进程之间创建一个unix domain socket套接口，然后调用sendmsg这个套接口发送一个特殊的消息，该消息由内核进行特殊的处理，从而把打开的描述字从发送进程传递到接收进程（采用recvmsg接收）

## 步骤

(1) 创建一个字节流或者数据报的unix domain socket套接口（父子进程可以用管道socketpair）

(2) 发送进程创建一个msghdr结构，其中将待传递的fd作为辅助数据发送，调用sendmsg发送fd。在发送完成以后，在发送进程即使关闭该fd也不会影响接收进程的fd，发送一个fd导致该fd的引用计数加1。（注意：至少一个字节的数据，该数据在接收过程中不做任何的处理）

(3) 接收进程调用recvmsg接收该fd，该fd在接收进程中的描述字号不同于在发送进程中的fd号是正常的，也就是说如果在发送进程中fd号是20，而在接收进程中对应的fd号可能被使用，该进程会分配一个不一样的fd号。

```c
int main(int argc, char *argv[]){
    int clifd, listenfd;
    struct sockaddr_un servaddr, cliaddr;
    int ret;
    socklen_t clilen;
    struct msghdr msg;
    struct iovec iov[1];
    char buf[100];
    char *testmsg = "test msg.\n";

    union {
        struct cmsghdr cm;
        char control[CMSG_SPACE(sizeof(int))];
    } control_un;
    struct cmsghdr *pcmsg;
    int recvfd;

    listenfd = socket(AF_UNIX, SOCK_STREAM, 0);
    if (listenfd < 0) {
        printf("socket failed.\n");
        return -1;
    }

    unlink(UNIXSTR_PATH);

    bzero(&servaddr, sizeof(servaddr));
    servaddr.sun_family = AF_UNIX;
    strcpy(servaddr.sun_path, UNIXSTR_PATH);

    ret = bind(listenfd, (SA *)&servaddr, sizeof(servaddr));
    if (ret < 0) {
        printf("bind failed. errno = %d.\n", errno);
        close (listenfd);
        return -1;
    }

    listen(listenfd, 5);

    while (1) {
        clilen = sizeof(cliaddr);
        clifd = accept(listenfd, (SA *)&cliaddr, &clilen);
        if (clifd < 0) {
            printf("accept failed.\n");
            continue;
        }

        msg.msg_name = NULL;
        msg.msg_namelen = 0;
        iov[0].iov_base = buf;
        iov[0].iov_len = 100;
        msg.msg_iov = iov;
        msg.msg_iovlen = 1;
        msg.msg_control = control_un.control;
        msg.msg_controllen = sizeof(control_un.control);

        ret = recvmsg(clifd, &msg, 0);
        if (ret <= 0) {
            return ret;
        }

        if ((pcmsg = CMSG_FIRSTHDR(&msg)) != NULL && (pcmsg->cmsg_len == CMSG_LEN(sizeof(int)))) {
            if (pcmsg->cmsg_level != SOL_SOCKET) {
                printf("cmsg_leval is not SOL_SOCKET\n");
                continue;
            }

            if (pcmsg->cmsg_type != SCM_RIGHTS) {
                printf("cmsg_type is not SCM_RIGHTS");
                continue;
            }

            recvfd = *((int *) CMSG_DATA(pcmsg));
            printf("recv fd = %d\n", recvfd);

            write(recvfd, testmsg, strlen(testmsg) + 1);
        }
    }  
    return 0;
}
```

客户端发送描述字:
```c
#include "unp.h"
#define OPEN_FILE  "test"
int main(int argc, char *argv[]){
    int clifd;
    struct sockaddr_un servaddr;
    int ret;
    struct msghdr msg;
    struct iovec iov[1];
    char buf[100];
    union {
        struct cmsghdr cm;
        char control[CMSG_SPACE(sizeof(int))];
    } control_un;
    struct cmsghdr *pcmsg;
    int fd;

    clifd = socket(AF_UNIX, SOCK_STREAM, 0);
    if (clifd < 0) {
        printf("socket failed.\n");
        return -1;
    }

    fd = open(OPEN_FILE, O_CREAT| O_RDWR, 0777);
    if (fd < 0) {
        printf("open test failed.\n");
        return -1;
    }

    bzero(&servaddr, sizeof(servaddr));
    servaddr.sun_family = AF_UNIX;
    strcpy(servaddr.sun_path, UNIXSTR_PATH);

    ret = connect(clifd,(SA *)&servaddr, sizeof(servaddr));
    if (ret < 0) {
        printf("connect failed.\n");
        return 0;
    }

    msg.msg_name = NULL;
    msg.msg_namelen = 0;
    iov[0].iov_base = buf;
    iov[0].iov_len = 100;
    msg.msg_iov = iov;
    msg.msg_iovlen = 1;
    msg.msg_control = control_un.control;
    msg.msg_controllen = sizeof(control_un.control);

    pcmsg = CMSG_FIRSTHDR(&msg);
    pcmsg->cmsg_len = CMSG_LEN(sizeof(int));
    pcmsg->cmsg_level = SOL_SOCKET;
    pcmsg->cmsg_type = SCM_RIGHTS;
    *((int *)CMSG_DATA(pcmsg)) = fd;

    ret = sendmsg(clifd, &msg, 0);
    printf("ret = %d.\n", ret);
    return 0;
}
```




# 字节序

```
htonl()--"Host to Network Long"
ntohl()--"Network to Host Long"
htons()--"Host to Network Short"
ntohs()--"Network to Host Short"


#include <arpa/inet.h>
uint32_t ntohl(uint32_t netlong);
```



# 实体信息hostent、netent、protoent、servent

hostent是host entry的缩写，该结构记录主机的信息，包括主机名、别名、地址类型、地址长度和地址列表。
```c
struct hostent{
    char * h_name;
    char ** h_aliases;
    short h_addrtype;
    short h_length;
    char ** h_addr_list;
};
struct hostent *gethostbyname(const char *name);

struct netent {char*n_name;/* 网络名 */char**n_aliases;/* 网络别名列表 */intn_addrtype;/* 网络地址类型 */uint32_tn_net;/* 网络号 */};
struct protoent{
    char *p_name;            /* Official protocol name.  */
    char **p_aliases;        /* Alias list.  */
    int p_proto;             /* Protocol number.  */
};
struct servent{char*s_name;/* 服务名 */char**s_aliases;/* 服务别名列表 */ints_port;/* 端口号 */char*s_proto;/* 使用的协议 */};
```
传入值是域名或者主机名，例如"www.google.cn"等等。



# 地址和字符串转化

```c
//适用于ipv4和ipv6
int inet_pton(int family,const char * strptr,void * addrptr);
const char * inet_ntop(int family,const void * addrptr,char * strptr,size_t len);

#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
int main(int argc,char ** argv){
    char dst[100];
    int sockfd = socket(AF_INET,SOCK_STREAM,0);

    struct sockaddr_in serv;    
    memset(&serv,0,sizeof(struct sockaddr_in));

    serv.sin_family = AF_INET;
    serv.sin_port = htons(5555);
    //serv.sin_addr.s_addr = INADDR_ANY;
    //以下serv.sin_addr.s_addr可替换为 serv.sin_addr
    if((inet_pton(AF_INET,"127.0.0.1",&serv.sin_addr.s_addr))==0)
        printf("inet_pton \n");    
    if((inet_ntop(AF_INET,&serv.sin_addr.s_addr,dst,sizeof(dst)))==NULL)
        printf("inet_ntop\n");
    printf("dst=%s,sizeof(dst)=%d\n",dst,sizeof(dst));

    bind(sockfd,(struct sockaddr *)&serv,sizeof(serv));
    listen(sockfd,15);
    return 0;
}



//只能用IPV4
//网络主机地址cp转为二进制数值
int inet_aton(const char *cp, struct in_addr *inp);

//网络主机地址（如192.168.1.10)转为网络字节序二进制值
in_addr_t inet_addr(const char *cp);

//函数返回指向点分开的字符串地址的指针
char *inet_ntoa(struct in_addr in);
```







