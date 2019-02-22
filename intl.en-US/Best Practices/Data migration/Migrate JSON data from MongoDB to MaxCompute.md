# Migrate JSON data from MongoDB to MaxCompute {#concept_z2b_fl2_xfb .concept}

This topic describes how to use the data integration feature of DataWorks to extract JSON fields from MongoDB to MaxCompute.

## Preparations {#section_qv1_nm2_xfb .section}

1.  Prepare an account.

    Create a user in the database in advance to add data sources in DataWorks. In this example, you can run the `db.createUser({user:"bookuser",pwd:"123456",roles:["root"]})` command to create a user named bookuser. The password of the user is 123456, and the permission is root.

2.  Prepare data.

    Upload data to your MongoDB. In this example, Alibaba Cloud ApsaraDB for MongoDB is used. The network type is VPC. \(An Internet IP address is required for MongoDB to communicate with the default resource group of DataWorks.\) The test data is as follows:

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

    Log on to the DMS console of MongoDB. In this example, the database name is admin and the collection is userlog. You can run the db.userlog.find\(\).limit\(10\) command in the query window to view the uploaded data.


## Use DataWorks to extract data to MaxCompute {#section_sk4_zq2_xfb .section}

-   **1. Add a MongoDB data source.**

    In the DataWorks console, go to the [Data Integration](../../../../../reseller.en-US/User Guide/Data integration/Data integration introduction/Data integration overview.md#) page and add a [MongoDB](../../../../../reseller.en-US/User Guide/Data integration/Data source configuration/Configure MongoDB data source.md#) data source.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/64919/155080408532876_en-US.png)

    The parameters are shown in the following figure. Click **Complete** after the connectivity test is successful. In this example, the network type of MongoDB is VPC. Therefore, the **Data Source Type** must be set to **Public IP Address Available**.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/64919/155080408532877_en-US.png)

    To obtain the IP address and the port number, log on to the and click the target instance. Example parameters are shown in the following figure.

-   **2. Create a data synchronization task.**

    In the DataWorks console, create a data synchronization node. For more information, see [Configure OSS Reader](../../../../../reseller.en-US/User Guide/Data integration/Task configuration/Configure reader plug-in/Configure OSS Reader.md#).

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/64919/155080408532879_en-US.png)

    At the same time, create a table named mqdata in DataWorks to store JSON data. For more information, see [Create a table](../../../../../reseller.en-US/User Guide/Data development/Table Management.md#).

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/64919/155080408538569_en-US.png)

    You can set the table parameters on the graphical interface. The mqdata table has only one column, which is named MQ data. The data type is string.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62284/155080408531545_en-US.png)

-   **3. Set the parameters.**

    After creating a table, you can set the data synchronization task parameters on the graphical interface. First, set the destination data source to odps\_first and the destination table to mqdata. Then, set the original data source to MongoDB and select mongodb\_userlog. After completing the preceding settings, click **Switch to script mode**. The following is an example of the code in script mode:

    ```
    
    {
        "type": "job",
        "steps": [
            {
                "stepType": "mongodb",
                "parameter": {
                    "datasource": "mongodb_userlog",
     //Data source name
                    "column": [
                        {
                            "name": "store.bicycle.color", //JSON field path. In this example, the value of color is extracted.
                            "type": "document.document.string" //The number of fields in this line must be the same as that in the preceding line (the name line). If the JSON field is a level-1 field, for example, the expensive field in this topic, enter the string.
                        }
                    ],
                    "collectionName //Collection name": "userlog"
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
                              "mqdata" //Table column name in MaxCompute
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

    After completing the preceding settings, click **Run**. If the following information is displayed, the code has run successfully.


## Verify the result {#section_cwq_bcf_xfb .section}

Create an ODPS SQL node in your [Business Flow](../../../../../reseller.en-US/User Guide/Data development/Business flow/Business flow.md#).

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/62284/155080408631551_en-US.png)

Enter the `SELECT * from mqdata;` statement to view the data in the mqdata table. You can also run the SELECT \* from mqdata; command on the [MaxCompute client](../../../../../reseller.en-US/Tools and Downloads/Client.md#) to view the data.

