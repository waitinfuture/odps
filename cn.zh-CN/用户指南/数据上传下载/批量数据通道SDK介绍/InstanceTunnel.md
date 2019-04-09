# InstanceTunnel {#concept_kn3_cxy_3hb .concept}

InstanceTunnel提供了使用Tunnel来下载SQL查询结果的功能。您可以通过它直接获取查询结果，下载Maxcompute Instance的数据。

InstanceTunnel接口定义如下，更多信息：请查看[Java-sdk-doc](http://repo.aliyun.com/java-sdk-doc/)。

```
public class InstanceTunnel{
 public DownloadSession createDownloadSession(String projectName, String instanceID);
 public DownloadSession createDownloadSession(String projectName, String instanceID, boolean limitEnabled);
 public DownloadSession getDownloadSession(String projectName, String id);
 }
```

参数说明：

-   projectName：Project名
-   instanceID：Instance ID

InstanceTunnel约束：虽然InstanceTunnel提供了非常方便的获取Instance执行结果的方式，但是为了保护用户数据安全，此功能也受到了权限约束。

-   使用InstanceTunnel下载数据时，若数据条数不超过1W，则只要对该Instance有Read权限的用户即可使用。此行为与使用Restful API获取查询数据的行为一致。
-   使用InstanceTunnel下载数据时，若数据条数超过1W，则需要对instance SQL查询语句中涉及到的所有源表进行权限检查，用户需要具有这些表的Read权限才可使用。

