# Type conversions {#concept_wyb_sgl_vdb .concept}

MaxCompute SQL allows for conversion between data types. The two conversion methods are explicit type conversion and implicit type conversion.

## Explicit conversion {#section_gvj_1jl_vdb .section}

Explicit conversions use CAST to convert a value type to another. The following table lists the types that can be explicitly converted in MaxCompute SQL.

|From/To|BIGINT|DOUBLE|STRING|DATETIME|BOOLEAN|DECIMAL|FLOAT|
|:------|:-----|:-----|:-----|:-------|:------|:------|-----|
|BIGINT|N/A|Yes|Yes|No|Yes|Yes|Yes|
|DOUBLE|Yes|N/A|Yes|No|Yes|Yes|Yes|
|STRING|Yes|Yes|N/A|Yes|Yes|Yes|Yes|
|DATETIME|No|No|Yes|N/A|No|No|No|
|BOOLEAN|Yes|Yes|Yes|No|N/A|Yes|Yes|
|DECIMAL|Yes|Yes|Yes|No|Yes|N/A|Yes|
|FLOAT|Yes|Yes|Yes|No|Yes|Yes|N/A|

Y means can be converted. N means cannot be converted. – means conversion is not required.

Example:

```
select cast(user_id as double) as new_id from user;
select cast('2015-10-01 00:00:00' as datetime) as new_date from user;
```

**Note:** 

-   To convert the Double type to the Bigint type, digits after the decimal point are dropped. For example, `cast(1.6 as bigint) = 1`.
-   To convert the STRING type that meets the Double format to the Bigint type, it is converted to the Double type, and then to the Bigint type. The digits after the decimal point are dropped. For example, `cast(“1.6” as bigint) = 1`.
-   The STRING type that meets the Bigint format can be converted to the Double type, and must keep one digit after the decimal point. For example, `cast(“1” as double) = 1.0`.
-   Explicit conversions of unsupported types may return an exception.
-   If a conversion fails during execution, the conversion is aborted with an exception.
-   To convert the DATETIME type, use the default format yyyy-mm-dd hh:mi:ss. For more information, see [Conversions between the STRING type and the DATETIME type](reseller.en-US/User Guide/SQL/Type conversions.md#section_qkb_fql_vdb).
-   Some types cannot be explicitly converted, but can be converted using built-in SQL functions. For example, the to\_char function can be used to convert values of the BOOLEAN type to the STRING type. For more information, see [TO\_CHAR](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md). The to\_date function can be used to convert values of the STRING type to the DATETIME type. For more information, see [TO\_DATE](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md).
-   For more information, see [CAST](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md).
-   If a DECIMAL value exceeds the value range, MSB overflow error or LSB overflow truncation may occur for CAST STRING TO DECIMAL.
-   MaxCompute supports conversion of complex data types. However, there are several conditions for conversions of complex data types. For the implicit conversion of complex data types, data subtypes must be supported. In Struct-type conversion, field names can be inconsistent, but the number of fields must be consistent, and the specified fields must be able to be converted implicitly or explicitly. The following are examples of complex data types and explanations as to whether they can be converted based on the preceding conditions:
    -   `array<bigINT>` can be implicitly or explicitly converted into `array<STRING>`.
    -   `array<bigINT>` can be explicitly converted into `array<INT>`, but cannot be implicitly converted.
    -   `array<bigINT>` cannot be implicitly or explicitly converted into `array<datetime>`.
    -   `struct<a:bigINT,b:INT>` can be implicitly converted into `struct<col1:STRING,col2:bigINT>`, but cannot be implicitly or explicitly converted into `struct<a:STRING>`.

## Implicit conversion and scope {#section_uhl_sml_vdb .section}

Implicit type conversion is an automatic type conversion performed by MaxCompute according to the usage context and type conversion rules. The following table lists the types that can be implicitly converted using MaxCompute.

| |BOOLEAN|TINYINT|SMALLINT|INT|BIGINT|FLOAT|double|Decimal|string|varchar|timestamp|binary|
|:-|:------|:------|:-------|:--|:-----|:----|------|-------|:-----|:------|:--------|:-----|
|boolean to|Yes|No|No|No|No|No|No|No|No|No|No|No|
|tinyint to|No|Yes|Yes|Yes|Yes|Yes|Yes|Yes|Yes|Yes|No|No|
|smallint to|No|No|Yes|Yes|Yes|Yes|Yes|Yes|Yes|Yes|No|No|
|int to|No|No|Yes|Yes|Yes|Yes|Yes|Yes|Yes|Yes|No|N/A|
|bigint to|No|No|No|No|Yes|Yes|Yes|Yes|Yes|Yes|No|No|
|float to|No|No|No|No|Yes|Yes|Yes|Yes|Yes|Yes|No|N/A|
|double to|No|No|No|No|No|No|Yes|Yes|Yes|Yes|No|No|
|decimal to|No|No|No|No|No|No|No|Yes|Yes|Yes|No|No|
|string to|No|No|No|No|No|No|Yes|Yes|Yes|Yes|No|No|
|varchar to|No|No|No|No|Yes|Yes|Yes|Yes|No|No|N/A|N/A|
|timestamp to|No|No|No|No|No|No|No|No|Yes|Yes|Yes|No|
|binary to|No|No|No|No|No|No|No|No|No|No|No|Yes|

Y means can be converted. N means cannot be converted.

**Note:** 

-   The DECIMAL type and DATETIME constant definition mode are added to MaxCompute2.0. 100BD indicates a DECIMAL, the value is 100. DATETIME`2017-11-11 00:00:00` indicates a constant of the DATETIME type. The constant definition is convenient because it can be directly used in values clauses and tables.
-   In the earlier version of MaxCompute, values of the DOUBLE type can be implicitly converted to the BIGINT type. Owing to some reasons, such conversions may lead to data loss, which is not allowed by common database systems.

Common use:

```
select user_id+age+'12345',
           concat(user_name,user_id,age)
  from user;
```

**Note:** 

-   Implicit conversions of unsupported types may cause an error.
-   If a conversion fails during execution, an exception occurs.
-   MaxCompute automatically performs implicit conversions based on the context environment. We recommend that you use CAST to perform an explicit conversion when the types do not match.
-   Implicit conversion rules are applicable to a specific range of scopes. In some scopes, only some rules can take effect. For more information, see the scopes of implicit conversions.

-   **Implicit conversions under relational operators** 

    Relational operators include equal to \(=\), not equal to \(<\>\), less than \(<\), less than or equal to \(<=\), greater than \(\>\), greater than or equal to \(\>=\), IS NULL, IS NOT NULL, LIKE, RLIKE, and IN. For the particularities, implicit conversion rules of LIKE, RLIKE, and IN are discussed separately. The following descriptions do not contain these three special operators.

    The following table describes implicit conversion rules when different types of data is involved in relational operations.

    |From/To|BIGINT|DOUBLE|STRING|DATETIME|BOOLEAN|DECIMAL|
    |:------|:-----|:-----|:-----|:-------|:------|:------|
    |BIGINT|N/A|DOUBLE|DOUBLE|No|No|DECIMAL|
    |DOUBLE|DOUBLE|N/A|DOUBLE|No|No|DECIMAL|
    |STRING|DOUBLE|DOUBLE|N/A|DATETIME|No|Decimal|
    |DATETIME|No|No|DATETIME|N/A|No|No|
    |BOOLEAN|No|No|No|No|N/A|No|
    |DECIMAL|DECIMAL|DECIMAL|DECIMAL|No|No|N/A|

    **Note:** 

    -   If two types cannot be implicitly converted, the relational operation is aborted by an error.
    -   For more information about the relational operators, see [Relational Operators](reseller.en-US/User Guide/SQL/Operators.md).
-   **Implicit conversions under special relational operators** 

    Special relational operators include LIKE, RLIKE, and IN.

    -   The usage of LIKE and RLIKE is as follows:

        ```
        source like pattern;  
        source rlike pattern;
        ```

        The following illustrates the notes for LIKE and RLIKE in implicit conversions:

        -   The source and pattern parameters of LIKE and RLIKE can only be of the STRING type.
        -   Other types can neither be involved in the operations nor be implicitly converted to the STRING type.
    -   The usage of IN is as follows:

```
key in (value1, value2, …)
```

        Implicit conversion rules of IN:

        -   Data in the value column must be consistent.
        -   To compare keys and values, if BIGINT, DOUBLE, and STRING types are compared, convert them to DOUBLE type. If the DATETIME and STRING types are compared, convert them to DATETIME type. Conversions between other types are not allowed.
-   **Implicit conversions under arithmetic operators** 

    Arithmetic operators include addition \(+\), subtraction \(-\), multiplication \(\*\), division \(/\), modulo \(%\), unary plus \(+\), and unary minus \(-\). Their implicit conversion rules are described as follows:

    -   Only the STRING, BIGINT, DOUBLE, and DECIMAL types can be involved in the operation.
    -   The STRING type are implicitly converted to the DOUBLE type before the operation.
    -   When the BIGINT and DOUBLE types are involved in the operation, the BIGINT type is implicitly converted to the DOUBLE type.
    -   The DATETIME and BOOLEAN types are not allowed in the arithmetic operation.
-   **Implicit conversions under logical operators** 

    Logical operators include AND, OR, and NOT. Their implicit conversion rules are as follows:

    -   Only the BOOLEAN type can be involved in the logical operation.
    -   Other types are not allowed in the logical operation, and cannot be implicitly converted to other types.

## Implicit conversions for Built-in functions {#section_mzm_1ql_vdb .section}

MaxCompute SQL provides numerous system functions. You can calculate one or multiple columns of any row and output data of any type. Their implicit conversion rules are described as follows:

-   To call a function, if the data type of an input parameter is different from that defined in the function, convert the data type of the input parameter to that defined in the function.
-   Parameters of different built-in functions of MaxCompute SQL have different requirements on implicit conversions. For more information, see [Built-in Functions](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md).

## Implicit conversions under CASE WHEN {#section_if3_cql_vdb .section}

For more information about CASE WHEN, see [CASE WHEN Expressions](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md). Its implicit conversion rules are listed as follows:

-   If the types of the returned values are BIGINT and DOUBLE, convert all to the DOUBLE type.
-   If a STRING type exists in return types, convert all to the STRING type. If the conversion fails \(such as BOOLEAN type conversion\), an error is returned.
-   Conversions between other types are not allowed.

## Conversions between the STRING Type and DATETIME Type {#section_qkb_fql_vdb .section}

MaxCompute supports conversions between the STRING type and DATETIME type. The conversion format is `yyyy-mm-dd hh:mi:ss`.

|Unit|STRING\(case-insensitive\)|Value range|
|:---|:-------------------------|:----------|
|Year|yyyy|0001 - 9999|
|Month|mm|01 - 12|
|Day|dd|01 - 28, 29, 30, 31|
|Hour|hh|00 - 23|
|Minute|mi|00 - 59|
|Second|ss|00 - 59|

**Note:** 

-   In the value range of each unit, if the first digit is 0, it cannot be ignored. For example, `2014-1-9 12:12:12` is an invalid DATETIME format and it cannot be converted from the STRING type to the DATETIME type. It must be written as `2014-01-09 12:12:12`.
-   Only the STRING type that meets the preceding format requirements can be converted to the DATETIME type. For example, `cast(“2013-12-31 02:34:34” as datetime)` converts`2013-12-31 02:34:34`of the STRING type to the DATETIME type. Similarly, when the DATETIME type is converted to the STRING type, the default conversion format is yyyy-mm-dd hh:mi:ss.

For example, the following conversions return an exception:

```
cast("2013/12/31 02/34/34" as datetime)  
cast("20131231023434" as datetime)  
cast("2013-12-31 2:34:34" as datetime)
```

The threshold of dd depends on the actual days of a month. If the value exceeds the actual days of the month, the conversion is aborted with an error.

**Example:** 

```
cast("2013-02-29 12:12:12" as datetime) -- Returns an error because February 29, 2013 does not exist.   
cast("2013-11-31 12:12:12" as datetime) -- Returns an exception because November 31, 2013 does not exist.
```

MaxCompute provides the TO\_DATE function to convert the STRING type that does not meet the DateDATETIME ime format to the DATETIME type. For more information, see [TO\_DATE](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md).

