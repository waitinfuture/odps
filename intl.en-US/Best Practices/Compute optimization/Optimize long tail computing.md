# Optimize long tail computing {#concept_bqm_4vd_5db .concept}

Long tail is a common and difficult problem in distributed computing. The workload of each node is different when data is distributed unevenly. The whole task does not end until the work on the slowest node is complete.

To address this problem, you can distribute a piece of task to multiple workers for execution, rather than designate a worker to run the heaviest task. This article describes how to address typical cases of long tail.

## Join long tail {#section_zxf_5vd_5db .section}

**Cause**:

When the Key of a Join statement has a large amount of data, a long tail occurs.

**Solution**:

You can solve the problem in four steps:

-   Verify that one or both of the tables are not small tables. If one table is large and the other small, you can use the mapjoin statement to cache the small table. For the syntax and relevant description, see [Introduction to the SELECT Syntax](../../../../../reseller.en-US/User Guide/SQL/Select Operation/Introduction to the SELECT Syntax.md#). If the job is a MapReduce job, you can use the resource table function to cache the small table.

-   If both tables are relatively large, reduce duplicated data as much as possible.

-   If the problem persists, consider service optimization to avoid the calculation of Cartesian product on the two keys with a large data volume.

-   Small table leftjoin large table, odps direct leftjoin is slow. At this point, you can first small the table and the large table mapjoin, so that you can get the intersection between the small table and the big table, and this intermediate table must not be greater than the large table \(as long as there is not a large key tilt not very large \). The small table then performs a leftjoin with this intermediate table, and the effect is equal to the larger table of the small leftjoin.


## Group By long tail {#section_byf_5vd_5db .section}

**Cause**

When a Key of the Group By statement has a large amount of computations, a long tail occurs.

**Solution**:

You can solve the problem in either of the following ways:

-   Rewrite the SQL statement and add random numbers to split the long key. As shown in the following:

    ```
    SELECT Key,COUNT(*) AS Cnt FROM TableName GROUP BY Key;
    ```

    Provided that the Combiner is skipped, data is shuffled from the M node to the R node. Then, the R node performs the Count operation. The execution plan is M \> R. If you want to redistribute work to the key with the long tail, modify the statement as follows:

    ```
    -- Assume that the key with the long tail is KEY001.
    SELECT a.Key
      , SUM(a.Cnt) AS Cnt
    FROM (
      SELECT Key
        , COUNT(*) AS Cnt
    FROM TableName
    GROUP BY Key, 
      CASE 
        WHEN Key = 'KEY001' THEN Hash(Random()) % 50
        ELSE 0
       END
    ) a
    GROUP BY a.Key;
    ```

    The execution plan is changed to M \> R \> R. The execution lengthens, but the consumed time may reduce because the long-tail key is processed in two steps.

    **Note:** 

    If you use the preceding method to add an R execution step for solving a long tail problem that is not serious, the consumed time may increase.

-   Set system parameters as follows:

    ```
    set odps.sql.groupby.skewindata=true.
    ```

    Parameter setting is a universal optimization method, but the results are not satisfying because this method does not consider specific services. You can rewrite the SQL statement in a more efficient manner based on the actual data.


## Distinct long tail {#section_myf_5vd_5db .section}

When a long tail occurs in a Distinct statement, the Key splitting method does not apply. You can consider other methods.

**Solution**:

```
--Original SQL statement, with the null UID skipped
SELECT COUNT(uid) AS Pv
    , COUNT(DISTINCT uid) AS Uv
FROM UserLog;
```

Rewrite the statement as follows:

```
SELECT SUM(PV) AS Pv
    , COUNT(*) AS UV
FROM (
    SELECT COUNT(*) AS Pv
      , uid
    FROM UserLog
    GROUP BY uid
) a;
```

The Distinct statement is rewritten to a Count statement to relieve the computing pressure on a single Reducer. This solution also enables Group By statement optimization and Combiner execution, greatly improving the performance.

## Long tail of a dynamic partition {#section_tyf_5vd_5db .section}

**Cause**:

-   To sort the data of small files, the dynamic partition function starts a Reduce task in the final stage of execution. A long tail occurs when the data written by the dynamic partition function is skewed.

-   Misuse of the dynamic partition function often results in long tails.


**Solution**:

If the target partition for data write is determined, you can specify this partition during the Insert operation, instead of using the dynamic partition function.

## Remove long tails using the Combiner {#section_vyf_5vd_5db .section}

For MapReduce jobs, it is a common practice to use the Combiner to remove long tails. This practice has been mentioned in the WordCount example. The Combiner can reduce the volume of data shuffle from the Mapper to the Reducer, thus sufficiently reduce the overhead of network transfer. The optimization is automatically done for MaxCompute SQL statements.

**Note:** 

The Combiner only supports optimization for Map. You must make sure that the results of Combiner execution are the same. For example, in WordCount, the results are the same for two \(KEY,1\)s and one \(KEY,2\). For example, in average value calculation, \(KEY,1\) and \(KEY,2\) cannot be directly merged into \(KEY,1.5\) in the Combiner.

## Remove long tails using system optimization {#section_wyf_5vd_5db .section}

In addition to the Local Combiner, you can use the optimization method provided by MaxCompute to remove long tails. For example, the following content \(+N backups\) is logged during the task running process:

```
M1_Stg1_job0:0/521/521[100%] M2_Stg1_job0:0/1/1[100%] J9_1_2_Stg5_job0:0/523/523[100%] J3_1_2_Stg1_job0:0/523/523[100%] R6_3_9_Stg2_job0:1/1046/1047[100%] 
M1_Stg1_job0:0/521/521[100%] M2_Stg1_job0:0/1/1[100%] J9_1_2_Stg5_job0:0/523/523[100%] J3_1_2_Stg1_job0:0/523/523[100%] R6_3_9_Stg2_job0:1/1046/1047[100%] 
M1_Stg1_job0:0/521/521[100%] M2_Stg1_job0:0/1/1[100%] J9_1_2_Stg5_job0:0/523/523[100%] J3_1_2_Stg1_job0:0/523/523[100%] R6_3_9_Stg2_job0:1/1046/1047(+1 backups)[100%] 
M1_Stg1_job0:0/521/521[100%] M2_Stg1_job0:0/1/1[100%] J9_1_2_Stg5_job0:0/523/523[100%] J3_1_2_Stg1_job0:0/523/523[100%] R6_3_9_Stg2_job0:1/1046/1047(+1 backups)[100%]
```

There total number of Reducers is 1,047, of which 1,046 are complete. The last one is incomplete. After the system identifies this condition, it starts a new Reducer to run the same data. Then, the system takes the data of the Reducer which completes first and combines the data to the result set.

## Remove long tails using service optimization {#section_yyf_5vd_5db .section}

The preceding optimization methods cannot solve all problems. You can analyze your services to find a better solution,the example is as follows:

-   A large amount of noisy data may exist in reality. For example, you want to check the behavior data in the access record of each user by visitor ID. You must remove crawler data first, even though crawlers become increasingly difficult to identify; otherwise, the crawler data may easily cause a long tail during the calculation process. A similar case is when you associate data by a specific xxid, you must check whether the field for association is null.

-   Special services exist.  For example, ISV operation records differ greatly from common user records in terms of data volume and behavior. You can use a special method to analyze and handle the ISV operation records for major accounts independently.

-   When data distribution is uneven, do not use a constant field as the Distribute by field for full sorting.


