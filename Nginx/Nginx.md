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

## nginx 启用 ssl
1. 下载openssl. 
https://wiki.openssl.org/index.php/Binaries
https://slproweb.com/products/Win32OpenSSL.html
2. 设置环境变量，路径为openssl安装目录下. set OPENSSL_CONF=C:\OpenSSL-Win64\bin\openssl.cfg 
3. 生成自签名证书

    >openssl genrsa -des3 -out server.key 1024
    
    输入密码（可自定义）：123456

    openssl req -new -key server.key -out server.csr

    输入国家：CN

    输入省份：GUANGDONG

    输入城市：SHENZHEN

    Org Name:TEST

    Org Unit Name:TEST

    Common Name:TEST

    Email:123456@mail.com

    challenge pwd:123456

    company name:TEST

    copy server.key server.key.org
    
    openssl rsa -in server.key.org -out server.key

    openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt

4. 把生成的server.key 和 server.crt copy到nginx/conf目录下
5. 修改Nginx.conf
    <pre>
    #HTTPS server		
    server {
        listen       443 ssl;
        server_name  localhost;

        ssl_certificate      server.crt;
        ssl_certificate_key  server.key.org;
        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;
        ssl_protocols  SSLv2 SSLv3 TLSv1;
        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;
        location / {
        proxy_pass http://myproject;
        }
    }
    </pre>