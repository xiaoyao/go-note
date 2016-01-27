# Web 会话管理
2016-01-19      七牛：沈达泱

(session manage)

## Cookie Only

## Browser Web Storage

## Server Session

客户端保存ID，服务端保存状态

1、In Memory

2、DataBase

3、Memory DataBase

## 过期策略



## 安全
Session

cookie



## OAuth 2.0


## 单点登录


***

## 1、session

session,中文经常翻译为会话,其本来的含义是指有始有终的一系列动作/消息,比如打电话是从拿起电话拨号到挂
断电话这中间的一系列过程可以称之为一个session。然而当session一词与网络协议相关联时,它又往往隐含了“面
向连接”和/或“保持状态”这样两个含义。

session在Web开发环境下的语义又有了新的扩展,它的含义是指一类用来在客户端与服务器端之间保持状态的解决方
案。有时候Session也用来指这种解决方案的存储结构。

session机制是一种服务器端的机制,服务器使用一种类似于散列表的结构(也可能就是使用散列表)来保存息。
但程序需要为某个客户端的请求创建一个session的时候,服务器首先检查这个客户端的请求里是否包含了一个
session标识-称为session id,如果已经包含一个session id则说明以前已经为此客户创建过session,服务器就按
照session id把这个session检索出来使用(如果检索不到,可能会新建一个,这种情况可能出现在服务端已经删除了
该用户对应的session对象,但用户人为地在请求的URL后面附加上一个JSESSION的参数)。如果客户请求不包含
session id,则为此客户创建一个session并且同时生成一个与此session相关联的session id,这个session id将在
本次响应中返回给客户端保存。

session机制本身并不复杂,然而其实现和配置上的灵活性却使得具体情况复杂多变。这也要求我们不能把仅仅某一
次的经验或者某一个浏览器,服务器的经验当作普遍适用的。
