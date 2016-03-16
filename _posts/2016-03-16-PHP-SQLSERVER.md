---
layout: post
title:  "PHP 如何连接 SQL Server"
date:   2016-03-16 20:06:09 +0800
categories: articles
keywords: php, sql server, freetds, mssql, mac, linux, windows
---
**Mac[Linux的方法应该大同小异]**

[解决方案]freetds + mssql

以下操作主要是针对我的开发环境来写的，如果你看到这篇文章，请不要完全照搬。先理解，再根据自己的情况处理。

{% highlight bash %}
$ brew install freetds
$ cd /usr/local/Cellar/freetds/0.95.80/etc
$ vi freetds.conf
{% endhighlight %}

在底部增加下面内容
{% highlight bash %}
[sql-server-2012]
    host = sql-server-ip
    port = sql-server-port
    tds version = 8.0
{% endhighlight %}

然后测试连接
{% highlight bash %}
$ cd /usr/local/Cellar/freetds/0.95.80/bin
$ ./tsql -S sql-server-2012 -U username -P password
{% endhighlight %}

如果连接成功，会显示
{% highlight bash %}
1>use db
2>go
{% endhighlight %}

出现[1>]则表示连接成功
如果在[2>]后面输入"go"以后，出现"The server principal "username" is not able to access the database "db" under the current security context."
则说明你对该db没有操作权限。

不要问我为什么会知道没有操作权限的提示。

官网下载对应当前php版本的源码[我的是5.6.19]
解压源码
这里我遇到个坑，和我自己的基础知识不牢固有关。我是使用的brew安装的php5.5和php5.6，通过php-version管理，编译以后发现编译的版本和现在php5.6版本不一致，所以以下命令都使用绝对路径
{% highlight bash %}
$ cd ./php-5.6.19/ext/mssql
$ /usr/local/Cellar/php56/5.6.19/bin/phpize
$ ./configure --with-php-config=/usr/local/Cellar/php56/5.6.19/bin/php-config --with-mssql=/usr/local/Cellar/freetds/0.95.80
$ make
$ make test
# 查看编译的是否正确
{% endhighlight %}

创建几个目录和文件，为了和brew安装的extension保持一致
{% highlight bash %}
$ mkdir /usr/local/Cellar/php56-mssql
$ cp ./php-5.6.19/ext/mssql/mssql.so /usr/local/Cellar/php56-mssql/
$ cd /usr/local/opt
$ ln -sfv ln -sfv /usr/local/Cellar/php56-mssql ./
$ cd /usr/local/etc/php/5.6/conf.d
$ vi ext-mssql.ini
{% endhighlight %}

把下面的字符串输进去

{% highlight bash %}
[mssql]
extension="/usr/local/opt/php56-mssql/mssql.so"
{% endhighlight %}

检查一下是否载入mssql模块
{% highlight bash %}
$ php -m | grep mssql
{% endhighlight %}

重启nginx和php-fpm，在phpinfo里看到mssql就说明成功了。


**Windows**

[解决方案]下载对应的dll，并安装sql server驱动。

[PHP文档](http://php.net/manual/zh/sqlsrv.installation.php)

根据自己的系统和php版本下载对应的sqlsrv扩展。
我的环境是win7，PHP 5.6，下的是3.0，[下载地址](https://www.microsoft.com/en-us/download/details.aspx?id=36434)

安装的时候会要求指定dll的目录，即php的扩展目录。

还有需要的sql server的[驱动](https://msdn.microsoft.com/en-us/library/cc296170.aspx)。
[[下载地址]](https://www.microsoft.com/en-us/download/details.aspx?id=20098)

在php.ini里增加以下的扩展。查看phpinfo里的Thread Safety，如果是enabled，则使用ts[线程安全]，否则使用nts[非线程安全]

extension=php_sqlsrv_56_ts.dll

extension=php_pdo_sqlsrv_56_ts.dll
