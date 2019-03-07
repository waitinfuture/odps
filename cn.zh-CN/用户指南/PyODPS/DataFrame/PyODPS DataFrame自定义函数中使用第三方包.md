# PyODPS DataFrame自定义函数中使用第三方包 {#concept_efd_byl_h2b .concept}

本文向您介绍如何在PyODPS DataFrame自定义函数中上传和使用pandas、scipy和scikit-learn第三方包。

[PyODPS DataFrame](cn.zh-CN/用户指南/PyODPS/DataFrame/创建DataFrame.md#)提供了类似pandas的接口来操作MaxCompute数据，同时也支持在本地使用pandas和使用数据库来执行。

PyODPS DataFrame不仅支持类似pandas的map和apply方法，也提供了MapReduce API来扩展pandas语法以适应大数据环境。

PyODPS的自定义函数是序列化到MaxCompute上执行，MaxCompute的Python环境仅包含numpy第三方包。现在，MaxCompute可以实现在自定义函数中使用pandas、scipy或scikit-learn等包含c代码的库。

**说明：** PyODPS需要0.7.4及以上版本 。

## 上传第三方包 {#section_my4_n1m_h2b .section}

**说明：** 您只需上传一次第三方包，当MaxCompute资源有了这些包，可直接跳过此步。

现在主流的Python包都提供了whl包，提供了各平台包含二进制文件的包，因此找到可以在MaxCompute上运行的包是第一步。

其次，要想在MaxCompute上运行，需要包含所有的依赖包，这个是比较繁琐的。各个包的依赖情况如下表所示。

|包名|依赖|
|:-|:-|
|pandas|numpy，python-dateutil，pytz，six|
|scipy|numpy|
|scikit-learn|numpy，scipy|

**说明：** 其中numpy已包含，您只需上传python-dateutil、pytz、pandas、scipy、sklearn、six包，pandas、scipy和scikit-learn即可使用。

您可进入[python-dateutils](http://mirrors.aliyun.com/pypi/simple/python-dateutil/)找到[python-dateutil-2.6.0.zip](http://mirrors.aliyun.com/pypi/packages/95/8e/71125f3f24771f50e630b5a6fa9fd209a9f167dcbc3aad65a48cb3dd5694/python-dateutil-2.6.0.zip#md5=530f7b56e36fa42ada6c02a17b15660c)进行下载。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/15204/15519475906659_zh-CN.png)

重命名为python-dateutil.zip，通过MaxCompute Console上传资源。

```
add archive python-dateutil.zip;
```

**说明：** pytz和six的上传方式同上，分别找到 [pytz-2017.2.zip](http://mirrors.aliyun.com/pypi/packages/a4/09/c47e57fc9c7062b4e83b075d418800d322caa87ec0ac21e6308bd3a2d519/pytz-2017.2.zip#md5=f89bde8a811c8a1a5bac17eaaa94383c)和[six-1.11.0.tar.gz](http://mirrors.aliyun.com/pypi/packages/16/d8/bc6316cf98419719bd59c91742194c111b6f2e85abac88e496adefaf7afe/six-1.11.0.tar.gz#md5=d12789f9baf7e9fb2524c0c64f1773f8)进行下载和上传资源操作。

对于pandas这种包含c的包，需要找到名字中包含cp27-cp27m-manylinux1\_x86\_64的whl包，这样才能在MaxCompute上正确执行。因此，您需要找到[pandas-0.20.2-cp27-cp27m-manylinux1\_x86\_64.whl](http://mirrors.aliyun.com/pypi/packages/44/39/e71009a0ebdbb6206b9fbde0367fc5cb5bb7fdb4521ae785ca7bd63d36aa/pandas-0.20.2-cp27-cp27m-manylinux1_x86_64.whl#md5=31a4d180048f72337d53cc7b87424568)进行下载，然后把后缀改成zip，在MaxCompute Console中执行`add archive pandas.zip;`进行上传。

其他包的操作同上，需下载资源如下表所示。

|包名|文件名|上传资源名|
|:-|:--|:----|
|python-dateutil|[python-dateutil-2.6.0.zip](http://mirrors.aliyun.com/pypi/packages/95/8e/71125f3f24771f50e630b5a6fa9fd209a9f167dcbc3aad65a48cb3dd5694/python-dateutil-2.6.0.zip#md5=530f7b56e36fa42ada6c02a17b15660c)|python-dateutil.zip|
|pytz|[pytz-2017.2.zip](http://mirrors.aliyun.com/pypi/packages/a4/09/c47e57fc9c7062b4e83b075d418800d322caa87ec0ac21e6308bd3a2d519/pytz-2017.2.zip#md5=f89bde8a811c8a1a5bac17eaaa94383c)|pytz.zip|
|six|[six-1.11.0.tar.gz](http://mirrors.aliyun.com/pypi/packages/16/d8/bc6316cf98419719bd59c91742194c111b6f2e85abac88e496adefaf7afe/six-1.11.0.tar.gz#md5=d12789f9baf7e9fb2524c0c64f1773f8)|six.tar.gz|
|pandas|[pandas-0.20.2-cp27-cp27m-manylinux1\_x86\_64.zip](http://mirrors.aliyun.com/pypi/packages/44/39/e71009a0ebdbb6206b9fbde0367fc5cb5bb7fdb4521ae785ca7bd63d36aa/pandas-0.20.2-cp27-cp27m-manylinux1_x86_64.whl#md5=31a4d180048f72337d53cc7b87424568)|pandas.zip|
|scipy|[scipy-0.19.0-cp27-cp27m-manylinux1\_x86\_64.zip](http://mirrors.aliyun.com/pypi/packages/ae/94/28ca6f9311e2351bb68da41ff8c1bc8f82bb82791f2ecd34efa953e60576/scipy-0.19.0-cp27-cp27m-manylinux1_x86_64.whl#md5=0e49f7fc8d31c1c79f0a4d63b29e8a1f)|scipy.zip|
|scikit-learn|[scikit\_learn-0.18.1-cp27-cp27m-manylinux1\_x86\_64.zip](http://mirrors.aliyun.com/pypi/packages/ca/dd/a18dba8ab879b13b43c3838a25887585a45101f4bffa398e1883e1e3d395/scikit_learn-0.18.1-cp27-cp27m-manylinux1_x86_64.whl#md5=b068bde57f00d285cc89eb0b8615fcae)|sklearn.zip|

**说明：** 您也可以使用PyODPS的资源上传接口来完成资源的上传，同样只需操作一遍。

## 编写代码验证 {#section_uzp_ppt_h2b .section}

1.  写一个简单的函数，里面用到所有的库，最好是在函数中import这些第三方库。

    ```
    def test(x):
        from sklearn import datasets, svm
        from scipy import misc
        import numpy as np
    
        iris = datasets.load_iris()
        assert iris.data.shape == (150, 4)
        assert np.array_equal(np.unique(iris.target),  [0, 1, 2])
    
        clf = svm.LinearSVC()
        clf.fit(iris.data, iris.target)
        pred = clf.predict([[5.0, 3.6, 1.3, 0.25]])
        assert pred[0] == 0
    
        assert misc.face().shape is not None
    
        return x
    ```

    **说明：** 上述代码只是示例，目标是用到上文所说的所有的包。

2.  写完函数后，写一个简单的map。

    **说明：** 运行时要确保打开isolation，如果在project级别没有打开，也可在运行时打开一个可以设置全局的选项。

    ```
    from odps import options
    
    options.sql.settings = {'odps.isolation.session.enable': True}
    ```

    您也可以在execute方法上指定本次执行打开isolation。

    同样，您可以在全局通过options.df.libraries指定用到的包，也可以在execute时指定。这里需要指定所有的包，包括依赖。

3.  调用定义的函数。

    ```
    hints = {
        'odps.isolation.session.enable': True
    }
    libraries = ['python-dateutil.zip', 'pytz.zip', 'six.tar.gz', 'pandas.zip', 'scipy.zip', 'sklearn.zip']
    
    iris = o.get_table('pyodps_iris').to_df()
    
    print iris[:1].sepal_length.map(test).execute(hints=hints, libraries=libraries)
    ```


## 总结 {#section_pfh_crt_h2b .section}

对于要用到的第三方库及其依赖，如果已经上传，可以直接编写代码，并指定用到的libraries即可。否则，需要按照上述操作上传第三方库。

## PyODPS相关资源 { .section}

-   相关文档请参见[PyODPS使用指南](http://pyodps.readthedocs.io/zh_CN/latest/)。
-   相关代码请参见[aliyun-odps-python-sdk](https://github.com/aliyun/aliyun-odps-python-sdk)。

