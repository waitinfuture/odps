# SELECT TRANSFORM {#concept_a45_b1c_wdb .concept}

This topic describes how to use the `SELECT TRANSFORM` statement to perform operations that MaxCompute SQL does not directly support. Specifically, by using SELECT TRANSFORM, you can start a specified child process and enter data of a required format into the child process through standard input \(stdin\). Then, you can parse the standard output \(stdout\) of the child process to obtain the final output. This process does not require you to compile user-defined functions \(UDFs\).

Example code:

```
SELECT TRANSFORM(arg1, arg2 ...) 
(ROW FORMAT DELIMITED (FIELDS TERMINATED BY field_delimiter (ESCAPED BY character_escape)?)? 
(LINES SEPARATED BY line_separator)? 
(NULL DEFINED AS null_value)?)?
USING 'unix_command_line' 
(RESOURCES 'res_name' (',' 'res_name')*)? 
( AS col1, col2 ...)?
(ROW FORMAT DELIMITED (FIELDS TERMINATED BY field_delimiter (ESCAPED BY character_escape)?)? 
(LINES SEPARATED BY line_separator)? (NULL DEFINED AS null_value)?)?
```

Parameters:

-   The `SELECT TRANSFORM` keyword can be replaced with the `MAP` or `REDUCE` keyword, but the meaning remains unchanged. However, to make the syntax clearer, we recommend that you use `SELECT TRANSFORM`.
-   `arg1,arg2...` indicates arguments in the `TRANSFORM` clause. The format of arguments in the `TRANSFORM` clause is similar to that of items in the `SELECT` clause. In the default format, the results of expressions for each argument are combined by using \\t after they are implicitly converted into strings, and then they are entered into the specified child process. The default format is configurable. For more information, see the `ROW FORMAT` clause described in the following section.
-   `USING`: specifies the command for starting a child process.

    **Note:** 

    -   In most MaxCompute SQL statements, the `USING` clause can only specify resources. However, with the `SELECT TRANSFORM` statement, the `USING` clause can specify commands to ensure compatibility with the syntax of Hive.
    -   The format of the `USING` clause is similar to the syntax of a Shell script. However, a Shell script is actually not executed to start the child process, and the child process is created according to the command input. Therefore, a number of Shell functions such as input and output redirection, pipe, and loop are unavailable. A Shell script can be used as the command for a child process if needed.
-   `RESOURCES`: specifies the resources that the specified child process can access. You can use one of the following two methods to specify resources:
    -   Use the `RESOURCES` clause, for example, `using 'sh foo.sh bar.txt' Resources 'foo.sh','bar.txt'`.
    -   Add the `set odps.sql.session.resources=foo.sh,bar.txt;` clause before SQL statements. This clause takes effect globally once it is specified, which means that all SELECT TRANSFORM statements can access the resources specified by this clause.
-   `ROW FORMAT`: specifies the input or output format.

    The syntax includes two `ROW FORMAT` clauses: One specifies the input format, and the other specifies the output format. By default, `\t` is used to separate columns, `\n` is used to separate rows, and `Null` is represented by `\N`.

    **Note:** 

    -   Only one character is accepted by field\_delimiter, character\_escape, and line\_separator. If you specify a string, the first character in the string takes priority over the others.
    -   Hive specifies the syntax for each format. Such syntaxes as inputRecordReader, outputRecordReader, and Serde are supported by MaxCompute. To use these formats, you need to enable compatibility with Hive by adding `set odps.sql.hive.compatible=true;` preceding SQL statements. For more information about the syntaxes supported by Hive, see [Hive](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Transform#LanguageManualTransform-TRANSFORMExamples).
    -   If you specify a syntax such as inputRecordReader or outputRecordReader supported by Hive, statements may be executed at lower speeds.
-   `AS`: specifies output columns.
    -   You can specify data types in the `AS` clause, for example, `as(col1:bigint, col2:boolean)`. If you do not specify data types, for example, `as(col1, col2)`, strings are returned by default.
    -   The output is obtained by parsing the stdout of the child process. If the specified data types do not include STRING, the system implicitly calls the `CAST` function, which may encounter runtime exceptions.
    -   You are not allowed to specify data types for only some of the specified columns, for example, `as(col1, col2:bigint)`.
    -   If you skip the `AS` clause, the field preceding the first `\t` in the stdout is a key, and all the following parts are values, equivalent to `as(key, value)`.

## Call Shell scripts {#section_q4t_mbc_wdb .section}

In the case that you execute a Shell script to generate 50 lines of data starting from 1 to 50, the output of the data field is as follows:

```
SELECT  TRANSFORM(script) USING 'sh' AS (data) 
FROM (
        SELECT  'for i in `seq 1 50`; do echo $i; done' AS script
      ) t
;
```

The Shell commands are used as the input of the `TRANSFORM` clause.

SELECT TRANSFORM is not only a language extension. Simple functions such as AWK, Python, Perl, and Shell allow you to compile scripts in commands, so that you do not need to compile independent script files or upload resources.

## Call Python scripts {#section_fx2_qbc_wdb .section}

Python is a simple function that can allow you to compile scripts in commands so that you do not need to compile independent script files or upload resources. The following is an example of how to call Python scripts.

1.  Compile a Python script file. In this example, the file name is myplus.py 

    ```
    #!/usr/bin/env python
    import sys
    line = sys.stdin.readline()
    while line:
        token = line.split('\t')
        if (token[0] == '\\N') or (token[1] == '\\N'):
            print '\\N'
        else:
            print int(token[0]) + int(token[1])
        line = sys.stdin.readline()
    ```

2.  Add the Python script file as a resource to MaxCompute.

    ```
    add py ./myplus.py -f;
    ```

    **Note:** You can add resources by using the DataWorks console.

3.  Use SELECT TRANSFORM to call the resource.

    ```
    Create table testdata(c1 bigint,c2 bigint); - Creates a test table.
    insert into Table testdata values (1,4),(2,5),(3,6); - Inserts test data into the test table.
    
    - Execute the SELECT TRANSFORM statement: 
    SELECT 
    TRANSFORM (testdata.c1, testdata.c2) 
    USING 'python myplus.py'resources 'myplus.py' 
    AS (result bigint) 
    FROM testdata;
    
    - Or:
    set odps.sql.session.resources=myplus.py;
    SELECT TRANSFORM (testdata.c1, testdata.c2) 
    USING 'python myplus.py' 
    AS (result bigint) 
    FROM testdata;
    ```


The returned information is as follows:

```
+-----+
| cnt |
+-----+
| 5   |
| 7   |
| 9   |
+-----+
```

Python scripts do not require MaxCompute to run in a Python framework, and they are not subject to any format requirements.

In MaxCompute, Python commands can be used as the input of the `TRANSFORM` clause. For example, you can call Shell scripts by running Python commands.

```
SELECT  TRANSFORM('for i in xrange(1, 50):  print i;') USING 'python' AS (data);
```

## Call Java scripts {#section_dsx_ccc_wdb .section}

Calling a Java script is similar to calling a Python script. In this example, you need to compile a Java script file, export it as a .jar package, and then run the add command to add the .jar package as a resource to MaxCompute. The resource will be called by using SELECT TRANSFORM.

1.  Compile a Java script file, and export it as a .jar package. In this example, the name of the .jar package is Sum.jar.

    ```
    package com.aliyun.odps.test;
    import java.util.Scanner
    public class Sum {
        public static void main(String[] args) {
            Scanner sc = new Scanner(System.in);
            while (sc.hasNext()) {
                String s = sc.nextLine();
                String[] tokens = s.split("\t");
                if (tokens.length < 2) {
                    throw new RuntimeException("illegal input");
                }
                if (tokens[0].equals("\\N") || tokens[1].equals("\\N")) {
                    System.out.println("\\N");
                }
                System.out.println(Long.parseLong(tokens[0]) + Long.parseLong(tokens[1]));
            }
        }
    }
    ```

2.  Add the .jar package as a resource to MaxCompute.

    ```
    add jar ./Sum.jar -f;
    ```

3.  Use SELECT TRANSFORM to call the resource.

    ```
    Create table testdata(c1 bigint,c2 bigint); - Creates a test table.
    insert into Table testdata values (1,4),(2,5),(3,6); - Inserts test data into the test table.
    - Execute the SELECT TRANSFORM statement: 
    SELECT TRANSFORM(testdata.c1, testdata.c2) USING 'java -cp Sum.jar com.aliyun.odps.test.Sum' resources 'Sum.jar' from testdata;
    - Or:
    set odps.sql.session.resources=Sum.jar;
    SELECT TRANSFORM(testdata.c1, testdata.c2) USING 'java -cp Sum.jar com.aliyun.odps.test.Sum' FROM testdata;
    ```


The returned information is as follows:

```
+-----+
| cnt |
+-----+
| 5   |
| 7   |
| 9   |
+-----+
```

You can run most Java utilities by using the preceding method.

**Note:** Although user-defined table-valued function \(UDTF\) frameworks are provided for Java and Python, compiling code by using SELECT TRANSFORM is easier. SELECT TRANSFORM is a simpler process because it is not subject to any format requirements and can be called offline. The paths for Java and Python offline scripts can be obtained from the JAVA\_HOME and PYTHON\_HOME environment variables, respectively.

## Call scripts of other languages {#section_p4m_pcc_wdb .section}

In addition to the language extensions mentioned above, SELECT TRANSFORM supports commonly used Unix command and script interpreters such as AWK and Perl.

An example of how to output column 2 by calling AWK:

```
SELECT TRANSFORM(*) USING "awk '//{print $2}'" as (data) from testdata;
```

An example of calling Perl:

```
SELECT TRANSFORM (testdata.c1, testdata.c2) USING "perl -e 'while($input = <STDIN>){print $input;}'" FROM testdata;
```

**Note:** Currently PHP and Ruby are not deployed in the MaxCompute cluster. Therefore, MaxCompute does not allow you to call PHP or Ruby scripts.

## Call scripts in tandem { .section}

SELECT TRANSFORM allows you to call scripts in tandem. For example, you can use distribute by and sort by to preprocess data.

```
SELECT TRANSFORM(key, value) USING 'cmd2' from 
(
    SELECT TRANSFORM(*) USINg 'cmd1' from 
    (
        SELECt * FROM data distribute by col2 sort by col1
    ) t distribute by key sort by value
) t2;
```

You can use either the map or reduce keywords with the same results.

```
@a := select * from data distribute by col2 sort by col1;
@b := map * using 'cmd1' distribute by col1 sort by col2 from @a;
reduce * using 'cmd2' from @b;
```

## SELECT TRANSFORM performance {#section_ymd_1dc_wdb .section}

The performance of SELECT TRANSFORM and UDTF varies depending on the specific scenario. In general, the performance of SELECT TRANSFORM is optimal when the data size is comparatively small, and the performance of UDTF is optimal when the data size is large.

SELECT TRANSFORM is easier to use, and therefore is more suitable for adhoc data analysis.

**Benefits of UDTF** 

-   In UDTF, the output and input must follow the specified data types, and therefore UDTF does not require data type conversion as is required by SELECT TRANSFORM.
-   In UDTF, processes are not suspended if the operating system pipe is empty or fully occupied as is the case for SELECT TRANSFORM.
-   In UDTF, constant parameters do not need to be transmitted, which is required by SELECT TRANSFORM.

**Benefits of SELECT TRANSFORM** 

-   SELECT TRANSFORM supports a child process and a parent process and therefore can take advantage of multiple cores in servers.
-   SELECT TRANSFORM calls underlying systems to read and write the data to be transmitted, which allows it to have performance that is higher than that of Java.
-   SELECT TRANSFORM supports tools such as AWK and can run native code. This offers SELECT TRANSFORM more advantages compared to Java.

