#nginx源码分析

###概述
nginx是一个开源的高性能Web服务器系统，事件驱动、多进程、异步非阻塞、模块化是其主要的特性。源代码层面，有很多自造的“轮子”，作者自己实现了诸如内存池、缓冲区、字符串、链表、红黑树等经典数据结构。全部代码仅有10万行。代码之间耦合的很厉害，很难抽出其中的一部分单独来看。

### 架构设计
nginx启动后，会有一个master进程和多个worker进程。master进程主要用来管理worker进程，包含：接受来自己外界的信号，向各个worker进程发送信号，监控worker进程的运行状态，自动重新启动新的worker进程。基本的网络事件则由worker进程处理。多个worker进程之间是对等的，共同竞争来自客户端的请求，一个请求只能在一个worker进程中处理。worker进程的数量是可以设置的，一般会设置与机器cpu核数一致。

**nginx的进程模型**，如下图所示：

![](img/ngx_thread_model.png "")

**事件模型**  

* 网络事件： 异步非阻塞
* 信号
* 定时器： epoll_wait函数的超时时间设置，定时器时间放在红黑树中

### 目录结构
	
* auto
  
  检查编译器版本、检查操作系统版本、检查标准库版本、检查模块依赖情况、安装、初始化、多线程检查  
  configure为此目录下的总驱动，调用脚本生成：
  * 版本信息头文件: ngx_auto_config.h ngx_auto_headers.h
  * 默认被包含的模块的声明代码: objs/ngx_modules.c
  * Makefile文件 	

* src 

  源码存放目录
  * core: 主干部分、基础数据结构和基础设施的源码
  * event:事件驱动模型和相关模块的源码
  * http:http server相关代码
  * mail:邮件代理和相关模块
  * misc: C++兼容性测试和google perftools模块
  * os: 依赖于操作系统实现的源码，master和worker创建 
  
* conf

  nginx相关的配置文件
  
### 基础概念

### 基本数据结构 

nginx实现中有很多结构体，一般命名为ngx_XXX_t。这些结构体分散在许多头文件中。src/core/ngx_core.h中把几乎所有的头文件都集合起来。也因此造成了nginx各部分源代码的耦合。但实际上nginx各个部分逻辑划分还是很明确的，整体上是一种松散的结构。

* ngx_str_t

		typedef struct{
			size_t len;
			u_char *data;
		}ngx_str_t;

* ngx_pool_t

		struct ngx_pool_s {

	
* ngx_hash_t
	* ngx_hash_t不像其他的hash表的实现，可以插入删除元素，只能一次初始化。
	* 解决冲突使用的是开链法，但实际上是开了一段连续的存储空间，和数组差不多。
	
			typedef struct {











### 配置系统

一个主配置文件+其他辅助的配置文件，nginx/conf。主配置文件nginx.conf包含若干配置项，每个配置项由配置指令和指令参数2各部分构成。  

指令上下文：

* main: 运行时与具体业务无关的一些参数。

	* user
	* worker_processes
	* error_log
	* events
	* http
	* mail

* http: 与提供http服务相关的一些配置参数。

	* server

* server：http服务支持若干虚拟主机。每个虚拟主机一个对应的Server配置。

	* listen
	* server_name
	* access_log
	* location
	* protocol
	* proxy
	* smtp_auth
	* xclient 

* location：http服务中，某些特定的url对应的一系列配置项。

	* index
	* root

* mail：实现Email相关的smtp/imap/pop3代理时，共享的一些配置项。

	* server
	* auth_http
	* imap_capabilities
	

### 模块化
nginx的内部结构是由核心部分和一系列的功能模块所组成的。其中核心除了web服务器的基础功能外，还包括web服务反向代理、email服务反向代理等功能。同时，nginx core实现了底层的通讯协议，为其他模块和nginx进程构建了基本的运行时环境，构建了高其他各模块的协作基础。
nginx是高度模块化的，除了核心部分的相关功能都是在其他模块中实现的。nginx将

### 启动过程

### 模块化

### 核心模块