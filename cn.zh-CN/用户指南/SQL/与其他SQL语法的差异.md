# 与其他SQL语法的差异 {#concept_fsf_mvf_vdb .concept}

本文将从SQL角度，将MaxCompute SQL与Hive、MySQL、Oracle、SQL Server进行对比，从而为您介绍MaxCompute不支持的DDL和DML语法。

## MaxCompute不支持的DDL语法 {#section_ed4_pvf_vdb .section}

|语法|MaxCompute|Hive|MySQL|Oracle|SQL Server|
|:-|:---------|:---|:----|:-----|:---------|
|CREATE TABLE—PRIMARY KEY|N|N|Y|Y|Y|
|CREATE TABLE—NOT NULL|N|N|Y|Y|Y|
|CREATE TABLE—CLUSTER BY|Y|Y|N|Y|Y|
|CREATE TABLE—EXTERNAL TABLE|Y\(OSS, OTS, TDDL\)|Y|N|N|N|
|CREATE TABLE—TEMPORARY TABLE|N|Y|Y|Y|Y （with \#prefix）|
|INDEX—CREATE INDEX|N|Y|Y|Y|Y|
|VIRTUAL COLUMN|N|N \(only 2 predefined\)|N|Y|Y|

## MaxCompute不支持的DML语法 {#section_ws2_vvf_vdb .section}

|语法|MaxCompute|Hive|MySQL|Oracle|SQL Server|
|:-|:---------|:---|:----|:-----|:---------|
|CTE|Y|Y|Y|Y|Y|
|SELECT—recursive CTE|N|N|N|Y|Y|
|SELECT—GROUP BY ROLL UP|Y|Y|Y|Y|Y|
|SELECT—GROUP BY CUBE|Y|Y|N|Y|Y|
|SELECT—GROUPING SET|Y|Y|N|Y|Y|
|SELECT—IMPLICT JOIN|Y|Y|N|Y|Y|
|SELECT—PIVOT|N|N|N|Y|Y|
|SEMI JOIN|Y|Y \(corelated expression must be in WHERE, EXISTS must be corelated\)|Y|N \(has IN and EXISTS, but no SEMI JOIN grammer\)|N \(has IN and EXISTS, but no SEMI JOIN grammer\)|
|SELEC TRANSFROM|Y|Y|N|N|N|
|SELECT—corelated subquery|Y|Y \(corelated expression must be in WHERE, EXISTS must be corelated\)|Y|Y|Y|
|ORDER BY NULLS FIRST/LAST|Y|Y|Y|Y|Y|
|LATERAL VIEW|Y|Y|N|Y \(LATERAL keyword\)|Y \(CROSS APPLY keyword\)|
|SET OPERATOR—UNION \(disintct\)|Y|Y|Y|Y|Y|
|SET OPERATOR—INTERSECT|Y|N|N|Y|Y|
|SET OPERATOR—MINUS/EXCEPT|Y|N|N|Y|Y（keyword EXCEPT）|
|INSERT INTO ... VALUES|Y|Y|Y|Y|Y|
|INSERT INTO \(ColumnList\)|Y|Y|Y|Y|Y|
|UPDATE … WHERE|N|Y|Y|Y|Y|
|UPDATE … ORDER BY LIMIT|N|N|Y|N|Y|
|DELETE … WHERE|N|Y|Y|Y|Y|
|DELETE … ORDER BY LIMIT|N|N|Y|N|N|
|ANALYTIC—reusable WINDOWING CLUSUE|Y|Y|N|N|N \(can implement with join\)|
|ANALYTIC—CURRENT ROW|Y|Y|N|Y|Y|
|ANALYTIC—UNBOUNDED|Y|N|Y|Y|Y|
|ANALYTIC—RANGE …|N|Y|N|Y|Y|
|WHILE DO|N|N|Y|Y|Y|

## MaxCompute不支持的SCRIPTING语法 {#section_ppz_kfq_kgb .section}

|语法|MaxCompute|Hive|MySQL|Oracle|SQL Server|
|--|----------|----|-----|------|----------|
|TABLE VARIABLE|Y|Y \(TEMPORARY TABLE\)|Y \(TEMPORARY TABLE\)|Y \(TEMPLORARY TABLE\)|Y|
|SCALER VARIABLE|Y|Y|Y \(DECLARE x INT\)|Y|Y|
|ERROR HANDLING—RAISE ERROR|N|N|Y|Y|Y|
|ERROR HANDLING—TRY CATCH|N|N|N|Y|Y|
|FLOW CONTROL—LOOP|N|N|Y|Y|Y|
|CURSOR|N|N|Y|Y|Y|

