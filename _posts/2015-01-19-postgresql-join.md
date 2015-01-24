---
layout: post
title: "PostgreSQL Join"
description: "PostgreSQL Join"
category: Postgresql
tags: [PostgreSQL,Join]
---
{% include JB/setup %}

交叉连接 内连接 左外连接 右外连接 全连接

#交叉连接CROSS JOIN
等同于t1里面的字段后面跟着所有t2的字段

		SELECT * FROM t1 CROSS JOIN t2;

#内连接 INNER  JOIN
对于 T1 中的每一行 R1 ，如果能在 T2 中找到一个或多个满足连接条件的行，那么这些满足条件的每一行都在连接表中生成一行

		SELECT * FROM t1 INNER JOIN t2 ON t1.num = t2.num;
		
#左外连接 LEFT OUTER JOIN
首先执行一次内连接。然后为每一个 T1 中无法在 T2 中找到匹配的行生成一行，该行中对应 T2 的列用 NULL 补齐。因此，生成的连接表里无条件地包含来自 T1 里的每一行至少一个副本。

		SELECT * FROM t1 LEFT JOIN t2 ON t1.num = t2.num;
		
#右外连接（RIGHT OUTER JOIN）
右外连接。首先执行一次内连接。然后为每一个 T2 中无法在 T1 中找到匹配的行生成一行，该行中对应 T1 的列用 NULL 补齐。因此，生成的连接表里无条件地包含来自 T2 里的每一行至少一个副本。

		SELECT * FROM t1 RIGHT JOIN t2 ON t1.num = t2.num;
		
#全连接（FULL OUTER JOIN）
首先执行一次内连接。然后为每一个 T1 与 T2 中找不到匹配的行生成一行，该行中无法匹配的列用 NULL 补齐。因此，生成的连接表里无条件地包含 T1 和 T2 里的每一行至少一个副本。

		 SELECT * FROM t1 FULL JOIN t2 ON t1.num = t2.num;
