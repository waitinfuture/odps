# 不同类型数据转换JSONString示例 {#concept_bv1_dth_mgb .concept}

在您的数据使用过程中，常常需要将不同类型的数据转换为JSON String。例如，假设您的数据是Array类型，由于Tunnel上传下载目前不支持复杂数据类型，可能需要先将Array类型数据转换为JSON String，再进行下载。

## 转换JSONStringUDF说明 {#section_pdj_n2q_3gb .section}

-   **函数名**：ToJsonString
-   **功能**

    将MaxCompute对象（字段）转换为JSON字符串，包括必要的字符串转义。您可以与PutJsonValues配合使用。

-   **参数**：任意一个参数，可以是除了datetime类型以外的任意类型。
-   **使用示例**

    ```
    SELECT tojson(array(2.0,3.0,4.0)) FROM dual;
    ```

    结果：

    ```
    [2.0,3.0,4.0]
    ```

    **说明：** 此处tojson是您使用[DataWorks](../../../../../cn.zh-CN/使用指南/数据开发/业务流程/注册函数.md#)或[Intellij IDEA](../../../../../cn.zh-CN/工具及下载/MaxCompute Studio/开发 Java 程序/打包、上传和注册.md#)注册的函数名称。

-   **注意事项**

    fastjson.JSON包可以手动在[https://mvnrepository.com/artifact/com.alibaba/fastjson](https://mvnrepository.com/artifact/com.alibaba/fastjson)下载

    实际在MaxCompute公共云运行时，由于沙箱等限制，需要把fastjson.JSON包和UDF jar包分别作为资源上传MaxCompute，并且在DataWorks创建函数时**同时引用两个资源**。如下图所示，本例中ODPSUDF-1.0-SNAPSHOT2.jar是UDF打包后生成的jar，而fastjson-1.2.28.odps.jar是fastjson.JSON包。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/106671/154795465837575_zh-CN.png)


## UDF代码示例 {#section_hl5_xfq_3gb .section}

```
import com.aliyun.odps.udf.UDF; //阿里云UDF
import java.math.BigDecimal;
import com.alibaba.fastjson.JSON;  //引入fastjson.JSON类
import java.util.Arrays;
import java.util.List;

public class ToJsonString extends UDF {
//下列代码调用fastjson.JSON的toJSONString方法，将各种类型的数据转换为JSON string,您可以根据您的需要增加更多的数据类型。
    public String evaluate(String obj) {
        return JSON.toJSONString(obj);
    }
    public String evaluate(Long obj) {
        return JSON.toJSONString(obj);
    }
    public String evaluate(Double obj) {
        return JSON.toJSONString(obj);
    }
    public String evaluate(Boolean obj) {
        return JSON.toJSONString(obj);
    }
    public String evaluate(BigDecimal obj) {
        return JSON.toJSONString(obj);
    }
//MaxCompute中Array类型数据在JAVA UDF中类型为List，例如Array<double>类型可以在UDF中通过List<Double>进行匹配
    public String evaluate(List<Double> obj) {
        return JSON.toJSONString(obj);
    }
    public String evaluate(double[] obj) {
        return JSON.toJSONString(obj);
    }
    public String evaluate(int[] obj) {
        return JSON.toJSONString(obj);
    }
}
```

## 单元测试 {#section_cwd_txp_mgb .section}

```
//下列代码为测试代码，用于测试数据转换是否生效，您在实际使用过程中可注释掉。
    public static void main(String[] args) {
        //System.out.println(new ToJsonString().evaluate(null));//java不能调用，但odps可以
        System.out.println(new ToJsonString().evaluate("中文\t"));
        System.out.println(new ToJsonString().evaluate(123L));
        System.out.println(new ToJsonString().evaluate(1.123));
        System.out.println(new ToJsonString().evaluate(true));
        //System.out.println(new ToJsonString().evaluate(new Date()));//udf/udaf不支持datetime类型
        System.out.println(new ToJsonString().evaluate(BigDecimal.TEN));
        System.out.println(new ToJsonString().evaluate(Arrays.asList(1.0,2.0,3.0,4.0)));
        double[] args1 = {1,2,3,4};
        System.out.println(new ToJsonString().evaluate(args1));
       }
    }
```

