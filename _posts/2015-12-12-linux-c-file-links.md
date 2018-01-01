---
layout: post
title: Linux下开发－揭秘文件链接数
categories:  C/C++
description: 
keywords: 
---



# struct stat

每一个文件，都可以通过一个struct stat的结构体来获得文件信息，其中一个成员st_nlink代表文件的链接数。
```c
struct stat {
    dev_t     st_dev;     /* ID of device containing file */
    ino_t     st_ino;     /* inode number */
    mode_t    st_mode;    /* protection */
    nlink_t   st_nlink;   /* number of hard links */
    uid_t     st_uid;     /* user ID of owner */
    gid_t     st_gid;     /* group ID of owner */
    dev_t     st_rdev;    /* device ID (if special file) */
    off_t     st_size;    /* total size, in bytes */
    blksize_t st_blksize; /* blocksize for file system I/O */
    blkcnt_t  st_blocks;  /* number of 512B blocks allocated */
    time_t    st_atime;   /* time of last access */
    time_t    st_mtime;   /* time of last modification */
    time_t    st_ctime;   /* time of last status change */
};
```



# open

当通过shell的touch命令或者在程序中open一个带有O_CREAT的不存在的文件时，文件的链接数为1。

通常**open**一个已存在的文件**不会影响文件的链接数**。open的作用**只是使调用进程与文件之间建立一种访问关系**，即open之后返回fd，调用进程可以通过fd来read、write 、 ftruncate等等一系列对文件的操作。



# close

close()就是**消除这种调用进程与文件之间的访问关系**。自然，**不会影响文件的链接数**。在调用close时，内核会检查打开该文件的进程数，如果此数为0，进一步检查文件的链接数，如果这个数也为0，那么就删除文件内容。



# link

**link函数创建一个新目录项，并且增加一个链接数**。



# unlink
```c
int unlink( constchar* pathname);
```

此函数**删除目录项**，并将由pathname所**引用文件的链接计数减1**。如果还有指向该文件的其它链接，则仍可通过其他链接访问该文件的数据。如果出错，则不对该文件做任何更改。

只有当链接计数达到0时，该文件的内容才可被删除。

关闭一个文件时，内核首先检查打开该文件的进程数。如果该数达到0，然后内核检查其链接数，如果这个数也是0，那么就删除该文件的内容。
```c
int main(void) { 
    int fd; 
    char buf[20] = {0}; 
    if ((fd =open("tempfile", O_RDWR)) < 0) 
        err_sys("open error"); 
    if (unlink("tempfile") < 0) 
        err_sys("unlink error"); 
    printf("file unlinked/n"); 
    read(fd, buf, sizeof(buf));//you could still read this after unlink 
    printf("%s/n", buf); 
}
```

unlink的这种性质经常被用来确保即使是在该程序崩溃时，它所创建的临时文件也不会遗留下来。进程用open或create创建一个文件，然后立即调用unlink。因为该文件仍旧是打开的，所以不会将其内容删除。只有当进程关闭该文件或终止时（在这种情况下，内核会关闭该进程打开的全部文件），该文件的内容才会被删除。

如果pahtname是符号链接，那么unlink删除该符号链接，而不会删除由该链接所引用的文件。



# remove

int remove(constchar* pathname);
我们也可以用remove函数解除对一个文件或目录的链接。对于文件，**remove的功能与unlink**相同。

ISO C指定remove函数删除一个文件，这更改了UNIX系统历来使用的名字unlink，其原因是实现C标准的大多数非UNIX系统并不支持文件链接。


# 总结

综上所诉，真正影响链接数的操作是link、unlink以及open的创建。删除文件内容的真正含义是文件的链接数为0，而这个操作的本质完成者是unlink。close能够实施删除文件内容的操作，必定是因为在close之前有一个unlink操作。

举个例子简单说明：通过shell命令touch一个文件test.txt
```c
1、stat("test.txt",&buf);
printf("1.link=%d\n",buf.st_nlink);//未打开文件之前测试链接数

2、fd=open("test.txt",O_RDONLY);//打开已存在文件test.txt
stat("test.txt",&buf);
printf("2.link=%d\n",buf.st_nlink);//测试链接数

3、close(fd);//关闭文件test.txt
stat("test.txt",&buf);
printf("3.link=%d\n",buf.st_nlink);//测试链接数

4、link("test.txt","test2.txt");//创建硬链接test2.txt
stat("test.txt",&buf);
printf("4.link=%d\n",buf.st_nlink);//测试链接数

5、unlink("test2.txt");//删除test2.txt
stat("test.txt",&buf);
printf("5.link=%d\n",buf.st_nlink);//测试链接数

6、重复步骤2  //重新打开test.txt

7、unlink("test.txt");//删除test.txt
fstat(fd,&buf);
printf("7.link=%d\n",buf.st_nlink);//测试链接数

8、close(fd);//此步骤可以不显示写出，因为进程结束时，打开的文件自动被关闭。
```

顺次执行以上8个步骤，结果如下：
```c
1.link=1
2.link=1   //open不影响链接数
3.link=1   //close不影响链接数
4.link=2   //link之后链接数加1
5.link=1   //unlink后链接数减1
2.link=1   //重新打开  链接数不变
7.link=0   //unlink之后再减1，此处我们改用fstat函数而非stat，因为unlilnk已经删除文件名，所以不可以通过 文件名访问，但是fd仍然是打开着的，文件内容还没有被真正删除，依旧可以使用fd获得文件信息。
执行步骤8，文件内容被删除
```




