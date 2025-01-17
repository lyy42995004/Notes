# main.cc

数据库信息

命令行解析

**调用**server初始化，日志，数据库，线程池，触发模式，监听，运行

# config.h

定义config类，ParseArgument()函数解析命令行参数

```C++
int port; 					// 端口号
int log_write;				// 日志写入方式
int close_log;				// 是否关闭日志
int trigger_mode; 			// 事件的触发模式，ET与LT
int listen_trigger_mode; 	// 监听套接字的触发模式
int connect_trigger_mode;	// 连接套接字的触发模式
int option_linger;			// 优雅关闭连接
int sql_num;				// 数据库连接池数量
int thread_num;				// 连接池线程数量
int actor_mode;				// 并发类型选择
```

# config.cc

congfig默认构造

ParseArgument调用getopt()解析命令行参数

p-port	l-log_write	m-trigger_mode	o-option_linger	s-sql_num	t-thread_num

c-close_log	a-actor_mode

# webserver.h