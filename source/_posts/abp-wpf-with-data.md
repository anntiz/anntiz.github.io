---
title: 在ABP项目中的WPF（二）：使用 MySql 数据库
categories:
  - ABP
tags:
  - ABP
  - WPF
  - MySql
date: 2019-06-05 19:47:28
---

## 添加 MySql 的 NuGet包
在项目的解决方案资源管理上，`鼠标右键` -> `管理解决方案的NuGet程序包`

{% asset_img untitled01.png Manage NuGet package %}

在 `浏览` 选择卡中搜索 `mysql`，选择 `MySql.Data.Entity`，版本选择 `6.8.8` (据说新版本会的有问题)， `Acme.ContosoUniversity.EntityFramework` 和 `Acme.ContosoUniversity.WPF` 项目打勾：

{% asset_img untitled02.png add NuGet package %}

## 修改迁移配置文件

打开 `Acme.ContosoUniversity.EntityFramework` 项目下 `Migrations/Configuration.cs` 文件，修改 `Configuration` 方法为如下代码：
```c#
        public Configuration()
        {
            AutomaticMigrationsEnabled = false;
            ContextKey = "ContosoUniversity";
            // 添加这一行代码
            SetSqlGenerator("MySql.Data.MySqlClient", new MySql.Data.Entity.MySqlMigrationSqlGenerator());
        }
```

## 修改数据上下文
打开 `Acme.ContosoUniversity.EntityFramework` 项目下的 `EntityFramework/ContosoUniversityDbContext.cs` 文件，修改前面部分如以下所示：

```c#
using System.Data.Common;
using System.Data.Entity;
using Abp.EntityFramework;
using Acme.ContosoUniversity.People;
using MySql.Data.Entity;   // 添加这一行引用

namespace Acme.ContosoUniversity.EntityFramework
{
    // 添加这一行代码
    [DbConfigurationType(typeof(MySqlEFConfiguration))]
    public class ContosoUniversityDbContext : AbpDbContext
    {
```

## 添加 MySql 数据库连接字符串

打开 `Acme.ContosoUniversity.WPF` 项目下的 `App.config` 文件，添加 MySql 连接字符串，内容类似以下代码所示：
```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <configSections>
    <!-- For more information on Entity Framework configuration, visit http://go.microsoft.com/fwlink/?LinkID=237468 -->
    <section name="entityFramework" type="System.Data.Entity.Internal.ConfigFile.EntityFrameworkSection, EntityFramework, Version=6.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" requirePermission="false" />
  </configSections>
  
  <!--添加下行的MySql连接字符串-->
  <connectionStrings>
    <add name="Default" connectionString="Server=127.0.0.1;Database=ContosoUniversity0602;userid=root;pwd=x5;port=3306;sslmode=none;" providerName="MySql.Data.MySqlClient" />
  </connectionStrings>
```

