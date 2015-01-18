---
layout: post
title: "Postgresql-admin"
date: 2015-01-07T08:53:35+08:00
categories: Postgresql
tags: [PostgreSQL,admin]

---

#配置文件
### postgresql.conf
常规设置： 内存分配、新数据存储地方、监听IP和日志地方
修改postgresql.conf设置`ALTER SYSTEM`

		ALTER SYSTEM set work_mem = 8192;
		SELECT pg_reload_conf();  <!--重启命令-->
		
查询postgresql.conf设置， 使用pg_settings视图
		
		SELECT name, context 1, unit 2, 
		    setting, boot_val, reset_val 3
		FROM pg_settings
		WHERE name IN (
			'listen_addresses',
			'max_connections',
			'shared_buffers',
			'effective_cache_size',
			'work_mem',
			'maintenance_work_mem'
		)
		ORDER BY context, name;

###pg_hba.conf
安全设置：用户、ip和认证模式

认证方法： ident, trust, md5,password，peer，reject
trust 信赖不需要密码 password 明文密码 md5 加密密码 ident 使用pg_ident.conf不需要password peer 客户端操作系统的用户名本地连接 reject 立刻断绝

		pg_ctl reload -D your_data_directory_here  <!--重新读取配置文件postgresql.conf, pg_hba.conf 不重启系统-->
		service postgresql-9.4 reload 

###pg_ident.conf
映射操作系统用户到数据库用户


超级用户查询配置文件存放地方

		SELECT name, setting FROM pg_settings WHERE category = 'File Locations';
		
#管理连接

		SELECT * FROM pg_stat_activity; <!--查询近期的连接，有最后的查询语句-->
		SELECT pg_cancel_backend(*procid*) <!--在某个连接上取消所有查询-->
		SELECT pg_terminate_backend(*procid*) <!--杀手连接-->
		SELECT pg_terminate_backend(pid) 
			FROM pg_stat_activity 
				WHERE usename = 'some_role'; <!--根据查询同时取消多个连接-->

#Roles
当前版本使用role和group role替代用户和组
默认postgres
例创建角色

		CREATE ROLE leo LOGIN PASSWORD 'king' CREATEDB VALID UNTIL 'infinity';<!--有创建数据库权限，无限期限-->
		CREATE ROLE regina LOGIN PASSWORD 'queen' SUPERUSER VALID UNTIL '2020-1-1 00:00';<!--创建超级角色，在某个时间结束-->
		CREATE ROLE royalty INHERIT; <!--创建组角色，所有成员集成组权限-->
		GRANT royalty TO leo;  <!--把组授予某个角色-->
		SET ROLE royalty;  <!--将自己临时"变成"该组成员-->
		SET SESSION AUTHORIZATION — 为当前会话设置会话用户标识符和当前用户标识符

#Database Creation
		CREATE DATABASE mydb; - 从template1复制
		CREATE DATABASE my_db TEMPLATE my_template_db; 
		UPDATE pg_database SET datistemplate = TRUE WHERE datname = 'mydb'; －设置某一数据库为模版数据库

#使用模式
可以按表或角色组织,在postgresql.conf设置，在查询时如果不在当前模式下需要在from里带上这个模式名

		search_path = '"$user",public'		# schema names
		show search_path -显示当前模式
		CREATE SCHEMA my_extensions; 在安装使用扩展前，建立模式把扩展的所有装入模式
		ALTER DATABASE mydb SET search_path='"$user", public, my_extensions'; 

#许可
细粒度，可以在一个表的每个列不同的许可
##许可种类
SELECT, INSERT, UPDATE, ALTER, EXECUTE, TRUNCATE 
数据库 CONNECT  CREATE TEMP TABLE ， 函数  EXECUTE , 语言 USAGE 

		CREATE ROLE mydb_admin LOGIN PASSWORD 'something'; 创建角色
		CREATE DATABASE mydb WITH owner = mydb_admin; 创建数据库在某个角色
		GRANT some_privilege TO some_role; 授予某个角色某些权利
		GRANT ALL ON ALL TABLES IN SCHEMA public TO mydb_admin WITH GRANT OPTION;  授予mydb_admin角色在public模式全部表全部许可
		GRANT SELECT, REFERENCES, TRIGGER ON ALL TABLES IN SCHEMA my_schema TO PUBLIC;  授予PUBLIC角色在my_schema模式全部表SELECT, REFERENCES, TRIGGER许可
		GRANT SELECT, UPDATE ON ALL SEQUENCES IN SCHEMA my_schema TO PUBLIC; 授予PUBLIC角色在my_schema模式对所有序列SELECT, UPDATE许可
		GRANT USAGE ON SCHEMA my_schema TO PUBLIC; 授予PUBLIC角色在my_schema模式使用许可
		REVOKE EXECUTE ON ALL FUNCTIONS IN SCHEMA my_schema FROM PUBLIC;回收角色PUBLIC在模式my_schema里运行所有函数许可

##默认许可

		GRANT USAGE ON SCHEMA my_schema TO PUBLIC;
		ALTER DEFAULT PRIVILEGES IN SCHEMA my_schema
		GRANT SELECT, REFERENCES ON TABLES TO PUBLIC;

		ALTER DEFAULT PRIVILEGES IN SCHEMA my_schema
		GRANT ALL ON TABLES TO mydb_admin WITH GRANT OPTION;

		ALTER DEFAULT PRIVILEGES IN SCHEMA my_schema
		GRANT SELECT, UPDATE ON SEQUENCES TO public;

		ALTER DEFAULT PRIVILEGES IN SCHEMA my_schema
		GRANT ALL ON FUNCTIONS TO mydb_admin WITH GRANT OPTION;

		ALTER DEFAULT PRIVILEGES IN SCHEMA my_schema
		GRANT USAGE ON TYPES TO PUBLIC;
		
#扩展
functions, tables, data types, casts, languages, operators classes

		SELECT * FROM pg_available_extensions;  查询扩展
		SELECT name, default_version, installed_version, left(comment,30) As 		comment
		FROM pg_available_extensions
		WHERE installed_version IS NOT NULL
		ORDER BY name;  查询已安装的扩展
		
		SELECT pg_catalog.pg_describe_object(d.classid, d.objid, 0) AS 		description
		FROM pg_catalog.pg_depend AS D INNER JOIN pg_catalog.pg_extension AS E
		ON D.refobjid = E.oid
		WHERE
		D.refclassid = 'pg_catalog.pg_extension'::pg_catalog.regclass AND
		deptype = 'e' AND E.extname = 'fuzzystrmatch'; fuzzystrmatch扩展里有哪些

安装扩展有两个目录

		psql -p 5432 -d postgres -f /usr/pgsql-9.0/share/contrib/adminpack.sql 安装扩展 操作系统命令
		CREATE EXTENSION fuzzystrmatch; 

##流行扩展
btree_gist btree_gin postgis fuzzystrmatch hstore pg_trgm (trigram) pgcrypto tsearch xml

#备份和恢复 
## pg_dump pg_dumpall 备份到plain SQL

		pg_dump -h localhost -p 5432 -U someuser -F c -b -v -f mydb.backup mydb 单一数据库压缩备份
		pg_dump -h localhost -p 5432 -U someuser -C -F p -b -v -f mydb.backup mydb  简单文本格式 单一数据库备份
		pg_dump -h localhost -p 5432 -U someuser -F c -b -v -t *.pay* -f pay.backup mydb 压缩备份表格名是pay
		pg_dump -h localhost -p 5432 -U someuser -F c -b -v -n hr -n payroll -f hr.backup mydb   压缩备份hr和payroll模式下所有对象
		pg_dump -h localhost -p 5432 -U someuser -F c -b -v -N public -f all_sch_except_pub.backup mydb  压缩备份 除了public模式其他所有备份
		pg_dump -h localhost -p 5432 -U someuser -j 3 -Fd -f /somepath/a_directory mydb  并行3个作业备份-j 3
		
		pg_dumpall -h localhost -U postgres --port=5432 -f myglobals.sql --globals-only  备份所有角色和表空间
		pg_dumpall -h localhost -U postgres --port=5432 -f myroles.sql --roles-only 仅备份所有角色

## 恢复 两种方法
###psql
		psql -U postgres -f myglobals.sql  用SQL语句恢复
		psql -U postgres --set ON_ERROR_STOP=on -f myglobals.sql 如果有错停下
		psql -U postgres -d mydb -f select_objects.sql 恢复一个指定数据库

###pg_restore   tar custom directory
		CREATE DATABASE mydb;  先建库
		pg_restore --dbname=mydb --jobs=4 --verbose mydb.backup 并行恢复
		pg_restore --dbname=postgres --create --jobs=4 --verbose mydb.backup 创建和恢复一步完成
		pg_restore --dbname=mydb2 --section=pre-data --jobs=4 mydb.backup 仅恢复结构
		
#用表空间管理磁盘存储
pg_default  all user data

pg_global all system data

		CREATE TABLESPACE secondary LOCATION 'C:/pgdata94_secondary'; 创建表空间
		CREATE TABLESPACE secondary LOCATION '/usr/data/pgdata94_secondary';
		ALTER DATABASE mydb SET TABLESPACE secondary;  移动数据库到secondary表空间
		ALTER TABLE mytable SET TABLESPACE secondary; 移动表到secondary表空间
		ALTER TABLESPACE pg_default MOVE ALL TO secondary; 
		
#其他
		path/to/your/bin/pg_ctl -D your_postgresql_data_folder 报错时启动数据库
 pg_log pg_xlog pg_clog 
 
 把超级用户权利给操作系统一般用户
 
 不要设置太大的共享缓冲shared_buffers
 
 在32位的windows中超过512M是动荡的，64位可以是1 GB，在linux内不要超过SHMMAX变量
