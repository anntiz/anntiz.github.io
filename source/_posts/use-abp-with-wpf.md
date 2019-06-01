---
title: 在ABP项目中使用WPF
categories:
  - ABP
tags:
  - ABP
  - WPF
date: 2019-06-01 16:11:57
---

## 创建 ABP 项目

在 [ASP.NET Boilerplate](https://aspnetboilerplate.com/Templates) 网站上创建一个 `ASP.NET MVC 5.x `的项目，如下图所示：

{% asset_img createsolution.PNG Create ASP.NET MVC 5.X Solution %}

将下载后的解决方案解压缩后使用 `Visual Studio 2019` 打开，并 `还原NuGet包`。

## 在解决方案中添加WPF项目

在 `Acme.ContosoUniversity.Web` 项目上`鼠标右键` -> `属性` ，查看项目的 `目标框架` 的版本

{% asset_img targetframeworks.PNG Add New Project %}

在 `Visual Studio 2019` 的解决方案资源管理器中选中解决方案，`鼠标右键` -> `添加` -> `新建项目`：

{% asset_img newproject.png Add New Project %}

在 `添加新项目` 窗口中选择 `WPF应用(.NET Framework)` 模板，然后 `下一步`：

{% asset_img addnewproject.png Add New Project %}

在 `配置新项目` 窗口中输入 WPF 项目的名称为：`Acme.ContosoUniversity.WPF`，框架为：`.NET Framework 4.6.1` (与前面查看的目标框架版本相同)

{% asset_img configurenewproject.PNG Add New Project %}

## 为WPF项目添加NuGet包

在 `Acme.ContosoUniversity.WPF` 项目上 `鼠标右键` -> `管理NuGet程序包`， 在打开的 `NuGet包管理器` 中搜索并添加以下包：

1、Abp

{% asset_img add_abp.PNG Add ABP %}
 
2、Abp.Castle.Log4Net

{% asset_img add_abp.castle.log4net.PNG Add Abp.Castle.Log4Net %}

3、EntityFramework 6.2.0

{% asset_img add_entityframework.PNG Add EntityFramework 6.2.0 %}

4、Microsoft.Bcl

{% asset_img add_microsoft.bcl.PNG Add Microsoft.Bcl %}

5、Microsoft.Bcl.Async

{% asset_img add_microsoft.bcl.async.PNG Add Microsoft.Bcl.Async %}

## 为WPF项目添加引用

在 `Acme.ContosoUniversity.WPF` 项目的 `引用` 上 `鼠标右键` -> `添加引用`

{% asset_img add_reference.png Add Reference %}

在 `引用管理器` 的左窗格选择 `项目`, 在右窗格中选中 `Application`、 `Core`、 `EntityFramework` 三个项目，然后 `确定`

{% asset_img reference_manager.png  Reference Manager %}

## 添加 log4net.config 配置文件

在 `Acme.ContosoUniversity.WPF` 项目上 `鼠标右键` -> `添加` -> `新建项`，模板选择 `应用程序配置文件`，输入文件名为：`log4net.config`

{% asset_img log4net.config.png  Add log4net.config %}

输入 `log4net.config` 文件的内容如以下代码所示：
```xml
<?xml version="1.0" encoding="utf-8" ?>
<log4net>
  <appender name="RollingFileAppender" type="log4net.Appender.RollingFileAppender" >
    <file value="Logs/Logs.txt" />
    <appendToFile value="true" />
    <rollingStyle value="Size" />
    <maxSizeRollBackups value="10" />
    <maximumFileSize value="10000KB" />
    <staticLogFileName value="true" />
    <layout type="log4net.Layout.PatternLayout">
      <conversionPattern value="%-5level %date [%-5.5thread] %-40.40logger - %message%newline" />
    </layout>
  </appender>
  <root>
    <appender-ref ref="RollingFileAppender" />
    <level value="DEBUG" />
  </root>
</log4net>
```

## 添加WPF项目模板类

在 `Acme.ContosoUniversity.WPF` 项目上 `鼠标右键` -> `添加` -> `类`，输入类文件名为：`ContosoUniversityWPFModule.cs`，输入如以下所示代码：
```c#
using Abp.Modules;
using System.Reflection;

namespace Acme.ContosoUniversity.WPF
{
    [DependsOn(typeof(ContosoUniversityDataModule), typeof(ContosoUniversityApplicationModule))]
    public class ContosoUniversityWPFModule : AbpModule
    {
        public override void Initialize()
        {
            IocManager.RegisterAssemblyByConvention(Assembly.GetExecutingAssembly());
            // base.Initialize();
        }
    }
}

```

## 编辑 App.xaml.cs 文件

打开文件 `App.xaml.cs`，修改为如以下所示代码：
```C#
using Abp;
using Abp.Castle.Logging.Log4Net;
using Castle.Facilities.Logging;
using System.Windows;

namespace Acme.ContosoUniversity.WPF
{
    /// <summary>
    /// App.xaml 的交互逻辑
    /// </summary>
    public partial class App : Application
    {
        private readonly AbpBootstrapper _bootstrapper;
        private MainWindow _mainWindow;

        public App()
        {
            _bootstrapper = AbpBootstrapper.Create<ContosoUniversityWPFModule>();
            _bootstrapper.IocManager.IocContainer.AddFacility<LoggingFacility>(
                f => f.UseAbpLog4Net().WithConfig("log4net.config"));
        }

        protected override void OnStartup(StartupEventArgs e)
        {
            _bootstrapper.Initialize();
            _mainWindow = _bootstrapper.IocManager.Resolve<MainWindow>();
            _mainWindow.Show();
            // base.OnStartup(e);
        }

        protected override void OnExit(ExitEventArgs e)
        {
            _bootstrapper.IocManager.Release(_mainWindow);
            _bootstrapper.Dispose();
           // base.OnExit(e);
        }
    }
}

```

## 编辑 App.xaml 文件

打开 `App.xaml` 文件，其内容为以下所示代码：
```xml
<Application x:Class="Acme.ContosoUniversity.WPF.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:local="clr-namespace:Acme.ContosoUniversity.WPF"
             StartupUri="MainWindow.xaml">
    <Application.Resources>
         
    </Application.Resources>
</Application>

```

将第5行的 `StartupUri="MainWindow.xaml"` 删除，否则运行程序时会打开两个 MainWindow 窗口。修改后代码如以下所示:
```xml
<Application x:Class="Acme.ContosoUniversity.WPF.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:local="clr-namespace:Acme.ContosoUniversity.WPF">
    <Application.Resources>
         
    </Application.Resources>
</Application>

```

## 修改 MainWindow.xaml.cs 文件

打开 `MainWindow.xaml.cs` 文件，打到以下行：

```c#
public partial class MainWindow : Window
```

将其修改为如下所示代码，并添加相应的名字空间引用：
```c#
public partial class MainWindow : Window, ISingletonDependency
```

修改后的完整代码，如以下所示：
```c#
using Abp.Dependency;
using System.Windows;

namespace Acme.ContosoUniversity.WPF
{
    /// <summary>
    /// MainWindow.xaml 的交互逻辑
    /// </summary>
    public partial class MainWindow : Window, ISingletonDependency
    {
        public MainWindow()
        {
            InitializeComponent();
        }
    }
}

```

## 运行WPF程序

将 `Acme.ContosoUniversity.WPF` 设为启动项目，并运行。此时出现 `未经处理的异常`，显示找不到文件 `Debug\log4net.config`。


解决办法：将 `log4net.config` 复制到 Debug 文件夹后，再次运行即可。