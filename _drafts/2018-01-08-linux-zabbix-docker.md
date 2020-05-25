---
layout: post
title: Centos7 下搭建 Docker 容器版本 Zabbix
categories: Centos7, Docker, Zabbix
description: 记录 Centos7 下搭建 Docker 容器版本 Zabbix
keywords: Centos7, Docker, Zabbix
---

## Yum 方式安装 Docker CE

1、删除以往安装的 docker 版本
```sh
$ sudo yum remove docker \
                  docker-common \
                  docker-selinux \
                  docker-engine
```
以上实测，删除 rpm 方式安装的 docker 不全，最好还是多执行一步
```sh
$ sudo rpm -qa | grep docker
$ sudo rmp -e docker-engine
```
2、安装些依赖组件
```sh
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```
3、安装官方 yum 源
```sh
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
还有附加的源可用，上面可用的情况，可以忽略
找开：
```sh
$ sudo yum-config-manager --enable docker-ce-edge
$ sudo yum-config-manager --enable docker-ce-test
```
关闭：
```sh
$ sudo yum-config-manager --disable docker-ce-edge
```
4、安装最新版本 Docker CE
```sh
$ sudo yum install docker-ce
```
可以安装指定版本，生产环境经常需要这样,先查一下版本
```sh
yum list docker-ce --showduplicates | sort -r
* updates: mirrors.aliyun.com
Loading mirror speeds from cached hostfile
Loaded plugins: fastestmirror
Installed Packages
* extras: mirrors.cn99.com
docker-ce.x86_64         18.01.0.ce-1.el7.centos                docker-ce-test  
docker-ce.x86_64         18.01.0.ce-1.el7.centos                docker-ce-edge  
docker-ce.x86_64         18.01.0.ce-1.el7.centos                @docker-ce-edge
docker-ce.x86_64         18.01.0.ce-0.1.rc1.el7.centos          docker-ce-test  
docker-ce.x86_64         17.12.0.ce-1.el7.centos                docker-ce-test  
docker-ce.x86_64         17.12.0.ce-1.el7.centos                docker-ce-stable
docker-ce.x86_64         17.12.0.ce-1.el7.centos                docker-ce-edge  
```
安装指定版本,示例安装 docker-ce-18.01.0.ce
```sh
$ sudo yum install docker-ce-18.01.0.ce
```
5、启动 Docker
```sh
$ sudo systemctl start docker
```
测试一下
```sh
$ sudo docker run hello-world
```

## 安装 Docker Compose
1、下载最新版本 Docker Compose
```sh
sudo curl -L https://github.com/docker/compose/releases/download/1.18.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```
2、斌与可执行权限
```sh
sudo chmod +x /usr/local/bin/docker-compose
```
3、测试下安装效果，查看 Docker Compose 版本
```sh
$ docker-compose --version
docker-compose version 1.18.0, build 1719ceb
```

## 下载编排文件和相关配置文件
```sh
cd /usr/local
git clone https://github.com/zabbix/zabbix-docker.git
```
编排文件和配置如下：
![compose](/images/posts/docker-compose/markdown-img-paste-20180122210825913.png)


## 执行编排文件，试运行容器组合
```sh
$ docker-compose -f ./docker-compose_v2_alpine_mysql_latest.yaml up -d
```
报错了
```sh
Traceback (most recent call last):
  File "bin/docker-compose", line 6, in <module>
  File "compose/cli/main.py", line 71, in main
  File "compose/cli/main.py", line 121, in perform_command
  File "compose/cli/command.py", line 37, in project_from_options
  File "compose/cli/command.py", line 91, in get_project
  File "compose/config/config.py", line 375, in load
  File "compose/config/config.py", line 506, in process_config_file
  File "compose/config/config.py", line 497, in interpolate_config_section
  File "compose/config/interpolation.py", line 44, in interpolate_environment_variables
  File "compose/config/interpolation.py", line 44, in <genexpr>
  File "compose/config/interpolation.py", line 39, in process_item
  File "compose/config/interpolation.py", line 39, in <genexpr>
  File "compose/config/interpolation.py", line 54, in interpolate_value
  File "compose/config/interpolation.py", line 74, in recursive_interpolate
  File "compose/config/interpolation.py", line 74, in <genexpr>
  File "compose/config/interpolation.py", line 70, in recursive_interpolate
  File "compose/config/interpolation.py", line 184, in convert
  File "compose/config/interpolation.py", line 141, in to_int
ValueError: invalid literal for int() with base 0: '512m'
Failed to execute script docker-compose
```


docker run --name zabbix-server-mysql \
 -p 10051:10051 \
 --net=host \
 -e DB_SERVER_HOST="数据库ip" \
 -e DB_SERVER_PORT=数据库端口 \
 -e MYSQL_USER="数据库用户名" \
 -e MYSQL_PASSWORD="数据库密码" \
 -d zabbix/zabbix-server-mysql

 docker run --name zabbix-server-mysql \
  -p 10051:10051 \
  --net=host \
  -e DB_SERVER_HOST="172.20.30.140" \
  -e DB_SERVER_PORT=3306 \
  -e MYSQL_USER="root" \
  -e MYSQL_PASSWORD="blf1#root" \
  -d zabbix/zabbix-server-mysql


  docker run --name zabbix-web-apache-mysql \
  -p 8088:80  \
  -e DB_SERVER_HOST="数据库ip" \
  -e DB_SERVER_PORT=数据库端口 \
  -e MYSQL_USER="数据库用户名" \
  -e MYSQL_PASSWORD="数据库密码" \
  -e ZBX_SERVER_HOST="zabbix服务器IP" \
  -e TZ="Asia/Shanghai" \
  -d zabbix/zabbix-web-apache-mysql

  docker run --name zabbix-web-apache-mysql \
  -p 8088:80  \
  -e DB_SERVER_HOST="172.20.30.140" \
  -e DB_SERVER_PORT=3306 \
  -e MYSQL_USER="root" \
  -e MYSQL_PASSWORD="blf1#root" \
  -e ZBX_SERVER_HOST="192.168.49.130" \
  -e TZ="Asia/Shanghai" \
  -d zabbix/zabbix-web-apache-mysql


  docker run --name some-zabbix-agent \
  -p 10050:10050 \
  -e ZBX_HOSTNAME="hostname" \
  -e ZBX_SERVER_HOST="zabbix服务器IP" \
  -e ZBX_SERVER_PORT=10051 \
  -d zabbix/zabbix-agent

  docker run --name some-zabbix-agent \
  -p 10050:10050 \
  -e ZBX_HOSTNAME="92.168.49.130" \
  -e ZBX_SERVER_HOST="192.168.49.130" \
  -e ZBX_SERVER_PORT=10051 \
  -d zabbix/zabbix-agent  



  docker run --name zabbix-mysql-server --hostname zabbix-mysql-server \
  -e MYSQL_ROOT_PASSWORD="123456" \
  -e MYSQL_USER="zabbix" \
  -e MYSQL_PASSWORD="123456" \
  -e MYSQL_DATABASE="zabbix" \
  -v /home/docker/mysql_3306/custom:/etc/mysql/conf.d \
  -v /home/docker/mysql_3306/data:/var/lib/mysql/ \
  -p 3306:3306 \
  -d registry.eyd.com:5000/library/mysql:5.7.14 \
  --character-set-server=utf8 --collation-server=utf8_bin

  docker run  --name zabbix-server-mysql --hostname zabbix-server-mysql \
--link zabbix-mysql-server:mysql \
-e DB_SERVER_HOST="mysql" \
-e MYSQL_USER="zabbix" \
-e MYSQL_DATABASE="zabbix" \
-e MYSQL_PASSWORD="123456" \
-v /etc/localtime:/etc/localtime:ro \
-v /data/docker/zabbix/alertscripts:/usr/lib/zabbix/alertscripts \
-v /data/docker/zabbix/externalscripts:/usr/lib/zabbix/externalscripts \
-p 10051:10051 \
-d \
zabbix/zabbix-server-mysql


docker run --name zabbix-web-nginx-mysql --hostname zabbix-web-nginx-mysql \
--link zabbix-mysql-server:mysql \
--link zabbix-server-mysql:zabbix-server \
-e DB_SERVER_HOST="mysql" \
-e MYSQL_USER="zabbix" \
-e MYSQL_PASSWORD="123456" \
-e MYSQL_DATABASE="zabbix" \
-e ZBX_SERVER_HOST="zabbix-server" \
-e PHP_TZ="Asia/Shanghai" \
-p 8000:80 \
-p 8443:443 \
-d \
zabbix/zabbix-web-nginx-mysql

docker run --name zabbix-agent --link zabbix-server-mysql:zabbix-server -d zabbix/zabbix-agent:latest

docker run --name zabbix-agent \
-e ZBX_HOSTNAME="test-02" \
-e ZBX_SERVER_HOST="192.168.49.130" \
-d zabbix/zabbix-agent:latest

docker run --name zabbix-agent \
--net=host \
-e ZBX_HOSTNAME="test-02" \
-e ZBX_SERVER_HOST="192.168.49.130" \
-d registry.eyd.com:5000/zabbix/zabbix-agent:latest

## 参考资料
* [Zabbix 官方文档 ](https://www.zabbix.com/documentation/3.4/zh/manual/installation/containers)
* [Docker 官方安装文档](https://docs.docker.com/engine/installation/linux/docker-ce/centos/#upgrade-docker-after-using-the-convenience-script)
* [Docker Compose 官方安装文档](https://docs.docker.com/compose/install/)
