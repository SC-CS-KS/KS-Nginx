# Nginx 原理

# 工作模式

* 单工作进程模式（默认）
除主进程外，还有一个工作进程，且工作进程是单线程的

* 多工作进程模	
每个工作进程包含多工作线程

## nginx事件处理
nginx需要将所有关心的fd注册到epoll
static ngx_int_t ngx_epoll_add_event(ngx_event_t *ev, ngx_int_t event, ngx_uint_t flags);
	ngx_event_t
		结构体指针，代表关心的一个读或者写事件
		nginx为事件可能会设置一个超时定时器，从而能够处理事件超时情况
		struct ngx_event_s {
    ngx_event_handler_pt  handler; //函数指针：事件的处理函数
    ngx_rbtree_node_t   timer;     //超时定时器，存储在红黑树中（节点的key即为事件的超时时间）
    unsigned         timedout:1;   //记录事件是否超时
};
	epoll_wait
		监听所有fd，处理发生的读写事件
		epoll_wait是阻塞调用
			最后一个参数timeout是超时时间，即最多阻塞timeout时间如果还是没有事件发生，方法会返回
	timeout
		从上面说的记录超时定时器的红黑树中查找最近要到时的节点
		以此作为epoll_wait的超时时间
			ngx_msec_t ngx_event_find_timer(void)
{
    node = ngx_rbtree_min(root, sentinel);
    timer = (ngx_msec_int_t) (node->key - ngx_current_msec);
 
    return (ngx_msec_t) (timer > 0 ? timer : 0);
}
		同时nginx在每次循环的最后
			会从红黑树中查看是否有事件已经过期
			如果过期，标记timeout=1，并调用事件的handler
			void ngx_event_expire_timers(void)
{
    for ( ;; ) {
        node = ngx_rbtree_min(root, sentinel);
 
        if ((ngx_msec_int_t) (node->key - ngx_current_msec) <= 0) {  //当前事件已经超时
            ev = (ngx_event_t *) ((char *) node - offsetof(ngx_event_t, timer));
 
            ev->timedout = 1;
 
            ev->handler(ev);
 
            continue;
        }
 
        break;
    }
}
nginx就是通过上面的方法实现了socket事件的处理，定时事件的处理