# Migrate JSON data from OSS to MaxCompute {#concept_gyw_dhd_5fb .concept}

This topic describes how to use the data integration feature of DataWorks to migrate JSON data from OSS to MaxCompute and use the built-in string function GET\_JSON\_OBJECT of MaxCompute to extract JSON information.

## Preparations {#section_fmj_3hd_5fb .section}

-   Upload data to OSS.

    Convert your JSON file to a TXT file and upload it to OSS. The following is a JSON file example:

    ```
    
    {
        "store": {
            "book": [
                 {
                    "category": "reference",
                    "author": "Nigel Rees",
                    "title": "Sayings of the Century",
                    "price": 8.95
                 },
                 {
                    "category": "fiction",
                    "author": "Evelyn Waugh",
                    "title": "Sword of Honour",
                    "price": 12.99
                 },
                 {
                     "category": "fiction",
                     "author": "J. R. R. Tolkien",
                     "title": "The Lord of the Rings",
                     "isbn": "0-395-19395-8",
                     "price": 22.99
                 }
              ],
              "bicycle": {
                  "color": "red",
                  "price": 19.95
              }
        },
        "expensive": 10
    }
    ```

    Upload the applog.txt file to OSS. In this example, the OSS bucket is located in China \(Shanghai\).


## Use DataWorks to migrate JSON data from OSS to MaxCompute {#section_zcj_s3d_5fb .section}

-   **1. Add an OSS data source.**

    In the DataWorks console, go to the [Data Integration](../../../../../reseller.en-US/User Guide/Data integration/Data integration introduction/Data integration overview.md#) page and add an OSS data source. For more information, see [Configure OSS data source](../../../../../reseller.en-US/User Guide/Data integration/Data source configuration/Configure OSS data source.md#).

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62284/155117497731532_en-US.png)

    The parameters are shown in the following figure. Click **Complete** after the connectivity test is successful. The endpoints in this topic include http://oss-cn-shanghai.aliyuncs.com and http://oss-cn-shanghai-internal.aliyuncs.com.

    **Note:** Because the OSS and DataWorks projects are located in the same region, the intranet endpoint `http://oss-cn-shanghai-internal.aliyuncs.com` is used.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62284/155117497731536_en-US.png)

-   **2. Create a data synchronization task.**

    In the DataWorks console, create a data synchronization node. For more information, see [Configure OSS Reader](../../../../../reseller.en-US/User Guide/Data integration/Task configuration/Configure reader plug-in/Configure OSS Reader.md#). At the same time, create a table named mqdata in DataWorks to store the JSON data. For more information, see [Create a table](../../../../../reseller.en-US/User Guide/Data development/Table Management.md#).

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62284/155117497731544_en-US.png)

    You can set the table parameters on the graphical interface. The mqdata table has only one column, which is named MQ data. The data type is string.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62284/155117497731545_en-US.png)

-   **3. Set the parameters.**

    After creating a table, you can set the data synchronization task parameters on the graphical interface, as shown in the following figure. First, set the destination data source to odps\_first and the destination table to mqdata. Then, set the original data source to OSS and enter the file path and name as the object prefix.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62284/155117497731546_en-US.png)

    **Note:** You can set the column delimiter to caret \(^\) or any other character that is not contained in the TXT file. DataWorks supports multiple column delimiters for the TXT data sources in OSS. Therefore, you can use characters such as %&%\#^$$^% to separate the data into a column.

    Select **Enable Same Line Mapping**.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62284/155117497731548_en-US.png)

    Click the script switching button in the upper-left corner to switch to the script mode. Set fileFormat to `"fileFormat":"binary"`. The following is an example of the code in script mode:

    ```
    
    {
        "type": "job",
        "steps": [
            {
                "stepType": "oss",
                "parameter": {
                    "fieldDelimiterOrigin": "^",
                    "nullFormat": "",
                    "compress": "",
                    "datasource": "OSS_userlog",
                    "column": [
                        {
                            "name": 0,
                            "type": "string",
                            "index": 0
                        }
                    ],
                    "skipHeader": "false",
                    "encoding": "UTF-8",
                    "fieldDelimiter": "^",
                    "fileFormat": "binary",
                    "object": [
                        "applog.txt"
                    ]
                },
                "name": "Reader",
                "category": "reader"
            },
            {
                "stepType": "odps",
                "parameter": {
                    "partition": "",
                    "isCompress": false,
                    "truncate": true,
                    "datasource": "odps_first",
                    "column": [
                        "mqdata"
                    ],
                    "emptyAsNull": false,
                    "table": "mqdata"
                },
                "name": "Writer",
                "category": "writer"
            }
        ],
        "version": "2.0",
        "order": {
            "hops": [
                {
                    "from": "Reader",
                    "to": "Writer"
                }
            ]
        },
        "setting": {
            "errorLimit": {
                "record": ""
            },
            "speed": {
                "concurrent": 2,
                "throttle": false,
                "dmu": 1
            }
        }
    }
    ```

    **Note:** In this step, after the JSON file is synchronized from OSS to MaxCompute, data in the file is saved in the same row. That is, data in the JSON file shares the same field. You can use the default values for other parameters.

    After completing the preceding settings, click **run**.


## Verify the result {#section_opc_bp3_pgb .section}

1.  Create an ODPS SQL node in your [Business Flow](../../../../../reseller.en-US/User Guide/Data development/Business flow/Business flow.md#).

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62284/155117497731551_en-US.png)

2.  Enter the `SELECT * from mqdata;` statement to view the data in the mqdata table.

    **Note:** You can also run the `SELECT * from mqdata;` command on the [MaxCompute client](../../../../../reseller.en-US/Tools and Downloads/Client.md#) to view the data and perform subsequent steps.

3.  Verify that the data imported to the table is correct and use `SELECT GET_JSON_OBJECT(mqdata.MQdata,'$.expensive') FROM mqdata;` to obtain the value of expensive in the JSON file.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62284/155117497731553_en-US.png)


## Additional information {#section_y4g_zlb_pgb .section}

To verify the result, you can also use the built-in string function [GET\_JSON\_OBJECT](../../../../../reseller.en-US/User Guide/SQL/Builtin functions/String functions.md#section_cdt_gxz_vdb) in MaxCompute to obtain the JSON data as needed.

