# Date functions {#concept_akb_2c2_5db .concept}

This article explains various functions that MaxCompute SQL offers to operate datetime types.

## DATEADD {#section_qjz_lrl_vdb .section}

Command format:

```
datetime dateadd(datetime date, bigint delta, string datepart)
```

Command description:

Modify the value of date according to a specified unit ‘datepart’ and specified scope ‘delta’.

Parameter description:

-   date: Datetime type, value of date.  If the input is string type, it is converted to ‘datetime’ type by implicit conversion. If it is another type, an exception is indicated.
-   delta: Bigint type, date scope to be modified.  If the input is ‘string’ type or ‘double’ type, it is converted to ‘bigint’ type by implicit conversion. If it is another data type, exception occurs.  If ‘delta’ is greater than zero, do ‘add’ operation, otherwise do ‘minus’ operation.
-   datepart: a String type constant. This field value follows ‘string’ and ‘datetime’ type conversion agreement, where, ‘yyyy’ indicates year; ‘mm’ indicates month.

    See Conversion between [String type and Datetime type](reseller.en-US/User Guide/SQL/SQL summary.md).  In addition, the extensional date format is also supported: year- ‘year’; month-‘month’ or ‘mon’; day-‘day’; hour-‘hour.  If it is not a constant or unsupported format or other data type, an exception is indicated.


Return value:Datetime type. If any input is NULL, return NULL.

**Note:** 

-   While increasing or decreasing ‘delta’ according to specified unit, it causes the carry or back space for higher unit. Day, month, hour, minute, second are calculated by 10 hexadecimal, 12 hexadecimal, 24 hexadecimal, 60 hexadecimal, 60 hexadecimal respectively.
-   If the unit of ‘delta’ is month, the calculation rule is shown as follows: 

    If the month part of ‘datetime’ does not cause the spillover of day after adding ‘delta’, then do not change the day, else the day value is set to the last day of the result month.

-   The value of ‘datepart’ follows ‘string’ and ‘datetime’ type conversion agreement, that is, ‘yyyy’ indicates year; ‘mm’ indicates month and so on. If no special description exists, related datetime built-in functions follow this agreement. Moreover, if no special instructions, the part of all datetime built-in functions supports extended date format: year- ‘year’; month-‘month’ or ‘mon’; day-‘day’; hour-‘hour.

For example:

```
 if trans_date = 2005-02-28 00:00:00
dateadd(trans_date, 1, 'dd') = 2005-03-01 00:00:00
-- Add one day. The result is beyond the last day in February. The actual value is the first day of next month.
dateadd(trans_date, -1, 'dd') = 2005-02-27 00:00:00
-- Minus one day.
dateadd(trans_date, 20, 'mm') = 2006-10-28 00:00:00
-- Add 20 months. The month spillover is caused and the year is added ‘1’.
If trans_date = 2005-02-28 00:00:00, dateadd(transdate, 1, 'mm') = 2005-03-28 00:00:00
If trans_date = 2005-01-29 00:00:00, dateadd(transdate, 1, 'mm') = 2005-02-28 00:00:00
-- No 29th is in Feb. of 2005. The date is intercepted to the last day of current month.
 If trans_date = 2005-03-30 00:00:00, dateadd(transdate, -1, 'mm') = 2005-02-28 00:00:00
```

**Note:** Here the value of trans\_date used only as an example. This simple expression is often used to present the datetime in this file.

In MaxCompute SQL, the datetime type has no direct constant representation, the following usage is wrong:

```
select dateadd(2005-03-30 00:00:00, -1, 'mm') from tbl1;
```

If you must describe the datetime type constant, try the following methods:

```
select dateadd(cast("2005-03-30 00:00:00" as datetime), -1, 'mm') from tbl1;
-- The String type constant is converted to datatime type by explicit conversion.
```

## DATEDIFF {#section_xl2_nsl_vdb .section}

Command format:

```
bigint datediff(datetime date1, datetime date2, string datepart)
```

Command description:

Calculate the difference between two datetime date1 and date2 in specified time unit ‘datepart’.

Parameter description:

-   datet1, date2: Datetime type, minuend, meiosis. If the input is ‘string’, it is converted to ‘datetime’ by implicit conversion. If it is another data type, an exception indicated.
-   datepart: a String type constant. The extensional date format is supported. If ‘datepart’ does not meet the specified format or is other data type, an exception is indicated.

Return value:

Returns the Bigint type. Any input parameter is NULL, return NULL. If date1 is less than date2, then the returned value may be negative.

**Note:** The lower unit part is cut off according to ‘datepart’ in the calculation process and then calculate the result.

For example:

```
If start = 2005-12-31 23:59:59, end = 2006-01-01 00:00:00:
    datediff(end, start, 'dd') = 1
    datediff(end, start, 'mm') = 1
    datediff(end, start, 'yyyy') = 1
    datediff(end, start, 'hh') = 1
    datediff(end, start, 'mi') = 1
    datediff(end, start, 'ss') = 1
    datediff('2013-05-31 13:00:00', '2013-05-31 12:30:00', 'ss') = 1800
    datediff('2013-05-31 13:00:00', '2013-05-31 12:30:00', 'mi') = 30
If start = 19:33:23. 234, end = 19:33:23. 250 .Dates with milliseconds do not belong to the standard datetime style, and cannot be converted implicitly directly.Explicit conversion is required here:
  
datediff(to_date('2018-06-04 19:33:23.250', 'yyyy-MM-dd hh:mi:ss.ff3'),to_date('2018-06-04 19:33:23.234', 'yyyy-MM-dd hh:mi:ss.ff3') , 'ff3') = 16
```

## DATEPART {#section_am4_xtl_vdb .section}

Command format:

```
bigint datepart(datetime date, string datepart)
```

Command format:

Extracts the value of the specified time unit ’datepart’ in ‘date’.

Parameter description:

Return value:

-   date: Datetime type. If the input is ‘string’ type, it is converted to ‘datetime’ type. If it is another data type, an exception is indicated.
-   datepart: String type constant. The extensional date format is supported. If ‘datepart’ does not meet the specified format or is other data type, an exception is indicated.

Returns the Bigint type. If any input is NULL, return NULL.

For example:

```
datepart('2013-06-08 01:10:00', 'yyyy') = 2013
datepart('2013-06-08 01:10:00', 'mm') = 6
```

## DATETRUNC {#section_zbr_d5l_vdb .section}

Command format:

```
datetime datetrunc (datetime date, string datepart)
```

Usage:：

Return the remained date value after the specified time unit ‘datepart’ has been intercepted.

Parameter description:：

-   date: Datetime type. If the input is ‘string’ type, it is converted to ‘datetime’ type. If it is another data type, an exception indicated.
-   datepart: String type constant. The extensional date format is supported.  If ‘datepart’ does not meet the specified format or is other data type, an exception is indicated.

Return value:

Datetime type. If any input is NULL, return NULL.

For example:

```
datetrunc('2011-12-07 16:28:46', 'yyyy') = 2011-01-01 00:00:00
datetrunc('2011-12-07 16:28:46', 'month') = 2011-12-01 00:00:00
datetrunc('2011-12-07 16:28:46', 'DD') = 2011-12-07 00:00:00
```

## GETDATE {#section_o4p_45l_vdb .section}

Command format:

```
datetime getdate()
```

Command description:

Get present system time.  Use UTC+8 as MaxCompute standard time.

Return value:

Datetime type, return present date and time.

**Note:** In a MaxCompute SQL task \(executed in a distributed manner\), ‘getdate’ always returns a fixed value.  The return result is any time in MaxCompute  SQL execution period and the precision of time is accurate to seconds.

## ISDATE {#section_rzl_s5l_vdb .section}

Command format:

```
boolean isdate(string date, string format)
```

Command description:

Determines whether a date string can be converted to a datetime value according to corresponding format string. If the conversion is successful, return TRUE, otherwise return FALSE.

Parameter description:

-   date: date value of String format. If the input is ‘bigint’, or ‘double’ or ‘datetime’, it is be converted to ‘string’ type. If it is another data type, an exception is indicated.
-   format: a String type constant. The extensional date format is not supported.  If redundant format strings appear in ‘format’, then get the datatime value corresponding to the first format string, other strings are taken as separators.  For example,  isdate \(‘1234-yyyy’, ‘yyyy-yyyy’\) returns ‘TRUE’.

Return value:

Boolean type. If any parameter is NULL, return NULL.

## LASTDAY {#section_vhk_w2m_vdb .section}

Command format:

```
datetime lastday(datetime date)
```

Command format:

Get the last day in the same month of the date, intercepted to day and the ‘hh:mm:ss’ part is ‘00:00:00’.

Parameter description:

date: Datetime type. If the input is ‘string’ type, it is converted to ‘datetime’ type. If it is another data type, an exception is reported.

Return value:

Datetime type. If the input is NULL, return NULL.

## TO\_DATE {#section_b3z_1fm_vdb .section}

Command format:

```
datetime to_date(string date, string format)
```

Command description:

Convert a string ‘date’ to the datetime value according to a specified format.

Parameter description:

-   date: String type, date value to be converted. If the input is ‘bigint’, or ‘double’ or ‘datetime’, it is converted to ‘string’ type by implicit conversion. If it is another data type or null, an exception is indicated.
-   format: String type constant, date format.  If it is not a constant or is other data type, the exception is caused.  The field ‘format’ does not support extensional format and other characters are ignored as invalid characters in analysis process. 

    The parameter format contains ‘yyyy’ at least; otherwise the expecion is indicated. If redundant format strings appear in format, then get the datatime value corresponding to the first format string, other strings are taken as separators.  For example, `to_date (‘1234-2234’, ‘yyyy-yyyy’)`  returns ‘1234-01-01 00:00:00’.

    Format format: yyyy is a four-digit year, mm is a two-digit month, DD is a two-digit day, HH is a 24-hour system, MI is a two-digit minute, SS is a two-digit second, FF3 is a three-digit precision millisecond.


Return value:

Datetime type, the format is yyyy-mm-dd hh: mi: ss. If any input is NULL, return NULL.

For example:

```
to_date('Alibaba2010-12*03', 'Alibabayyyy-mm*dd') = 2010-12-03 00:00:00
to_date('20080718', 'yyyymmdd') = 2008-07-18 00:00:00
to_date('200807182030','yyyymmddhhmi')=2008-07-18 20:30:00
to_date('2008718', 'yyyymmdd') = null
-- The format does not meet the requirements. An exception is thrown.
to_date('Alibaba2010-12*3', 'Alibabayyyy-mm*dd') = null
-- Format is not compatible and exception is thrown.
to_date('2010-24-01', 'yyyy') = null
-- Format is not compatible and exception is thrown.
to_date('20181030 15-13-12.345','yyyymmdd hh-mi-ss.ff3')=2018-10-30 15:13:12
```

## TO\_CHAR {#section_a2d_rfm_vdb .section}

Command format:

```
string to_char(datetime date, string format)
```

Command description:

Convert the ‘date’ of datetime type to a string according to a specified format.

Parameter description:

-   date: Datetime type, the date value to be converted. If the input is ‘string’ type, it is converted to ‘datetime’ type by implicit conversion. If it is another data type, an exception indicated.
-   format: String type constant.  If it is not a constant or is other data type, the exception is indicated.  In ‘format’, the date format part is replaced with the corresponding data and other characters are output directly.

Return value: 

Returns the String type. Any input parameter is NULL, return NULL.

For example:

```
to_char('2010-12-03 00:00:00', 'Alibabayyyy-mm*dd') = 'Alibaba2010-12*03'
to_char('2008-07-18 00:00:00', 'yyyymmdd') = '20080718' 
to_char('Alibaba2010-12*3', 'Alibabayyyy-mm*dd') -- Format is not compatible and exception is thrown.
to_char('2010-24-01', 'yyyy') -- Format is not compatible and exception is thrown.
to_char('2008718', 'yyyymmdd') -- Format is not compatible and exception is thrown.
```

See [TO\_CHAR](reseller.en-US/User Guide/SQL/Builtin functions/String functions.md) for conversion from other types to string type.

## UNIX\_TIMESTAMP {#section_k4r_zfm_vdb .section}

Command format:

```
bigint unix_timestamp(datetime date)
```

Command description:

Convert the date of Datetime type to UNIX format date of Bigint type.

Parameter description:

date: Datetime type date value. If the input is ‘string’ type, it is converted to ‘datetime’ type and involved in calculation. If it is another type, an exception indicated.

Return value:

Bigint type, it indicates UNIX format date value. If ‘date’ is NULL, return NULL.

## FROM\_UNIXTIME {#section_yzv_j5l_vdb .section}

Command format:

```
datetime from_unixtime(bigint unixtime)
```

Command description:

Convert the numeric UNIX time value ‘unixtime’ to datetime value.

Parameter description:

unixtime: Bigint type, number of seconds, UNIX format date time value. If the input is ‘string’, ‘double’, it is converted to ‘bigint’ type by implicit conversion.

Return value:

Datetime type date value. If ‘unixtime’ is NULL, return NULL.

For example:

```
from_unixtime(123456789) = 1973-11-30 05:33:09
```

## WEEKDAY {#section_g41_2gm_vdb .section}

Command format:

```
bigint weekday(datetime date)
```

Command description:

 Return the nth day of present week corresponding to the date.

Parameter description:

date: Datetime type. If the input is ‘string’ type, it is converted to ‘datetime’ type and then involved in operation. If it is another date type, an exception indicated.

Return value:

Bigint type. If the input parameter is NULL, return NULL.  Monday is the first day of a week and the return value is 0. Days are in ascending order starting from 0. If the day is Sunday, then return is 6.

## WEEKOFYEAR {#section_rjv_hgm_vdb .section}

Command format:

```
bigint weekofyear(datetime date)
```

Command description:

Return the nth week of a year which the date is included in.  Monday is taken as the first day of a week.

**Note:** Whether this week belongs to this year, or the next year, it depends on which year \(4 days or more\)  most of the time of this week belongs to.

Parameter description:

date: Datetime type. If the input is ‘string’ type, it is converted to ‘datetime’ type and then involved in operation. If it is another date type, an exception is indicated.

Return value:

Bigint type.  If the input is NULL, return NULL.

For example:

```
select weekofyear(to_date("20141229", "yyyymmdd")) from dual;  
Result:  
+------------+
| _c0 |
+------------+
| 1 |
+------------+
 -Although 20141229 belongs to 2014, most of the dates of the week are in 2015, therefore, the return result is 1, indicating that it is the first week of 2015.    
 select weekofyear(to_date("20141231", "yyyymmdd")) from dual；
-- Return 1.  
 select weekofyear(to_date("20141229", "yyyymmdd")) from dual；
-- Return 53.
```

## Maxcomputerte2.0 New Extended Mathematical Functions {#section_kj2_zhm_vdb .section}

With the upgraded version of MaxCompute 2.0, some new date functions are added to the product.  If the functions are used to design a new data type compatible with the Hive mode, you must add the following two set statements before the SQL statement of the new functions:

```
set odps.sql.type.system.odps2=true;--Enable the new type.
```

If you want to submit both at the same time, run the following statements:

```
set odps.sql.type.system.odps2=true;
select year('1970-01-01 12:30:00')=1970 from dual;
```

The new extended functions are described as follows.

## YEAR {#section_gb4_g3m_vdb .section}

Command format:

```
INT year(string date)
```

Command description:

Returns the year of a date.

Parameter description:

date: String-type date value. The format must at least include ‘yyyy-mm-dd’ and cannot include additional strings. Otherwise, null is returned.

Return value:

INT type.

For example:

```
year('1970-01-01 12:30:00') = 1970
year('1970-01-01') = 1970
year('70-01-01') = 70
year(1970-01-01) = null
year('1970/03/09') = null
year(null) Returns an exception
```

## QUARTER {#section_ckz_m3m_vdb .section}

Command format:

```
INT quarter(datetime/timestamp/string date )
```

**Note:** Before the SQL statement which uses the QUARTER function,add `set odps.sql.type.system.odps2=true;` to use the new data type function normally.

Command description:

Returns the quarter of a date, range: 1–4.

Parameter description:

date: Datetime, Timestamp, or String-type date value. The format must at least include ‘yyyy-mm-dd’. Otherwise, null is returned.

Return value:

Int type, null input returns null.

For example:

```
quarter('1970-11-12 10:00:00') = 4
quarter('1970-11-12') = 4
```

## MONTH {#section_atw_s3m_vdb .section}

Command format:

```
INT month(string date)
```

**Note:** Before the SQL statement which uses the MONTH function, add `set odps.sql.type.system.odps2=true;` to use the new data type function normally.

Command description:

Returns the month of a date.

Parameter description:

date: String-type date value. Other value types return an exception.

Return value:

INT type.

For example:

```
month('2014-09-01') = 9
month('20140901') = null
```

## DAY {#section_lt5_w3m_vdb .section}

Command format:

```
INT day(string date)
```

**Note:** Before the SQL statement which uses the function, add `set odps.sql.type.system.odps2=true;` to use the new data type function normally.

Command description:

Returns the day of a date.

Parameter description:

date: String-type date value. Other  value types return an exception.

Return value:

INT type.

For example:

```
day('2014-09-01') = 1
day('20140901') = null
```

## DAYOFMONTH {#section_k4p_fjm_vdb .section}

Command format:

```
INT dayofmonth(date)
```

**Note:** Before the SQL statement which uses the DAYOFMONTH function, add `set odps.sql.type.system.odps2=true;` to use the new data type function normally.

Command description:

Returns the day of a date.

For example,  after  command `int dayofmonth(2017-10-13)` runs, 13 returns.

Parameter description:

date: String-type date value. Other value types return an exception.

Return value:

INT type.

For example:

```
dayofmonth('2014-09-01') = 1
dayofmonth('20140901') = null
```

## HOUR {#section_lrl_jjm_vdb .section}

Command format:

```
INT hour(string date)
```

**Note:** Before the SQL statement which uses the HOUR function, add `set odps.sql.type.system.odps2=true;` to use the new data type function normally.

Command description:

Returns the hour of a date.

Parameter description:

date: String-type date value. Other value types return an exception.

Return value:

Int type.

For example:

```
hour('2014-09-01 12:00:00')=12
hour('12:00:00')=12
hour('20140901120000')=null
```

## MINUTE {#section_o15_mjm_vdb .section}

Command format:

```
INT minute(string date)
```

**Note:** Before the SQL statement which uses the MINUTE function, add `set odps.sql.type.system.odps2=true;` to use the new data type function normally.

Command description:

Returns the minute of a date.

Parameter description:

date: String-type date value. Other value types return an exception.

Return value:

Int type.

For example:

```
minute('2014-09-01 12:30:00') = 30
minute('12:30:00') = 30
minute('20140901120000') = null
```

## SECOND {#section_ylr_pjm_vdb .section}

Command format:

```
INT second(string date)
```

**Note:** Before the SQL statement which uses the SECOND function, add `set odps.sql.type.system.odps2=true;` to use the new data type function normally.

Command description:

Returns the second of a date.

Parameter description:

date: String-type date value. Other value types return an exception.

Return value:

INT type.

For example:

```
second('2014-09-01 12:30:45') = 45
second('12:30:45') = 45
second('20140901123045') = null
```

## CURRENT\_TIMESTAMP {#section_r1r_sjm_vdb .section}

Command format:

```
timestamp current_timestamp()
```

**Note:** Before the SQL statement which uses the CURRENT\_TIMESTAMP function, add `set odps.sql.type.system.odps2=true;` to use the new data type function normally.

Command description:

Returns the current timestamp as a Timestamp-type value. The value is not fixed.

Return value:

Timestamp type.

For example:

```
select current_timestamp() from dual;--Returns '2017-08-03 11:50:30.661'
```

## ADD\_MONTHS {#section_psn_vjm_vdb .section}

Command format:

```
string add_months(string startdate, int nummonths)
```

**Note:** Before the SQL statement which uses the ADD\_MONTHS function, add `set odps.sql.type.system.odps2=true;` to use the new data type function normally.

Command description:

Returns the date given by startdate plus the nummonths value.

Parameter description:

-   startdate: String-type value. The format must at least include the **‘yyyy-mm-dd’** date. Otherwise, null is returned.
-   num\_months: Int-type value.

Return value:

A String-type date, in the format ‘yyyy-mm-dd’.

For example:

```
Add_months ('2017-02-14', 3) = '2017-05-14'
add_months('17-2-14',3) = '0017-05-14'
add_months('2017-02-14 21:30:00',3) = '2017-05-14'
add_months('20170214',3) = null
```

## LAST\_DAY {#section_idn_1km_vdb .section}

Command format:

```
string last_day(string date)
```

**Note:** Before the SQL statement which uses the LAST\_DAY function, add `set odps.sql.type.system.odps2=true;` to use the new data type function normally.

Command description:

Returns the date of the last day of the month that contains the given date.

Parameter description:

date: String type, with the format ‘yyyy-MM-dd HH:mi:ss’ or ‘yyyy-MM-dd’.

Return value:

A String-type date, in the format ‘yyyy-mm-dd’.

For example:

```
last_day('2017-03-04') = '2017-03-31'
last_day('2017-07-04 11:40:00') = '2017-07-31'
last_day('20170304') = null
```

## NEXT\_DAY {#section_eqf_2km_vdb .section}

Command format:

```
string next_day(string startdate, string week)
```

**Note:** Before the SQL statement which uses the NEXT\_DAY function, add `set odps.sql.type.system.odps2=true;` to use the new data type function normally.

Command description:

Returns the first date larger than the specified startdate that matches the day of the week given by the week parameter. It is the date of a specific day in the next week.

Parameter description:

-   startdate: String type, with the format ‘yyyy-MM-dd HH:mi:ss’ or  ‘yyyy-MM-dd’.
-   week: String type, the first two or three letters of a day of the week, or the full name of the day of the week. For example: Mo, TUE, or FRIDAY.

Return value:

A String-type date, in the format ‘yyyy-mm-dd’.

For example:

```
next_day('2017-08-01','TU') = '2017-08-08'
next_day('2017-08-01 23:34:00','TU') = '2017-08-08'
Next_day ('20170801 ', 'tu') = NULL
```

## MONTHS\_BETWEEN {#section_cbq_4km_vdb .section}

Command format:

```
double months_between(datetime/timestamp/string date1, datetime/timestamp/string date2)
```

**Note:** Before the SQL statement which uses the MONTHS\_BETWEEN function, add `set odps.sql.type.system.odps2=true;` to use the new data type function normally.

Command description:

Returns the number of months between date1 and date2.

Parameter description:

-   date1: Datetime, Timestamp, or String type, with the format ‘yyyy-MM-dd  HH:mi:ss’ or ‘yyyy-MM-dd’.
-   date2: Datetime, Timestamp, or String type, with the format ‘yyyy-MM-dd  HH:mi:ss’ or ‘yyyy-MM-dd’.

Return Value:

Returns the Double type.

-   When date1 is later than date2, the returned value is positive. When date2 is later than date1, the returned value is negative.
-   When date1 and date2 correspond to the last days of two months, the returned value is an integer representing the number of months. Otherwise, the formula is \(date1 - date2\)/31.

Examples: 

```
months_between('1997-02-28 10:30:00', '1996-10-30') = 3.9495967741935485
months_between('1996-10-30','1997-02-28 10:30:00' ) = -3.9495967741935485
months_between('1996-09-30','1996-12-31') = -3.0
```

