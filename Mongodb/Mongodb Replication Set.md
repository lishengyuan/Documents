# Mongodb 副本集 Replication Set

`Mongodb 官方不推荐使用master-slaver模式，并且Master-slaver的master 宕机后，slaver不会制动切换成master.ReplSet 官方建议奇数个实例，最少需要3个实例`
1. 新建实例的db log pid目录
   mkdir /usr/local/mongodb/data/db1
   mkdir /usr/local/mongodb/data/db2
   mkdir /usr/local/mongodb/data/db3

   mkdir /usr/local/mongodb/log

   mkdir /usr/local/mongodb/pid
2. 新建3个配置文件
   touch mongodb1.conf mongodb2.conf mongodb3.conf

   mongodb1.conf

   <pre>
    dbpath=/usr/local/mongodb/data/db1
    port=29031
    fork=true
    logappend=true
    pidfilepath=/usr/local/mongodb/log/mongo1.pid
    logpath=/usr/local/mongodb/pid/mongodb1.log
    nohttpinterface=true
    # 副本集名称
    replSet=rs1
   </pre>

    mongodb2.conf
   
   <pre>
    dbpath=/usr/local/mongodb/data/db2
    port=29032
    fork=true
    logappend=true
    pidfilepath=/usr/local/mongodb/log/mongo2.pid
    logpath=/usr/local/mongodb/pid/mongodb2.log
    nohttpinterface=true
    # 副本集名称
    replSet=rs1
   </pre>

    mongodb3.conf
   
   <pre>
    dbpath=/usr/local/mongodb/data/db3
    port=29033
    fork=true
    logappend=true
    pidfilepath=/usr/local/mongodb/log/mongo3.pid
    logpath=/usr/local/mongodb/pid/mongodb3.log
    nohttpinterface=true
    # 副本集名称
    replSet=rs1
   </pre>

3. 分别启动三个节点
   <pre>
    ./mongod -f mongodb1.conf     
    ./mongod -f mongodb2.conf     
    ./mongod -f mongodb3.conf 
   </pre>
4. 查看ReplSet状态
   随便登录一个实例

   <pre>
   ./mongo -port=29031

   show dbs 
   </pre>
    这个时候回抛异常，因为没有初始化ReplSet
    <pre>
    config= {
        _id:"rs1",
        members:[ 
            {_id:0,host:"127.0.0.1:29031"}, 
            {_id:1,host:"127.0.0.1:29032"}, 
            {_id:2,host:"127.0.0.1:29033"}
        ] 
    }
    rs.initiate(config)
    </pre>
    上面的`_id:"rs1"`需要与之前3个配置文件设置的replSet的名称一致

    查看集群状态
    <pre>
     rs.status() 
     { 
        "set" : "rs1", 
        "date" : ISODate("2017-11-14T00:49:25.782Z"), 
        "myState" : 1, 
        "members" : [ 
            { 
            "_id" : 0, 
            "name" : "127.0.0.1:29031", 
            "health" : 1, 
            "state" : 1, 
            "stateStr" : "PRIMARY", 
            "uptime" : 695, 
            "optime" : Timestamp(1510620540, 1), 
            "optimeDate" : ISODate("2017-11-14T00:49:00Z"), 
            "electionTime" : Timestamp(1510620544, 1), 
            "electionDate" : ISODate("2017-11-14T00:49:04Z"), 
            "configVersion" : 1, 
            "self" : true
            }, 
            { 
            "_id" : 1, 
            "name" : "127.0.0.1:29032", 
            "health" : 1, 
            "state" : 2, 
            "stateStr" : "SECONDARY", 
            "uptime" : 25, 
            "optime" : Timestamp(1510620540, 1), 
            "optimeDate" : ISODate("2017-11-14T00:49:00Z"), 
            "lastHeartbeat" : ISODate("2017-11-14T00:49:24.739Z"), 
            "lastHeartbeatRecv" : ISODate("2017-11-14T00:49:24.762Z"), 
            "pingMs" : 0, 
            "configVersion" : 1 
            }, 
            { 
            "_id" : 2, 
            "name" : "127.0.0.1:29033", 
            "health" : 1, 
            "state" : 2, 
            "stateStr" : "SECONDARY", 
            "uptime" : 25, 
            "optime" : Timestamp(1510620540, 1), 
            "optimeDate" : ISODate("2017-11-14T00:49:00Z"), 
            "lastHeartbeat" : ISODate("2017-11-14T00:49:24.739Z"), 
            "lastHeartbeatRecv" : ISODate("2017-11-14T00:49:24.762Z"), 
            "pingMs" : 0, 
            "configVersion" : 1 
            } 
        ], 
        "ok" : 1 
    }
    </pre>
    整个副本及已经搭建成功了！
5. 测试副本集数据复制功能
   
   在主节点登录mongodb
   <pre>
    ./mongo -port=29031 
    use test
    db.testdb.insert({"key":"this test item"}) 
    exit
   </pre>
   在副本节点登录查看数据：
   <pre>
    ./mongo -port=290312
    use test
    db.getMongo().setSlaveOk() #mongodb默认是从主节点读写数据的，副本节点上不允许读，需要设置副本节点可以读 
    db.testdb.find() 
    { 
        "_id" : ObjectId("5a0a3e8d40637405ab003b39"),
         "key" : "this test item" 
    }
   </pre>

6. 测试副本集Master故障后自动选举新Master (failover功能)
   查看集群状态
   <pre>
    rs.status()
   </pre>
   找到PRIMARY，把现在的主节点停掉，查看集群状态
   <pre>
    ./mongod --shutdown -f mongodb1.conf
    rs.status()
    { 
        "set" : "repset", 
        "date" : ISODate("2017-11-14T00:57:14.753Z"), 
        "myState" : 2, 
        "members" : [ 
            { 
            "_id" : 0, 
            "name" : "127.0.0.1:29031", 
            "health" : 0, 
            "state" : 8, 
            "stateStr" : "(not reachable/healthy)", 
            "uptime" : 0, 
            "optime" : Timestamp(0, 0), 
            "optimeDate" : ISODate("1970-01-01T00:00:00Z"), 
            "lastHeartbeat" : ISODate("2017-11-14T00:57:13.304Z"), 
            "lastHeartbeatRecv" : ISODate("2017-11-14T00:57:09.285Z"), 
            "pingMs" : 0, 
            "lastHeartbeatMessage" : "Failed attempt to connect to 192.168.221.161:27017; couldn't connect to server 192.168.221.161:27017 (192.168.221.161), connection attempt failed", 
            "configVersion" : -1 
            }, 
            { 
            "_id" : 1, 
            "name" : "127.0.0.1:29032", 
            "health" : 1, 
            "state" : 2, 
            "stateStr" : "SECONDARY", 
            "uptime" : 1167, 
            "optime" : Timestamp(1510620813, 2), 
            "optimeDate" : ISODate("2017-11-14T00:53:33Z"), 
            "infoMessage" : "could not find member to sync from", 
            "configVersion" : 1, 
            "self" : true
            }, 
            { 
            "_id" : 2, 
            "name" : "127.0.0.1:29033", 
            "health" : 1, 
            "state" : 1, 
            "stateStr" : "PRIMARY", 
            "uptime" : 17, 
            "optime" : Timestamp(1510620813, 2), 
            "optimeDate" : ISODate("2017-11-14T00:53:33Z"), 
            "lastHeartbeat" : ISODate("2017-11-14T00:57:13.317Z"), 
            "lastHeartbeatRecv" : ISODate("2017-11-14T00:57:13.330Z"), 
            "pingMs" : 0, 
            "electionTime" : Timestamp(1510621032, 1), 
            "electionDate" : ISODate("2017-11-14T00:57:12Z"), 
            "configVersion" : 1 
            } 
        ], 
        "ok" : 1 
    }
   </pre>
   可以看到29033变成PRIMARY

Replication Set 就完成了