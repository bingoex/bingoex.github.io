---
layout: post
title: Nginx源码浅析－主流程
categories: Nginx C/C++
description: Nginx源码浅析－主流程
keywords: 
---

# 主流程


![](/images/posts/2016-10-01-nginx-main.md/1.png)




# 前期初始化


- 1、ngx_strerror_init初始化所有系统错误字符串。

- 2、ngx_get_options获取**命令行参数**（-s等）。

- 3、ngx_time_init（线程安全，精确到毫秒级别）

调用ngx_time_update。上锁，获取当前时间并提前设置好相关字符串变量，从而方便后面使用。

- 4、ngx_regex_init

- 5、ngx_log_init初始化错误日志log

- 6、ngx_ssl_init

- 7、初始化ngx_cycle结构（[ngx_cycle_t](https://bingoex.github.io/2016/10/07/nginx-data-struct/)）

创建并设置内存池、保存程序运行参数和环境变量、设置前缀和配置文件等路径字符串。
```C
struct ngx_cycle_s {
    void ****conf_ctx;
    ngx_pool_t *pool;

    ngx_log_t *log;
    ngx_log_t new_log;

    ngx_uint_t log_use_stderr; /* unsigned log_use_stderr:1; */

    ngx_connection_t **files;
    ngx_connection_t *free_connections;
    ngx_uint_t free_connection_n;

    ngx_module_t **modules;
    ngx_uint_t modules_n;
    ngx_uint_t modules_used; /* unsigned modules_used:1; */

    ngx_queue_t reusable_connections_queue;
    ngx_uint_t reusable_connections_n;

    ngx_array_t listening;
    ngx_array_t paths;

    ngx_array_t config_dump;
    ngx_rbtree_t config_dump_rbtree;
    ngx_rbtree_node_t config_dump_sentinel;

    ngx_list_t open_files;
    ngx_list_t shared_memory;

    ngx_uint_t connection_n;
    ngx_uint_t files_n;

    ngx_connection_t *connections;
    ngx_event_t *read_events;
    ngx_event_t *write_events;

    ngx_cycle_t *old_cycle;

    ngx_str_t conf_file;
    ngx_str_t conf_param;
    ngx_str_t conf_prefix;
    ngx_str_t prefix;
    ngx_str_t lock_file;
    ngx_str_t hostname;
};
```

![](/images/posts/2016-10-01-nginx-main.md/2.jpeg)

- 8、ngx_os_init

获取系统名和版本号、设置网络io回调（ngx_linux_io）、获取cpu核数（ngx_ncpu）、最大socket数（ngx_max_sockets）、CpuCacheLine大小（ngx_cacheline_size）、内存页大小（ngx_pagesize）、设置随机数种子。

- 9、ngx_add_inherited_sockets

如果环境变量有NGINX_VAR值，则从环境变量中取出socket的fd信息并设置到ngx_cycle的ngx_listening_t变量listening中，设置ngx_inherited值为1。

- 10、ngx_preinit_modules简单初始化ngx_module_t数组ngx_modules

# 11、ngx_init_cycle配置初始化


![](/images/posts/2016-10-01-nginx-main.md/3.png)

从步骤7中的ngx_cycle变量中拷贝值到新的ngx_cycle_t变量，包括设置内存池、log、前缀字符串、配置文件字符串、路径数组、dump相关、打开文件句柄list（ngx_open_file_t）、共享内存list（ngx_shm_zone_t）、监听数组（ngx_listening_t）、重用连接queue、配置文件上下文、hostname主机名、模块数组ngx_module_t。

- 11.0、**初始化每个核心模块（type为NGX_CORE_MODULE），调用其create_conf回调方法，将返回值（生成模块配置文件上下文）保存到cycle->conf_ctx相应模块下标中**。

初始化ngx_conf_t变量conf（type为NGX_MAIN_CONF）

```C
struct ngx_conf_s {
    char *name;
    ngx_array_t *args;

    ngx_cycle_t *cycle;
    ngx_pool_t *pool;
    ngx_pool_t *temp_pool;
    ngx_conf_file_t *conf_file;
    ngx_log_t *log;

    void *ctx;
    ngx_uint_t module_type;
    ngx_uint_t cmd_type;

    ngx_conf_handler_pt handler;
    char *handler_conf;
};
```

- 11.1、ngx_conf_param（解析ngx_cycle_s的conf_param参数值，通过-g指定）

**调用ngx_conf_parse，解析（状态机，各种if、case）并设置到conf中参数args**，对于每个配置参数都进行如下步骤。

回调第三方自定义指令解析机制conf的handler方法，如果有，一般没有。

```C
rv = (*cf->handler)(cf, NULL, cf->handler_conf);
```

调用ngx_conf_handler方法。**轮询所有模块ngx_module_t（type为NGX_CONF_MODULE）的所有ngx_command_t与从文件中解析出来的args比较**。并检测其合法性（参数类型、个数、出现的位置），**并调用cmd的set回调方法**。
```C
rc = ngx_conf_handler(cf, rc);
rv = cmd->set(cf, cmd, conf);

struct ngx_command_s {
    ngx_str_t name;
    ngx_uint_t type;
    char *(*set)(ngx_conf_t *cf, ngx_command_t *cmd, void *conf);
    ngx_uint_t conf;
    ngx_uint_t offset;
    void *post;
};
```

- 11.2、ngx_conf_parse（解析ngx_cycle_s的**conf_file文件**，同8.1）

- 11.3、调用每个**核心模块**（type为NGX_CORE_MODULE）的**init_conf回调方法**。

- 11.4、获取核心模块ngx_core_conf_t，并生成和删除相关pid文件

```C
ccf = (ngx_core_conf_t *) ngx_get_conf(cycle->conf_ctx, ngx_core_module);
#define ngx_get_conf(conf_ctx, module) conf_ctx[module.index]
```

```C
typedef struct {
    ngx_flag_t daemon;
    ngx_flag_t master;

    ngx_msec_t timer_resolution;
    ngx_msec_t shutdown_timeout;

    ngx_int_t worker_processes;
    ngx_int_t debug_points;

    ngx_int_t rlimit_nofile;
    off_t rlimit_core;

    int priority;

    ngx_uint_t cpu_affinity_auto;
    ngx_uint_t cpu_affinity_n;
    ngx_cpuset_t *cpu_affinity;

    char *username;
    ngx_uid_t user;
    ngx_gid_t group;

    ngx_str_t working_directory;
    ngx_str_t lock_file;

    ngx_str_t pid;
    ngx_str_t oldpid;

    ngx_array_t env;
    char **environment;
} ngx_core_conf_t;
```

- 11.5、删除相关锁文件、生成相关path文件夹并设置权限和用户、打开新文件open_files并关闭无用open_files、打开shared_memory及初始化slab、关闭无用shared_memory、处理listening数组并设置socket参数（SO_REUSEPORT、SO_REUSEADDR、IPV6_V6ONLY、noblock、unix域chmod、SO_RCVBUF、SO_SNDBUF、SO_KEEPALIVE、TCP_KEEPIDLE、TCP_KEEPINTVL、TCP_KEEPCNT、SO_SETFIB、TCP_FASTOPEN、SO_ACCEPTFILTER、TCP_DEFER_ACCEPT、IP_RECVDSTADDR、IP_PKTINFO、IPV6_RECVPKTINFO）、开启监听（bind、listen）并关闭无用listening

调用ngx_init_modules，**回调各个模块的init_module方法**。
```C
cycle->modules[i]->init_module(cycle)
```

# 信号处理


- 12、**如果是-s启动（ngx_signal），则发送完信号后退出程序**

读取pid文件，取出pid。发送相应信号给pid进程。

具体信号处理函数ngx_signal_handler会设置标志位变量，然后在各进程while循环中处理。

- 13、注册信号处理函数、daemon初始化、生成pid文件

# 子进程启动


- 14、如果**NGX_PROCESS_SINGLE**（单进程）模式启动则调用ngx_single_process_cycle
设置环境变量ngx_set_environment、调用每个模块的init_process方法。

进入无限循环。
- 调用ngx_process_events_and_timers，处理事件和定时器timer
- 调用ngx_shmtx_trylock，获取共享内存锁后轮询所有listening的conn，把符合条件的socket添加epoll读事件（ngx_event_actions.add）。
- 调用ngx_process_events（ngx_event_actions.process_events）。
- 调用ngx_event_process_posted，回调所有ngx_posted_accept_events队列里面的ngx_event_t的handle方法。
- 调用ngx_shmtx_unlock，释放锁。
- 调用ngx_event_expire_timers，处理所有超时事件。
- 调用ngx_event_process_posted，回调所有ngx_posted_events队列里面的ngx_event_t的handle方法。

处理ngx_terminate和ngx_quit信号
- 调用每个模块的exit_process方法。
- 调用ngx_master_process_exit。删除pid文件、回调所有模块的exit_master方法、关闭监听socket、关闭连接池回调其cleanup。

处理ngx_reconfigure信号，调用ngx_init_cycle重新读取配置文件。

处理ngx_reopen信号，重新打开open_files，并设置用户权限等参数？？


<br/>
<br/>
<br/>

- 15、如果其他模式启动则调用ngx_master_process_cycle。

sigprocmask屏蔽相关信号


- 15.0、调用ngx_start_worker_processes启动子进程

以**NGX_PROCESS_RESPAWN**模式创建子进程。每个子进程调用socketpair创建双向通信管道并设置异步io相关属性。**调用fork，其中子进程调用ngx_worker_process_cycle函数后无限循环**。master每生成一个子进程就调用ngx_pass_open_channel把其信息（pid，数组下标、NGX_CMD_OPEN_CHANNEL等ngx_channel_t）通过管道告诉其他已经生成的子进程，子进程收到后会写入自己的ngx_processes数组。

其中**ngx_worker_process_cycle**。

设置基本参数（ngx_process＝NGX_PROCESS_WORKER、ngx_worker下标）。调用ngx_worker_process_init，设置环境变量、setpriority设置优先级、setrlimit设置句柄等资源参数、如果是root用户则设置user和group、设置cpu亲密性、更换进程运行目录、**调用每个模块的init_process函数**、关闭其他与自己无关的管道、通过ngx_event_actions.add*设置管道监听事件回调ngx_channel_handler。

进入while循环，处理信号。
- 处理ngx_exiting，等所有timer事件都cancelable时调用ngx_worker_process_exit退出。
- 调用ngx_process_events_and_timers。
- 处理ngx_terminate，调用ngx_worker_process_exit退出。
- 处理ngx_quit，设置ngx_exiting为1。增加timer（ngx_shutdown_timer_handler，超时时间shutdown_timeout）超时后尽量读取conntion中的数据。关闭所有listen socket。尽快关闭idel的连接。
- 处理ngx_reopen，重新打开open_files。


- 15.1、调用ngx_start_cache_manager_processes

如果有ngx_cycle的path设置了manager和loader参数，则**fork两个子进程（NGX_PROCESS_RESPAWN模式），子进程调用ngx_cache_manager_process_cycle**，并通知其他子进行自己的信息（同woker）

其中**ngx_cache_manager_process_cycle**

关闭listening、增加一个timer、while循环处理ngx_terminate、ngx_quit、ngx_reopen、调用ngx_process_events_and_timers（同上）。



- 15.2、master处理while循环

处理SIGALRM（**ngx_sigalrm**）信号，定时触发（超时时间delay初始50，＊2递增），辅助ngx_terminate信号。

调用**sigsuspend阻塞master进程等待信号发生**。

处理SIGCHLD信号（**ngx_reap**），调用ngx_reap_children。关闭所有状态exited为1的woker进程的管道，每关闭一个就通知其他woker进程（NGX_CMD_CLOSE_CHANNEL），同时调用ngx_spawn_process开启新的woker进程。如果还有子进程存活则设置live=1。

如果live为0，表明没有子进程存活，则master调用ngx_master_process_exit。

处理SIGINT信号（**ngx_terminate**），调用ngx_signal_worker_processes通过管道发送相应信息（一开始发送SIGTERM/NGX_CMD_TERMINATE，后面delay值大于1000后发送SIGKILL）给子进程，并通过kill系统调用发送相应信号给wokers。

处理SIGQUIT信号（**ngx_quit**）。发送SIGQUIT/NGX_CMD_QUIT信号给所有woker进程，并关闭监听端口。

处理SIGHUP信号（ngx_reconfigure）。
- 如果ngx_new_binary不为0，则调用ngx_start_worker_processes（**NGX_PROCESS_RESPAWN**）、ngx_start_cache_manager_processes（**NGX_PROCESS_RESPAWN**）。
- 如果ngx_new_binary为0，则调用ngx_init_cycle、ngx_start_worker_processes（**NGX_PROCESS_JUST_RESPAWN**）、ngx_start_cache_manager_processes（**NGX_PROCESS_JUST_RESPAWN**）。
- 睡眠100ms让新进程启动。
- 发送信号（NGX_SHUTDOWN_SIGNAL/SIGQUIT/NGX_CMD_QUIT）给所有子进程。

处理ngx_restart，调用ngx_start_worker_processes启动新woker进程（**NGX_PROCESS_RESPAWN**），调用**ngx_start_cache_manager_processes启动新manger和loader进程**。

处理SIGINFO信号（**ngx_reopen**），重新打开open_file设置用户属性，并发送相应信号（NGX_REOPEN_SIGNAL/SIGINFO/NGX_CMD_REOPEN）给所有woker进程。

处理XCPU信号（**ngx_change_binary**），调用ngx_exec_new_binary以NGX_PROCESS_DETACHED热代码替换模式创建进程。并设置ngx_new_binary为新创建进程pid。

处理WINCH信号（**ngx_noaccept**），发送信号（NGX_SHUTDOWN_SIGNAL/SIGQUIT/NGX_CMD_QUIT）给子进程。


# 备注
- ngx_开头的均为全局变量
- <http://www.lenky.info/archives/2011/09/22>
- <http://blog.csdn.net/u013009575/article/details/18800875>
- <http://tengine.taobao.org/book/chapter_11.html>
- <http://blog.csdn.net/fjslovejhl/article/details/8124510>
- <http://www.cnblogs.com/ourroad/p/4861096.html>
- <http://blog.csdn.net/jackywgw/article/details/51454006>


