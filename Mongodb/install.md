# 在Liunx Server上安装MongoDB
## 目录

>1. [安装连接Linux server的工具](#安装连接Linux-server的工具)
>2. [下载Mongodb for Linux server安装包](#下载Mongodb-for-Linux-server安装包)
>3. [把mongodb安装包从本地传输到服务器](#把mongodb安装包从本地传输到服务器)
>4. [安装](#安装MongoDB)
>5. [数据备份](#MongoDB数据备份)
>6. [启用身份认证](#启用身份认证)

## 安装连接Linux server的工具
1. mRemoteNG.exe
2. PuTTY
3. SecurityCRT

## 下载Mongodb for Linux server安装包
到官网下载Linux安装包 [Download Link](https://www.mongodb.com/download-center#previous)

> 例如 mongodb-linux-x86_64-rhel70-2.6.7.tgz

## 把mongodb安装包从本地传输到服务器

* 打开CMD命令行,cd 到putty的安装目录

	<pre>
	cd C:\Program Files\PuTTY
	pscp C:\MongoDBTool\mongodb-linux-x86_64-rhel70-2.6.7.tgz lisheng@oac-lxmdcmdb204:/home/lisheng
	</pre>

回车后输入Linux服务器的登录密码
## 安装MongoDB

1. 解压安装包
	<pre>
	tar –zxvf mongodb-linux-x86_64-rhel70-2.6.7.tgz
	</pre>


2. 把安装包移动到 /usr/local/下

	<pre>
	mv mongodb-linux-x86_64-rhel70-2.6.7 /usr/local/mongodb
	cd /usr/local/mongodb
	</pre>

3. 新建目录data/db和logs

	<pre>
	mkdir -p data/db
	mkdir logs
	</pre>

4. 然后在/usr/local/mongodb/bin下新建一个mongodb的配置文件

	<pre>
	cd /usr/local/mongodb/bin
	touch mongodb.conf
	vi mongodb.conf
	</pre>

	按一下解盘i 启用编辑
	把下面配置复制上去
	
	<pre>
	dbpath=/usr/local/mongodb/db 
	logpath=/usr/local/mongodb/logs/mongodb.log 
	port=27017 
	fork=true 
	nohttpinterface=true
	</pre>
	按esc键然后输入:wq 回车。保存退出
5. 然后用配置文件启动
	<pre>
	./mongod –f mongodb.conf
	</pre>
## MongoDB数据备份

1. mongodump命令备份数据库
	<pre>
	mongodump -h IP --port 端口 -u 用户名 -p 密码 -d 数据库 -o 文件存在路径
	//如果没有用户，可以去掉-u和-p,如果在Linux上用户密码中含有特殊字符在其前面加\
	//如果导出本机的数据库，可以去掉-h。
	//如果是默认端口，可以去掉--port。
	//如果想导出所有数据库，可以去掉-d。
	//如果不指定-o，文件备份在当前目录下
	</pre>
2. mongorestore还原数据库
	<pre>
	mongorestore -h IP --port 端口 -u 用户名 -p 密码 -d 数据库 --drop 文件存在路径
	//--drop的意思是，先删除所有的记录，然后恢复。
	</pre>

## 启用身份认证

1. 启动mongodb(如果已启动请忽略)
	<pre>
	cd /usr/local/mongodb/bin
	./mongod –f mongodb.conf
	</pre>
2. 在admin库下创建一个管理员帐号
	<pre>
	./mongo
	use admin
	db.createUser(
		{
			user: "root",
			pwd: "root",
			roles: [ { role: "userAdminAnyDatabase", db: "admin" },{ role: "readWriteAnyDatabase", db: "admin" } ]
		}
	)
	Exit
	</pre>
3. 关闭MongoDB server，修改mongodb.conf配置
	<pre>
	cd /usr/local/mongodb/bin
	./mongod –f mongodb.conf –shutdown
	vi mongodb.conf
	按i进行编辑，并用下面内容替换原来的内容
	dbpath=/usr/local/mongodb/data/db
	logpath=/usr/local/mongodb/logs/mongodb.log
	pidfilepath=/usr/local/mongodb/logs/mongodb.pid
	port=27017
	fork=true
	auth=true //启用身份认证
	nohttpinterface=true
	</pre>
4. 使用新的配置文件重启
	<pre>
	./mognod –f mongodb.conf
	</pre>
5. 为每个数据库单独设置用户和权限

	> 由于我们刚刚只创建了一个admin权限的用户，实际在生产环境中是应该是使用单独用户和最小的权限去操作数据库的。下面我们为每个数据库创建一个只有读写权限的用户。

	<pre>
	cd /usr/local/mongodb/bin
	./mongo
	use admin
	db.auth(“root”,”root”)
	use testdb
	db.createUser(
		{
			user: "admin",
			pwd: "admin",
			roles: [ { role: "readWrite", db: "testdb" } ]
		}
	)
	</pre>
