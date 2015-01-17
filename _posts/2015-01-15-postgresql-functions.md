---
layout: post
title: "PostgreSQL Functions"
description: "Postgresql-Functions"
categories: Postgresql
tags: [Postgresql,Functions]
---
{% include JB/setup %}

把一系列的SQL语句串联出统称为函数，其他数据库称之为存储过程等，还能控制PostgreSQL的语言

#解剖PostgreSQL函数
##基本函数
一个简单的结构

		CREATE OR REPLACE FUNCTION func_name(arg1 arg1_datatype DEFAULT arg1_default)
		RETURNS some type | set of some type | TABLE (..) AS
		$$
		BODY of function
		$$
		LANGUAGE language_of_function

查询安装在数据库的语言
		
		SELECT lanname FROM pg_language

###VOLATILITY 变化 

不变IMMUTABLE 同样的输入同样的输出

稳定STABLE 使用相同的输入查询同样的输出

变化VOLATILE  每次输出不同即便同样的输入

限制STRICT  如果输入null输出就为null，小心使用STRICT

###COST成本

SQL 和 PL/pgSQL 函数默认  100  C函数 默认 1

###ROWS
仅作为返回多少行数使用

###SECURITY DEFINER安全定义
这个子句使得函数在安全上下文使用

##触发器和触发器函数
把触发器放在表和视图上，触发器有语句级和行级，语句级每执行一次语句就运行触发器一次，行级是每更新一行就运行触发器一次有500行就运行触发器500次

触发器有三种  BEFORE, AFTER, INSTEAD OF 。BEFORE, AFTER在表上使用，INSTEAD OF在视图上使用

也可以用在WHEN子句控制多少行触发或在更新语句使用

触发器和一般函数结构一样，区别是在输入参数和输出类型上不同，触发器没有参数，总是产生一个输出结果trigger，能在不同触发器重用同一个触发器

对一个事件可以触发多个触发器，按字母顺序，每一个触发器可以读取上一个触发器修改的数据，如果其中一个触发器回滚，在这个事件上的所有触发器全部回滚。

##聚集
不仅仅MIN, MAX, AVG, SUM,COUNT
创建结构

		CREATE AGGREGATE my_agg (input data type) (
		SFUNC=state function name,
		STYPE=state type,
		FINALFUNC=final function name,
		INITCOND=initial state value, SORTOP=sort_operator
		);
		
##信赖的和非信赖的语言
###信赖的：不能执行操作系统命令 SQL, PL/pgSQL, PL/Perl 

###非信赖的：可以和操作系统交互，超级用户，语言结尾有个U PL/PerlU, PL/PythonU,

##用SQL写函数
用现有的sql语句加上函数头和结尾

###基本SQL函数

创建一个在表中插入一行返回标量值
	
		CREATE OR REPLACE FUNCTION 
		write_to_log(param_user_name varchar, param_description text)
		RETURNS integer AS
		$$
		INSERT INTO logs(user_name, description) VALUES($1, $2)
		RETURNING log_id;
		$$
		LANGUAGE 'sql' VOLATILE;
		
调用函数

		SELECT write_to_log('alejandro', 'Woke up at noon.') As new_id;

创建更新函数

		CREATE OR REPLACE FUNCTION
		update_logs(log_id int, param_user_name varchar, param_description text)
		RETURNS void AS
		$$
		UPDATE logs SET user_name = $2, description = $3
		 , log_ts = CURRENT_TIMESTAMP WHERE log_id = $1;
		$$
		LANGUAGE 'sql' VOLATILE;
		
调用函数

		SELECT update_logs(12, 'alejandro', 'Fell back asleep.');

三种返回设置

1. Using RETURNS TABLE:

		CREATE OR REPLACE FUNCTION select_logs_rt(param_user_name varchar)
		RETURNS TABLE (log_id int, user_name varchar(50), description text, log_ts timestamptz) AS
		$$
		SELECT log_id, user_name, description, log_ts FROM logs WHERE user_name = $1;
		$$
		LANGUAGE 'sql' STABLE;
		
2. Using OUT parameters:

		CREATE OR REPLACE FUNCTION select_logs_out(param_user_name varchar, OUT log_id int
		 , OUT user_name varchar, OUT description text, OUT log_ts timestamptz)
		RETURNS SETOF record AS
		$$
		SELECT * FROM logs WHERE user_name = $1;
		$$
		LANGUAGE 'sql' STABLE;
		
3. Using a composite type:

		CREATE OR REPLACE FUNCTION select_logs_so(param_user_name varchar)
		RETURNS SETOF logs AS
		$$
		SELECT * FROM logs WHERE user_name = $1;
		$$
		LANGUAGE 'sql' STABLE;
		
调用上述三种方式

		SELECT * FROM select_logs_xxx('alejandro');
		
###写SQL聚集函数

用到两个子函数：

状态转移函数加总日志

		CREATE OR REPLACE FUNCTION geom_mean_state(prev numeric[2], next numeric)
		RETURNS numeric[2] AS
		$$
		SELECT
		  CASE
		    WHEN $2 IS NULL OR $2 = 0 THEN $1
		    ELSE ARRAY[COALESCE($1[1],0) + ln($2), $1[2] + 1]
		  END;
		$$
		LANGUAGE sql IMMUTABLE;
		
最终函数取幂

		CREATE OR REPLACE FUNCTION geom_mean_final(numeric[2])
		RETURNS numeric AS
		$$
		SELECT CASE WHEN $1[2] > 0 THEN exp($1[1]/$1[2]) ELSE 0 END;
		$$
		LANGUAGE sql IMMUTABLE;
		
定义聚集调用上述两个
		
		CREATE AGGREGATE geom_mean(numeric) (
		SFUNC=geom_mean_state,
		STYPE=numeric[],
		FINALFUNC=geom_mean_final,
		INITCOND='{0,0}'
		);

具体使用

		SELECT left(tract_id,5) As county, geom_mean(val) As div_county
		FROM census.vw_facts
		WHERE category = 'Population' AND short_name != 'white_alone'
		GROUP BY county
		ORDER BY div_county DESC LIMIT 5;
		
		county |     div_county
		-------+---------------------
		25025  | 85.1549046212833364
		25013  | 79.5972921427888918
		25017  | 74.7697097102419689
		25021  | 73.8824162064128504
		25027  | 73.5955049035237656
		
把聚集函数作为窗口函数

		WITH X AS (SELECT
		  tract_id,
		  left(tract_id,5) As county,
		  geom_mean(val) OVER (PARTITION BY tract_id) As div_tract,
		  ROW_NUMBER() OVER (PARTITION BY tract_id) As rn,
		  geom_mean(val) OVER(PARTITION BY left(tract_id,5)) As div_county
		FROM census.vw_facts WHERE category = 'Population' AND short_name != 'white_alone'
		)
		SELECT tract_id, county, div_tract, div_county
		FROM X
		WHERE rn = 1
		ORDER BY div_tract DESC, div_county DESC LIMIT 5;
		
		tract_id    | county |      div_tract       |     div_county
		------------+--------+----------------------+---------------------
		25025160101 | 25025  | 302.6815688785928786 | 85.1549046212833364
		25027731900 | 25027  | 265.6136902148147729 | 73.5955049035237656
		25021416200 | 25021  | 261.9351057509603296 | 73.8824162064128504
		25025130406 | 25025  | 260.3241378371627137 | 85.1549046212833364
		25017342500 | 25017  | 257.4671462282508267 | 74.7697097102419689

##写PL/pgSQL函数
###基本PL/pgSQL函数

		CREATE FUNCTION select_logs_rt(param_user_name varchar)
		RETURNS TABLE (log_id int, user_name varchar(50), description text, log_ts timestamptz) AS
		$$
		BEGIN
			RETURN QUERY
		    SELECT log_id, user_name, description, log_ts FROM logs
		     WHERE user_name = param_user_name;
		END;
		$$
		LANGUAGE 'plpgsql' STABLE;
		
###用PL/pgSQL写触发器函数
创建一个触发器

		CREATE OR REPLACE FUNCTION trig_time_stamper() RETURNS trigger AS 
		$$
		BEGIN
		    NEW.upd_ts := CURRENT_TIMESTAMP;
		    RETURN NEW;
		END;
		$$
		LANGUAGE plpgsql VOLATILE;
		CREATE TRIGGER trig_1
		BEFORE INSERT OR UPDATE OF session_state, session_id 
		ON web_sessions
		FOR EACH ROW EXECUTE PROCEDURE trig_time_stamper();

##写PL/Python函数
##写PL/V8, PL/CoffeeScript, PL/LiveScript函数
安装扩展
		
		CREATE EXTENSION plv8;
		CREATE EXTENSION plcoffee;
		CREATE EXTENSION plls;

###基本函数
创建

		CREATE OR REPLACE FUNCTION
		validate_email(email text) returns boolean as
		$$
		 var re = /\S+@\S+\.\S+/;
		 return re.test(email);
		$$ LANGUAGE plv8 IMMUTABLE STRICT;
		
调用函数

		SELECT email, validate_email(email) AS is_valid
		 FROM (VALUES ('alexgomezq@gmail.com')
		 ,('alexgomezqgmail.com'),('alexgomezq@gmailcom')) AS x (email);
		         email         | is_valid
		 ----------------------+----------
		  alexgomezq@gmail.com | t
		  alexgomezqgmail.com  | f
		  alexgomezq@gmailcom  | f

创建一个plcoffee函数

		CREATE OR REPLACE FUNCTION
		validate_email(email text) returns boolean as
		$$
		    re = /\S+@\S+\.\S+/
		    return re.test email
		$$
		LANGUAGE plcoffee IMMUTABLE STRICT;
		
###用PL/V8写聚集函数
状态转移函数

		CREATE OR REPLACE FUNCTION geom_mean_state(prev numeric[2], next numeric)
		RETURNS numeric[2] AS
		$$
		    return (next == null || next == 0) ? prev :
		    [(prev[0] == null)? 0: prev[0] + Math.log(next), prev[1] + 1];
		$$
		LANGUAGE plv8 IMMUTABLE;

最终函数
		
		CREATE OR REPLACE FUNCTION geom_mean_final(in_num numeric[2])
		RETURNS numeric AS
		$$
		  return in_num[1] > 0 ? Math.exp(in_num[0]/in_num[1]) : 0;
		$$
		LANGUAGE plv8 IMMUTABLE;
		
The final CREATE AGGREGATE 
	
		CREATE AGGREGATE geom_mean(numeric) (
		  SFUNC=geom_mean_state,
		  STYPE=numeric[],
		  FINALFUNC=geom_mean_final,
		  INITCOND='{0,0}'
		);

		


