---
layout: post
title: "PostgreSQL Replication External"
description: "PostgreSQL Replication and External"
category: Postgresql
tags: [PostgreSQL,Replication,External]
---
{% include JB/setup %}

#复制综述
##复制行话
###Master
当前是一个主，规划在以后的版本有多个主服务器
###Slave
类似于订阅或代理,只读
###Write-ahead log WAL
日志跟踪全部事务处理，阶段性复制就是系统把日志给Slave，再由他运行
###Synchronous
直到一个从服务更新完成，主服务的事务才算完成。
###异步
有可能在主机宕机时部分事务在从服务失败
###Streaming
流复制模式，依赖PostgreSQL连接协议传递WAL
###层叠复制
收到附近的从服务代替主服务，当一个既扮演接受角色又扮演发送角色称为cascading standby
###Remastering
使得一个Slave成为Master，9.2需要WAL代替流复制，需要所有从服务去再克隆，9.3只读流，从服务不需要再克隆，9.4还需要重启，未来将改变
##PostgreSQL复制演变
###第三方复制
Slony Bucardo， Postgres-XC类似于 Oracle RAC.
##设置复制
###配置主服务
创建一个复制角色	

		 CREATE ROLE pgrepuser REPLICATION LOGIN PASSWORD 'woohoo';
		 
修改postgresql.conf配置

		listen_addresses = *
		wal_level = hot_standby
		archive_mode = on
		max_wal_senders = 2
		wal_keep_segments = 10
		
在postgresql.conf增加archive_command命令配置

		archive_command = 'cp %p ../archive/%f'
		
		archive_command = 'rsync -av %p postgres@192.168.0.10:archive/%f'

在pg_hba.conf包含规则允许slave扮演主服务的复制agent

		host replication pgrepuser 192.168.0.0/24 md5
		
停下系统，复制在data目录下所有文件除了 pg_xlog 和 pg_log到从服务，确保pg_xlog 和 pg_log都出现在从服务

###配置从服务
创建一个新的instance，同样版本包括小版本，同样操作系统同样补丁

停下从服务 

把主服务的Data数据覆盖从服务

在postgresql.conf 增加下列配置

		hot_standby = on
		
修改监听端口，可以和主服务默认端口不一样

在Data 目录创建一个新文件recovery.conf，在下面第二行列出主服务的参数

		standby_mode = 'on'
		primary_conninfo = 'host=192.168.0.1 port=5432 user=pgrepuser password=woohoo'
		trigger_file = 'failover.now'
		
如果发现从服务不能很快地返回WAL，在recovery.conf加一行到缓冲

		restore_command = 'cp %p ../archive/%f'
		
###初始化复制过程
启动从服务在主服务前

#外部数据包装器FDW