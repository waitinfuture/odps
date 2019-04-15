# Announcements {#concept_p4b_ndc_5db .concept}

This topic lists updates to MaxCompute in descending chronological order.

## March 26, 2019: MaxCompute SQL is upgraded. {#section_v9x_myl_w17 .section}

-   The `GROUPING SETS` clause \(together with the `CUBE` and `ROLLUP` subclauses\) can be used to aggregate and analyze data of multiple dimensions. For more information, see [Grouping Sets](../../../../../reseller.en-US/User Guide/SQL/Select Operation/Grouping Sets.md#).
-   The `INTERSECT`, `MINUS`, and `EXCEPT` clauses gained support. For more information, see [UNION, INTERSECT, and EXCEPT](../../../../../reseller.en-US/User Guide/SQL/Select Operation/UNION, INTERSECT, and EXCEPT.md#).
-   When the system reads files in ORC or Parquet format by using external tables, it can crop the columns in the files, which can reduce I/O and resource usage, thereby lowering overall computing costs.
-   Systems that run in the Java UDX framework are enhanced to support writable parameters. For more information, see [Java UDF](../../../../../reseller.en-US/User Guide/SQL/UDF/Java UDF.md#).

## January 15, 2019: The underlying structure of MaxCompute in China \(Hong Kong\) is optimized from 16:00 to 20:00. {#section_pts_yhk_lgb .section}

The underlying metadata warehouse of MaxCompute in China \(Hong Kong\) is optimized from 16:00 to 20:00 on January 15, 2019 to improve the performance and stability of MaxCompute. During the release window, users in the **Hong Kong** region may encounter submission delays or failures for tasks, which may last about **one minute**. In the worst cases, the application may be **unavailable for 30 minutes** \(or half an hour\). Therefore, we recommend that you do not submit any tasks during the release window. If you have any questions, contact us through DingTalk or by opening a ticket. Users in other regions are not affected.

