# Other operations {#concept_in2_nbd_5db .concept}

## Alias command {#section_kcp_t2f_vdb .section}

The ALIAS command reads different resources \(data\) using a fixed resource name in [MapReduce](intl.en-US/User Guide/MapReduce/Summary/MapReduce.md) or [UDF](intl.en-US/User Guide/SQL/UDF/UDF Summary.md) without modifying the code.

Command format:

```
ALIAS <alias>=<real>;
```

Action:

Create alias for a resource.

Example:

```
ADD TABLE src_part PARTITION (ds='20121208') AS res_20121208;
ADD TABLE src_part PARTITION (ds='20121209') AS res_20121209;
ALIAS resName=res_20121208;
jar -resources resName -libjars work.jar -classpath ./work.jar com.company.MainClass args ... ;//job 1
ALIAS resName=res_20121209;
jar -resources resName -libjars work.jar -classpath ./work.jar com.company.MainClass args ... ;//job 2
```

In the preceding example, resource alias **resName** refers to different resource tables in two jobs. Different data can be read without modifying the code.

## Cost SQL {#section_xm2_sgf_vdb .section}

Command format:

```
cost sql <SQL Sentence>;
```

Action:

Estimate an SQL measurement message, including the size of the input data, the number of UDFs, and the SQL complexity level.

**Note:** Use the following information for reference purpose only. Refrain from using it as an actual charging standard.

Example:

```
odps@ $odps_project >cost sql select distinct project_name, user_name from meta.m_security_users distribute by project_name sort by project_name;  
ID = 20150715113033121xxxxxxxx
Input:65727592 Bytes
UDF:0
Complexity:1.0
```

