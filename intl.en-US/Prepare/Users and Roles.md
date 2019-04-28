# Users and Roles {#concept_v1c_zzx_5db .concept}

[Project](../../../../reseller.en-US/User Guide/Definitions/Project.md) are the basic unit of multi-tenant architecture, data management, computing, and are also a subject of billing in MaxCompute. Once user becomes a project owner, all objects \(tables, instances, resources, UDFs\) in this project belongs to the user. If project owner does not grant permissions to others, no one else has permission to access this project.

**Note:** For RAM sub-accounts that already have roles in a MaxCompute or Dateworks project, revoke the sub-account role in the project and remove the sub-account in the project space before deleting the RAM sub-account. Otherwise, the sub-account will be residual in the project space, displayed as " p4\_*xxxxxxxxxxxxxxxxxxxx*" and cannot be removed in the project space \(does not affect normal function usage\) \). Sample:

```
Residual sub-account in the project space:
odps@ MaxCompute>list users;
p4_2652900xxxxxxxxxx
The residual sub-account could not be deleted in space:
odps@ MaxCompute_DOC>remove user p4_2652900xxxxxxxxxx;
Confirm to "remove user p4_2652900xxxxxxxxxx
;" (yes/no)? yes
FAILED: lack of account provider
You can still see the RAM sub-account on the dataworks project member management page at this time.
The right approach: revoke the role of the sub-account first:
odps@ MaxCompute>revoke role_project_security, role_project_admin, role_project_dev, role_project_pe, role_project_deploy, role_project_guest from RAM$MainCount:hanmeimei;
OK
Then remove the sub-account in project space:
odps@ MaxCompute>remove user RAM$MainCount:hanmeimei;
Then the RAM sub-account can be deleted normally.
				
```

-   If you want to add and authorize users using DataWorks, see [Add users and roles](../../../../reseller.en-US/Preparation/Administrator Operations/Add users and roles.md#).
-   If you use the MaxCompute security management commands for [user management](../../../../reseller.en-US//User management.md), see User Management for how to create, authorize, and delete users \(including RAM sub-accounts\).
-   If you use the MaxCompute security management commands for role management, see [Role Management](../../../../reseller.en-US//Role management.md) for how to create, authorize, and delete roles.
-   For more information about authorization and permission check, see [Authorization](../../../../reseller.en-US//Authorization.md) and [Permission Check](../../../../reseller.en-US//Permission check.md).

