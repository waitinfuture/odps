# Announcements {#concept_p4b_ndc_5db .concept}

This topic lists updates to MaxCompute in descending chronological order.

## March 1, 2019: External tables of MaxCompute begin to incur charges. {#section_prm_zcd_5gb .section}

Starting from March 1, 2019, SQL external tables \(which are used to process OSS data and Table Store data\) of MaxCompute begin to incur charges.

The charging policy is as follows:

```
One-time SQL computing fee = Input data volume x SQL complexity x SQL price
```

The SQL price is 0.0044 USD/GB/Complexity. The complexity coefficient is 1. All the fees are charged on the next day, and you will receive an account bill.

For more information, see [Billing](../../../../../intl.en-US/Pricing/Billing.md#).

If you have any questions, open a ticket.

## January 15, 2019: The underlying structure of MaxCompute in China \(Hong Kong\) is optimized from 16:00 to 20:00. {#section_pts_yhk_lgb .section}

The underlying metadata warehouse of MaxCompute in China \(Hong Kong\) is optimized from 16:00 to 20:00 on January 15, 2019 to improve the performance and stability of MaxCompute. During the release window, users in the **Hong Kong** region may encounter submission delays or failures for tasks, which may last about **one minute**. In the worst cases, the application may be **unavailable for 30 minutes** \(or half an hour\). Therefore, we recommend that you do not submit any tasks during the release window. If you have any questions, contact us through DingTalk or by opening a ticket. Users in other regions are not affected.

## December 24, 2018: MaxCompute supports time zone configuration. {#section_wjq_mrc_ggb .section}

MaxCompute project uses the UTC+8 time zone by default. The time-related built-in functions and datetime, timestamp, date fields are calculated based on UTC+8. From December 24, 2018, users can configure time zones in MaxCompute using either of the following methods:

-   Session level: Submit the `set odps.sql.timezone=<timezoneid>;` SQL statement and a calculation statement. The following is an example:

    ```
    set odps.sql.timezone=Asia/Tokyo;
    select getdate();
    --Results:
    output:
    +------------+
    | _c0        |
    +------------+
    | 2018-10-30 23:49:50 |
    +------------+
    ```

-   Project level: The project owner runs the `setProject odps.sql.timezone=<timezoneid>;` SQL statement using a CLI. After a project is configured, the corresponding time zone is used automatically, which will affect the data of existing tasks. Therefore, we recommend that you do not configure the existing projects. Instead, you can configure new projects as needed.

**Note:**

-   The time zone configuration supports SQL built-in date functions, UDF, UDT, UDJ, and select transform.
-   The time zone format such as Asia/Shanghai \(daylight saving is considered\) is supported. The GMT+9 format is not supported.
-   If the time in the SDK time zone differs from that in the project time zone, you need to configure the GMT time zone so as to convert the date data to a string.
-   After the time zone is configured, there might be difference between the real time and the output time when you run the related SQL statements through MaxCompute. Between the years of 1900 and 1928, the time difference is 352 seconds. Before the year of 1900, the time difference is 9 seconds.
-   To ensure the datetime data accuracy in different regions, we will upgrade MaxCompute and the Java SDK and related console versions with the `-oversea` suffix. After the upgrade, the display of existing datetime data \(before the year of 1928\) stored in MaxCompute might be affected.
-   When you upgrade MaxComput, we recommend that you upgrade the Java SDK and console versions if the local time zone is not UTC+8, so as to ensure the accuracy and consistency between the SQL computing result and the Tunnel transferred data after '1900-01-01'. For the datetime data before '1900-01-01', the SQL computing output and the Tunnel transferred data might differ by 343 seconds. For the existing datetime data before '1928-01-01', the time difference is 352 seconds.
-   If you continue using the SDK and console versions without the `-oversea` suffix, you might encounter time difference between the SQL output and the Tunnel transferred data. The time difference before '1900-01-01' is 9 seconds, and the time difference between '1900-01-01' and '1928-01-01' is 352 seconds.

    **Note:** 

    When you update or configure the time zone of Java SDK and console versions, the time zone of DataWorks remains unchanged. Therefore, there might be time difference and you need to evaluate the impact of task scheduling in DataWorks. In the Japan region, the time zone of DataWorks is GMT+9. In the Singapore region, the time zone of DataWorks is GMT+8.

-   If you are using a third-party client connected by JDBC, you need to set the time zone on the client to ensure the time consistency.
-   MapReduce supports time zone configuration.
-   Spark supports time zone configuration.
    1.  For tasks that are submitted to ODPS computing clusters, the project time zone can be automatically obtained.
    2.  For settings that are made through the yarn-client method, such as spark-shell, spark-sql, and pyspark, you need to configure the spark-defaults.conf parameter of the driver and add `spark.driver.extraJavaOptions -Duser.timezone=America/Los_Angeles`. The 'timezone' in the preceding statement is the time zone to be used.
-   PAI supports time zone configuration.
-   Graph supports time zone configuration.

