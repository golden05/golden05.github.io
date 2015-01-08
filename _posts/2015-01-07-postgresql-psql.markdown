---
layout: post
title: "Postgresql-psql"
date: 2015-01-07T20:32:58+08:00
categories: Postgresql psql
---

#Environment Variables
PGHOST PGPORT PGUSER PGPASSWORD
PSQL_HISTORY 
PSQLRC  .psqlrc 配置文件地方和名称

		\set PROMPT1 '%n@%M:%>%x %/# ' logged  (%n), host server (%M)  port (%>)  transaction status (%x)  database (%/)
		\connect postgis_book change to another database
		\timing on  每次执行的时间
		
		\set AUTOCOMMIT off  关闭提交事务
		UPDATE census.facts SET short_name = 'This is a mistake.';  批处理
		ROLLBACK;  取消更新
		COMMIT;  执行更新
		
		\set eav 'EXPLAIN ANALYZE VERBOSE'  设置快捷键
		:eav SELECT COUNT(*) FROM pg_tables;  使用快捷键
		
		\set HISTSIZE 10  历史命令次数
		\set HISTFILE ~/.psql_history- :HOST - :DBNAME  按主机和数据库名保留历史命令
		
		\!  执行操作系统命令
		
		\watch  间隔秒数
		SELECT datname, waiting, query
		FROM pg_stat_activity
		WHERE state = 'active' AND pid != pg_backend_pid(); \watch 10
		
		\dt+ pg_catalog.pg_t*  list all tables 以pg_t开头
		\d+  表名 显示一个表的结构细节
		
		\copy  
##\? 查看可用命令
##\h 
		
		psql -f some_script_file  执行相关文件
		psql -d postgresql_book -c "DROP TABLE IF EXISTS dross; CREATE SCHEMA staging;" 多个命令
		