---
layout: post
title: Nginx源码浅析－常用数据结构
categories: Nginx 数据结构
description: nginx源码浅析--常用数据结构
keywords: 
---


# ngx_str_t
```c
typedef struct {
    size_t len;
    u_char *data;
} ngx_str_t;
```
src/core/ngx_string.h

# ngx_log_t

```c
struct ngx_open_file_s {
    ngx_fd_t fd;
    ngx_str_t name;

    void (*flush)(ngx_open_file_t *file, ngx_log_t *log);
    void *data;
};

struct ngx_log_s {
    ngx_uint_t log_level;
    ngx_open_file_t *file;

    ngx_atomic_uint_t connection;

    time_t disk_full_time;

    ngx_log_handler_pt handler;
    void *data;

    ngx_log_writer_pt writer;
    void *wdata;

    /*  
     * we declare "action" as "char *" because the actions are usually
     * the static strings and in the "u_char *" case we have to override
     * their types all the time
     */

    char *action;

    ngx_log_t *next;
};
```
src/core/ngx_log.h

<http://www.jb51.net/article/88557.htm>


# ngx_file_t

```c
typedef struct stat ngx_file_info_t;

struct ngx_file_s {
    ngx_fd_t fd;
    ngx_str_t name;
    ngx_file_info_t info;

    off_t offset;
    off_t sys_offset;

    ngx_log_t *log;

#if (NGX_THREADS || NGX_COMPAT)
    ngx_int_t (*thread_handler)(ngx_thread_task_t *task,
            ngx_file_t *file);
    void *thread_ctx;
    ngx_thread_task_t *thread_task;
#endif

#if (NGX_HAVE_FILE_AIO || NGX_COMPAT)    
    ngx_event_aio_t *aio;      
#endif

    unsigned valid_info:1;
    unsigned directio:1;
};
```
<http://blog.csdn.net/apelife/article/details/53043275>


# ngx_buf_t
```c
typedef void * ngx_buf_tag_t;
struct ngx_buf_s {
    u_char *pos;
    u_char *last;
    off_t file_pos;
    off_t file_last;

    u_char *start; /* start of buffer */
    u_char *end; /* end of buffer */
    ngx_buf_tag_t tag;
    ngx_file_t *file;
    ngx_buf_t *shadow;


    /* the buf's content could be changed */
    unsigned temporary:1;

    /*  
     * the buf's content is in a memory cache or in a read only memory
     * and must not be changed
     */
    unsigned memory:1;

    /* the buf's content is mmap()ed and must not be changed */
    unsigned mmap:1;

    unsigned recycled:1;
    unsigned in_file:1;
    unsigned flush:1;
    unsigned sync:1;
    unsigned last_buf:1;
    unsigned last_in_chain:1;

    unsigned last_shadow:1;
    unsigned temp_file:1;

    /* STUB */ int num;
};
```
<http://sofar.blog.51cto.com/353572/1327728>


# ngx_pool_t
```c
typedef struct {
    u_char *last;
    u_char *end;
    ngx_pool_t *next;
    ngx_uint_t failed;
} ngx_pool_data_t;

struct ngx_chain_s {
    ngx_buf_t *buf;
    ngx_chain_t *next;
};

struct ngx_pool_large_s {
    ngx_pool_large_t *next;
    void *alloc;
};

struct ngx_pool_cleanup_s {
    ngx_pool_cleanup_pt handler;
    void *data;
    ngx_pool_cleanup_t *next;
};

struct ngx_pool_s {
    ngx_pool_data_t d;
    size_t max;
    ngx_pool_t *current;
    ngx_chain_t *chain;
    ngx_pool_large_t *large;
    ngx_pool_cleanup_t *cleanup;
    ngx_log_t *log;
};
```

![](/images/posts/2016-10-07-nginx-data-struct.md/1.png)

<http://blog.csdn.net/chen19870707/article/details/41015613>


# ngx_list_t
```c
struct ngx_list_part_s {          
    void *elts;       
    ngx_uint_t nelts;
    ngx_list_part_t *next;       
};

typedef struct {
    ngx_list_part_t *last;
    ngx_list_part_t part;       
    size_t size;       
    ngx_uint_t nalloc;     
    ngx_pool_t *pool;
} ngx_list_t;
```

![](/images/posts/2016-10-07-nginx-data-struct.md/2.jpeg)

<http://blog.csdn.net/livelylittlefish/article/details/6599065>


# ngx_queue_t 
```c
struct ngx_queue_s {
    ngx_queue_t *prev;
    ngx_queue_t *next;
};

#define ngx_queue_data(q, type, link) \
    (type *) ((u_char *) q - offsetof(type, link))
```
src/core/ngx_queue.h


# ngx_array_t
```c
typedef struct {
    void *elts;
    ngx_uint_t nelts;
    size_t size;
    ngx_uint_t nalloc;
    ngx_pool_t *pool;
} ngx_array_t;
```


# ngx_rbtree_t
```c
struct ngx_rbtree_node_s {       
    ngx_rbtree_key_t key;  
    ngx_rbtree_node_t *left; 
    ngx_rbtree_node_t *right;
    ngx_rbtree_node_t *parent;
    u_char color;
    u_char data;  
};

typedef struct ngx_rbtree_s ngx_rbtree_t;

typedef void (*ngx_rbtree_insert_pt) (ngx_rbtree_node_t *root,
        ngx_rbtree_node_t *node, ngx_rbtree_node_t *sentinel);

struct ngx_rbtree_s {
    ngx_rbtree_node_t *root;  
    ngx_rbtree_node_t *sentinel;
    ngx_rbtree_insert_pt insert;
};
```
<http://blog.csdn.net/chen19870707/article/details/40515287>



