# Jupyter Notebook增强 {#concept_ybt_rgg_cfb .concept}

PyODPS针对Jupyter Notebook下的探索性数据分析进行了增强， 包括结果探索功能以及进度展示功能。

## 结果探索 {#section_rmm_yh4_cfb .section}

PyODPS在Jupyter Notebook中为SQL Cell和DataFrame提供了数据探索功能。对于已拉到本地的数据，可使用交互式的数据探索工具 浏览数据，交互式地绘制图形。

当执行结果为DataFrame时，PyODPS会读取执行结果，并以分页表格的形式展示出来。单击页号或前进 / 后退按钮可在数据中导航， 如下图。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21195/155236327411733_zh-CN.png)

结果区的顶端为模式选择区。除数据表外，也可以选择柱状图、饼图、折线图和散点图。下图为使用默认字段选择（即前三个字段） 绘制的散点图。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21195/155236327411734_zh-CN.png)

在绘图模式下，单击右上角的配置按钮可以修改图表设置。如下图中，将name设置为分组列，X 轴选择为petallength，Y轴选择为petalwidth，则图表变为下图。可见在petallength - petalwidth维度下，数据对name有较好的区分度。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21195/155236327411735_zh-CN.png)

对于柱状图和饼图，值字段支持选择聚合函数。PyODPS对柱状图的默认聚合函数为sum，对饼图则为count。如需修改聚合函数， 可在值字段名称后的聚合函数名上单击，此后选择所需的聚合函数即可。

对于折线图，需要避免X轴包含空值，否则图像可能不符合预期。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21195/155236327411736_zh-CN.png)

完成绘图后，可单击“下载”保存绘制的图表。

**说明：** 注意：使用此功能需要安装Pandas ，并保证ipywidgets被正确安装。

## 进度展示 {#section_f2y_p34_cfb .section}

大型作业执行通常需要较长的时间，因而PyODPS提供了进度展示功能。当DataFrame、机器学习作业或通过%sql编写的SQL语句在Jupyter Notebook中执行作业时，会显示当前正在执行的作业列表及总体进度，如下图：

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21195/155236327411737_zh-CN.png)

当点击某个作业名称上的链接时，会弹出一个对话框，显示该作业中每个Task的具体执行进度，如图：

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21195/155236327511738_zh-CN.png)

当作业运行成功后，浏览器将给出提醒信息，告知作业是否成功：

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/21195/155236327511739_zh-CN.png)

