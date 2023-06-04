##### 命令行远程调用dubbo接口：
1. telnet ip port 连接Dubbo服务
2. ls -l 列出当前dubbo服务可以用的接口有那些
3. 使用invoke service.method(params)进行Dubbo接口调用 其中参数使用JSON格式化