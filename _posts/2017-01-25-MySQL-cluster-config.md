---
layout: post
title:  "MySQL Cluster 基础配置"
date:   2017-02-02 23:00:00 +0800
categories: articles
keywords: MySQL Cluster 集群
---

本文主要讲的是 MySQL Cluster 的配置过程。

MySQL Cluster 官方推荐最少6台机器，2台管理节点，2台SQL节点，2台数据节点。
不过测试的时候都放到一个节点也可以。

我的配置如下:

管理节点  192.168.3.21

SQL节点  192.168.3.22

数据节点1  192.168.3.23

数据节点2  192.168.3.24

我的操作系统是 Ubuntu 16.04, 使用的MySQL Cluster是7.5, 以下步骤都切换到root用户执行。

## 0 每个节点相同部分
{% highlight bash %}
$ apt-get update
$ apt-get install libaio1
{% endhighlight %}

下载 MySQL Cluster 版
{% highlight bash %}
$ wget https://cdn.mysql.com//Downloads/MySQL-Cluster-7.5/mysql-cluster-gpl-7.5.5-linux-glibc2.5-x86_64.tar.gz
$ tar -xzvf mysql-cluster-gpl-7.5.5-linux-glibc2.5-x86_64.tar.gz
$ mv mysql-cluster-gpl-7.5.5-linux-glibc2.5-x86_64/ mysql
{% endhighlight %}

## 1 管理节点
{% highlight bash %}
$ cp mysql/bin/ndb_mgm* /usr/local/bin/
$ chmod 755 /usr/local/bin/ndb_mgm*
$ mkdir -p /var/lib/mysql-cluster/
$ vi /var/lib/mysql-cluster/config.ini
{% endhighlight %}

在配置文件/var/lib/mysql-cluster/config.ini中写入以下内容，注意修改各个节点IP
{% highlight ini %}
[ndbd default]
NoOfReplicas=2
# Memory to allocate for data storage
DataMemory=2G
# Memory to allocate for index storage
IndexMemory=512M

[mysqld default]

[ndb_mgmd default]

[tcp default]

# Management VPS
[ndb_mgmd]
# Enter the hostname or IP address of the Management VPS
hostname=192.168.3.21

# SQL VPS
[mysqld]
# Enter the hostname or IP address of the SQL VPS
hostname=192.168.3.22

# Data1 VPS
[ndbd]
# Enter the hostname or IP address of the Data1 VPS
hostname=192.168.3.23
DataDir= /var/lib/mysql-cluster

# Data2 VPS
[ndbd]
# Enter the hostname or IP address of the Data2 VPS
hostname=192.168.3.24
DataDir=/var/lib/mysql-cluster
{% endhighlight %}

启动管理节点
{% highlight bash %}
$ ndb_mgmd -f /var/lib/mysql-cluster/config.ini --configdir=/var/lib/mysql-cluster/
$ ndb_mgmd -f /var/lib/mysql-cluster/config.ini --configdir=/var/lib/mysql-cluster/ --initial # 有修改节点的时候
{% endhighlight %}

启动成功会提示:
{% highlight bash %}
MySQL Cluster Management Server mysql-5.7.17 ndb-7.5.5
{% endhighlight %}

将启动管理节点命令写入开机执行, 编辑vi /etc/rc.local, 在exit 0前面加入
{% highlight configure %}
ndb_mgmd -f /var/lib/mysql-cluster/config.ini --configdir=/var/lib/mysql-cluster/
# 以上命令在exit 0之前
{% endhighlight %}

执行下面命令，可以看到ndb_mgmd启动，SQL节点和数据节点还未连接
{% highlight bash %}
$ netstat -plntu
$ ndb_mgm -e show
{% endhighlight %}

![]({{ site.url }}/assets/img/mysql-cluster/mgm1.png){:style="width:550px"}
![]({{ site.url }}/assets/img/mysql-cluster/mgm2.png){:style="width:550px"}

## 2 数据节点
两个数据节点的操作相同
{% highlight bash %}
$ groupadd mysql
$ useradd -g mysql mysql
$ mv mysql /usr/local
$ cd /usr/local/mysql
$ chown -R root:mysql /usr/local/mysql/
$ cd /usr/local/mysql/
$ mv bin/* /usr/local/bin/
$ rm -rf bin/
$ ln -s /usr/local/bin /usr/local/mysql/
{% endhighlight %}

编辑配置文件 /etc/my.cnf
{% highlight cnf %}
[mysqld]
datadir=/usr/local/mysql/data
socket=/tmp/mysql.sock
user=mysql

ndbcluster
ndb-connectstring=192.168.3.21

[mysql_cluster]

ndb-connectstring=192.168.3.21

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
{% endhighlight %}

执行下面的命令
{% highlight bash %}
$ mkdir -p /var/lib/mysql-cluster
$ chown -R mysql /var/lib/mysql-cluster
$ cd /var/lib/mysql-cluster
$ ndbd --initial
{% endhighlight %}

## 3 SQL 节点
{% highlight bash %}
$ groupadd mysql
$ useradd -g mysql mysql
$ mv mysql /usr/local
$ cd /usr/local/mysql
$ mv bin/* /usr/local/bin/
$ rm -rf bin/
$ ln -s /usr/local/bin /usr/local/mysql/
$ mysqld --initialize --user=mysql
$ chown -R root:mysql /usr/local/mysql/
$ chown -R mysql /usr/local/mysql/data
$ cp support-files/mysql.server /etc/init.d/mysql
$ /lib/systemd/systemd-sysv-install enable mysql
{% endhighlight %}

编辑配置文件 /etc/my.cnf
{% highlight cnf %}
[mysqld]
datadir=/usr/local/mysql/data
socket=/tmp/mysql.sock
user=mysql

ndbcluster
ndb-connectstring=192.168.3.21

[mysql_cluster]

ndb-connectstring=192.168.3.21

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
{% endhighlight %}

最后开启 MySQL
{% highlight bash %}
$ service mysql start
{% endhighlight %}

## 4 检查
在管理节点执行
{% highlight bash %}
$ ndb_mgm -e show
{% endhighlight %}

看到如下图，则表示连接成功。
![]({{ site.url }}/assets/img/mysql-cluster/mgm3.png){:style="width:550px"}
