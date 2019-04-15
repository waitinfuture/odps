# 基于Package的资源分享 {#concept_kyz_531_wdb .concept}

本文为您介绍基于Package的资源分享语句说明。

## 分享资源 {#section_ofl_bk1_wdb .section}

|语句|说明|
|:-|:-|
|create package `<pkgName>`|创建一个Package|
|delete package `<pkgName>`|删除一个Package|
|add `<objType>``<objName>` to package `<pkgName>` \[with privileges privs\]|向Package中添加需要分享的资源|
|remove `<objType>``<objName>` from package `<pkgName>`|从Package中删除已分享的资源|
|allow project `<prjName>` to install package `<pkgName>` \[using label `<num>`\]|许可某个项目空间使用您的Package|
|disallow project `<prjName>` to install package `<pkgName>`|禁止某个项目空间使用您的Package|

## 使用资源 {#section_sg2_dk1_wdb .section}

|语句|说明|
|:-|:-|
|install package `<pkgName>`|安装Package|
|uninstall package `<pkgName>`|卸载Package|

## 查看Package {#section_fyp_2k1_wdb .section}

|语句|说明|
|:-|:-|
|show packages|列出所有创建和安装的Packages|
|describe package `<pkgName>`|查看package的详细信息|

**说明：** 

如果在DataWorks执行生产project授权相关的请求时：

1.  project owner通过临时查询方式执行，不能提交到生产环境执行。因为生产环境是由生产账号执行，而这个账号没有授权的权限。
2.  查询语句前加use <生产project\>;语句，并与命令一起提交。因为DataWorks数据开发默认当前的project是\_dev结尾的开发project。通过命令行执行授权命令时，请project owner先执行：

    ```
     use project_name;--进入对应的project
    ```


