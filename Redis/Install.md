## Install Redis 5.0
1. 安装依赖工具
    <pre>
    yum install gcc
    yum install gcc-c++
    yum install tcl
    </pre>
2. 安装redis
    <pre>
    wget http://download.redis.io/releases/redis-5.0.0.tar.gz
    tar xzf redis-5.0.0.tar.gz
    cd redis-5.0.0
    make
    make test
    </pre>
3. 启动-简单模式
    <pre>
    src/redis-server
    </pre>
4. 启动-带配置
    <pre>
    src/redis-server redis.conf
    </pre>
5. 开启防火墙
    <pre>
    firewall-cmd --permanent --zone=public --add-port=6379/tcp //永久
    firewall-cmd --zone=public --add-port=6379/tcp //临时
    </pre>
6. 设置系统开机自启动

编辑/etc/rc.d/rc.local,在文件后面加上如下这行
<pre>
/opt/redis/redis-5.0.0/src/redis-server /opt/redis/redis-5.0.0/redis.conf
</pre>