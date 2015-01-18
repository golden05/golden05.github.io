---
layout: post
title: "PostgreSQL Performance"
description: "PostgreSQL Performance"
category: Postgresql
tags: [PostgreSQL,Functions]

---
{% include JB/setup %}

#EXPLAIN
##EXPLAIN Options
参数  ANALYZE 与期望值的比较
	 VERBOSE 报告到列级
	 BUFFERS 
##样本运行和输出
		
		ALTER TABLE census.hisp_pop DROP CONSTRAINT IF EXISTS hisp_pop_pkey;
		
		EXPLAIN (ANALYZE) SELECT tract_id, hispanic_or_latino
		FROM census.hisp_pop
		WHERE tract_id = '25025010103';
		Seq Scan on hisp_pop
		    (cost=0.00..33.48 rows=1 width=16)    0.00启动成本  33.48这一步的总成本
		    (actual time=0.205..0.339 rows=1 loops=1)    
		    Filter: ((tract_id)::text = '25025010103'::text)
		    Rows Removed by Filter: 1477
		Total runtime: 0.360 ms
		
9.4版输出 
		
		Seq Scan on hisp_pop
		    (cost=0.00..33.48 rows=1 width=16) 
			(actual time=0.213..0.346 rows=1 loops=1)
		    Filter: ((tract_id)::text = '25025010103'::text)
		    Rows Removed by Filter: 1477
		Planning time: 0.095 ms                把时间分为计划时间和运行时间
		Execution time: 0.381 ms
		
增加主键索引

		ALTER TABLE census.hisp_pop ADD CONSTRAINT hisp_pop_pkey PRIMARY KEY(tract_id);
		
		Index Scan using idx_hisp_pop_tract_id_pat on hisp_pop
		    (cost=0.28..8.29 rows=1 width=16)      成本减少从33.48到8.29
			(actual time=0.018..0.019 rows=1 loops=1)
		    Index Cond: ((tract_id)::text = '25025010103'::text)
		Planning time: 0.110 ms
		Execution time: 0.046 ms
		
更复杂的查询，包括更多的子步骤 GROUP BY SUM

		EXPLAIN (ANALYZE)
		SELECT left(tract_id,5) AS county_code, SUM(white_alone) As w
		FROM census.hisp_pop
		WHERE tract_id BETWEEN '25025000000' AND '25025999999'
		GROUP BY county_code;
		
		HashAggregate     总的父步骤 
		    (cost=29.57..32.45 rows=192 width=16) 
			(actual time=0.664..0.664 rows=1 loops=1)
		    Group Key: "left"((tract_id)::text, 5)
		    -> Bitmap Heap Scan on hisp_pop         子步骤
		        (cost=10.25..28.61 rows=192 width=16) 
				(actual time=0.441..0.550 rows=204 loops=1)
		        Recheck Cond:
		            (((tract_id)::text >= '25025000000'::text) AND
		            ((tract_id)::text <= '25025999999'::text))
		        Heap Blocks: exact=15
		        -> Bitmap Index Scan on hisp_pop_pkey    孙子步骤
		            (cost=0.00..10.20 rows=192 width=0) 
					(actual time=0.421..0.421 rows=204 loops=1)
		            Index Cond:
		                (((tract_id)::text >= '25025000000'::text) AND
		                ((tract_id)::text <= '25025999999'::text))
		Planning time: 4.835 ms
		Execution time: 0.732 ms
		
#收集统计
使用pg_stat_statements扩展，在postgresql.conf设置
	
		shared_preload_libraries = 'pg_stat_statements'		
		pg_stat_statements.max = 10000 
		pg_stat_statements.track = all

在使用数据库中应用
		
		CREATE EXTENSION pg_stat_statements;
		
有一个 pg_stat_statements的视图，显示当前用户能读取的所有数据库

有一个pg_stat_statements_reset函数，刷新查询log，被超级用户使用

例如显示在指定数据库中5个最费时的查询

		SELECT
		    query, calls, total_time, rows,
		    100.0*shared_blks_hit/nullif(shared_blks_hit+shared_blks_read,0) AS hit_percent
		FROM pg_stat_statements As s INNER JOIN pg_database As d On d.oid = s.dbid
		WHERE d.datname = 'postgresql_book'
		ORDER BY total_time DESC LIMIT 5;
		
#指导查询计划
考虑元素：索引表现、cost设置、策略设置和数据分布
##策略设置
默认所有策略设置都启用，给予最大灵活性，通过禁用，两个偶尔被禁用enable_nestloop和 enable_seqscan，最耗时、最后手段
###索引是否有用
检查查询是否使用索引 查询 pg_stat_user_indexes 和 pg_stat_user_tables 视图 

创建了一个索引

		CREATE INDEX idx_lu_fact_types ON census.lu_fact_types USING gin (fact_subcats);

设置了全表顺序扫描分析
		
		set enable_seqscan = true;
		EXPLAIN (ANALYZE)
		SELECT *
		FROM census.lu_fact_types
		WHERE fact_subcats && '{White alone, Black alone}'::varchar[];
		
		Seq Scan on lu_fact_types
		    (cost=0.00..2.85 rows=2 width=200) (actual time=0.066..0.076 rows=2 loops=1)
			Filter: (fact_subcats && '{"White alone","Black alone"}'::character varying[])
			Rows Removed by Filter: 66
		Planning time: 0.182 ms
		Execution time: 0.108 ms

关闭顺序扫描，发现运用了索引
		
		set enable_seqscan = false;
		EXPLAIN (ANALYZE)
		SELECT *
		FROM census.lu_fact_types
		WHERE fact_subcats && '{White alone, Black alone}'::varchar[];
		
		Bitmap Heap Scan on lu_fact_types
			(cost=12.02..14.04 rows=2 width=200) (actual time=0.058..0.058 rows=2 loops=1)
			Recheck Cond: (fact_subcats && '{"White alone","Black alone"}'::character varying[])
			Heap Blocks: exact=1
				-> Bitmap Index Scan on idx_lu_fact_types
		        (cost=0.00..12.02 rows=2 width=0) (actual time=0.048..0.048 rows=2 loops=1)
		        Index Cond: (fact_subcats && '{"White alone","Black alone"}'::character varying[])
		Planning time: 0.230 ms
		Execution time: 0.119 ms

对比发现是顺序扫描比索引成本少，一般会采用顺序扫描

###表统计
planner依赖数据的聚集统计，设置STATISTICS值

		SELECT
		    attname As colname,
		    n_distinct,
		    most_common_vals AS common_vals,
		    most_common_freqs As dist_freq
		FROM pg_stats
		WHERE tablename = 'facts'
		ORDER BY schemaname, tablename, attname;
		colname      | n_distinct | common_vals      | dist_freq
		-------------+------------+------------------+-----------------
		fact_type_id |         68 | {135,113...      | {0.0157,0.0156333,...
		perc         |        985 | {0.00,...        | {0.1845,0.0579333,0.056...
		tract_id     |       1478 | {25025090300...  | {0.00116667,0.00106667,0.0...
		val          |       3391 | {0.000,1.000,2...| {0.2116,0.0681333,0...
		yr           |          2 | {2011,2010}      | {0.748933,0.251067}
		
pg_stats表在后台不断更新，当有大数据被更新，应手动运行VACUUM ANALYZE 从表里迅速删除行

对于经常参加join的和重度被使用where子句的列应该增加设置STATISTICS值增加样本数据

		ALTER TABLE census.facts ALTER COLUMN fact_type_id SET STATISTICS 1000;
		
###Random Page Cost 和 Quality of Drives
随机页成本是比较顺序读取和随机读取的成本，一般更快的物理磁盘，比例值较低 机械磁盘rpc默认4，固态硬盘、san或云存储要调整
可以按每个表空间、每个服务器、每个数据库设置rpc值

在数据库层是设置在postgresql.conf ，

在不同的磁盘设置在表空间 

		ALTER TABLESPACE pg_default SET (random_page_cost=2);
		
High-end NAS/SAN: 2.5 or 3.0

Amazon EBS and Heroku: 2.0

iSCSI and other mediocre SANs: 6.0, but varies widely

SSDs: 2.0 to 2.5

NvRAM (or NAND): 1.5

##缓冲
安装pg_buffercache扩展，使用缓冲用pg_buffercache视图
		
		CREATE EXTENSION pg_buffercache;

我的表行是否在缓冲里
		
		SELECT
		    C.relname,
		    COUNT(CASE WHEN B.isdirty THEN 1 ELSE NULL END) As dirty_buffers,
		    COUNT(*) As num_buffers
		FROM
		    pg_class AS C INNER JOIN
		    pg_buffercache B ON C.relfilenode = B.relfilenode INNER JOIN
		    pg_database D ON B.reldatabase = D.oid AND D.datname = current_database()
		WHERE C.relname IN ('facts','lu_fact_types')
		GROUP BY C.relname;
		
在postgresql.conf 设置 shared_buffers来修改缓冲大小
在9.4版安装pg_prewarm扩展来加载已运行的表格数据到内存里

##写更好的查询
一直使用左Join，没有使用内部Join
###过度使用子查询
		
		SELECT tract_id,
		    (SELECT COUNT(*) FROM census.facts As F WHERE F.tract_id = T.tract_id) As num_facts,
		    (SELECT COUNT(*)
		    FROM census.lu_fact_types As Y
		    WHERE Y.fact_type_id IN (
		        SELECT fact_type_id
		        FROM census.facts F
		        WHERE F.tract_id = T.tract_id
		    )
		    ) As num_fact_types
		FROM census.lu_tracts As T;

简化过度使用子查询
		
		SELECT T.tract_id,
		    COUNT(f.fact_type_id) As num_facts,
		    COUNT(DISTINCT fact_type_id) As num_fact_types
		FROM census.lu_tracts As T LEFT JOIN census.facts As F ON T.tract_id = F.tract_id
		GROUP BY T.tract_id;
		
###避免SELECT *

		CREATE OR REPLACE VIEW vw_stats AS
		SELECT tract_id,
		    (SELECT COUNT(*) FROM census.facts As F WHERE F.tract_id = T.tract_id) As num_facts,
		    (SELECT COUNT(*)
		    FROM census.lu_fact_types As Y
		    WHERE Y.fact_type_id IN (
		        SELECT fact_type_id
		        FROM census.facts F
		        WHERE F.tract_id = T.tract_id
		    )
		    ) As num_fact_types
		FROM census.lu_tracts As T;

好的应用

		SELECT tract_id FROM vw_stats;

差的应用

		SELECT * FROM vw_stats;
		
###利用好CASE
使用子查询代替CASE

		SELECT T.tract_id, COUNT(*) As tot, type_1.tot AS type_1
		FROM
		    census.lu_tracts AS T LEFT JOIN
		    (SELECT tract_id, COUNT(*) As tot
		        FROM census.facts
		        WHERE fact_type_id = 131
		        GROUP BY tract_id
		    ) As type_1 ON T.tract_id = type_1.tract_id LEFT JOIN
		    census.facts AS F ON T.tract_id = F.tract_id
		GROUP BY T.tract_id, type_1.tot;

使用CASE代替子查询
		
		SELECT T.tract_id, COUNT(*) As tot,
		    COUNT(CASE WHEN F.fact_type_id = 131 THEN 1 ELSE NULL END) AS type_1
		FROM census.lu_tracts AS T LEFT JOIN census.facts AS F
		ON T.tract_id = F.tract_id
		GROUP BY T.tract_id;
		
###利用 Filter 代替 CASE
9.4版新特性，在聚集表达式代替，不仅语法好，且性能更好

		SELECT T.tract_id, COUNT(*) As tot,
		    COUNT(*) FILTER(WHERE F.fact_type_id = 131) AS type_1
		FROM census.lu_tracts AS T LEFT JOIN census.facts AS F
		ON T.tract_id = F.tract_id
		GROUP BY T.tract_id;