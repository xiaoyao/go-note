# GoWeb

## web工作方式的几个概念

以下均是服务器端的几个概念

Request：用户请求的信息，用来解析用户的请求信息，包括post、get、cookie、url等信息

Response：服务器需要反馈给客户端的信息

Conn：用户的每次请求链接

Handler：处理请求和生成返回信息的处理逻辑



## 分析http包运行机制

1. 创建Listen Socket, 监听指定的端口, 等待客户端请求到来。

2. Listen Socket接受客户端的请求, 得到Client Socket, 接下来通过Client Socket与客户端通信。

3. 处理客户端的请求, 首先从Client Socket读取HTTP请求的协议头, 如果是POST方法, 还可能要读取客户端提交的数据, 然后交给相应的handler处理请求, handler处理完毕准备好客户端需要的数据, 通过Client Socket写给客户端。

这整个的过程里面我们只要了解清楚下面三个问题，也就知道Go是如何让Web运行起来了

    - 如何监听端口？
    - 如何接收客户端请求？
    - 如何分配handler？

## Go代码的执行流程 
通过对http包的分析之后,现在让我们来梳理一下整个的代码执行过程。

### 1、首先调用Http.HandleFunc
按顺序做了几件事:

1 调用了DefaultServerMux的HandleFunc

2 调用了DefaultServerMux的Handle

3 往DefaultServeMux的map[string]muxEntry中增加对应的handler和路由规则
### 2、其次调用http.ListenAndServe(":9090", nil)
按顺序做了几件事情:

1 实例化Server

2 调用Server的ListenAndServe()

3 调用net.Listen("tcp", addr)监听端口

4 启动一个for循环,在循环体中Accept请求

5 对每个请求实例化一个Conn,并且开启一个goroutine为这个请求进行服务go c.serve()

6 读取每个请求的内容w, err := c.readRequest()

7 判断handler是否为空,如果没有设置handler(这个例子就没有设置handler),handler就设置为 DefaultServeMux

8 调用handler的ServeHttp

9 在这个例子中,下面就进入到DefaultServerMux.ServeHttp

10 根据request选择handler,并且进入到这个handler的ServeHTTP mux.handler(r).ServeHTTP(w, r)

11 选择handler:

    A 判断是否有路由能满足这个request(循环遍历ServerMux的muxEntry)

    B 如果有路由满足,调用这个路由handler的ServeHttp

    C 如果没有路由满足,调用NotFoundHandler的ServeHttp
