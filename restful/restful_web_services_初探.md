# RESTful Web Services初探

近几年，RESTful Web Services渐渐开始流行，大量用于解决异构系统间的通信问题。很多网站和应用提供的API，都是基于RESTful风格的Web Services，比较著名的包括Twitter、Google以及项目管理工具Redmine。本文将简要介绍RESTful Web Service，希望能对读者有所帮助。

## 一、 RESTful Web Services是什么

REST的全称是Representation State Transfer，它描述了一种设计Web应用的架构风格，它是一组架构约束条件和原则，满足这些约束条件和原则的应用程序或设计就是 RESTful风格的。而符合RESTful风格的Web Services，就是我们所说的RESTful Web Services。REST原则如下：

### 1、资源由URI来指定

在Web应用中，所有的事物都应该拥有唯一的ID，代表ID的统一概念是：URI。URI构成了一个全局命名空间，使用URI标识你的关键资源意味着它们获得了一个唯一、全局的ID。



### 2、显式的使用HTTP方法

REST 要求开发人员显式地使用 HTTP 方法，并且使用方式与协议定义一致。 这个基本 REST 设计原则建立了创建、读取、更新和删除（create, read, update, and delete，CRUD）操作与 HTTP 方法之间的一对一映射。 根据此映射：

若要在服务器上创建资源，应该使用 POST 方法。
若要检索某个资源，应该使用 GET 方法。
若要更改资源状态或对其进行更新，应该使用 PUT 方法。
若要删除某个资源，应该使用 DELETE 方法。

### 3、资源的多重表述

针对不同的需求提供资源多重表述。这里所说的多重表述包括XML、JSON、HTML等。即服务器端需要向外部提供多种格式的资源表述，供不同的客户端使用。比如移动应用可以使用XML或JSON和服务器端通信，而浏览器则能够理解HTML。

### 4、无状态

对服务器端的请求应该是无状态的，完整、独立的请求不要求服务器在处理请求时检索任何类型的应用程序上下文或状态。无状态约束使服务器的变化对客户 端是不可见的，因为在两次连续的请求中，客户端并不依赖于同一台服务器。一个客户端从某台服务器上收到一份包含链接的文档，当它要做一些处理时，这台服务 器宕掉了，可能是硬盘坏掉而被拿去修理，可能是软件需要升级重启——如果这个客户端访问了从这台服务器接收的链接，它不会察觉到后台的服务器已经改变了。

## 二、 RESTful Web Services与基于SOAP的Web Services的比较

通过前文，读者应该大致了解了RESTful风格的Web Services，也有了一点深入了解的兴趣。但读者或许会问，基于SOAP的Web Services也是解决异构系统间通信问题的常用方案，那么，RESTful Web Services相对于基于SOAP 的Web Services，有什么优势呢？或者说，我们为什么要开始学习RESTful Web Services，使用已经流行很久的基于SOAP的Web Services不就好了么？

### RESTful web Services接口更易于使用

RESTful Web Services使用标准的 HTTP 方法 (GET/PUT/POST/DELETE) 来抽象所有 Web 系统的服务能力，而不同的是，SOAP 应用都通过定义自己个性化的接口方法来抽象 Web 服务。相对来说，RESTful Web Services接口更简单。

RESTful Web Services使用标准的 HTTP 方法的优势，从大的方面来讲：标准化的 HTTP 操作方法，结合其他的标准化技术，如 URI，HTML，XML 等，将会极大提高系统与系统之间整合的互操作能力。尤其在 Web 应用领域，RESTful Web Services所表达的这种抽象能力更加贴近 Web 本身的工作方式，也更加自然。

同时，使用标准 HTTP 方法实现的 RESTful Web Services也带来了 HTTP 方法本身的一些优势：

无状态性
HTTP 协议从本质上说是一种无状态的协议，客户端发出的 HTTP 请求之间可以相互隔离，不存在相互的状态依赖。基于 HTTP 的 ROA，以非常自然的方式来实现无状态服务请求处理逻辑。对于分布式的应用而言，任意给定的两个服务请求 Request 1 与 Request 2，由于它们之间并没有相互之间的状态依赖，就不需要对它们进行相互协作处理，其结果是：Request 1 与 Request 2 可以在任何的服务器上执行，这样的应用很容易在服务器端支持负载平衡 (load-balance)。

安全操作与幂指相等特性
HTTP 的 GET、HEAD 请求本质上应该是安全的调用，即：GET、HEAD 调用不会有任何的副作用，不会造成服务器端状态的改变。对于服务器来说，客户端对某一 URI 做 n 次的 GET、HAED 调用，其状态与没有做调用是一样的，不会发生任何的改变。

HTTP 的 PUT、DELTE 调用，具有幂指相等特性 , 即：客户端对某一 URI 做 n 次的 PUT、DELTE 调用，其效果与做一次的调用是一样的。HTTP 的 GET、HEAD 方法也具有幂指相等特性。

HTTP 这些标准方法在原则上保证你的分布式系统具有这些特性，以帮助构建更加健壮的分布式系统。

### RESTful Web Services更容易实现缓存

众所周知，对于基于网络的分布式应用，网络传输是一个影响应用性能的重要因素。如何使用缓存来节省网络传输带来的开销，这是每一个构建分布式网络应用的开发人员必须考虑的问题。

HTTP 协议带条件的 HTTP GET 请求 (Conditional GET) 被设计用来节省客户端与服务器之间网络传输带来的开销，这也给客户端实现 Cache 机制 ( 包括在客户端与服务器之间的任何代理 ) 提供了可能。HTTP 协议通过 HTTP HEADER 域：If-Modified-Since/Last- Modified，If-None-Match/ETag 实现带条件的 GET 请求。

REST 的应用可以充分地挖掘 HTTP 协议对缓存支持的能力。当客户端第一次发送 HTTP GET 请求给服务器获得内容后，该内容可能被缓存服务器 (Cache Server) 缓存。当下一次客户端请求同样的资源时，缓存可以直接给出响应，而不需要请求远程的服务器获得。而这一切对客户端来说都是透明的。

## 三、 总结

在日常应用中，我们有大量的场合可以使用到RESTful Web Services，包括Web系统间的交互，移动客户端与Web服务器端的通信等。只有在日常工作中更多的实践RESTful，才能更好的理解RESTful Web Services。

本文仅仅是对RESTful Web Services的一个粗略介绍，希望能帮助读者初步的了解这一领域，如果需要更深入的了解，还需要读者从其他方面阅读更多的信息并在实践中大量应用。

转自：http://express.ruanko.com/ruanko-express_37/technologyexchange6.html