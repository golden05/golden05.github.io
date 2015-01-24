---
layout: post
title: "PostgreSQL SQL"
description: "PostgreSQL SQL"
category: Postgresql
tags: [PostgreSQL,SQL]
---
{% include JB/setup %}

#Views
版本9.3推出了可更新视图，就是通过视图修改下面的表数据，还推出了物化视图只有在你发布刷新命令时才重新查询源数据，版本9.4对物化视图加以提升
##单表视图
总是有主键

		CREATE OR REPLACE VIEW census.vw_facts_2011 AS
		SELECT fact_type_id, val, yr, tract_id FROM census.facts WHERE yr = 2011;

在9.3版里可以插入、更新和删除数据，但更新和删除必须要满足创建view的WHERE条件子句
下列语句将仅删除WHERE yr = 2011的并且val = 0的数据

		DELETE FROM census.vw_facts_2011 WHERE val = 0;

下列语句将不删除记录，因为yr = 2012 与视图条件不匹配

		UPDATE census.vw_facts_2011 SET val = 1 WHERE val = 0 AND yr = 2012;

可以插入或更新原来满足视图条件的到其他值		

		UPDATE census.vw_facts_2011 SET yr = 2012 WHERE yr = 2011;

版本9.4加强了限制 子句 WITH CHECK OPTION,例如条件是

		CREATE OR REPLACE VIEW census.vw_facts_2011 AS
		SELECT fact_type_id, val, yr, tract_id
		FROM census.facts WHERE yr = 2011 WITH CHECK OPTION;

由于条件不匹配，报错		

		UPDATE census.vw_facts_2011 SET yr = 2012 WHERE val > 2942;

##在更新view使用触发器
在多表view更新下面基础表的数据

		CREATE OR REPLACE VIEW census.vw_facts AS
		SELECT
			y.fact_type_id, y.category, y.fact_subcats, y.short_name,
			x.tract_id, x.yr, x.val, x.perc
		FROM census.facts As x INNER JOIN census.lu_fact_types As y
		ON x.fact_type_id = y.fact_type_id

创建函数返回触发器	

TG_OP是一个说明触发触发器的操作的字符串	INSERT，UPDATE 或者 DELETE

OLD 数据类型是 RECORD，该变量为 INSERT/UPDATE 操作时保存行（ROW）一级的触发器旧的数据库行。 在语句级别的触发器里，这个变量是 NULL

		CREATE OR REPLACE FUNCTION census.trig_vw_facts_ins_upd_del() RETURNS trigger AS
		$$
		BEGIN
		    IF (TG_OP = 'DELETE') THEN DELETE FROM census.facts AS f
		        WHERE
		            f.tract_id = OLD.tract_id AND f.yr = OLD.yr AND
		            f.fact_type_id = OLD.fact_type_id;
		        RETURN OLD;
		    END IF;
		    IF (TG_OP = 'INSERT') THEN INSERT INTO census.facts(tract_id, yr, fact_type_id, val, perc)
		        SELECT NEW.tract_id, NEW.yr, NEW.fact_type_id, NEW.val, NEW.perc;
		        RETURN NEW;
		    END IF;
		    IF (TG_OP = 'UPDATE') THEN IF
		            ROW(OLD.fact_type_id, OLD.tract_id, OLD.yr, OLD.val, OLD.perc) !=
		            ROW(NEW.fact_type_id, NEW.tract_id, NEW.yr, NEW.val, NEW.perc)
		        THEN UPDATE census.facts AS f
		            SET
		                tract_id = NEW.tract_id,
		                yr = NEW.yr,
		                fact_type_id = NEW.fact_type_id,
		                val = NEW.val,
		                perc = NEW.perc
		            WHERE
		                f.tract_id = OLD.tract_id AND
		                f.yr = OLD.yr AND
		                f.fact_type_id = OLD.fact_type_id;
		            RETURN NEW;
		        ELSE
		            RETURN NULL;
		        END IF;
		    END IF;
		END;
		$$
		LANGUAGE plpgsql VOLATILE;

绑定函数到触发器		

		CREATE TRIGGER census.trig_01_vw_facts_ins_upd_del
		INSTEAD OF INSERT OR UPDATE OR DELETE ON census.vw_facts
		FOR EACH ROW EXECUTE PROCEDURE census.trig_vw_facts_ins_upd_del();

##物化视图
使用在需要长时间查询并且平时对时间不敏感，不大更新的，构建olap应用时

		CREATE MATERIALIZED VIEW census.vw_facts_2011_materialized AS
		SELECT fact_type_id, val, yr, tract_id FROM census.facts WHERE yr = 2011;

创建索引可以提高速度		

		CREATE UNIQUE INDEX ix
		ON census.vw_facts_2011_materialized (tract_id, fact_type_id, yr);

物理索引可以使用ORDER BY和 cluster index ,ORDER BY不需要刷新命令，但平时要消耗更新时间
命名创建

		CLUSTER census.vw_facts_2011_materialized; 

使用cluster重新对表进行物理分布 安装索引 每次刷新使用

		CLUSTER census.vw_facts_2011_materialized USING ix; 

刷新物化视图

		REFRESH MATERIALIZED VIEW census.vw_facts_2011_materialized;

9.4版本刷新时避免锁表		

		REFRESH MATERIALIZED VIEW CONCURRENTLY census.vw_facts_2011_materialized;

物化视图的局限
不能创建和更新已存在的物化视图，必须删除重建
必须使用刷新物化视图命令重建缓冲

#方便的结构
##DISTINCT ON
哪一个列明确DISTINCT，剩下的列排序ORDER BY配合使用
		SELECT DISTINCT ON (left(tract_id, 5))
		    left(tract_id, 5) As county, tract_id, tract_name
		FROM census.lu_tracts
		ORDER BY county, tract_id;
		county |  tract_id   |                     tract_name
		-------+-------------+----------------------------------------------------
		25001  | 25001010100 | Census Tract 101, Barnstable County, Massachusetts
		25003  | 25003900100 | Census Tract 9001, Berkshire County, Massachusetts
		25005  | 25005600100 | Census Tract 6001, Bristol County, Massachusetts
		25007  | 25007200100 | Census Tract 2001, Dukes County, Massachusetts
		25009  | 25009201100 | Census Tract 2011, Essex County, Massachusetts

##LIMIT 和 OFFSET
LIMIT 代表 多少行 
OFFSET 代表跳过多少行

		SELECT DISTINCT ON (left(tract_id, 5))
		    left(tract_id, 5) As county, tract_id, tract_name
		FROM census.lu_tracts
		ORDER BY county, tract_id LIMIT 3 OFFSET 2;
		county |  tract_id   |                     tract_name
		-------+-------------+----------------------------------------------------
		25005  | 25005600100 | Census Tract 6001, Bristol County, Massachusetts
		25007  | 25007200100 | Census Tract 2001, Dukes County, Massachusetts
		25009  | 25009201100 | Census Tract 2011, Essex County, Massachusetts

##快捷转化Shorthand Casting
标准sql转化  CAST('2011-1-11' AS date) PostgreSQL的时'2011-1-1'::date 把字符转化为日期
someXML::text::integer 不可以直接要有过渡
##多行插入
VALUES 后面看作是一个值列表

		INSERT INTO logs_2011 (user_name, description, log_ts)
		VALUES
		    ('robe', 'logged in', '2011-01-10 10:15 AM EST'),
		    ('lhsu', 'logged out', '2011-01-11 10:20 AM EST');

值列表可以看作是一个虚拟表，但要明确它的列名

		SELECT *
		FROM (
		    VALUES
		        ('robe', 'logged in', '2011-01-10 10:15 AM EST'::timestamptz),
		        ('lhsu', 'logged out', '2011-01-11 10:20 AM EST'::timestamptz)
		    ) AS l (user_name, description, log_ts);

##ILIKE 大小写不敏感的查询
由于PostgreSQL是大小写敏感的

		SELECT tract_name FROM census.lu_tracts WHERE tract_name ILIKE '%duke%';

##返回函数

在select子句使用 有返回集的函数

		CREATE TABLE interval_periods (i_type interval);
		INSERT INTO interval_periods (i_type)
		VALUES ('5 months'), ('132 days'), ('4862 hours');
		
		SELECT i_type,
		    generate_series('2012-01-01'::date,'2012-12-31'::date,i_type) As dt
		FROM interval_periods;
		
		i_type     |           dt
		-----------+------------------------
		5 months   | 2012-01-01 00:00:00-05
		5 months   | 2012-06-01 00:00:00-04
		5 months   | 2012-11-01 00:00:00-04
		132 days   | 2012-01-01 00:00:00-05
		132 days   | 2012-05-12 00:00:00-04
		132 days   | 2012-09-21 00:00:00-04
		4862 hours | 2012-01-01 00:00:00-05
		4862 hours | 2012-07-21 15:00:00-04

##限制从继承表内DELETE, UPDATE, SELECT
一般在进行父表查询时会对继承表进行查询，使用ONLY关键字可以不要钻入继承表查询
##DELETE USING
删除表内数据也想删除基于关联表的本表数据，在WHERE子句使用USING子句

		DELETE FROM census.facts
		USING census.lu_fact_types As ft
		WHERE facts.fact_type_id = ft.fact_type_id AND ft.short_name = 's01';

##返回影响用户的数据
用RETURNING子句返回更新的数据，在删除、插入和更新SQL语句中使用

		UPDATE census.lu_fact_types AS f
		SET short_name = replace(replace(lower(f.fact_subcats[4]),' ','_'),':','')
		WHERE f.fact_subcats[3] = 'Hispanic or Latino:' AND f.fact_subcats[4] > ''
		RETURNING fact_type_id, short_name;
		96            | white_alone
		97            | black_or_african_american_alone
		98            | american_indian_and_alaska_native_alone
		99            | asian_alone
		100           | native_hawaiian_and_other_pacific_islander_alone
		101           | some_other_race_alone
		102           | two_or_more_races

##查询组合类型
系统自动为按表名查询建立组合类型

		SELECT x FROM census.lu_fact_types As x LIMIT 2;
		x
		------------------------------------------------------------------
		(86,Population,"{D001,Total:}",d001)
		(87,Population,"{D002,Total:,""Not Hispanic or Latino:""}",d002)

生成数组，在使用array_agg函数(把多行结果转为一个数组)和hstore扩展包内的函数非常有用，json

		SELECT array_to_json(array_agg(f)) As cat FROM (
		    SELECT MAX(fact_type_id) As max_type, category FROM census.lu_fact_types
		    GROUP BY category
		) As f;
		cats
		----------------------------------------------------
		[{"max_type":102,"category":"Population"},
		{"max_type":153,"category":"Housing"}]

json_agg函数把array_agg函数和array_to_json结合起来，从多行结果直接到json

		SELECT json_agg(f) As cats
		FROM (
		    SELECT MAX(fact_type_id) As max_type, category
		    FROM census.lu_fact_types
		    GROUP BY category
		) As f;

##DO命令
把一段存储过程代码放到运行的SQL中

		set search_path=census;
		DROP TABLE IF EXISTS lu_fact_types;
		CREATE TABLE lu_fact_types (
		    fact_type_id serial,
		    category varchar(100),
		    fact_subcats varchar(255)[],
		    short_name varchar(50),
		    CONSTRAINT pk_lu_fact_types PRIMARY KEY (fact_type_id)
		);

		DO language plpgsql
		$$
		DECLARE var_sql text;
		BEGIN
		    var_sql := string_agg(
		        'INSERT INTO lu_fact_types(category, fact_subcats, short_name)
		        SELECT
		            ''Housing'',
		            array_agg(s' || lpad(i::text,2,'0') || ') As fact_subcats,
		            ' || quote_literal('s' || lpad(i::text,2,'0')) || ' As short_name
		        FROM staging.factfinder_import
		        WHERE s' || lpad(I::text,2,'0') || ' ~ ''^[a-zA-Z]+'' ', ';'
		    )
		    FROM generate_series(1,51) As I; 
		    EXECUTE var_sql; 
		END
		$$;

#FILTER子句用于聚合
没有使用过滤

		SELECT student,
		    array_agg(CASE WHEN subject ='algebra' THEN score ELSE NULL END) As algebra,
		    array_agg(CASE WHEN subject ='physics' THEN score ELSE NULL END) As physics
		FROM test_scores
		GROUP BY student;


使用过滤
		
		SELECT student,
		    array_agg(score) FILTER (WHERE subject ='algebra') As algebra,
		    array_agg(score) FILTER (WHERE subject ='physics') As physics
		FROM test_scores
		GROUP BY student;

#Window函数
窗口函数在和当前行相关的一组表行上执行计算。 这相当于一个可以由聚合函数完成的计算类型。但不同于常规的聚合函数， 使用的窗口函数不会导致行被分组到一个单一的输出行；行保留其独立的身份。 在后台，窗口函数能够访问的不止查询结果的当前行。
比较每个员工的工资和在他或她的部门的平均工资

		SELECT depname, empno, salary, avg(salary) OVER (PARTITION BY depname) FROM empsalary;
		 depname   | empno | salary |          avg          
		-----------+-------+--------+-----------------------
		 develop   |    11 |   5200 | 5020.0000000000000000
		 develop   |     7 |   4200 | 5020.0000000000000000
		 develop   |     9 |   4500 | 5020.0000000000000000
		 develop   |     8 |   6000 | 5020.0000000000000000
		 develop   |    10 |   5200 | 5020.0000000000000000
		 personnel |     5 |   3500 | 3700.0000000000000000
		 personnel |     2 |   3900 | 3700.0000000000000000
		 sales     |     3 |   4800 | 4866.6666666666666667
		 sales     |     1 |   5000 | 4866.6666666666666667

前三输出列直接来自表empsalary，并有一个针对表中的每一行的输出行。 第四列将代表所有含有相同的depname值的表行的平均值作为当前值。 （这实际上与标准avg聚合函数的功能相同， 但是OVER子句使其被视为一个窗口函数并在一组合适的行上执行计算。）

窗口函数的调用总是包含一个OVER子句，后面直接跟着窗口函数的名称和参数。 这是它在语法上区别于普通函数或聚合功能的地方。 OVER子句决定如何将查询的行进行拆分以便给窗口函数处理。 OVER子句内的PARTITION BY列表指定将行划分成组或分区， 组或分区共享相同的PARTITION BY表达式的值。 对于每一行，窗口函数在和当前行落在同一个分区的所有行上进行计算。
		
		
在对当前行使用时还要计算到其他行，在聚集时比较有用，诸如row_number rank
下面例子OVER设置了使用WHERE的边界,可以在不使用GROUP BY聚集，也可以在没有join使用聚集

		SELECT tract_id, val, AVG(val) OVER () as val_avg
		FROM census.facts
		WHERE fact_type_id = 86;
		tract_id   |    val    |        val_avg
		------------+-----------+-----------------------
		25001010100 |  2942.000 | 4430.0602165087956698
		25001010206 |  2750.000 | 4430.0602165087956698
		25001010208 |  2003.000 | 4430.0602165087956698
		25001010304 |  2421.000 | 4430.0602165087956698

可以把所有的SQL聚集函数作为window函数
##PARTITION BY
可以Window函数使用PARTITION BY子句，而不是整个表
下面例子针对tract_id前5位分区

		SELECT tract_id, val, AVG(val) OVER (PARTITION BY left(tract_id,5)) As val_avg_county
		FROM census.facts WHERE fact_type_id = 2 ORDER BY tract_id;
		  tract_id   |   val    |    val_avg_county
		-------------+----------+-----------------------
		 25001010100 | 1765.000 | 1709.9107142857142857
		 25001010206 | 1366.000 | 1709.9107142857142857
		 25001010208 |  984.000 | 1709.9107142857142857
		 :
		 25003900100 | 1920.000 | 1438.2307692307692308
		 25003900200 | 1968.000 | 1438.2307692307692308
		 25003900300 | 1211.000 | 1438.2307692307692308
		 :

##ORDER BY
window函数允许在OVER子句使用ORDER BY

		SELECT ROW_NUMBER() OVER (ORDER BY tract_name) As rnum, tract_name
		FROM census.lu_tracts
		ORDER BY rnum LIMIT 4;
		rnum |                    tract_name
		-----+--------------------------------------------------
		1    | Census Tract 1, Suffolk County, Massachusetts
		2    | Census Tract 1001, Suffolk County, Massachusetts
		3    | Census Tract 1002, Suffolk County, Massachusetts
		4    | Census Tract 1003, Suffolk County, Massachusetts

在每个分区里进行排序，并得到在一个分区内逐行累计	

		SELECT tract_id, val,
		    SUM(val) OVER (PARTITION BY left(tract_id,5) ORDER BY val) As sum_county_ordered
		FROM census.facts
		WHERE fact_type_id = 2
		ORDER BY left(tract_id,5), val;
		  tract_id   |   val    | sum_county_ordered
		-------------+----------+--------------------
		 25001014100 |  226.000 |            226.000
		 25001011700 |  971.000 |           1197.000
		 25001010208 |  984.000 |           2181.000
		 :
		 25003933200 |  564.000 |            564.000
		 25003934200 |  593.000 |           1157.000
		 25003931300 |  606.000 |           1763.000
		 :
window命名在后面WINDOW wt，在前面使用OVER( wt )	 

		 SELECT * FROM (
		     SELECT
		         ROW_NUMBER() OVER( wt ) As rnum, substring(tract_id,1, 5) As county_code,
		         tract_id,
		         LAG(tract_id,2) OVER wt As tract_2_before,
		         LEAD(tract_id) OVER wt As tract_after
		     FROM census.lu_tracts
		     WINDOW wt AS (PARTITION BY substring(tract_id,1, 5) ORDER BY tract_id) ) As x
		 WHERE rnum BETWEEN 2 and 3 AND county_code IN ('25007','25025')
		 ORDER BY county_code, rnum;
		 rnum | county_code |  tract_id   | tract_2_before | tract_after
		 -----+-------------+-------------+----------------+-------------
		 2    | 25007       | 25007200200 |                | 25007200300
		 3    | 25007       | 25007200300 | 25007200100    | 25007200400
		 2    | 25025       | 25025000201 |                | 25025000202
		 3    | 25025       | 25025000202 | 25025000100    | 25025000301
在PostgreSQL里所有聚集函数都可以作为一个window函数

#通用表表达式CTEs
定义一个查询在一个大的查询里重用，作为临时表
三种使用CTE的方法
##基本CTE
更好阅读，中间结果提高性能

		WITH cte AS (
		    SELECT
		        tract_id, substring(tract_id,1, 5) As county_code,
		        COUNT(*) OVER(PARTITION BY substring(tract_id,1, 5)) As cnt_tracts
		    FROM census.lu_tracts
		)
		SELECT MAX(tract_id) As last_tract, county_code, cnt_tracts
		FROM cte
		WHERE cnt_tracts > 100
		GROUP BY county_code, cnt_tracts;
分离几个	cte，在第二个里使用第一个	

		WITH
		    cte1 AS (
		        SELECT
		            tract_id,
		            substring(tract_id,1, 5) As county_code,
		            COUNT(*) OVER (PARTITION BY substring(tract_id,1,5)) As cnt_tracts
		        FROM census.lu_tracts
		    ),
		    cte2 AS (
		        SELECT
		            MAX(tract_id) As last_tract,
		            county_code,
		            cnt_tracts
		        FROM cte1
		        WHERE cnt_tracts < 8 GROUP BY county_code, cnt_tracts
		    )
		SELECT c.last_tract, f.fact_type_id, f.val
		FROM census.facts As f INNER JOIN cte2 c ON f.tract_id = c.last_tract;
##可写CTE
基本CTE的扩充，带有UPDATE, INSERT, DELETE 
先创建表

		CREATE TABLE logs_2011_01_02 (
		    PRIMARY KEY (log_id),
		    CONSTRAINT chk
		        CHECK (log_ts >= '2011-01-01' AND log_ts < '2011-03-01')
		)
		INHERITS (logs_2011);

创建更新	CTE	 RETURNING子句可以返回这个删除结果形成的临时表，该临时表内容被下面的INSERT语句使用

		WITH t AS (
		    DELETE FROM ONLY logs_2011 WHERE log_ts < '2011-03-01' RETURNING *
		)
		INSERT INTO logs_2011_01_02 SELECT * FROM t;
##递归CTE
UNION ALL组合表

		WITH RECURSIVE tbls AS (
		    SELECT
		        c.oid As tableoid,
		        n.nspname AS schemaname,
		        c.relname AS tablename FROM
		        pg_class c LEFT JOIN
		        pg_namespace n ON n.oid = c.relnamespace LEFT JOIN
		        pg_tablespace t ON t.oid = c.reltablespace LEFT JOIN
		        pg_inherits As th ON th.inhrelid = c.oid
		    WHERE
		        th.inhrelid IS NULL AND
		        c.relkind = 'r'::"char" AND c.relhassubclass
		    UNION ALL
		    SELECT
		        c.oid As tableoid,
		        n.nspname AS schemaname,
		        tbls.tablename || '->' || c.relname AS tablename  FROM
		        tbls INNER JOIN
		        pg_inherits As th ON th.inhparent = tbls.tableoid INNER JOIN
		        pg_class c ON th.inhrelid = c.oid LEFT JOIN
		        pg_namespace n ON n.oid = c.relnamespace LEFT JOIN
		    pg_tablespace t ON t.oid = c.reltablespace
		)
		SELECT * FROM tbls ORDER BY tablename;
		tableoid| schemaname |            tablename
		--------+------------+----------------------------------
		3152249 | public     | logs
		3152260 | public     | logs->logs_2011
		3152272 | public     | logs->logs_2011->logs_2011_01_02
#横向Lateral Join
允许在两个表的列上共享数据，只是单向的

		SELECT * FROM census.facts L INNER JOIN LATERAL
		    (SELECT * FROM census.lu_fact_types
		      WHERE category = CASE WHEN L.yr = 2011 THEN 'Housing' ELSE category END) R
		    ON L.fact_type_id = R.fact_type_id;
		
		SELECT i_type, dt
		FROM
		    interval_periods CROSS JOIN LATERAL
		    generate_series('2012-01-01'::date, '2012-12-31'::date, i_type) AS dt
		WHERE NOT (dt = '2012-01-01' AND i_type = '132 days'::interval);


