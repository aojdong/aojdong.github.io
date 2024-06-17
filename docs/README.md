# 主页

## 访问 `GitHub` 的办法
尝试过很多办法，最后还是`FastGithub`稳定，网上搜索 `FastGithub` 即可。

对于该软件，原作者已删库。

参考链接：
1. https://zhuanlan.zhihu.com/p/432414619


## nginx/openresty 归档
### 安装
* nginx安装: https://blog.csdn.net/weixin_47661174/article/details/126415892
    - yum安装
    - 容器安装
    - 编译安装

* OpenResty安装
    - 官网：https://openresty.org/cn/
    - 菜鸟教程：https://www.runoob.com/w3cnote/openresty-intro.html

OpenResty(又称：ngx_openresty) 是一个基于 NGINX 的可伸缩的 Web 平台，由中国人章亦春发起，提供了很多高质量的第三方模块。
安装依赖库：
```
apt-get install libreadline-dev libpcre3-dev libssl-dev perl
```
或
```
yum install readline-devel pcre-devel openssl-devel
```

在官方下载最新的 OpenResty 源码包并解压编译安装。

Docker拉取：
```
docker pull openresty/openresty:buster
```

使用Dockerfile:
```
FROM openresty/openresty:buster
```

openresty是nginx的luaJit的扩展，openresty的启动、停止、启动操作，实际执行nginx的操作就可以了。
```
启动：nginx -c <configuration file>
快速停止nginx：nginx -s stop
完整有序的停止nginx：nginx -s quit
修改配置后重新加载生效：nginx -s reload
重新打开日志文件：nginx -s  reopen
```

### nginx.conf 配置文件
菜鸟教程：https://www.runoob.com/w3cnote/nginx-setup-intro.html
```yaml
... #全局块: 配置影响nginx全局的指令。一般有运行nginx服务器的用户组，nginx进程pid存放路径，日志存放路径，配置文件引入，允许生成worker process数等。
 
events {
   ... #events块: 配置影响nginx服务器或与用户的网络连接。有每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。
}
 
http
{
    ...   #http全局块: 可以嵌套多个server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。如文件引入，mime-type定义，日志自定义，是否使用sendfile传输文件，连接超时时间，单连接请求数等。
    server
    { 
        ...       #server全局块: 配置虚拟主机的相关参数，一个http中可以有多个server。
        location [PATTERN]  
        {
            ...  #location块: 配置请求的路由，以及各种页面的处理情况。
        }
        location [PATTERN]  #多个location
        {
            ...
        }
    }
    server  #多个server
    {
      ...
    }
    ...     #http全局块
}
```

#### 配置项解释：
https://blog.csdn.net/kenzo2017/article/details/105877846
```yaml
# 请求行+请求头的标准大小为1k
client_header_buffer_size 1k;
# 请求行+请求头的最大大小为32k
large_client_header_buffers 4 8k;
```

https://www.cnblogs.com/feixiangmanon/p/10129018.html
```yaml
#客户端请求服务器最大允许大小，在“Content-Length”请求头中指定。如果请求的正文数据大于client_max_body_size，HTTP协议会报错 413 Request Entity Too Large。就是说如果请求的正文大于client_max_body_size，一定是失败的。如果需要上传大文件，一定要修改该值。
client_max_body_size 10G;
#Nginx分配给请求数据的Buffer大小，如果请求的数据小于client_body_buffer_size直接将数据先在内存中存储。如果请求的值大于client_body_buffer_size小于client_max_body_size，就会将数据先存储到临时文件中; 临时文件是client_body_temp 指定的路径中，默认该路径值是/tmp/.
client_body_buffer_size 16k;
```

https://www.cnblogs.com/lemon-flm/p/8352194.html
```yaml
#指定等待client发送一个请求头的超时时间 秒
client_header_timeout 60
# 设置请求体（request body）的读超时时间
client_body_timeout 60
# 第一个参数指定了与client的keep-alive连接超时时间。服务器将会在这个时间后关闭连接。可选的第二个参数指定了在响应头Keep-Alive: timeout=time中的time值。这个头能够让一些浏览器主动关闭连接，这样服务器就不必要去关闭连接了。没有这个参数，nginx不会发送Keep-Alive响应头（尽管并不是由这个头来决定连接是否“keep-alive”）
keepalive_timeout 75 120
# lingering_close生效后，在关闭连接前，会检测是否有用户发送的数据到达服务器，如果超过lingering_timeout时间后还没有数据可读，就直接关闭连接；否则，必须在读取完连接缓冲区上的数据并丢弃掉后才会关闭连接
lingering_timeout 5
# 设置DNS解析超时时间
resolver_timeout 30
# 设置与upstream server的连接超时时间，有必要记住，不能超过75秒
proxy_connect_timeout 60
# 设置与代理服务器的读超时时间。它决定了nginx会等待多长时间来获得请求的响应。这个时间不是获得整个response的时间，而是两次reading操作的时间。
proxy_read_timeout 60
# 设置了发送请求给upstream服务器的超时时间。超时设置不是为了整个发送期间，而是在两次write操作期间。如果超时后，upstream没有收到新的数据，nginx会关闭连接。
proxy_send_timeout 60
```

### lua注意事项
http://www.hangdaowangluo.com/archives/2674
- 变量申明后，默认的值是nil；将 nil 赋给变量后，相当于删除变量。注意nil 和 ngx.null的区别。
- 在 lua中只有 false和 nil 值为 false ,其他值都为 true ，包括0,””等
- lua中的数字（一切数字）都是 number类型。
- table 的下标从 1 开始。
- 逻辑运算符是 and 、or 、not
- 字符串的连接可使用string.format ，或者 table.concat，普通..连接消耗性能比较大（每次连接生成新的更大字符串）
- lua中没有 switch 的语法，但可想办法使用 table 实现。
- lua循环语法中没有continue，但有break
- return只能出现在语句块的最后。或者使用 do return end
- 养成定义局部变量的习惯（local）
- Lua 内部实际采用哈希表和数组，分别保存键值对、普通值，所以不推荐混合使用这两种赋值方式。
- 数组中不要使用 nil,如果删除数组某一key采用 table.remove
- 调用代码前先定义函数
- Lua module 只会在第一次请求时加载一次（除非显式禁用了 lua_code_cache 配置指令）

### ngx_lua模块
http://www.hangdaowangluo.com/archives/2686 
![流程](http://www.hangdaowangluo.com/wp-content/uploads/2017/11/77d1c09e-1a37-11e6-97ef-d9767035fc3e.png)

#### 常量
http://www.hangdaowangluo.com/archives/2692 
```yaml
ngx.HTTP_CONTINUE               = 100
ngx.HTTP_SWITCHING_PROTOCOLS    = 101
ngx.HTTP_OK                     = 200
ngx.HTTP_CREATED                = 201
ngx.HTTP_ACCEPTED               = 202
ngx.HTTP_NO_CONTENT             = 204
ngx.HTTP_PARTIAL_CONTENT        = 206
ngx.HTTP_SPECIAL_RESPONSE       = 300
ngx.HTTP_MOVED_PERMANENTLY      = 301
ngx.HTTP_MOVED_TEMPORARILY      = 302
ngx.HTTP_SEE_OTHER              = 303
ngx.HTTP_NOT_MODIFIED           = 304
ngx.HTTP_TEMPORARY_REDIRECT     = 307
ngx.HTTP_BAD_REQUEST            = 400
ngx.HTTP_UNAUTHORIZED           = 401
ngx.HTTP_PAYMENT_REQUIRED       = 402
ngx.HTTP_FORBIDDEN              = 403
ngx.HTTP_NOT_FOUND              = 404
ngx.HTTP_NOT_ALLOWED            = 405
ngx.HTTP_NOT_ACCEPTABLE         = 406
ngx.HTTP_REQUEST_TIMEOUT        = 408
ngx.HTTP_CONFLICT               = 409
ngx.HTTP_GONE                   = 410
ngx.HTTP_UPGRADE_REQUIRED       = 426
ngx.HTTP_TOO_MANY_REQUESTS      = 429
ngx.HTTP_CLOSE                  = 444
ngx.HTTP_ILLEGAL                = 451
ngx.HTTP_INTERNAL_SERVER_ERROR  = 500
ngx.HTTP_METHOD_NOT_IMPLEMENTED = 501
ngx.HTTP_BAD_GATEWAY            = 502
ngx.HTTP_SERVICE_UNAVAILABLE    = 503
ngx.HTTP_GATEWAY_TIMEOUT        = 504
ngx.HTTP_VERSION_NOT_SUPPORTED  = 505
ngx.HTTP_INSUFFICIENT_STORAGE   = 507
```

#### api：
- 官网：https://github.com/openresty/lua-nginx-module#ngxexit
- 博客：http://www.hangdaowangluo.com/archives/2694
