# Type conversions {#concept_wyb_sgl_vdb .concept}

MaxCompute SQL allows conversion between data types. The two conversion methods are explicit type conversion and implicit type conversion.

## Explicit conversion {#section_gvj_1jl_vdb .section}

Explicit conversions use CAST to convert a value type to another. The following table lists the types that can be explicitly converted in MaxCompute SQL.

|From/To|Bigint|Double|String|Datetime|Boolean|Decimal|
|:------|:-----|:-----|:-----|:-------|:------|:------|
|Bigint|–|Y|Y|N|N|Y|
|Double|Y|–|Y|N|N|Y|
|String|Y|Y|–|Y|N|Y|
|Datetime|N|N|Y|–|N|N|
|Boolean|N|N|N|N|–|N|
|Decimal|Y|Y|Y|N|N|–|

Y means can be converted. N means cannot be converted. – means conversion is not required.

Example:

```
select cast(user_id as double) as new_id from user;
select cast('2015-10-01 00:00:00' as datetime) as new_date from user;
```

**Note:** 

-   To convert the Double type to the Bigint type, digits after the decimal point are dropped. For example, `cast(1.6 as bigint) = 1`.
-   To convert the String type that meets the Double format to the Bigint type, it is converted to the Double type, and then to the Bigint type. The digits after the decimal point are dropped. For example, `cast(“1.6” as bigint) = 1`. 
-   The String type that meets the Bigint format can be converted to the Double type, and must keep one digit after the decimal point. For example, `cast(“1” as double) = 1.0`.
-   Explicit conversions of unsupported types may return an exception.
-   If a conversion fails during execution, the conversion is aborted with an exception.
-   To convert the Datetime type, use the default format yyyy-mm-dd hh:mi:ss. For more information, see [Conversions between the String type and the Datetime type](reseller.en-US/User Guide/SQL/Type conversion.md#section_qkb_fql_vdb).
-   Some types cannot be explicitly converted, but can be converted using built-in SQL functions. For example, the to\_char function can be used to convert values of the Boolean type to the String type. For more information, see [TO\_CHAR](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md). The to\_date function can be used to convert values of the String type to the Datetime type. For more information, see [TO\_DATE](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md).
-   For more information, see [CAST](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md).
-   If a DECIMAL value exceeds the value range, MSB overflow error or LSB overflow truncation may occur for CAST STRING TO DECIMAL.

## Implicit conversion and scope {#section_uhl_sml_vdb .section}

Implicit type conversion is an automatic type conversion performed by MaxCompute according to the usage context and type conversion rules. The following table lists the types that can be implicitly converted using MaxCompute.

| |boolean|tinyint|smallint|int|bigint|float|double|Decimal|string|varchar|timestamp|binary|
|:-|:------|:------|:-------|:--|:-----|:----|------|-------|:-----|:------|:--------|:-----|
|boolean to|Y|N|N|N|N|N|N|N|N|N|N|N|
|tinyint to|N|Y|Y|Y|Y|Y|Y|Y|Y|Y|N|N|
|smallint to|N|N|Y|Y|Y|Y|Y|Y|Y|Y|N|N|
|int to|N|N|Y|Y|Y|Y|Y|Y|Y|Y|N|–|
|bigint to|N|N|N|N|Y|Y|Y|Y|Y|Y|N|N|
|float to|N|N|N|N|Y|Y|Y|Y|Y|Y|N|–|
|double to|N|N|N|N|N|N|Y|Y|Y|Y|N|N|
|decimal to|N|N|N|N|N|N|N|Y|Y|Y|N|N|
|string to|N|N|N|N|N|N|Y|Y|Y|Y|N|N|
|varchar to|N|N|N|N|Y|Y|Y|Y|N|N|–|–|
|timestamp to|N|N|N|N|N|N|N|N|Y|Y|Y|N|
|binary to|N|N|N|N|N|N|N|N|N|N|N|Y|

Y means can be converted. N means cannot be converted.

**Note:** 

-   The DECIMAL type and Datetime constant definition mode are added to MaxCompute2.0. 100BD indicates a DECIMAL, the value is 100. Datetime `2017-11-11 00:00:00` indicates a constant of the Datetime type. The constant definition is convenient because it can be directly used in values clauses and tables.
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

    Relational operators include equal to \(=\), not equal to \(<\>\), less than \(<\), less than or equal to \(<=\), greater than \(\>\), greater than or equal to \(\>=\), IS NULL, IS NOT  NULL, LIKE, RLIKE, and IN. For the particularities, implicit conversion rules of LIKE, RLIKE, and IN are discussed separately. The following descriptions do not contain these three special operators.

    The following table describes implicit conversion rules when different types of data is involved in relational operations.

    |From/To|Bigint|Double|String|Datetime|Boolean|Decimal|
    |:------|:-----|:-----|:-----|:-------|:------|:------|
    |Bigint|–|Double|Double|N|N|Decimal|
    |Double|Double|–|Double|N|N|Decimal|
    |String|Double|Double|–|Datetime|N|Decimal|
    |Datetime|N|N|Datetime|–|N|N|
    |Boolean|N|N|N|N|–|N|
    |Decimal|Decimal|Decimal|Decimal|N|N|-|

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

        -   The source and pattern parameters of LIKE and RLIKE can only be of the String type.
        -   Other types can neither be involved in the operations nor be implicitly converted to the String type.
    -   The usage of IN is as follows:

```
key in (value1, value2, …)
```

        Implicit conversion rules of IN:

        -   Data in the value column must be consistent.
        -   To compare keys and values, if Bigint, Double, and String types are compared, convert them to Double type. If the Datetime and String types are compared, convert them to Datetime type. Conversions between other types are not allowed.
-   **Implicit conversions under arithmetic operators**

    Arithmetic operators include addition \(+\), subtraction \(-\), multiplication \(\*\), division \(/\), modulo \(%\), unary plus \(+\), and unary minus \(-\). Their implicit conversion rules are described as follows:

    -   Only the String, Bigint, Double, and Decimal types can be involved in the operation.
    -   The String type are implicitly converted to the Double type before the operation.
    -   When the Bigint and Double types are involved in the operation, the Bigint type is implicitly converted to the Double type.
    -   The Datetime and Boolean types are not allowed in the arithmetic operation.
-   **Implicit conversions under logical operators**

    Logical operators include AND, OR, and NOT. Their implicit conversion rules are as follows:

    -   Only the Boolean type can be involved in the logical operation.
    -   Other types are not allowed in the logical operation, and cannot be implicitly converted to other types.

## Implicit conversions for Built-in functions {#section_mzm_1ql_vdb .section}

MaxCompute SQL provides numerous system functions. You can calculate one or multiple columns of any row and output data of any type. Their implicit conversion rules are described as follows:

-   To call a function, if the data type of an input parameter is different from that defined in the function, convert the data type of the input parameter to that defined in the function.
-   Parameters of different built-in functions of MaxCompute SQL have different requirements on implicit conversions. For more information, see [Built-in Functions](reseller.en-US/User Guide/SQL/Builtin functions/Mathematical functions.md).

## Implicit conversions under CASE WHEN {#section_if3_cql_vdb .section}

For more information about CASE WHEN, see [CASE WHEN Expressions](reseller.en-US/User Guide/SQL/Builtin functions/Other functions.md).  Its implicit conversion rules are listed as follows:

-   If the types of the returned values are Bigint and Double, convert all to the Double type.
-   If a String type exists in return types, convert all to the String type. If the conversion fails \(such as Boolean type conversion\), an error is returned.
-   Conversions between other types are not allowed.

## Conversions between the String Type and Datetime Type {#section_qkb_fql_vdb .section}

MaxCompute supports conversions between the String type and Datetime type.  The conversion format is `yyyy-mm-dd hh:mi:ss`.

|Unit|String \(case-insensitive\)|Value range|
|:---|:--------------------------|:----------|
|Year|yyyy|0001 - 9999|
|Month|mm|01 - 12|
|Day|dd|01 - 28,29,30,31|
|Hour|hh|00 - 23|
|Minute|mi|00 - 59|
|Second|ss|00 - 59|

**Note:** 

-   In the value range of each unit, if the first digit is 0, it cannot be ignored. For example, `2014-1-9 12:12:12` is an invalid Datetime format and it cannot be converted from the STRING type to the Datetime type. It must be written as `2014-01-09  12:12:12`.
-   Only the String type that meets the preceding format requirements can be converted to the Datetime type. For example, `cast(“2013-12-31 02:34:34” as  datetime)` converts`2013-12-31 02:34:34`of the String type to the Datetime type.  Similarly, when the Datetime type is converted to the String type, the default conversion format is yyyy-mm-dd hh:mi:ss.

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

MaxCompute provides the TO\_DATE function to convert the String type that does not meet the Datetime format to the Datetime type. For more information, see [TO\_DATE](reseller.en-US/User Guide/SQL/Builtin functions/Date functions.md).

