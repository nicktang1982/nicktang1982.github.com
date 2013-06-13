---
layout: post
title: "安装Postfix+Policyd"
date: 2013-06-09 00:09
updated: 2013-06-09 00:09
comments: true
keywords: postfix,policyd,cluebringer
description: "邮件外发使用Postfix+Policyd，实现对用户外发邮件数量控制"
categories: postfix
tags: [postfix, policyd]
---

邮件外发使用Postfix+Policyd，实现对用户外发邮件数量控制。

<!--more-->

## 安装Postfix

	make makefiles CCARGS='-DDEF_CONFIG_DIR=\"/var/postfix\"'
	
	mv /usr/sbin/sendmail /usr/sbin/sendmail.OFF
	mv /usr/bin/newaliases /usr/bin/newaliases.OFF
	mv /usr/bin/mailq /usr/bin/mailq.OFF
	chmod 755 /usr/sbin/sendmail.OFF /usr/bin/newaliases.OFF \
	/usr/bin/mailq.OFF
	
	groupadd postfix
	useradd -g postfix postfix
	groupadd postdrop
	
	make install

## 配置Postfix

	cd /var/postfix
	vi main.cf

以下为`main.cf`的内容，推荐用于外发：

``` ini main.cf
	myhostname = mx.example.com
	#myorigin = example.com
	mydestination =
	mynetworks = 10.100.0.0/23
	local_recipient_maps =
	local_transport = error:local mail delivery is disabled
	
	maximal_queue_lifetime = 12h
	bounce_queue_lifetime = 6h
	
	transport_maps = hash:/var/postfix/transport
	
	smtpd_recipient_restrictions =
	        check_policy_service inet:127.0.0.1:10031,
	        permit_mynetworks,
	        permit_sasl_authenticated,
	        reject_non_fqdn_hostname,
	        reject_non_fqdn_sender,
	        reject_non_fqdn_recipient,
	        reject_unauth_destination,
	        reject_unauth_pipelining,
	
	smtpd_end_of_data_restrictions =
	        check_policy_service inet:127.0.0.1:10031,
```

## 管理Postfix

启动：`postfix start`  
停止：`postfix stop`  
重载：`postfix reload`  
重新投递邮件：`postfix flush`  
显示配置：`postconf`  
`postmap`  
`mailq`  
删除所有邮件：`postsuper -d ALL`  

## 配置Logrotate

`/etc/logrotate.d/maillog`文件

``` ini maillog
/var/log/maillog {
    daily
    rotate 61
    copytruncate
    compress
    notifempty
    missingok
    postrotate
    /usr/sbin/postfix reload > /dev/null 2>&1 || true
    endscript
}
```
取消`/etc/logrotate.d/syslog`中`maillog`相关设置  
手动执行一下`logrotate -f /etc/logrotate.conf -v`

## Policyd环境要求

### LAMP

- php5+mysql+apache
- web管理需要pdo-mysql支持

### Perl

- Net::Server >= 0.96
- Net::CIDR
- Config::IniFiles (Debian based: libconfig-inifiles-perl, RPM based: perl-Config-IniFiles)
- Cache::FastMmap (Debian based: libcache-fastmmap-perl, RPM based: perl-Cache-FastMmap)
- Mail::SPF

## 安装LAMP

	useradd -g mysql mysql
	tar -xzf  mysql-5.1.60.tar.gz
	cd mysql-5.1.60
	
	./configure \
	  --prefix=/var/mysql \
	  --enable-static \
	  --with-charset=utf8 \
	  --with-extra-charsets=gbk,gb2312,utf8 \
	  --enable-thread-safe-client \
	  --without-debug \
	  --with-fast-mutexes \
	  --enable-assembler \
	  --with-plugins=innobase,partition
	
	make
	make install
	cp support-files/my-huge.cnf /etc/my.cnf
	cd /var/mysql
	bin/mysql_install_db --user=mysql
	chown -R root  .
	chown -R mysql var
	chgrp -R mysql .
	bin/mysqld_safe --user=mysql &
	bin/mysqladmin -u root password abc123!

建立数据库

	bin/mysql -u root -p
	create database policyd;

## 安装cluebringer

	tar zxvf cluebringer-2.0.11.tar.gz
	cd cluebringer-2.0.11\database
	
	for i in  core.tsql access_control.tsql quotas.tsql amavis.tsql checkhelo.tsql checkspf.tsql greylisting.tsql
	do
	       ./convert-tsql mysql $i
	done > policyd.mysql
	
	/var/mysql/bin/mysql -u root -p policyd < policyd.mysql

若mysql5.5，需更改convert-tsql，将TYPE=InnoDB改成ENGINE=InnoDB

	cp cluebringer.conf /etc
	mkdir /usr/local/lib/policyd-2.0
	cp -r cbp /usr/local/lib/policyd-2.0/
	cp cbpadmin /usr/local/bin/
	cp cbpolicyd /usr/local/sbin/
	mkdir /var/apache/htdocs/cluebringer
	cp webui/* -r /var/apache/htdocs/cluebringer

更改/etc/cluebringer.conf中的相关数据库设置（填写数据库root密码）

``` ini cluebringer.conf
	[database]
	#DSN=DBI:SQLite:dbname=policyd.sqlite
	DSN=DBI:mysql:database=policyd;host=localhost
	Username=root
	Password=abc123!
```

更改/var/apache/htdocs/cluebringer/includes/config.php中相关数据库设置（更改数据库名称及root密码）

``` php config.php
	<?php
	
	# mysql:host=xx;dbname=yyy
	#
	# pgsql:host=xx;dbname=yyy
	#
	# sqlite:////full/unix/path/to/file.db?mode=0666
	#
	#$DB_DSN="sqlite:////tmp/cluebringer.sqlite";
	$DB_DSN="mysql:host=localhost;dbname=policyd";
	$DB_USER="root";
	$DB_PASS="abc123!";
	
	
	#
	# THE BELOW SECTION IS UNSUPPORTED AND MEANT FOR THE ORIGINAL SPONSOR OF V2
	#
	
	#$DB_POSTFIX_DSN="mysql:host=localhost;dbname=postfix";
	#$DB_POSTFIX_USER="root";
	#$DB_POSTFIX_PASS="";
	
	?>
```

## 安装Perl环境

	cpan -i Net::Server
	cpan -i Net::CIDR
	cpan -i Config::IniFiles
	cpan -i Cache::FastMmap
	cpan -i Mail::SPF
	
	wget http://search.cpan.org/CPAN/authors/id/C/CA/CAPTTOFU/DBD-mysql-4.020.tar.gz
	perl Makefile.PL "--libs=-L/var/mysql/lib -lmysqlclient -lz -lm -lcrypt -lnsl" --cflags=-I/var/mysql/include
	make
	make test
	make install

## 启动Policyd

	cbpolicyd

## 管理Policyd

管理页面：http://localhost/cluebringer

	crontab -e

``` ini crontab
0 23 * * * (/usr/local/bin/cbpadmin --cleanup)
```

