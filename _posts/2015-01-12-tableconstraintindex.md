---
layout: post
title: "TableConstraintIndex"
description: "Postgresql Tables Constraints and Index"
category: Postgresql
tags: [PostgreSQL,Table,Constraint,Index]
---
{% include JB/setup %}

#Tables
##表的创建
		CREATE TABLE logs (
			log_id serial PRIMARY KEY, 
			user_name varchar(50), 
			description text, 
			log_ts timestamp with time zone NOT NULL DEFAULT current_timestamp); 
		CREATE INDEX idx_logs_log_ts ON logs USING btree (log_ts);

##表的继承
###父子设计是非常好的分割数据的方式
###PostgreSQL 能记住父表的变化并自动传递给子表
###查询父表时自动包含子表所有行
###主键、唯一性约束和索引不继承 
###检查约束继承 并且子表可以有附加自己的检查约束 check constraints

		CREATE TABLE logs_2011 (PRIMARY KEY(log_id)) INHERITS (logs);
		CREATE INDEX idx_logs_2011_log_ts ON logs USING btree(log_ts);
		ALTER TABLE logs_2011 ADD CONSTRAINT chk_y2011
		 CHECK (log_ts >= '2011-1-1'::timestamptz
		   AND log_ts < '2012-1-1'::timestamptz );

##不加日志的表
不太重要的表，不需要重建  UNLOGGED 高速度

		CREATE UNLOGGED TABLE web_sessions (
			session_id text PRIMARY KEY,
			add_ts timestamptz,
			upd_ts timestamptz,
			session_state xml);

##类型
根据自定义数据类型创建表

		CREATE TYPE basic_user AS (user_name varchar(50), pwd varchar(10));
		CREATE TABLE super_users OF basic_user (CONSTRAINT pk_su PRIMARY KEY (user_name));

当修改类型后，对应的表结构自动调整 类似于继承		

		ALTER TYPE basic_user ADD ATTRIBUTE phone varchar(10) CASCADE;
		
#约束
##外键约束
创建一个在表facts和fact_types之间的外键关系，增加了在更新fact_type_id的层叠删除规则
主键和唯一约束会自动建立索引，所以这里要手动创建索引

		set search_path=census, public;
		ALTER TABLE facts ADD CONSTRAINT fk_facts_1 FOREIGN KEY (fact_type_id)
		REFERENCES lu_fact_types (fact_type_id) 
		ON UPDATE CASCADE ON DELETE RESTRICT;  
		CREATE INDEX fki_facts_1 ON facts (fact_type_id);
##唯一约束
不可以有null值

		ALTER TABLE logs_2011 ADD CONSTRAINT uq UNIQUE (user_name,log_ts);

##检查约束 Check constraints

		ALTER TABLE logs ADD CONSTRAINT chk CHECK (user_name = lower(user_name));

##排他约束exclusion constraints
通常使用 GiST索引，组合使用B-Tree 安装btree_gist扩展

		CREATE TABLE schedules(id serial primary key, room smallint, time_slot tstzrange);
		ALTER TABLE schedules ADD CONSTRAINT ex_schedules
		EXCLUDE USING gist (room WITH =, time_slot WITH &&);

#索引
允许在一个表里使用多个索引，在一个模式里索引名必须唯一
##B-Tree
默认索引
##GiST
全文搜索、空间数据、科学数据、非结构化数据和层次化数据
##GIN
倒排的针对全文搜索、jsonb。hstore、pg_trim扩展利用它 是GiST 的无损派生
##SP-GiST 分割空间
几何、点、盒子和文本，PostGIS空间扩展使用
##hash 避免使用
##B-Tree-GiST/B-Tree-GIN
多个列的组合索引使用
##操作类operator classes
每个索引的数据类型一个操作类，没有用默认的
系统表pg_opclass提供全部操作类
btree支持哪一个数据类型和操作类

		SELECT am.amname AS index_method, opc.opcname AS opclass_name,
		opc.opcintype::regtype AS indexed_type, opc.opcdefault AS is_default
		FROM pg_am am INNER JOIN pg_opclass opc ON opc.opcmethod = am.oid
		WHERE am.amname = 'btree'
		ORDER BY index_method, indexed_type, opclass_name;

明确操作类创建索引		

		CREATE INDEX idx1 ON census.lu_tracts USING btree (tract_name text_pattern_ops);

使用默认的操作类创建索引		

		CREATE INDEX idx2 ON census.lu_tracts USING btree (tract_name);
##函数索引
文本索引，针对列的函数

		CREATE INDEX fidx ON featnames_short
		 USING btree (upper(fullname) varchar_pattern_ops);

##Partial部分索引
过滤索引  能满足 where 子句 条件

		CREATE TABLE subscribers (
		 id serial PRIMARY KEY,
		 name varchar(50) NOT NULL, type varchar(50),
		 is_active boolean);
		CREATE UNIQUE INDEX uq ON subscribers USING btree(lower(name)) WHERE is_active;

查询时，使用的条件必须是创建索引条件的子集

		CREATE OR REPLACE VIEW vw_subscribers_current AS
		SELECT id, lower(name) As name FROM subscribers WHERE is_active = true;

具体查询		

		SELECT * FROM vw_active_subscribers WHERE user_name = 'sandy';
##多列索引
		CREATE INDEX idx ON subscribers 
		  USING btree (type, upper(name) varchar_pattern_ops);

使数据库只扫描已索引的 index-only scans 不管基础表
 
