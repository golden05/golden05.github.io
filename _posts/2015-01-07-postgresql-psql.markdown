---
layout: post
title: "Postgresql-psql"
date: 2015-01-07T20:32:58+08:00
categories: Postgresql
tags: [postgresql,psql]
---

#Environment Variables 环境变量
PGHOST PGPORT PGUSER PGPASSWORD

PSQL_HISTORY  历史命令 默认~/.psql_history

PSQLRC  .psqlrc 配置文件地方和名称

.psqlrc 配置文件内容

		\set PROMPT1 '%n@%M:%>%x %/# ' 自定义提示符 登录用户(%n), 服务器机器(%M)  port (%>)  事务状态(%x)  数据库 (%/)
		\connect postgis_book 连接另一个数据库
		\timing on  每次执行的时间
		
		\set AUTOCOMMIT off  关闭提交事务
		UPDATE census.facts SET short_name = 'This is a mistake.';  批处理
		ROLLBACK;  取消更新
		COMMIT;  执行更新
		
		\set eav 'EXPLAIN ANALYZE VERBOSE'  设置快捷键
		:eav SELECT COUNT(*) FROM pg_tables;  使用快捷键
		
		\set HISTSIZE 10  历史命令次数
		\set HISTFILE ~/.psql_history- :HOST - :DBNAME  按主机和数据库名保留历史命令

##psql 执行命令		

		\!  执行操作系统命令
		
		\watch  间隔秒数
		SELECT datname, waiting, query
		FROM pg_stat_activity
		WHERE state = 'active' AND pid != pg_backend_pid(); \watch 10
		
		\dt+ pg_catalog.pg_t*  显示pg_catalog模式下所有以pg_t开头的表
		\d+  表名 显示一个表的结构细节
		
		\copy  导入导出数据 
		导入用from 
		\connect postgresql_book
		\cd /postgresql_book/ch03
		\copy staging.factfinder_import FROM DEC_10_SF1_QTH1_with_ann.csv CSV
		\copy sometable FROM somefile.txt DELIMITER '|';
		\copy sometable FROM somefile.txt NULL As '';
		
		CREATE TABLE dir_list (filename text);
		\copy dir_list FROM PROGRAM 'dir C:\projects /b' 从操作系统命令导入 curl ls wget
		
		导出用to
		\connect postgresql_book
		\copy (SELECT * FROM staging.factfinder_import  WHERE s01 ~ E'^[0-9]+' ) TO '/test.tab'
		WITH DELIMITER E'\t' CSV HEADER
		\copy staging.factfinder_import TO '/test.csv' WITH CSV HEADER QUOTE '"' FORCE QUOTE *
		
		
##基本报表 
		psql -d postgresql_book -H -c
		"SELECT category, count(*) As num_per_cat
		FROM pg_settings
		WHERE category LIKE '%Query%'
		GROUP BY category
		ORDER BY category;" -o test.html  html格式输出
		
##\? 查看可用命令
##\h 
		
		psql -f some_script_file  执行相关文件
		psql -d postgresql_book -c "DROP TABLE IF EXISTS dross; CREATE SCHEMA staging;" 多个命令
		