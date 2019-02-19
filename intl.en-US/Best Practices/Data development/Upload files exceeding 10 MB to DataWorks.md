# Upload files exceeding 10 MB to DataWorks {#concept_jtw_kfy_bfb .concept}

This topic describes how to upload a JAR package or resource file exceeding 10 MB to DataWorks when running a MapReduce job.

Methods:

1.  Upload resources exceeding 10 MB through the [MaxCompute CLI client](../../../../../reseller.en-US/Tools and Downloads/Client.md#). Before you can upload the resource, first complete the following procedure.
    -   Download the MaxCompute CLI client. For more information, see [Client](../../../../../reseller.en-US/Tools and Downloads/Client.md#).
    -   Set AccessKeys and endpoints for the MaxCompute CLI client. For more information, see [Install and configure a client](../../../../../reseller.en-US/Prepare/Install and configure a client.md#).

        Run the following command on the MaxCompute CLI client:

        ```
        //Add resources.
        add jar C:\test_mr\test_mr.jar -f;
        ```

2.  Currently, resources uploaded through the MaxCompute CLI client cannot be viewed in the resource list in the DataWorks console. You can view and confirm the resources by running the list resources command.

    ```
    //View resources.
    list resources;
    ```

3.  Reduce the JAR file size and run the MapReduce jobs on DataWorks on your local server. You need to keep only one main function.

    ```
    jar 
    -resources test_mr.jar,test_ab.jar --The list resources command directly runs after the function is registered on the MaxCompute CLI client.
    -classpath test_mr.jar --Size reduction policy: A mapper and a reducer that are related to the main function are required to submit the size reduction policy to a gateway. The third-party dependency is not required. Other resources can be stored on the resources directory.
    com.aliyun.odps.examples.mr.test_mr wc_in wc_out;
    ```


With the preceding methods, you can use the scheduling feature to run MapReduce jobs exceeding 10 MB on DataWorks regularly.

