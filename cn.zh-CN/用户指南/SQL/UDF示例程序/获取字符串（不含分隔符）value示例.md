# 获取字符串（不含分隔符）value示例 {#concept_qby_2zw_tgb .concept}

本文向您介绍如何获取不包含分隔符的字符串中指定key对应的value值的Java UDF示例。

## 获取字符串\(不含分隔符\)value的UDF说明 {#section_eqm_rkc_5gb .section}

-   函数名：UDFKeyValue
-   功能

    从字符串中获得指定key的value值

    **说明：** 该UDF不适用于字符串本身包含分隔符的情况，若需要处理该情况请使用[UDFKeyValueEx](cn.zh-CN/用户指南/SQL/UDF示例程序/获取字符串（含有分隔符）value示例.md#)。

-   参数说明

    `UDFKeyValue(String str, String splitor1, String splitor2, String key)`

    -   @str：字符串
    -   @splitor1：通过splitor1分割出key:value对
    -   @splitor2：对分割出来的key:value对，使用splitor2进行分割
    -   @key：要获取的参数值
    `UDFKeyValue(String keyValues, String split)`

    -   @str：字符串
    -   @key：要获取的参数值，splitor1为`;`splitor2为`:`

## UDF代码示例 {#section_zh3_mqc_5gb .section}

```language-java
//package名称，可以根据您的情况定义
package com.aliyun.odps.examples.udf;
import com.aliyun.odps.io.Text;
import com.aliyun.odps.udf.UDF;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public class UDFKeyValue extends UDF{
        // 存储所有键：值对
	private Map<String, String> mKeyValueache = new ConcurrentHashMap<String, String>();
	private Text result = new Text();

	public UDFKeyValue() {
	}

	public String evaluate(String str, String keyname){
		// 默认split1=; split2=:
		return evaluate(str, ";", ":", keyname);
	}

	public String evaluate(String str, String split1, String split2, String keyname) {
		try {   
                        // 通过splitor1分割出key:value对
			if (str == null || "".equals(str))
				return null;
			String[] values1 = str.split(split1);
			mKeyValueache.clear();
			int i = 0;
                        // 对分割出来的key:value对，使用splitor2进行分割
			while (i < values1.length) {
				storeKeyValue(values1[i], split2);
				i++;
			}
			String resultValue = getKeyValue(keyname);
			if (resultValue == null)
				return null;
			result.set(new Text(resultValue));
                        // 获取结果值
			return result.toString();
		} catch (Exception e) {
			return null;
		}
	}

	private boolean storeKeyValue(String keyValues, String split) {
		if (keyValues == null || "".equals(keyValues))
			return false;
		if (mKeyValueache == null)
			mKeyValueache = new ConcurrentHashMap<String, String>();
		String[] keyValueArr = keyValues.split(split);
		if (keyValueArr.length == 2) {
			mKeyValueache.put(keyValueArr[0], keyValueArr[1]);
			return true;
		}
		return false;
	}

	private String getKeyValue(String keyName) {
		if (keyName == null || 
		    "".equals(keyName) || 
		    mKeyValueache == null || 
		    mKeyValueache.size() == 0)
			return null;
		return mKeyValueache.get(keyName);
	}
}
```

## UDF使用示例 {#section_pq4_rzc_5gb .section}

1.  注册函数

    UDFKeyValue.java测试通过后，将其注册函数，如下图。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/124620/155014078038827_zh-CN.png)

    **说明：** 

    -   studio提供了一键发布功能，即依次执行mvn clean package，上传jar和注册UDF三个步骤。您也可以在DataWorks选择**资源管理**，点击**资源上传**，选择Jar包完成上传，详细内容请参见[资源](../../../../../cn.zh-CN/使用指南/数据开发/手动业务流程/资源.md#)。
    -   通过客户端使用常用命令进行资源上传操作，请参见[资源操作](cn.zh-CN/用户指南/常用命令/资源操作.md#)。
2.  使用示例

    成功注册UDF后，执行命令：

    -   示例一

        ```language-sql
        select UDFKeyValue('a:1;b:2;','a') from dual;
        ```

        -   结果：

            ```language-sql
            +-----+
            | _c0 |
            +-----+
            | 1   |
            +-----+
            ```

    -   示例二

        ```language-sql
        select UDFKeyValue('a:1;b:2;','\;',':','a') from dual;
        ```

        -   结果：

            ```language-sql
            +-----+
            | _c0 |
            +-----+
            | 1   |
            +-----+
            ```


