# JSON字符串增加键值示例 {#concept_ql4_rlp_mgb .concept}

在您的数据使用过程中，常常需要向字符串中增加键值或者嵌套JSON格式数据。本文为您介绍如何实现向基础JSON字符串中增加（覆盖）一个或多个键值对（支持嵌套JSON），且接受奇数个参数的代码示例。

## JSON字符串增加键值UDF说明 {#section_s4l_hpp_mgb .section}

-   **函数名**：PutJsonValues
-   **功能**

    向基础JSON串中增加（覆盖）一个或多个键值对支持嵌套JSON，接受奇数个参数。可以和[不同类型数据转换JSONString示例](cn.zh-CN/用户指南/SQL/UDF示例程序/不同类型数据转换JSONString示例.md#)中的ToJsonObeject配合使用（请先完成ToJsonObeject函数的注册）。

-   **参数**：
    -   参数1：基础JSON串
    -   参数2：键1
    -   参数3：值1
    -   \[参数4：键2\]
    -   \[参数5：值2\]
-   **使用示例**

    ```language-sql
    SELECT tojson(NULL)
    ,  tojson(concat('123', chr(3)))
    ,  tojson(123)
    ,  tojson(12.3)
    ,  tojson(true)
    ,  tojson(CAST(123  AS  DECIMAL))
    ,  putJsonValuesTest('{}', 'a', '123', 'b', tojson('abc'), 'c', '{"c1":567}')
    FROM dual;
    ```

    结果：

    ```
    null
    "123\u0003"
    123
    12.3
    true
    123
    {"a":123,"b":"abc","c":{"c1":567}}
    ```

    **说明：** null代表"null"，其他类推最后一个例子里，a的值是数值没有引号，b的值是字符串，需要使用tojson转义为json元素（json字符串）， '\{"c1":567\}'本身已经是一个合法的json元素，因此可以直接传入。

    此处putJsonValuesTest、[tojson](cn.zh-CN/用户指南/SQL/UDF示例程序/JSON字符串获取示例.md#)是您使用[DataWorks](../../../../../cn.zh-CN/使用指南/数据开发/业务流程/注册函数.md#)或[Intellij IDEA](../../../../../cn.zh-CN/工具及下载/MaxCompute Studio/开发 Java 程序/打包、上传和注册.md#)注册的函数名称。

-   **注意事项**

    fastjson.JSON包可以手动在[https://mvnrepository.com/artifact/com.alibaba/fastjson](https://mvnrepository.com/artifact/com.alibaba/fastjson)下载

    实际在MaxCompute公共云运行时，由于沙箱等限制，需要把fastjson.JSON包和UDF jar包分别作为资源上传MaxCompute，并且在DataWorks创建函数时**同时引用两个资源**。如下图所示，本例中PutJsonValues.jar是UDF打包后生成的jar，ODPSUDF-1.0-SNAPSHOT2.jar是上例ToJsonString打包后生成的jar，而fastjson-1.2.28.odps.jar是fastjson.JSON包。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/112666/154795469437650_zh-CN.png)


## UDF代码示例 {#section_txm_csp_mgb .section}

```language-java
import com.alibaba.fastjson.JSON; //阿里云UDF
import com.alibaba.fastjson.JSONObject; //引入fastjson.JSON类
import com.aliyun.odps.udf.UDF;

public class PutJsonValues extends UDF {
//MaxCompute中String类型数据与JAVA UDF中类型一致，String... keyValues为可变形参，表示参数个数不定
    public  PutJsonValues(){
    }

    public  String evaluate(String jsonBase, String... keyValues) {
        if (jsonBase == null) {
            return null;
        }
        JSONObject outputObj = JSON.parseObject(jsonBase);
        for (int i = 1; i < keyValues.length; i = i + 2) {
            String key = keyValues[i - 1];
            String value = keyValues[i];
            if (key == null || value == null) {
                continue;
            }
            Object valueObj = JSON.parse(value);
            outputObj.put(key, valueObj);
        }
        return JSON.toJSONString(outputObj);
    }
}
```

## 单元测试 {#section_cwd_txp_mgb .section}

```
//下列代码为测试代码，用于测试数据转换是否生效，您在实际使用过程中可注释掉。
    public static void main(String[] args) {
        PutJsonValues putJsonValues=new PutJsonValues();
        System.out.println(putJsonValues.hashCode();

        String js=putJsonValues.evaluate("{}", "k", "null");
        System.out.println(js);
        System.out.println(new PutJsonValues().evaluate("{}", "a", "123", "b", "234", "c", "345"));
        System.out.println(new PutJsonValues().evaluate("{}", "k", "\"abc\""));
        System.out.println(new PutJsonValues().evaluate("{}", "k", "{\"kk\":123}"));
    }
}
```

