# IntelliJ IDEA插件安装步骤 {#concept_k5m_kpy_5db .concept}

MaxCompute Studio是IntelliJ IDEA的插件，本文为您介绍如何安装MaxCompute Studio。

## 环境要求 {#section_ozv_n5y_5db .section}

IntelliJ IDEA支持在*Windows*， *Mac*， *Linux*操作系统上安装，硬件及系统环境要求请参见 [Requirements for IntelliJ IDEA](https://www.jetbrains.com/help/idea/2016.3/requirements-for-intellij-idea.html) 。基于IntelliJ IDEA平台的MaxCompute Studio也可以安装在这些操作系统的客户端上。

MaxCompute Studio对用户环境有以下要求：

-   Windows，Mac OS，或者Linux系统客户端。
-   安装IntelliJ IDEA v18.2.4以上版本 （支持Ultimate版本、PyCharm版本和免费的 [Community 版本](https://www.jetbrains.com/idea/download/)）。
-   已安装JRE 1.8 （最新的IntelliJ IDEA版本捆绑了JRE 1.8）。
-   已安装JDK 1.8 （*可选*：如果需要开发和调试Java UDF，则需安装 JDK）。

**说明：** 客户端从0.28.0版开始支持JDK 1.9，在这之前的版本只支持JDK 1.8。

## 安装方式 {#section_qzv_n5y_5db .section}

MaxCompute Studio是IntelliJ IDEA的插件，有以下两种安装方式：

-   通过插件库在线安装（推荐）。
-   通过本地文件安装。

## 在线安装（推荐） {#section_szv_n5y_5db .section}

MaxCompute Studio插件已对全部公网用户开放，您可以通过IntelliJ官方插件库安装。

**操作步骤**

1.  在IntelliJ IDEA中打开插件配置页面 （Windows/Linux用户导航至 **File** \> **Settings** \> **Plugins**，Mac用户导航至**IntelliJ IDEA** \> **Preferences** \> **Plugins** ）。
2.  单击**Browse repositories…**按钮，然后搜索`MaxCompute Studio`。

3.  找到MaxCompute Studio插件页面，单击绿色**Install**按钮进行安装。

4.  确认安装后，重新启动IntelliJ IDEA，完成安装。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12117/15541875611555_zh-CN.png)


## 本地安装 {#section_vzv_n5y_5db .section}

MaxCompute Studio也可以在本地环境中进行安装。

**操作步骤**

1.  进入[MaxCompute Studio 插件页面](https://plugins.jetbrains.com/plugin/9193?spm=5176.doc44555.2.1.4hXBG1)下载插件包。

2.  运行IntelliJ IDEA。

    -   如果是第一次运行，会出现欢迎界面，单击欢迎界面中的**configure**（配置），选中弹出菜单中的**Plugins**（插件），如下图所示：

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12117/15541875611556_zh-CN.png)

    -   如果不是第一次运行，可以依次单击菜单**File** \> **Settings** \> **Plugins**进入相同的界面，如下图所示：

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12117/15541875611557_zh-CN.png)

        ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12117/15541875611558_zh-CN.png)

3.  在插件页面，单击**Install plugin from disk…**（从本地磁盘安装插件），如下图所示：

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12117/15541875611559_zh-CN.png)

4.  在弹出窗口中，通过单击目录名称前的灰色图标进行导航，找到插件文件并选中，单击**OK**。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12117/15541875611561_zh-CN.png)

5.  回到插件首页后，单击**OK**，开始安装本地插件。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12117/15541875611562_zh-CN.png)

6.  安装完成后，弹出重新启动的提示窗口，单击**Restart** ，重新启动IntelliJ IDEA。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12117/15541875611563_zh-CN.png)

7.  重新启动后，界面如下所示：

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/12117/15541875611564_zh-CN.png)


## 后续步骤 {#section_g1w_n5y_5db .section}

现在，您已经学习了如何安装MaxCompute Studio插件，您可以继续学习下一个教程。在该教程中您将学习如何配置MaxCompute Project连接管理数据和资源。详情请参见 [新建MaxCompute项目空间连接](intl.zh-CN/工具及下载/MaxCompute Studio/项目空间连接管理.md)。

