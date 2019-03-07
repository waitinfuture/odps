# UDJ {#concept_e3d_ck4_hfb .concept}

目前基于MaxCompute 2.0计算引擎，MaxCompute在UDF框架中新近引入的一种新扩展机制：UDJ\(User Defined Join\)，来实现灵活的跨表、多表自定义操作，同时减少不得不通过MR等方式对分布式系统底层细节的操作。这是MaxCompute基于新一代体系机构发展NewSQL数据处理框架的重要一步。

## UDJ概述 {#section_p1f_kk4_hfb .section}

目前MaxCompute内置了多种[Join](intl.zh-CN/用户指南/SQL/SELECT操作/JOIN.md#)操作，包括inner/right join、outer/left jion、outer/full join、outer/semi/anti-semi join等。这些内置的join操作功能强大，能够满足很大一部分需求，但是其标准的join实现，无法满足很多跨表操作的需求。

很多情况下，可以通过UDF（User Defined Function）描述您的代码框架，但现有的UDF/UDTF/UDAF接口主要是针对在单个数据表上的操作而设计，一旦涉及多表的自定义操作，经常还需要依赖于内置**join +**各种UDF/UDTF, 并配合比较复杂的SQL语句来完成。甚至在一些多表操作的场景上，您不得不放弃SQL而转向传统的完全自定义MapReduce，才能完成所需的计算。

无论是join+各种UDF/UDTF+复杂SQL还是自定义MR门槛都比较高，同时还会带来一些问题：

-   对于计算平台而言，多个复杂的join和散布在SQL语言各处的代码揉合在一起，将带来多处的逻辑黑盒，这点不利于生成最优的执行计划。
-   使用MR，不仅更大程度上限制了系统进行执行优化的可能性，而且由于MR绝大部分代码由Java完成，在执行效率上会远低于MaxCompute基于LLVM 代码生成器产生的深度优化native运行时。

## UDJ使用 {#section_pww_vk4_hfb .section}

接下来用一个样例来详细介绍MaxCompute的UDJ使用。

-   样例定义

    两个日志表，分别是payment和user\_client\_log。

    -   payment \(user\_id string,time datetime,pay\_info string\)表中保存了用户的支付记录，一笔支付记录包含用户ID、支付时间和支付内容。
    -   user\_client\_log\(user\_id string,time datetime,content string\)表保存了用户的客户端日志，每一条日志包含了用户ID、日志时间和日志内容。
    需求：对于每一条客户端日志，找出该用户在payment表里时间最接近的一条支付记录，将其中的支付内容和日志内容合并输出。

    样例数据如下：

    payment

    |user\_id|time|pay\_info|
    |:-------|:---|:--------|
    |2656199|2018-02-13 22:30:00|gZhvdySOQb|
    |8881237|2018-02-13 08:30:00|pYvotuLDIT|
    |8881237|2018-02-13 10:32:00|KBuMzRpsko|

    user\_client\_log

    |user\_id|time|content|
    |:-------|:---|:------|
    |8881237|2018-02-13 00:30:00|click MpkvilgWSmhUuPn|
    |8881237|2018-02-13 06:14:00|click OkTYNUHMqZzlDyL|
    |8881237|2018-02-13 10:30:00|click OkTYNUHMqZzlDyL|

    其中user\_client\_log的一条记录

    |user\_id|time|content|
    |:-------|:---|:------|
    |8881237|2018-02-13 00:30:00|click MpkvilgWSmhUuPn|

    和payment中时间最接近的一条是

    |user\_id|time|pay\_info|
    |--------|----|---------|
    |8881237|2018-02-13 08:30:00|pYvotuLDIT|

    因此，这两条记录合并为：

    |8881237|2018-02-13 00:30:00|click MpkvilgWSmhUuPn, pay pYvotuLDIT|

    上面样例数据全部合并的结果如下：

    |user\_id|time|content|
    |:-------|:---|:------|
    |8881237|2018-02-13 00:30:00|click MpkvilgWSmhUuPn, pay pYvotuLDIT|
    |8881237|2018-02-13 06:14:00|click OkTYNUHMqZzlDyL, pay pYvotuLDIT|
    |8881237|2018-02-13 10:30:00|click OkTYNUHMqZzlDyL, pay KBuMzRpsko|

-   使用内置JOIN

    若使用标准join解决这个需求，会发现这个需求除了需要按照user\_id进行关联以外，还需要知道payment中哪一条记录和user\_client\_log中的记录的时间差异值最小，勉强写SQL伪代码则类似于：

    ```
    SELECT
      p.user_id,
      p.time,
      merge(p.pay_info, u.content)
    FROM
      payment p RIGHT OUTER JOIN user_client_log u
    ON p.user_id = u.user_id and abs(p.time - u.time) = min(abs(p.time - u.time))
    ```

    关联时需要知道相同user\_id下的p.time与u.time差异最小的值，而聚合函数不能出现在关联条件上。因此，这个看似简单的需求，无法通过标准的关联操作实现。

    但在分布式系统中，Join操作实际上是将两个表按照某个\(或多个\)字段进行分组，并将同组数据集中到一处，而标准SQL中的Join对于关联后的操作有限，此时，若能提供一个通用编程语言接口，使用过程通过插件式的方式实现这个接口，在这个接口中将关联后的分组数据进行自定义处理并输出既可解决这个问题，这也是UDJ要解决的问题。

-   使用Java编写UDJ代码

    接下来详细介绍如何通过UDJ实现这个内置join无法实现的需求。

    前置条件

    这是一个新功能，我们需要一个比较新的SDK：

    ```
    <dependency>
      <groupId>com.aliyun.odps</groupId>
      <artifactId>odps-sdk-udf</artifactId>
      <version>0.29.10-public</version>
      <scope>provided</scope>
    </dependency>
    ```

    新的SDK中包含了一个新的抽象类UDJ，我们通过继承这个类，来实现UDJ的功能：

    ```
    package com.aliyun.odps.udf.example.udj;
    import com.aliyun.odps.Column;
    import com.aliyun.odps.OdpsType;
    import com.aliyun.odps.Yieldable;
    import com.aliyun.odps.data.ArrayRecord;
    import com.aliyun.odps.data.Record;
    import com.aliyun.odps.udf.DataAttributes;
    import com.aliyun.odps.udf.ExecutionContext;
    import com.aliyun.odps.udf.UDJ;
    import com.aliyun.odps.udf.annotation.Resolve;
    import java.util.ArrayList;
    import java.util.Iterator;
    /** 对于右表的每个记录，找到最近的左表记录和
     * 合并两条记录。
     */
    @Resolve("->string,bigint,string")
    public class PayUserLogMergeJoin extends UDJ {
      private Record outputRecord;
      /** 将在数据处理阶段之前调用。 用户可以实施
       * 这个方法做初始化工作。
       */
      @Override
      public void setup(ExecutionContext executionContext, DataAttributes dataAttributes) {
        //
        outputRecord = new ArrayRecord(new Column[]{
          new Column("user_id", OdpsType.STRING),
          new Column("time", OdpsType.BIGINT),
          new Column("content", OdpsType.STRING)
        });
      }
      /** 重写此方法以实现连接逻辑。
       * @param key 当前连接键
       * @param 左表对应当前键的记录组
       * @param right对应当前键的右表记录组
       * @param output用于输出UDJ的结果
       */
      @Override
      public void join(Record key, Iterator<Record> left, Iterator<Record> right, Yieldable<Record> output) {
        outputRecord.setString(0, key.getString(0));
        if (!right.hasNext()) {
          // 空右组，什么都不做。
          return;
        } else if (!left.hasNext()) {
          // 空左组。 输出右侧组的所有记录而不合并。
          while (right.hasNext()) {
            Record logRecord = right.next();
            outputRecord.setBigint(1, logRecord.getDatetime(0).getTime());
            outputRecord.setString(2, logRecord.getString(1));
            output.yield(outputRecord);
          }
          return;
        }
        ArrayList<Record> pays = new ArrayList<>();
        // 左侧记录组将从开始到结束迭代
        // 对于右组的每个记录，但迭代器无法重置。
        // 所以我们将左边的每个记录保存到ArrayList。
        left.forEachRemaining(pay -> pays.add(pay.clone()));
        while (right.hasNext()) {
          Record log = right.next();
          long logTime = log.getDatetime(0).getTime();
          long minDelta = Long.MAX_VALUE;
          Record nearestPay = null;
          // 迭代左边的所有记录，找到有的记录
          // 时间上的微小差异。
          for (Record pay: pays) {
            long delta = Math.abs(logTime - pay.getDatetime(0).getTime());
            if (delta < minDelta) {
              minDelta = delta;
              nearestPay = pay;
            }
          }
          // 将日志记录与最近的支付记录合并，并输出到结果。
          outputRecord.setBigint(1, log.getDatetime(0).getTime());
          outputRecord.setString(2, mergeLog(nearestPay.getString(1), log.getString(1)));
          output.yield(outputRecord);
        }
      }
      String mergeLog(String payInfo, String logContent) {
        return logContent + ", pay " + payInfo;
      }
      @Override
      public void close() {
      }
    }
    ```

    **说明：** 在本例中没有处理记录中的NULL值，为了使程序简洁便于演示，这里假设数据中没有NULL值。

    从代码中可以看到，在每次调用UDJ的join方法时，两张表中各有一组对应着同一个key数据，提供给了我们。因此，只需遍历右表\(user\_client\_log\)的分组，对于每一个log记录，遍历一遍左表\(payment\)的分组，找出时间相差最小的记录，将日志内容合并然后输出即可。

    这里假设同一个用户的支付记录数比较少，可以预先将左表分组全部加载到内存\(通常情况下，内存足以存放一个用户在一天内产生的支付数据\)。但若这个假设不成立应该如何解决？文章后面内容使用[SORT BY预排序](intl.zh-CN/用户指南/SQL/UDJ.md#section_p1f_kk4_hfb)会解决这个问题。

-   在MaxCompute中创建UDJ

    编写完UDJ的Java代码后，需要将这部分代码插件式的嵌入到MaxCompute SQL中进行执行。再此之前，需要将代码注册到MaxCompute。假设上述代码打包成了odps-udj-example.jar。

    通过add jar命令将其当做jar资源上传到MaxCompute:

    ```
    add jar odps-udj-example.jar;
    ```

    通过create function语句注册UDJ函数，指定UDJ在SQL中的函数名`pay_user_log_merge_join`，以及关联上它对应的jar资源odps-udj-example.jar和在jar包中的类名

    `com.aliyun.odps.udf.example.udj.PayUserLogMergeJoin:`

    ```
    create function pay_user_log_merge_join
      as 'com.aliyun.odps.udf.example.udj.PayUserLogMergeJoin'
      using 'odps-udj-example.jar';
    ```

-   使用MaxCompute SQL进行UDJ查询

    UDJ注册好了以后，就可以在MaxCompute SQL中使用了。

    1.  创建示例源表：

        ```
        create table payment(user_id string,time datetime,pay_info string);
        create table user_client_log(user_id string,time datetime,content string);
        ```

    2.  制造演示数据：

        ```language-xml
        --制造payment表数据
        INSERT OVERWRITE TABLE payment VALUES
        ('1335656', datetime '2018-02-13 19:54:00', 'PEqMSHyktn'),
        ('2656199', datetime '2018-02-13 12:21:00', 'pYvotuLDIT'),
        ('2656199', datetime '2018-02-13 20:50:00', 'PEqMSHyktn'),
        ('2656199', datetime '2018-02-13 22:30:00', 'gZhvdySOQb'),
        ('8881237', datetime '2018-02-13 08:30:00', 'pYvotuLDIT'),
        ('8881237', datetime '2018-02-13 10:32:00', 'KBuMzRpsko'),
        ('9890100', datetime '2018-02-13 16:01:00', 'gZhvdySOQb'),
        ('9890100', datetime '2018-02-13 16:26:00', 'MxONdLckwa')
        ;
        --制造user_client_log表数据
        INSERT OVERWRITE TABLE user_client_log VALUES
        ('1000235', datetime '2018-02-13 00:25:36', 'click FNOXAibRjkIaQPB'),
        ('1000235', datetime '2018-02-13 22:30:00', 'click GczrYaxvkiPultZ'),
        ('1335656', datetime '2018-02-13 18:30:00', 'click MxONdLckpAFUHRS'),
        ('1335656', datetime '2018-02-13 19:54:00', 'click mKRPGOciFDyzTgM'),
        ('2656199', datetime '2018-02-13 08:30:00', 'click CZwafHsbJOPNitL'),
        ('2656199', datetime '2018-02-13 09:14:00', 'click nYHJqIpjevkKToy'),
        ('2656199', datetime '2018-02-13 21:05:00', 'click gbAfPCwrGXvEjpI'),
        ('2656199', datetime '2018-02-13 21:08:00', 'click dhpZyWMuGjBOTJP'),
        ('2656199', datetime '2018-02-13 22:29:00', 'click bAsxnUdDhvfqaBr'),
        ('2656199', datetime '2018-02-13 22:30:00', 'click XIhZdLaOocQRmrY'),
        ('4356142', datetime '2018-02-13 18:30:00', 'click DYqShmGbIoWKier'),
        ('4356142', datetime '2018-02-13 19:54:00', 'click DYqShmGbIoWKier'),
        ('8881237', datetime '2018-02-13 00:30:00', 'click MpkvilgWSmhUuPn'),
        ('8881237', datetime '2018-02-13 06:14:00', 'click OkTYNUHMqZzlDyL'),
        ('8881237', datetime '2018-02-13 10:30:00', 'click OkTYNUHMqZzlDyL'),
        ('9890100', datetime '2018-02-13 16:01:00', 'click vOTQfBFjcgXisYU'),
        ('9890100', datetime '2018-02-13 16:20:00', 'click WxaLgOCcVEvhiFJ')
        ;
        ```

    3.  在MaxCompute SQL中使用刚刚创建好的UDJ函数：

        ```
        SELECT r.user_id, from_unixtime(time/1000) as time, content FROM (
        SELECT user_id, time as time, pay_info FROM payment
        ) p JOIN (
        SELECT user_id, time as time, content FROM user_client_log
        ) u
        ON p.user_id = u.user_id
        USING pay_user_log_merge_join(p.time, p.pay_info, u.time, u.content)
        r
        AS (user_id, time, content)
        ;
        ```

        UDJ的语法与标准Join语法类似，主要差异多了using字句：

        -   pay\_user\_log\_merge\_join是注册的UDJ在SQL中的函数名；
        -   \(p.time, p.pay\_info, u.time, u.content\)是UDJ中分别用到的左右表的列；
        -   r是UDJ结果的别名，用于其他地方引用UDJ的结果；
        -   \(user\_id, time, content\)是UDJ产生的结果的列名。
        运行上面这条SQL，结果为：

        ```language-sql
        +---------+------------+---------+
        | user_id | time       | content |
        +---------+------------+---------+
        | 1000235 | 2018-02-13 00:25:36 | click FNOXAibRjkIaQPB |
        | 1000235 | 2018-02-13 22:30:00 | click GczrYaxvkiPultZ |
        | 1335656 | 2018-02-13 18:30:00 | click MxONdLckpAFUHRS, pay PEqMSHyktn |
        | 1335656 | 2018-02-13 19:54:00 | click mKRPGOciFDyzTgM, pay PEqMSHyktn |
        | 2656199 | 2018-02-13 08:30:00 | click CZwafHsbJOPNitL, pay pYvotuLDIT |
        | 2656199 | 2018-02-13 09:14:00 | click nYHJqIpjevkKToy, pay pYvotuLDIT |
        | 2656199 | 2018-02-13 21:05:00 | click gbAfPCwrGXvEjpI, pay PEqMSHyktn |
        | 2656199 | 2018-02-13 21:08:00 | click dhpZyWMuGjBOTJP, pay PEqMSHyktn |
        | 2656199 | 2018-02-13 22:29:00 | click bAsxnUdDhvfqaBr, pay gZhvdySOQb |
        | 2656199 | 2018-02-13 22:30:00 | click XIhZdLaOocQRmrY, pay gZhvdySOQb |
        | 4356142 | 2018-02-13 18:30:00 | click DYqShmGbIoWKier |
        | 4356142 | 2018-02-13 19:54:00 | click DYqShmGbIoWKier |
        | 8881237 | 2018-02-13 00:30:00 | click MpkvilgWSmhUuPn, pay pYvotuLDIT |
        | 8881237 | 2018-02-13 06:14:00 | click OkTYNUHMqZzlDyL, pay pYvotuLDIT |
        | 8881237 | 2018-02-13 10:30:00 | click OkTYNUHMqZzlDyL, pay KBuMzRpsko |
        | 9890100 | 2018-02-13 16:01:00 | click vOTQfBFjcgXisYU, pay gZhvdySOQb |
        | 9890100 | 2018-02-13 16:20:00 | click WxaLgOCcVEvhiFJ, pay MxONdLckwa |
        +---------+------------+---------+
        ```

        如上述代码，通过使用UDJ完成了内置join无法轻松完成的需求。

-   UDJ预排序功能

    前面的UDJ代码抛出过这样的问题：为了找到payment中相差最小的一条记录，需要反复对payment表的数据进行iterator遍历，所以事先将相同user\_id的payment记录全部加载到了ArrayList，当同一个用户一天之内的支付行为比较少时，这个方式可用，但在其它场景中，有时候同组内的数据可能非常大，大到无法在内存中放下，此时就需要通过其他方式解决。本小节将介绍通过SORT BY预排序解决这个场景。

    当某个用户的支付数据量非常巨大导致无法将payment放在内存中时，仔细分析会发现组内所有数据如果已经按照time排好序，那么这个问题可以这样解决：只需要比较两边iterator最”顶端”的数据，就可以实现这个功能。

    java UDJ代码如下：

    ```language-java
    @Override
    public void join(Record key, Iterator<Record> left, Iterator<Record> right, Yieldable<Record> output) {
      outputRecord.setString(0, key.getString(0));
      if (!right.hasNext()) {
        return;
      } else if (!left.hasNext()) {
        while (right.hasNext()) {
          Record logRecord = right.next();
          outputRecord.setBigint(1, logRecord.getDatetime(0).getTime());
          outputRecord.setString(2, logRecord.getString(1));
          output.yield(outputRecord);
        }
        return;
      }
      long prevDelta = Long.MAX_VALUE;
      Record logRecord = right.next();
      Record payRecord = left.next();
      Record lastPayRecord = payRecord.clone();
      while (true) {
        long delta = logRecord.getDatetime(0).getTime() - payRecord.getDatetime(0).getTime();
        if (left.hasNext() && delta > 0) {
          //两个记录之间的时间差正在减少，我们仍然可以操作
          //探索左侧组以尝试获得更小的增量。
          lastPayRecord = payRecord.clone();
          prevDelta = delta;
          payRecord = left.next();
        } else {
         //到达最小delta点。 检查最后的工资记录，
         //输出合并结果并准备处理下一条记录
         //右组.
          Record nearestPay = Math.abs(delta) < prevDelta ? payRecord : lastPayRecord;
          outputRecord.setBigint(1, logRecord.getDatetime(0).getTime());
          String mergedString = mergeLog(nearestPay.getString(1), logRecord.getString(1));
          outputRecord.setString(2, mergedString);
          output.yield(outputRecord);
          if (right.hasNext()) {
            logRecord = right.next();
            prevDelta = Math.abs(
              logRecord.getDatetime(0).getTime() - lastPayRecord.getDatetime(0).getTime()
            );
          } else {
            break;
          }
        }
      }
    }
    ```

    SQL语句中，只需要对前面的例子稍作修改，在UDJ语句尾部增加SORT BY子句，指定UDJ组内左右表分别都按照各自的time字段进行排序。

    **说明：** 注意：UDJ代码修改后需要更新UDJ对应的jar包：

    ```
    SELECT r.user_id, from_unixtime(time/1000) as time, content FROM (
      SELECT user_id, time as time, pay_info FROM payment
    ) p JOIN (
      SELECT user_id, time as time, content FROM user_client_log
    ) u
    ON p.user_id = u.user_id
    USING pay_user_log_merge_join(p.time, p.pay_info, u.time, u.content)
    r
    AS (user_id, time, content)
    SORT BY p.time, u.time
    ;
    ```

    执行结果将和前面的方式执行结果一样。

    该方式主要是使用SORT BY子句对UDJ的数据进行预排序，在这个过程中最多只需要同时缓存3条记录，就可以实现和之前算法的相同的功能。


## UDJ的性能 {#section_u3k_nm4_hfb .section}

当前通常情况下，在面对复杂的跨表计算需求时，因为缺乏UDJ这种机制，不得不借助MR来实现整套跨表计算的分布式流程。这类场景业务复杂的BU（比如搜索，广告等）尤为常见。

在此用一个线上线上真实的MR作业进行测试验证UDJ的性能。该MR作业实现一套比较复杂的算法将两个表合并一起，用UDJ对该MR进行改写，并且验证新的UDJ实现结果的正确性。性能方面，在并发度相同的情况下两者性能对比如下：

![](images/13237_zh-CN.jpeg)

由图可见，UDJ接口的引入，一方面更加方便的描述对多表数据进行操作的复杂逻辑，一方面大幅度提高了性能（代码只有在UDJ内被调用，其上下游的逻辑（比如这个例子中的整个mapper逻辑）则完全通过MaxCompute高效的native运行时完成）。在Java代码中，由于UDJ对MaxCompute运行时引擎和Java接口之间的数据交互逻辑做了深度的优化，通过UDJ实现的join逻辑，也比其对等的reducer更高效。

