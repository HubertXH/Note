### 官网地址：https://nginx.org/en/

1. main 全局块：配置影响nginx全局的指令。一般有运行nginx服务器的用户组，nginx进程pid存放路径，日志存放路径，配置文件引入，允许生成worker process数等。
2. events块：配置影响nginx服务器或与用户的网络连接。有每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。
3. http块：可以嵌套多个server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。如文件引入，mime-type定义，日志自定义，是否使用sendfile传输文件，连接超时时间，单连接请求数等。
4. server块：配置虚拟主机的相关参数，一个http中可以有多个server。
5. location块：配置请求的路由，以及各种页面的处理情况。


```
user administrator administrators
worker_processes 2
pid /nginx/pid/nginx.pid;
error_log log/error.log debug
```
- user administrator administrators 配置用户或者组，默认为nobody nobody。
- worker_processes 2;  允许生成的进程数，默认为1,建议设置为CPU的核数，在1.2.5和1.3.8版本开始可以配置为 "auto"参数，NG会自动进行配置
- pid /nginx/pid/nginx.pid; 指定nginx进程运行文件存放地址
- error_log log/error.log debug;制定日志路径，级别。这个设置可以放入全局块，http块，server块，级别以此为：debug|info|notice|warn|error|crit|alert|emerg

```
events {
    accept_mutex on;
    multi_accept on;
    #use epoll;
    worker_connections  1024;  
}
```
1. accept_mutex 置网路连接序列化，防止[惊群现象](https://wenfh2020.com/2021/09/29/nginx-thundering-herd/)发生，默认为OFF。
   - 如果值为On时当新的请求到达时NG会以串行的方式来处理请求，即唤醒一个子进程来处理请求，其余子进程是处理睡眠状态。
   - 如果值为OFF则当新的请求进来的时主进程会唤醒所有子进程，但是只能有一个子进程获取到新的链接，其余进程又会新进入到睡眠状态。
   - NG默认为关闭状态，主要是因为NG设置 worker_processes 的数值一般都不是很大，只有几十个，相对而言代价较小。且如果服务的并发量较高,accept_mutex开启的情况下会影响系统的吞吐量。
2. multi_accept 设置一个进程是否同时接受多个网络连接，默认为off
3. use epoll 事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport, 在不同的平台NG会自动选择最优的驱动模型
4. worker_connections 工作进程可以同时打开的最大连接数，默认为512,包括所有连接（例如，与代理服务器的连接等），而不仅仅是与客户端的连接。另一个注意事项是，实际同时连接数不能超过最大打开文件数的当前限制，可以通过worker_rlimit_nofile更改。
```
http {
    include       mime.types;   #文件扩展名与文件类型映射表
    default_type  application/octet-stream; #默认文件类型，默认为text/plain
    #access_log off; #取消服务日志  
    log_format myFormat '$remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for'; #自定义格式
    access_log log/access.log myFormat;  #combined为日志格式的默认值
    sendfile on;   #允许sendfile方式传输文件，默认为off，可以在http块，server块，location块。
    sendfile_max_chunk 100k;  #每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
    keepalive_timeout 65;  #连接超时时间，默认为75s，可以在http，server，location块。

    upstream backend {
        # round_robin,least_conn,ip_hash,hash $request_uri consistent,random 
        server backend1.example.com weight=5;
        server backend2.example.com slow_start=30s;
        server backend3.example.com down;
        server 192.0.0.1 backup;  
    }
    error_page 404 https://www.baidu.com; #错误页
    server {
        keepalive_requests 120; #单连接请求上限次数。
        listen       4545;   #监听端口
        server_name  127.0.0.1;   #监听地址     
        location  ~*^.+$ {       #请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。
           #root path;  #根目录
           #index vv.txt;  #设置默认页
           proxy_pass  http://mysvr;  #请求转向mysvr 定义的服务器列表
           deny 127.0.0.1;  #拒绝的ip
           allow 172.18.5.54; #允许的ip         
        } 
    }
}

```
- keepalive_requests： 在长连接时一个链接建立后处理的最大请求数量，当链接建立后，NG对这个链接会有一个计数器，每处理一次请求计数器+1，当计数器的值超过该值后NG会强制性断开链接，强制客户端及建立新的链接。
官方文档不建议该值配置的很大理由是:周期性的断开链接可以释放链接占用的内存空间，如果该值过大会导致占用超额的内存。但是在实际中如果该值较小且应用的请求量较大会造成链接的频繁建立和断开会
产生大量的TIME_WAIT该配置为多少需要根据实际情况来决定。

##### Rewrite规则

rewrite功能就是，使用nginx提供的全局变量或自己设置的变量，结合正则表达式和标志位实现url重写以及重定向。rewrite只能放在server{},location{},if{}中，并且只能对域名后边的除去传递的参数外的字符串起作用

如果相对域名或参数字符串起作用，可以使用全局变量匹配，也可以使用proxy_pass反向代理。

表明看rewrite和location功能有点像，都能实现跳转，主要区别在于rewrite是在同一域名内更改获取资源的路径，而location是对一类路径做控制访问或反向代理，可以proxy_pass到其他机器。很多情况下rewrite也会写在location里，它们的执行顺序是：

- 执行server块的rewrite指令
- 执行location匹配
- 执行选定的location中的rewrite指令

> 如果其中某步URI被重写，则重新循环执行1-3，直到找到真实存在的文件；循环超过10次，则返回500 Internal Server Error错误。

###### flag标志位

- last : 停止处理当前匹配模块，开始重新进行匹配。相当于Apache的[L]标记，表示完成rewrite
- break : 停止执行当前虚拟主机的后续rewrite指令集
- redirect : 返回302临时重定向，地址栏会显示跳转后的地址
- permanent : 返回301永久重定向，地址栏会显示跳转后的地址

> 因为301和302不能简单的只返回状态码，还必须有重定向的URL，这就是return指令无法返回301,302的原因了。这里 last 和 break 区别有点难以理解：

- last一般写在server和if中，而break一般使用在location中
- last不终止重写后的url匹配，即新的url会再从server走一遍匹配流程，而break终止重写后的匹配
- break和last都能组织继续执行后面的rewrite指令
