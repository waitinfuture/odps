# Set operations {#concept_yys_xby_rfb .concept}

This topic describes how to use the set command to set MaxCompute or user-defined system variables and how to clear the settings.

## set command {#section_pqs_bff_vdb .section}

**Format**

```
set <KEY>=<VALUE>
```

**Action**

You can use the set command to configure operations in MaxCompute.

MaxCompute supports the following system variables:

```
--The set commands supported by MaxCompute SQL and the latest MapReduce.
set odps.sql.allow.fullscan=  --Set whether to allow a partitioned table is fully scanned. The value 'true' means yes, and the value 'false' means not.
set odps.stage.mapper.mem=    --Set the memory size of each map worker. The default value is 1,024 MB.
set odps.stage.reducer.mem=   --Set the memory size of each reduce worker. The default value is 1,024 MB.
set odps.stage.joiner.mem=    --Set the memory size of each join worker. The default value is 1,024 MB.
set odps.stage.mem =    --Set the memory size of all workers in specified MaxCompute jobs. This set key has lower priority than that of the preceding three ones and does not have a default value.
set odps.stage.mapper.split.size=    -- Modify the input data quantity of each map worker (the input file shard size) to manage the number of workers at each map stage. The default value is 256 MB.
set odps.stage.reducer.num=       --Modify the number of workers at each reduce stage. This set key does not have a default value.
set odps.stage.joiner.num=        --Modify the number of workers at each join stage. This set key does not have a default value.
set odps.stage.num=               --Modify the worker concurrency at all stages in specified MaxCompute jobs. This set key has lower priority than that of the preceding three ones and does not have a default value.
set odps.sql.type.system.odps2=   -- The default value is false. The value must be set to 'true' if the SQL statement contains new data types such as TINYINT, SMALLINT, INT, FLOAT, VARCHAR, and TIMESTAMP BINARY.
set odps.sql.executionengine.coldata.deep.buffer.size.max=   --Adjust the buffer size of a complex type column in a table that is written in MaxCompute.
set odps.sql.hive.compatible=false|true  -- Set the session HIVE compatible or not.
```

**Note:** Only lowercase letters can be used in the set command.

**Example**

`set odps.sql.executionengine.coldata.deep.buffer.size.max`

-   The preceding SQL statement is used to adjust the buffer size of a complex type column in a table that is written in MaxCompute.
-   **Applicable scenarios**
    1.  An output table contains many complex data types.
    2.  The size of a single complex variable contained in the output table exceeds the specified size.
-   **Description**
    -   The default buffer size is 67,108,864 byte \(64 MB\).
    -   If an output table has 3 columns whose schema is of a complex type, such as string, map, struct, array, and binary, MaxCompute reserves 64 MB for each column by default. The buffer that is applied for each column is used to store data for the corresponding batch row count row.
    -   We recommend that you set a proper value according to the estimated size of the complex variables in a table. For example, if the size of each complex variable does not exceed 1,024 byte and the batch row count value is 1024 \(the default value\), you can set the flag to 1024 × 1024. The following is an example:

        ```language-sql
        set odps.sql.executionengine.coldata.deep.buffer.size.max=1048576;
        ```

    -   If the value of each complex variable is between 7 MB and 8 MB, and the value of batch row count is 32, you can set this value to 8 MB × 32.
    -   If the output of a task has a complex type, or if the mapjoin table of a task has a complex type, adjusting the value affects the memory during task running. An excessive large value might cause out of memory \(OOM\).

## show flags {#section_ft4_jff_vdb .section}

**Format**

```
show flags; --Display parameters set by using the set command.
```

**Action**

You can run the Use Project command to clear the settings.

