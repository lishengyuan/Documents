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