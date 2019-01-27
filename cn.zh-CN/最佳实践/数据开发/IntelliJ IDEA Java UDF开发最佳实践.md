# IntelliJ IDEA Java UDF开发最佳实践 {#task_lph_3mw_dgb .task}

IntelliJ IDEA是Java语言的集成开发环境，可以帮助我们快速的开发Java程序。本文为您详细介绍如何使用IntelliJ IDEA进行Java UDF开发。

在开始UDF开发实践之前，您需要做如下准备工作：

1.  准备IntelliJ IDEA开发工具，请参见[安装Studio](../../../../../cn.zh-CN/工具及下载/MaxCompute Studio/工具安装与版本信息/安装IntelliJ IDEA.md)。
2.  通过IntelliJ IDEA MaxCompute Studio[创建MaxCompute项目连接](../../../../../cn.zh-CN/工具及下载/MaxCompute Studio/项目空间连接管理.md)。
3.  连接MaxCompute项目成功后，您需要[创建MaxCompute Java Module](../../../../../cn.zh-CN/工具及下载/MaxCompute Studio/开发 Java 程序/创建MaxCompute Java Module.md)。

开发环境准备完成后即可开发UDF，下面将为您介绍一个字符小写转换功能的UDF实现示例。

**说明：** 更多UDF开发的相关资料请参见[Java UDF](../../../../../cn.zh-CN/用户指南/SQL/UDF/Java UDF.md#)。

1.  **创建Java UDF Project** 

    首先在您的IntelliJ IDEA中展开已创建的MaxCompute Java Module目录，导航至**src** \> **main** \> **java** \> **New**，单击**MaxCompute Java** ，如下图所示。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/154856829434451_zh-CN.png)

    填写**Name**，输入`package名称.文件名`，**Kind**选择UDF，单击**OK** ，如下图所示。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/154856829434456_zh-CN.png)

    **说明：** 

    -   Name：填写创建的MaxCompute Java Class名称，如果还没创建package，可以在此处填写packagename.classname，会自动生成package。
    -   Kind：选择类型。目前支持的类型有：自定义函数（UDF/UDAF/UDTF）、MapReduce（Driver/Mapper/Reducer）、非结构化开发（StorageHandler/Extractor）等。
2.  **编辑Java UDF代码** 在您新建的Java UDF项目（本例中的porjectLower）中编辑代码，如下图。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/154856829434458_zh-CN.png)

    示例代码如下。

    ```
    package <package名称>;
    import com.aliyun.odps.udf.UDF;
    public class Lower extends UDF {
       public String evaluate(String s) {
          if (s == null) { 
              return null; 
          }
          return s.toLowerCase();
       }
    }
    ```

    **说明：** 这里的代码模板可在您的Intellij IDEA中自定义，具体操作路径：**Settings** \> **Editor** \> **File Code Templates**，然后在Code标签页中寻找MaxCompute对应的模板修改。

3.  **测试UDF** 开发UDF完成后，可通过单元测试和本地运行两种方式进行测试，看是否符合预期结果，操作如下。
    1.  **单元测试** 在您的Modul项目中examples目录下有各种类型的单元测试示例，您可参考示例编写自己的Unit Test。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/154856829534470_zh-CN.png)

        测试结果如下。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/154856829534473_zh-CN.png)

        我们可以看到大写字母"ALIYUN"已经成功转换成小写字母"aliyun"输出。

    2.  **本地运行** 在您的IntelliJIDEA中本地运行UDF时，需要指定运行数据源，有以下两种方式设定测试数据源：

        -   MaxCompute Studio通过Tunnel服务自动下载指定项目下的表数据到warehouse目录下。
        -   提供Mock项目及表数据，即您可参考warehouse下的example\_project自行设置。
        操作步骤

        1.  为了测试Java UDF代码，我们可以首先在MaxCompute上存放一些大写字母作为输入数据。您可以利用script脚本文件或者odpscmd客户端使用SQL语句`create table upperABC(upper string);`新建一个名为upperABC的测试表格，如图。

            ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/154856829534592_zh-CN.png)

        2.  右击UDF类，单击**Run '类名.main\(\)'**，弹出run configurations对话框，如下图。

            ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/154856829534483_zh-CN.png)

            **说明：** 

            -   UDF/UDAF/UDTF一般作用于select子句中表的某些列，需要配置MaxCompute project，table和column（元数据来源于project explorer和warehouse下的Mock项目）。复杂类型的调试也是支持的。
            -   如果指定项目下的表数据未被下载到warehouse中，需要先下载数据，默认下载100条。默认下载100条，如需更多数据，可配置Download record limit项。
            -   UDF的local run框架会将warehouse中指定列的数据作为UDF的输入，开始本地运行UDF，您可以在控制台看到日志输出和结果打印。
            -   如果采用Mock项目或已下载数据，则直接运行。
        单击**OK**，运行结果如下图。

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/154856829534510_zh-CN.png)

4.  **发布UDF** 此时我们的Lower.java测试通过，接下来我们要将其打包成jar资源上传到MaxCompute服务端上。一个UDF要想发布到服务端供生产使用，要经过**打包** \> **上传** \> **注册**三个步骤。针对此，IntelliJ IDEA MaxCompute Studio提供了一键发布功能（Studio会依次执行maven clean package，上传jar和注册UDF三个步骤，一次完成）。具体的操作如下。右键单击UDF的Java文件，选择**Deploy to server**，弹框里选择注册到哪一个MaxCompute project，依次输入Function name和Resource name，Resource name可以修改，如下图。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/154856829534564_zh-CN.png)

    **说明：** 

    如果您想了解打包、上传和注册的详细操作步骤，请参见[打包、上传和注册](../../../../../cn.zh-CN/工具及下载/MaxCompute Studio/开发 Java 程序/打包、上传和注册.md#)。

    填写完成后，单击**OK**即可完成注册，成功后会有提示。您可在连接的MaxCompute项目下找到已经注册好的Function函数，如图所示。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/154856829534572_zh-CN.png)

5.  **试用UDF** 成功注册UDF后，即可试用UDF。在您的Module项目中打开SQL脚本，执行命令`select Lower_test('ALIYUN');`，显示结果如下图所示。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/154856829534603_zh-CN.png)

    您也可以在odpscmd客户端使用select Lower\_test\('ALIYUN'\) from uppperABC;命令测试您的Java UDF函数。到此，您使用IntelliJIDEA上开发的Java UDF函数Lower\_test已经可用了。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/80567/154856829534582_zh-CN.png)


如果您要使用Eclipse开发工具完成完整的Java UDF开发流程，请参见[Eclipse Java UDF开发最佳实践](cn.zh-CN/最佳实践/数据开发/Eclipse Java UDF开发最佳实践.md#)。

