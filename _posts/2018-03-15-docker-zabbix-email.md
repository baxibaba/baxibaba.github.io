---
layout: post
title: Docker 容器版的 zabbix 监控报警系统搭建
date: 2018-03-15 09:30:30 +0800
description: 使用 docker 容器来发布 zabbix 监控报警系统
categories: docker, zabbix, centos7, mysql
keywords: docker, zabbix, centos7, mysql
---


zabbix 是一个流行的开源监控报警系统，这次使用 docker 容器来发布 zabbix ，以下分享安装的整个过程。

### 监控的目的

主要关注点是系统层，服务器CPU、memory、IO这些基础指标参数，并实现一个邮件报警功能，邮件关联企业微信，这样就能方便的在移动端及时收到报警邮件了。

---

### 安装 zabbix 监控的数据库

[zabbix-db-mariadb官网](https://hub.docker.com/r/monitoringartist/zabbix-db-mariadb/)

网上大部分资料都是从创建一个 mariadb 数据库开始的：

```sh
$ docker rm -f zabbix-db

$ docker run -d --name zabbix-db -p 5000:3306 --restart=always \
  --env="MARIADB_USER=设置一个库用户名" \
  --env="MARIADB_PASS=设置一个密码" \
  --env="DB_innodb_buffer_pool_size=512M" \
  --env="DB_innodb_log_file_size=64M" \
  -v /etc/localtime:/etc/localtime:ro \
  -v /home/docker/zabbix-db/data:/var/lib/mysql \
  -v /home/docker/zabbix-db/backups:/backups \
  reg.blf1.org/monitoringartist/zabbix-db-mariadb:latest

# 查看日志输出
$ docker logs -f zabbix-db
=> An empty/uninitialized MariaDB volume is detected in /var/lib/mysql
=> Installing MariaDB...
=> Installing MariaDB... Done!
----------------- Previous error log -----------------
171025 11:34:29 [Note] InnoDB: Starting shutdown...
171025 11:34:30 [Note] InnoDB: Waiting for page_cleaner to finish flushing of buffer pool
171025 11:34:31 [Note] InnoDB: Shutdown completed; log sequence number 1616697
171025 11:34:32 [Note] InnoDB: Using mutexes to ref count buffer pool pages
171025 11:34:32 [Note] InnoDB: The InnoDB memory heap is disabled
171025 11:34:32 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
171025 11:34:32 [Note] InnoDB: GCC builtin __atomic_thread_fence() is used for memory barrier
171025 11:34:32 [Note] InnoDB: Compressed tables use zlib 1.2.7
171025 11:34:32 [Note] InnoDB: Using Linux native AIO
171025 11:34:32 [Note] InnoDB: Using CPU crc32 instructions
171025 11:34:32 [Note] InnoDB: Initializing buffer pool, size = 512.0M
171025 11:34:32 [Note] InnoDB: Completed initialization of buffer pool
171025 11:34:32 [Note] InnoDB: Highest supported file format is Barracuda.
171025 11:34:32 [Note] InnoDB: 128 rollback segment(s) are active.
171025 11:34:32 [Note] InnoDB: Waiting for purge to start
171025 11:34:32 [Note] InnoDB:  Percona XtraDB (http://www.percona.com) 5.6.36-82.1 started; log sequence number 1616697
171025 11:34:32 [Note] InnoDB: FTS optimize thread exiting.
171025 11:34:32 [Note] InnoDB: Starting shutdown...
171025 11:34:33 [Note] InnoDB: Waiting for page_cleaner to finish flushing of buffer pool
171025 11:34:34 [Note] InnoDB: Shutdown completed; log sequence number 1616707
----------------- Previous error log ends -----------------

Waiting for DB service...
Still waiting for DB service...
171025 11:34:35 mysqld_safe Logging to '/var/lib/mysql/error.log'.
171025 11:34:35 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
171025 11:34:35 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
171025 11:34:35 [Warning] options --log-slow-admin-statements, --log-queries-not-using-indexes and --log-slow-slave-statements have no effect if --log_slow_queries is not set
171025 11:34:35 [Note] /usr/sbin/mysqld (mysqld 10.0.32-MariaDB) starting as process 341 ...
171025 11:34:35 [Warning] Could not increase number of max_open_files to more than 4096 (request: 4407)
171025 11:34:35 [Note] InnoDB: Using mutexes to ref count buffer pool pages
171025 11:34:35 [Note] InnoDB: The InnoDB memory heap is disabled
171025 11:34:35 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
171025 11:34:35 [Note] InnoDB: GCC builtin __atomic_thread_fence() is used for memory barrier
171025 11:34:35 [Note] InnoDB: Compressed tables use zlib 1.2.7
171025 11:34:35 [Note] InnoDB: Using Linux native AIO
171025 11:34:35 [Note] InnoDB: Using CPU crc32 instructions
171025 11:34:35 [Note] InnoDB: Initializing buffer pool, size = 512.0M
171025 11:34:35 [Note] InnoDB: Completed initialization of buffer pool
171025 11:34:35 [Note] InnoDB: Highest supported file format is Barracuda.
171025 11:34:35 [Note] InnoDB: 128 rollback segment(s) are active.
171025 11:34:35 [Note] InnoDB: Waiting for purge to start
171025 11:34:35 [Note] InnoDB:  Percona XtraDB (http://www.percona.com) 5.6.36-82.1 started; log sequence number 1616707
171025 11:34:35 [Note] Plugin 'FEEDBACK' is disabled.
171025 11:34:35 [Note] Server socket created on IP: '0.0.0.0'.
171025 11:34:35 [Warning] 'user' entry 'root@fc28ae2c6c51' ignored in --skip-name-resolve mode.
171025 11:34:35 [Warning] 'user' entry '@fc28ae2c6c51' ignored in --skip-name-resolve mode.
171025 11:34:35 [Warning] 'proxies_priv' entry '@% root@fc28ae2c6c51' ignored in --skip-name-resolve mode.
171025 11:34:35 [Note] Reading of all Master_info entries succeded
171025 11:34:35 [Note] Added new Master_info '' to hash table
171025 11:34:35 [Note] /usr/sbin/mysqld: ready for connections.
Version: '10.0.32-MariaDB'  socket: '/var/lib/mysql/mysql.sock'  port: 3306  MariaDB Server
171025 11:34:35 [Note] /usr/sbin/mysqld: ready for connections.
Securing and tidying DB...
Securing and tidying DB... Done!
Showing DB status...

--------------
mysql  Ver 15.1 Distrib 10.0.32-MariaDB, for Linux (x86_64) using readline 5.1

Connection id:          6
Current database:
Current user:           root@127.0.0.1
SSL:                    Not in use
Current pager:          stdout
Using outfile:          ''
Using delimiter:        ;
Server:                 MariaDB
Server version:         10.0.32-MariaDB MariaDB Server
Protocol version:       10
Connection:             localhost via TCP/IP
Server characterset:    utf8
Db     characterset:    utf8
Client characterset:    utf8
Conn.  characterset:    utf8
TCP port:               3306
Uptime:                 1 sec

Threads: 1  Questions: 16  Slow queries: 1  Opens: 1  Flush tables: 1  Open tables: 64  Queries per second avg: 16.000
--------------

Creating DB admin user...

=> Creating MariaDB user 'admin' with 'zabbixxxx2017' password.
========================================================================
    You can now connect to this MariaDB Server using:                   
    mysql -uadmin -pzabbixAdmin2017 -h<host>                      

    For security reasons, you might want to change the above password.  
    The 'root' user has no password but only allows local connections   
========================================================================
```

测试数据库连接（mariadb 跟 mysql 类似，这里直接使用 mysql 的客户端进行连接测试）

```sh
$ mysql -h172.20.32.44 -uadmin -pzabbixxxx2017 -P5000 -e"show databases"
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+

$ mysql -h172.20.32.44 -uadmin -pzabbixAdmin2017 -P5000 -e"select version()"
+-----------------+
| version()       |
+-----------------+
| 10.0.32-MariaDB |
+-----------------+
```

#### 数据库备份

```sh
$ docker exec -it zabbix-db \
  /zabbix-backup/zabbix-mariadb-dump -u admin -p zabbixxxx2017 -o /backups
Configuration:
 - host:     127.0.0.1 (localhost)
 - database: zabbix
 - user:     admin
 - output:   /backups

Fetching list of existing tables...
Starting table backups...
0%.......12%12%.......24%.......36%.......48%.......60%60%.......72%72%.......84%.......96%..

For the following large tables only the schema (without data) was stored:
 - acknowledges
 - alerts
 - auditlog
 - auditlog_details
 - events
 - history
 - history_log
 - history_str
 - history_text
 - history_uint
 - trends
 - trends_uint

Compressing backup file...

Backup Completed:
/backups/zabbix_cfg_localhost_20171025-1650_db-3.4.5.sql.gz

$ ls /home/docker/zabbix-db/backups/
zabbix_cfg_localhost_20171025-1650_db-3.4.5.sql.gz
```


#### 数据库还原

其实就是 mysqldump 相关的操作，这里暂不折腾。

---

### 安装 Zabbix Server

[Zabbix Server官方介绍](https://hub.docker.com/r/zabbix/zabbix-server-mysql/)

What is Zabbix server?
Zabbix server is the central process of Zabbix software.

The server performs the polling and trapping of data, it calculates triggers, sends notifications to users. It is the central component to which Zabbix agents and proxies report data on availability and integrity of systems. The server can itself remotely check networked services (such as web servers and mail servers) using simple service checks.

Zabbix server 是此套监控的服务端，核心组件。

```sh
$ docker rm -f zabbix-server

$ docker run  --name zabbix-server --hostname zabbix-server \
--link zabbix-db:mysql \
-e DB_SERVER_HOST="mysql" \
-e MYSQL_USER="库用户名" \
-e MYSQL_DATABASE="zabbix" \
-e MYSQL_PASSWORD="库密码" \
-v /etc/localtime:/etc/localtime:ro \
-v /data/docker/zabbix/alertscripts:/usr/lib/zabbix/alertscripts \
-v /data/docker/zabbix/externalscripts:/usr/lib/zabbix/externalscripts \
-p 10051:10051 \
--restart=always \
-d \
zabbix/zabbix-server-mysql:3.4.5
```

部署 zabbix-web-nginx
```sh
docker run --name zabbix-web-nginx --hostname zabbix-web-nginx \
--link zabbix-db:mysql \
--link zabbix-server:zabbix-server \
-e DB_SERVER_HOST="mysql" \
-e MYSQL_USER="库账号" \
-e MYSQL_PASSWORD="库密码" \
-e MYSQL_DATABASE="zabbix" \
-e ZBX_SERVER_HOST="zabbix-server" \
-e PHP_TZ="Asia/Shanghai" \
-p 8000:80 \
-p 8443:443 \
-d \
zabbix/zabbix-web-nginx-mysql:3.4.5
```

Wait ~30 seconds for Zabbix initialization

Zabbix web will be available on the port 8000, Zabbix server on the port 9000

Zabbix server Default credentials: Admin/zabbix

在容器起来以后，可以通过 `http://172.20.32.44:8000`（此处IP地址为容器所在宿主机IP，需要改成自己环境的） 访问 zabbix 的 web 界面了：

![zabbix22](/images/posts/zabbix/markdown-img-paste-20180323151202889.png)

登录成功后，可以看到 zabbix server 的界面：

![zabbixx23](/images/posts/zabbix/markdown-img-paste-20180323151322730.png)

点击右上角的用户图标，可以进行用户的设置，比如：更改字符集为中文（对于新手会比较友好）

![zabbix24](/images/posts/zabbix/markdown-img-paste-20180323151436299.png)

#### 使用自建的 mysql 数据库

经过初步验证，发现直接使用已搭建好的 mysql 数据库也是可以的，这样数据就比较集中，也不用操心数据备份的事情的，数据库备份策略会统一做。

```sh
docker rm -f zabbix-mysql

docker run --name zabbix-mysql --hostname zabbix-mysql \
-e MYSQL_ROOT_PASSWORD="库的root密码" \
-e MYSQL_USER="zabbix" \
-e MYSQL_PASSWORD=“zabbix用户密码” \
-e MYSQL_DATABASE="zabbix" \
-v /home/docker/mysql_3306/custom:/etc/mysql/conf.d \
-v /home/docker/mysql_3306/data:/var/lib/mysql/ \
-p 3306:3306 \
--restart=always \
-d registry.eyd.com:5000/library/mysql:5.7.14 \
--character-set-server=utf8 --collation-server=utf8_bin
```

熟悉 mysql 的小伙伴，选择自建的 mysql 库做为 zabbix server 的后端存储，很不错哦。

---

### 安装 Zabbix Agent

[zabbix_agent官网说明](https://hub.docker.com/r/zabbix/zabbix-agent/)

What is Zabbix agent?

Zabbix agent is deployed on a monitoring target to actively monitor local resources and applications (hard drives, memory, processor statistics etc).

Zabbix Agent 是被监控目标端上的数据采集工具。

在每个客户端运行：

```
$ docker rm -f zabbix-agent

$ docker run --name zabbix-agent \
--net=host \
-e ZBX_HOSTNAME="给个计算名" \
-e ZBX_SERVER_HOST="Zabbix server IP" \
--restart=always \
-d zabbix/zabbix-agent:3.4.5

```

您可以使用几乎任何代理配置参数，只需添加前缀ZBX_

如果不指定自定义设置变量，则将使用默认的 Zabbix agent 设置。

例如，要使用 StartAgents=10，只需添加环境变量-e "ZBX_STARTAGENTS=10"。


容器重启，停用，删除都是可以的，不影响使用。 客户端启动成功后，发现已经被自动注册到服务端，如果没有，可以手工创建主机。

![zabbix21](/images/posts/zabbix/markdown-img-paste-20180323114041815.png)

---

### 配置 Zabbix 邮件告警功能

#### 使用自带的 Email

参见 [官方文档 Zabbix Documentation 3.4](https://www.zabbix.com/documentation/3.4/zh/manual/config/notifications/media/script)

1、设置报警媒介，通俗讲就是设置一个发件邮箱
![zabbix13](/images/posts/zabbix/markdown-img-paste-20180323095530905.png)
![zabbix14](/images/posts/zabbix/markdown-img-paste-20180323095853923.png)

关于邮箱的选择，原则上任何一家服务商提供的邮箱都可以，相信大家都想申请一个专用邮箱用来发送报警邮件，包括博主我也是。**在尝试了QQ，163邮箱后，发现新注册的邮箱都是没有开通SMTP这些功能的（没有此功能，设置了无法正常发送邮件，坑），想要使用此功能，15天后才能申请开通，真是太揪心了。**绕了一圈，还是公司的企业邮箱（也是企鹅厂的产品）最好用。

2、配置--动作，设置邮件触发条件和邮件格式的地方
![zabbix15](/images/posts/zabbix/markdown-img-paste-20180323103958238.png)
![zabbix16](/images/posts/zabbix/markdown-img-paste-2018032310403210.png)
![zabbix17](/images/posts/zabbix/markdown-img-paste-20180323104100226.png)

选择触发条件，以固定邮件格式告警，这里贴几个改过的格式出来，添加了中文，增强可读性。

```sh
# 操作
[告警触发] {TRIGGER.NAME}
告警时间：{EVENT.DATE} {EVENT.TIME}
告  警  IP：{HOST.IP}
告警主机：{HOSTNAME1}
告警等级：{TRIGGER.SEVERITY}
告警信息：{TRIGGER.NAME}
告警项目：{TRIGGER.KEY1}
问题详情：{ITEM.NAME}
当前状态：{TRIGGER.STATUS}
当  前  值：{ITEM.VALUE1}
事  件  ID：{EVENT.ID}

# 恢复操作
[告警恢复] {TRIGGER.NAME}
恢复时间：{EVENT.DATE} {EVENT.TIME}
告  警  IP：{HOST.IP}
告警主机：{HOSTNAME1}
告警等级：{TRIGGER.SEVERITY}
告警信息：{TRIGGER.NAME}
告警项目：{TRIGGER.KEY1}
问题详情：{ITEM.NAME}
当前状态：{TRIGGER.STATUS}
当  前  值：{ITEM.VALUE1}
原事件ID：{EVENT.ID}
{TRIGGER.URL}

# 确认操作
[告警确认]  {TRIGGER.NAME}
{USER.FULLNAME} acknowledged problem at {ACK.DATE} {ACK.TIME} with the following message:
{ACK.MESSAGE}

Current problem status is {EVENT.STATUS}

```

3、管理--用户，设置收件人
![zabbix18](/images/posts/zabbix/markdown-img-paste-20180323105433740.png)
![zabbix19](/images/posts/zabbix/markdown-img-paste-20180323105529240.png)
![zabbix20](/images/posts/zabbix/markdown-img-paste-20180323105603871.png)

这个地方注意，**如果没有邮件组，要给多人发送邮件的话，得添加多条，默认一条只支持设置一个邮箱，如果设置多个邮箱，亲测无效。**


#### 使用自定义告警功能

[zabbix3.0.4 邮件告警详细配置](http://www.cnblogs.com/rysinal/p/5834421.html)

除了自带的邮件告警功能，zabbix 还支持自定义告警，如使用 sendEmail 工具，也是可以成功的。相比自带的 Email 增加了对 html 格式邮件的支持。

```sh
$ docker exec -it zabbix bash

$ wget http://caspian.dotconf.net/menu/Software/SendEmail/sendEmail-v1.56.tar.gz

$ tar zxf sendEmail-v1.56.tar.gz -C /usr/src

$ cd /usr/src/sendEmail-v1.56

$ cp -a sendEmail /usr/local/bin

$ chmod +x /usr/local/bin/sendEmail

# 测试邮件功能
$ /usr/local/bin/sendEmail \
  -f 你的邮件账号 \
  -t 你的邮件账号 \
  -s 你的邮件服务器 \
  -u "我是邮件主题" \
  -o message-content-type=html \
  -o message-charset=utf8 \
  -o tls=no \
  -xu 你的邮件账号 \
  -xp 这里是密码 \
  -m "我是邮件内容"

# 增加自定义警告发送脚本
$  tee /usr/local/share/zabbix/alertscripts/sendEmail.sh <<-'EOF'
#!/bin/bash

to=$1
subject=$2
body=$3

/usr/local/bin/sendEmail  
  -f 你的邮件账号 \
  -t "$to"
  -s 你的邮件服务器 \
  -u "$subject"
  -o message-content-type=html \
  -o message-charset=utf8 \
  -o tls=no \
  -xu 你的邮件账号 \
  -xp 这里是密码 \
  -m "$body"
EOF

$ chmod +x /usr/local/share/zabbix/alertscripts/sendEmail.sh

$ chown zabbix.zabbix /usr/local/share/zabbix/alertscripts/sendEmail.sh
```

然后按照教程一步步在管理界面配置就能成功，需要一些耐心。

相较而言，zabbix 自带的邮件告警功能已经能满足基本需求，不用安装额外的软件，相对简单。有些 zabbix 容器中并不支持安装软件，需要重新封装镜像来支持，还是蛮花时间的，不推荐。

---

### 监控服务器内存

zabbix默认的剩余内存报警，值设置过小，小于20M才报警太晚了。
```sh
Average Lack of available memory on server {HOST.NAME}{Template OS Linux:vm.memory.size[available].last(0)}<20M
```

实际上，当内存不足10%的时候就需要配置报警，下面我们来完成此项配置。
1、创建item(监控项)
```sh
Configuration-->Templates-->Template OS Linux-->items-->create item

name:
Ava memory percent
type:
Calculated #计算类型
key:
vm.memory.free[percent]
Formula：
100*last("vm.memory.size[available]")/last("vm.memory.size[total]")
Applications：
Memory
```

![z01](/images/posts/zabbix/markdown-img-paste-20180321085405470.png)

2、创建trigger（触发器）
```sh
Configuration-->Templates-->Template OS Linux-->Triggers-->create trigger
Name:
free mem less 10%
Expression:
{Template OS Linux:vm.memory.free[percent].last()}<10
```

![zabbix02](/images/posts/zabbix/markdown-img-paste-20180321090120105.png)

3、报警信息效果图

![zabbix03](/images/posts/zabbix/markdown-img-paste-2018032109073556.png)

4、更进一步
内存不足10% 这个时候服务器内存资源已经比较紧张，可以利用这个触发器触发一个脚本来重启占用内存比较多的服务，单实例无集群的服务不能用此方法，最好对服务也做好监控，要不然自动重启服务失败，就尴尬了。


### 监控服务器CPU

需要的效果，linux主机cpu使用率超过90%的时候报警。linux默认的模板中无此项，只有cpu的负载，当负载在一定时间内5以上报警。我们在cpu utilization中找到一个cpu idle时间，即cpu的空闲时间，当空闲时间小于10%的时候就是cpu大于90%的时候。

在linux模板中添加触发器

```sh
Name:
cpu user percent gt 90%

Expression:
{Template OS Linux:system.cpu.util[,idle].avg(1m)}<10
```

![zabbix03](/images/posts/zabbix/markdown-img-paste-20180321094921538.png)

---

### 监控流量

为了防止某台机器流量过高导致网络慢，或者因为中病毒或木马等原因，导致流量很高，可使用zabbix的流量告警功能来对流量进行告警。

配置告警路径：
Configuration--Templates--Template OS Linux--Discovery--Network interface discovery--Trigger Prototypes--Create trigger prototype

![zabbix04](/images/posts/zabbix/markdown-img-paste-2018032110381992.png)

在Add，添加Item时要选择Select prototype；Function选择Average value of a period T is > N ; N 的值为50000000（我这是设置为超过50MB才告警，请自行根据需要设置大小）

![zabbix05](/images/posts/zabbix/markdown-img-paste-20180321104721552.png)

![zabbix06](/images/posts/zabbix/markdown-img-paste-2018032110450356.png)

![zabbix07](/images/posts/zabbix/markdown-img-paste-2018032110494494.png)

![zabbix08](/images/posts/zabbix/markdown-img-paste-20180321105022384.png)


---

### 监控磁盘空间
对磁盘容量的监控，zabbix自带的linux模板中已有配置，让我们来看看配置在哪里，另外稍改下配置来做做实验，测试一下效果。
位置在这里
![zabbix09](/images/posts/zabbix/markdown-img-paste-20180321113837317.png)

![zabbix10](/images/posts/zabbix/markdown-img-paste-20180321171901104.png)

修改触发器的值，20改为80，让可用磁盘空间超过 20%就触发报警，这是做测试用，测试OK后再改回来

![zabbix11](/images/posts/zabbix/markdown-img-paste-20180321172112297.png)

收到的邮件报警效果

![zabbix12](/images/posts/zabbix/markdown-img-paste-2018032117223285.png)



---



### 参考资料

[官方文档 Zabbix Documentation 3.4](https://www.zabbix.com/documentation/3.4/zh/manual/config/notifications/media/script)

[ZABBIX中文社区](http://www.zabbix.org.cn/)

[zabbix3.0.4 邮件告警详细配置](http://www.cnblogs.com/rysinal/p/5834421.html)

[如何使用sendEmail发送邮件](http://www.ttlsa.com/linux/use-sendemail/)

[reblue520的zabbix专栏](http://blog.csdn.net/reblue520/article/category/6403502)

[用 Prometheus 来监控你的 Kubernetes 集群](https://www.kubernetes.org.cn/1954.html)

---

**转载**请注明出处，本文采用 [CC4.0](http://creativecommons.org/licenses/by-nc-nd/4.0/) 协议授权，版权归 [able615](https://able615.github.io) 所有。
