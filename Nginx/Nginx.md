# Nginx快速入门
## 安装&启动
1. [下载程序包](http://nginx.org/en/download.html)
2. 解压程序包
3. 编辑conf文件夹下nginx.conf
    <pre>
    #定义负载均衡
    upstream myproject{
        #max_fails=2; 当这台机器请求两次失败后就在 [fail_timeout] 定义的时间内不再分发请求给它
        #fail_timeout=120s
        server 127.0.0.1:8081 max_fails=2 fail_timeout=120s;
        server 127.0.0.1:8082 max_fails=2 fail_timeout=120s;
    }
    #定义负载均衡2
    upstream admineproject{
        server 127.0.0.1:8083;
    }
    
    server {
        listen       8888;
        server_name  localhost;
        #Url匹配
        location / {
            proxy_pass http://myproject;
        }
        #Url匹配2
        location /admin {
            proxy_pass http://admineproject;
        }
    }
    </pre>
4. 启动
    <pre>
    nginx.exe -c conf\nginx.conf    
    </pre>
