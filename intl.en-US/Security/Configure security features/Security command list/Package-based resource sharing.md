# Package-based resource sharing {#concept_kyz_531_wdb .concept}

This article gives you a description of resource sharing statements based on Package.

## Share resources {#section_ofl_bk1_wdb .section}

|Statement|Description|
|:--------|:----------|
|Create package <pkgname\> `<pkgName>`|Create a package.|
|Delete package <pkgname\> `<pkgName>`|Delete a package.|
|add `<objType>``<objName>` to package `<pkgName>` \[with privileges privs\]|Add resources to be shared to a package.|
|remove `<objType>``<objName>` from package `<pkgName>`|Remove shared resources from a package.|
|allow project `<prjName>` to install package `<pkgName>` \[using label `<num>`\]|Allow a project to use a user package.|
|disallow project `<prjName>` to install package `<pkgName>`|Disallow a project from using a user package.|

## Use Resources {#section_sg2_dk1_wdb .section}

|Statement|Description:|
|:--------|:-----------|
|Install package <pkgname\> `<pkgName>`|Install a package.|
|uninstall package `<pkgName>`|Uninstall a package.|

## View a package {#section_fyp_2k1_wdb .section}

|Statement|Description:|
|:--------|:-----------|
|show packages|List all created and installed packages.|
|describe package `<pkgName>`|View details of a package.|

**Note:** 

If you execute a production plan authorization related request in DataWorks：

1.  Project owner is executed by temporary query and cannot be submitted to the production environment for execution.Because the production environment is executed by the production account, which has no authorized authority.
2.  Add the use<production project\>; statement before the query and submit it with the command. Because DataWorks data development defaults the current project is the development project ending in \_dev. When executing authorization commands from the command line, ask project owner to execute them first:

    ```
    use project_name;--进入对应的project
    ```


