# Docker 安装
1. 缷载Docker
<pre>
sudo yum remove docker \
docker-client \
docker-client-latest \
docker-common \
docker-latest \
docker-latest-logrotate \
docker-logrotate \
docker-selinux \
docker-engine-selinux \
docker-engine
</pre>
2. 安装依赖包
<pre>
sudo yum install -y yum-utils \
device-mapper-persistent-data \
lvm2
</pre>

3. 设置Docker的安装库
<pre>
sudo yum-config-manager \
--add-repo \
https://download.docker.com/linux/centos/docker-ce.repo
</pre>