# JSON字符串值判断示例 {#concept_ak4_vzb_mgb .concept}

本文向您介绍如何通过JAVA UDF实现根据path路径获取JSON字符串并对指定的值进行判断是否存在的代码示例。

## JSON字符串值判断UDF说明 {#section_w1p_g1c_mgb .section}

-   **函数名**：UDFGetValue
-   **功能**

    根据路径path得到json串中的值，与[JSON字符串获取示例](cn.zh-CN/用户指南/SQL/UDF示例程序/JSON字符串获取示例.md#)get\_json\_object类似。另外还能判断指定的path中，是否包含有value。$代表根节点，.取子节点，\[\]取数组下标。path只支持字母、数字、下划线。

-   **参数说明**

    `UDFGetJsonObject(String jsonString, String pathString)`

    -   @jsonString：json字符串
    -   @pathString：路径

## UDF使用示例 {#section_btp_3yj_5gb .section}

-   资源上传

    实际在MaxCompute公共云运行时，由于沙箱等限制，需要把org.json包和UDF jar包分别作为资源上传MaxCompute，并且在DataWorks创建函数时**同时引用两个资源**，org.json包可以手动到[MVN](https://mvnrepository.com/artifact/org.json/json)下载。

-   注册函数

    本例中TestJson.jar是UDF打包后生成的jar，而json-20180813.jar是org.json包，如下图。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/105851/155047936137569_zh-CN.png)

-   使用示例

    成功注册UDF后，执行命令：

    1.  ```language-sql
select TestJson('{"LogA\":[{"id":"11562,20508"}, {"id":"11563,20509"}]}', "LogA[0]") from dual;
```

        结果：`{"id":"11562,20508"}`

    2.  ```language-sql

select bi_udf:bi_get_value('{"LogA\":[{"id":"11562,20508"}, {"id":"11563,20509"}]}', "LogA[0].id"， "20508") from dual;

```

        结果：true

    3.  ```language-sql
select bi_udf:bi_get_value('{"LogA\":[{"id":"11562,20508"}, {"id":"11563,20509"}]}', "LogA[0].id"， "20508"， "&") from dual;
```

        结果：true


## UDF代码示例 {#section_ucz_g5g_mgb .section}

```language-java
package com.aliyun.odps.examples.udf; //package名称，可以根据您的情况定义
import com.aliyun.odps.io.Text;
import com.aliyun.odps.udf.UDF; //阿里云UDF
import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;

import java.util.ArrayList;
import java.util.Iterator;
import java.util.LinkedHashMap;
import java.util.Map;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class UDFGetValue extends UDF {
    //声明私有不可变Pattern正则表达式模式
    private  final  Pattern  patternKey  =  Pattern.compile("^([a-zA-Z0-9_\\-]+).*");  
    private  final  Pattern  patternIndex  =  Pattern.compile("\\[([0-9]+|\\*)\\]");
    //使用HashCache构造方法创建Map类型缓存区域
    private  static  Map<String,  Object>  extractObjectCache  =  new  HashCache<String,  Object>();
    private  static  Map<String,  String[]>  pathExprCache  =  new  HashCache<String,  String[]>();
    private  static  Map<String, ArrayList<String>>  indexListCache  =  new  HashCache<String,  ArrayList<String>>();
    private  static  Map<String,  String>  mKeyGroup1Cache  =  new  HashCache<String,  String>();
    private  static  Map<String,  Boolean>  mKeyMatchesCache  =  new  HashCache<String,  Boolean>();
    private  Text  result  =  new Text();
    private  ArrayList<Object>  jsonList  =  new  ArrayList<Object>();

    // An LRU cache using a linked hash map, LRU 表缓存
    //定义HashCache静态内部子类，继承LinkedHashMap
    private  static  class  HashCache<K,  V>  extends LinkedHashMap<K,  V> {
        private  static  final  int  CACHE_SIZE  =  16;
        private  static  final  int  INIT_SIZE  =  32;
        //声明私有常量HashMap的LOAD_FACTOR装载因子
        private  static  final  float  LOAD_FACTOR  =  0.6f;
        private  static  final  long  serialVersionUID  =  1;
        HashCache()  {
            super(INIT_SIZE,  LOAD_FACTOR);
        }
        //重载LinkedHashMap方法，用于判断是否超过Cache大小并清理缓存
        @Override
        protected  boolean  removeEldestEntry(Map.Entry<K,  V>  eldest)  {
            return  size()  >  CACHE_SIZE;
        }
    }

    public UDFGetValue()  {
    }

    /**
     *  @param  jsonStr  传入的Json字符串
     *  @param  path  Json路径
     *  @return  返回对应的值
     */
    public  String  evaluate(String  jsonStr,  String  path)  {
        //创建evaluate方法，与HIVE一样，MaxCompute的UDF通常使用evaluate方法，jsonString作为输入的JSON字符串，pathString作为输入的路径
        //判断如果输入的JSON字符串路径为空则返回null
        if  (jsonStr  ==  null  ||  jsonStr  ==  ""  ||  path  ==  null  ||  path  ==  "")  {
            return null;
        }
        String  jResult  =  evaluateJson(jsonStr,  path);
        if  (jResult  ==  null  ||  "".equals(jResult))
            return  null;
        result.set(jResult);
        return result.toString();
    }

    /**
     * @param jsonStr 传入的Json字符串
     * @param path Json路径
     * @param value
     * @return 判断Json字符串中，指定的path中，是否包含有value(完全包含),返回"true" 或者"false"
     */
    public String evaluate(String jsonStr, String path, String value) {
        if (jsonStr == null || jsonStr == "" || path == null
                || path == "" || value == null || "".equals(value)) {
            return null;
        }
        String jResult = evaluateJson(jsonStr, path);
        if (jResult == null || "".equals(jResult)) {
            result.set("false");
            return result.toString();
        }
        String[] sV = value.split(",");
        String[] dV = jResult.split(",");
        int sameCount = 0;
        for (int i = 0; i < sV.length; i++) {
            for (int j = 0; j < dV.length; j++) {
                if (sV[i].equals(dV[j])) {
                    sameCount++;
                    break;
                }
            }
        }
        if (sameCount == sV.length) {
            result.set("true");
        } else {
            result.set("false");
        }
        return result.toString();
    }

    /**
     * @param jsonStr 传入的Json字符串
     * @param path Json路径
     * @param value
     * @param flag 可以取值"|"或者"&","|"表示value中任意一个存在即可，"&"表示value中的值都需要包含
     * @return 判断Json字符串中，指定的path中，是否包含有value,返回"true" 或者"false"
     */
    public String evaluate(String jsonStr, String path, String value, String flag) {
        if ("&".equals(flag))
            return evaluate(jsonStr, path, value);
        if (jsonStr == null || jsonStr.equals("") || path == null || path.equals("")
                || value == null || "".equals(value) || flag == null
                || !"|".equals(flag)) {
            result.set("false");
            return result.toString();
        }
        String jResult = evaluateJson(jsonStr, path);
        if (jResult == null || "".equals(jResult)) {
            result.set("false");
            return result.toString();
        }
        String[] sV = value.split(",");
        String[] dV = jResult.split(",");
        for (int i = 0; i < sV.length; i++) {
            for (int j = 0; j < dV.length; j++) {
                if (sV[i].equals(dV[j])) {
                    result.set("true");
                    return result.toString();
                }
            }
        }
        result.set("false");
        return result.toString();
    }

    /**
     * Extract json object from a json string based on json path specified, and
     * return json string of the extracted json object. It will return null if
     * the input json string is invalid.
     *
     * A limited version of JSONPath supported: $ : Root object . : Child
     * operator [] : Subscript operator for array * : Wildcard for []
     *
     * Syntax not supported that's worth noticing: '' : Zero length string as
     * key .. : Recursive descent &amp;#064; : Current object/element () :
     * Script expression ?() : Filter (script) expression. [,] : Union operator
     * [start:end:step] : array slice operator
     *
     * @param jsonString the json string.
     * @param pathString the json path expression.
     * @return json string or null when an error happens.
     */
    private String evaluateJson(String jsonString, String pathString) {
        try {
            pathString = "$." + pathString;
            // Cache pathExpr
            //获取string数组类型的输入路径
            String[] pathExpr = pathExprCache.get(pathString);
            if (pathExpr == null) {
                //用“.”作为分隔,获取输入路径的string数组pathExpr，-1表示如果最后几位是分隔符也会继续切割
                pathExpr = pathString.split("\\.", -1);
                //使用put方法将pathString、pathExpr缓存到pathExprCache
                pathExprCache.put(pathString, pathExpr);
            }
            //如果path首位不为$，则返回null
            if (!pathExpr[0].equalsIgnoreCase("$")) {
                return null;
            }
            // Cache extractObject
            Object extractObject = extractObjectCache.get(jsonString);
            if (extractObject == null) {
                extractObject = new JSONObject(jsonString);
                Object put = extractObjectCache.put(jsonString, extractObject);
            }
            for (int i = 1; i < pathExpr.length; i++) {
                //根据路径参数pathExpr使用extract方法取出JSON字符串
                extractObject = extract(extractObject, pathExpr[i]);
            }
            //将获取的object类型的JSON字符串以string类型输出
            return extractObject.toString();
        } catch (Exception e) {
            //发生异常时返回null
            return null;
        }
    }

    private Object extract(Object json, String path) throws JSONException {
        // Cache patternkey.matcher(path).matches()
        Matcher mKey = null;
        Boolean mKeyMatches = mKeyMatchesCache.get(path);
        //如果缓存中不存在path
        if (mKeyMatches == null) {
            mKey = patternKey.matcher(path);
            //如果路径可以匹配到JSON字符串，返回True，如果无法匹配，返回False
            mKeyMatches = mKey.matches() ? Boolean.TRUE : Boolean.FALSE;
            mKeyMatchesCache.put(path, mKeyMatches);
        }
        if (!mKeyMatches.booleanValue()) {
            return null;
        }

        // Cache mkey.group(1)
        //根据path获取对应的JSON
        String mKeyGroup1 = mKeyGroup1Cache.get(path);
        //判断缓存中是否已存在JSON
        if (mKeyGroup1 == null) {
            if (mKey == null) {
                mKey = patternKey.matcher(path);
            }
            mKeyGroup1 = mKey.group(1);
            // 根据path缓存对应的JSON
            mKeyGroup1Cache.put(path, mKeyGroup1);
        }
        //获取JSON
        json = extract_json_withkey(json, mKeyGroup1);

        // Cache indexList
        //根据获取对应的数组类型JSON
        ArrayList<String> indexList = indexListCache.get(path);
        //判断缓存中是否已存在数组类型JSON
        if (indexList == null) {
            Matcher mIndex = patternIndex.matcher(path);
            indexList = new ArrayList<String>();
            while (mIndex.find()) {
                indexList.add(mIndex.group(1));
            }
            indexListCache.put(path, indexList);
        }

        if (indexList.size() > 0) {
            // 根据path缓存对应数组类型的JSON
            json = extract_json_withindex(json, indexList);
        }

        return json;
    }
    
    private Object extract_json_withindex(Object json, ArrayList<String> indexList) throws JSONException {

        jsonList.clear();
        jsonList.add(json);
        Iterator<String> itr = indexList.iterator();
        while (itr.hasNext()) {
            String index = itr.next();
            //创建数组类型JSON对象
            ArrayList<Object> tmp_jsonList = new ArrayList<Object>();
            //创建获取数组类型JSON的方法 throws JSONException {
            if (index.equalsIgnoreCase("*")) {
                for (int i = 0; i < (jsonList).size(); i++) {
                    try {
                        JSONArray array = (JSONArray) (jsonList).get(i);
                        for (int j = 0; j < array.length(); j++) {
                            tmp_jsonList.add(array.get(j));
                        }
                    } catch (Exception e) {
                        continue;
                    }
                }
                jsonList = tmp_jsonList;
              //如果不存在通配符，则遍历JSON
            } else {
                for (int i = 0; i < (jsonList).size(); i++) {
                    try {
                        tmp_jsonList.add(((JSONArray) (jsonList).get(i))
                                .get(Integer.parseInt(index)));
                    } catch (ClassCastException e) {
                        continue;
                    } catch (JSONException e) {
                        return null;
                    }
                    jsonList = tmp_jsonList;
                }
            }
        }
        return (jsonList.size() > 1) ? new JSONArray(jsonList) : jsonList
                .get(0);
    }
    //创建获取普通JSON的方法
    private Object extract_json_withkey(Object json, String path) throws JSONException {
        if (json.getClass() == JSONArray.class) {
            JSONArray jsonArray = new JSONArray();
            for (int i = 0; i < ((JSONArray) json).length(); i++) {
                Object josn_elem = ((JSONArray) json).get(i);
                try {
                    Object json_obj = ((JSONObject) josn_elem).get(path);
                    if (json_obj.getClass() == JSONArray.class) {
                        for (int j = 0; j < ((JSONArray) json_obj).length(); j++) {
                            jsonArray.put(((JSONArray) json_obj).get(j));
                        }
                    } else {
                        jsonArray.put(json_obj);
                    }
                } catch (Exception e) {
                    continue;
                }
            }
            return (jsonArray.length() == 0) ? null : jsonArray;
        } else {
            return ((JSONObject) json).get(path);
        }
    }
}
```

