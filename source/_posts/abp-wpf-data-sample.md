---
title: 在ABP项目中的WPF（三）：简单的数据操作示例
categories:
  - ABP
tags:
   - ABP
   - WPF
   - MySql
date: 2019-06-05 20:35:37
---

## 添加实体
在 `Acme.ContosoUniversity.Core` 项目中添加文件夹 `People`， 并在该文件夹中添加类：`Person`， 内容如以下所示：
```c#
using Abp.Domain.Entities;
using System.ComponentModel.DataAnnotations.Schema;

namespace Acme.ContosoUniversity.People
{
    [Table("AppPeople")]
    public class Person : Entity
    {
        public virtual string Name { get; set; }
    }
}
```

## 修改 数据上下文

打开 `Acme.ContosoUniversity.EntityFramework` 项目下的 `EntityFramework/ContosoUniversityDbContext.cs` 文件，在数据库上下文中添加 `IDbSet<Person>` 属性，如以下代码所示：

```cs
namespace Acme.ContosoUniversity.EntityFramework
{    
    [DbConfigurationType(typeof(MySqlEFConfiguration))]
    public class ContosoUniversityDbContext : AbpDbContext
    {
        // 添加这一行代码
        public virtual IDbSet<Person> People { get; set; }
        
        // 后续原有的代码省略
     }
 }
 ```
 
## 添加 Application 服务类
在 `Acme.ContosoUniversity.Application` 项目中添加 `People` 文件夹，在该文件夹中添加类： `IPersonAppService.cs`， 文件内容如以下所示：
```C#
using Abp.Application.Services;
using Acme.ContosoUniversity.People.Dto;
using System.Threading.Tasks;

namespace Acme.ContosoUniversity.People
{
    public interface IPersonAppService: IApplicationService
    {
        Task<GetPeopleOutput> GetAllPeopleAsync();
        Task AddNewPerson(AddNewPersonInput input);
    }
}
```

在 `People` 文件夹中添加类：`PersonAppService`，文件内容如以下代码所示：
```C#
using Abp.Domain.Repositories;
using Acme.ContosoUniversity.People.Dto;
using AutoMapper;
using Castle.Core.Logging;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace Acme.ContosoUniversity.People
{
    public class PersonAppService : ContosoUniversityAppServiceBase, IPersonAppService
    {
        private readonly IRepository<Person> _personRepository;
        public ILogger Logger { get; set; }

        public PersonAppService(IRepository<Person> personRepository)
        {
            _personRepository = personRepository;
            Logger = NullLogger.Instance;
        }


        public async Task AddNewPerson(AddNewPersonInput input)
        {
            Logger.Debug("Adding a new person:" + input.Name);
            await _personRepository.InsertAsync(new Person { Name = input.Name });
            // throw new NotImplementedException();
        }

        public async Task<GetPeopleOutput> GetAllPeopleAsync()
        {
            Logger.Debug("Getting all people");
            return new GetPeopleOutput
            {
                People = Mapper.Map<List<PersonDto>>(await _personRepository.GetAllListAsync())
            };
            // throw new NotImplementedException();
        }
    }
}

```

在 `People` 文件夹中添加子文件夹 `Dto`，在该文件夹中添加三个类：`PersonDto`， `GetPeopleOutput`， `AddNewPersonInput`， 文件内容分别如以下所示：


PersonDto.cs
```C#
using Abp.Application.Services.Dto;

namespace Acme.ContosoUniversity.People.Dto
{
    public class PersonDto : EntityDto
    {
        public string Name { get; set; }
    }
}

```


GetPeopleOutput.cs
```C#
using System.Collections.Generic;

namespace Acme.ContosoUniversity.People.Dto
{
    public class GetPeopleOutput
    {
        public List<PersonDto> People { get; set; }
    }
}

```

AddNewPersonInput.cs
```C#
using System.ComponentModel.DataAnnotations;

namespace Acme.ContosoUniversity.People.Dto
{
    public class AddNewPersonInput
    {
        [Required]
        public string Name { get; set; }
    }
}

```

## 创建 DTO 映射关系

修改 `Acme.ContosoUniversity.Application` 项目中的 `ContosoUniversityApplicationModule.cs`文件，内容如以下代码所示：
```C#
using System.Reflection;
using Abp.Modules;
using Acme.ContosoUniversity.People;
using Acme.ContosoUniversity.People.Dto;
using AutoMapper;

namespace Acme.ContosoUniversity
{
    [DependsOn(typeof(ContosoUniversityCoreModule))]
    public class ContosoUniversityApplicationModule : AbpModule
    {
        public override void Initialize()
        {
            IocManager.RegisterAssemblyByConvention(Assembly.GetExecutingAssembly());
            // 低版本的AutoMapper用这一行
            // Mapper.CreateMap<Person, PersonDto>();  
            
            // 高版本的AutoMapper 改成这样的写法
            Mapper.Initialize(x => x.CreateMap<Person, PersonDto>()); 
        }
    }
}

```
 
## 迁移和更新数据库
在 `Visual Studio 2019` 中菜单中选择：`工具` -> `NuGet包管理器` -> `程序包管理器控制台`，在打开的控制台中输入以下两个命令执行数据库迁移和更新：
```bash
add-migration init
update-database
```

## 在 WPF 中操作数据
打开 `Acme.ContosoUniversity.WPF` 项目中的 `MainWindow.xaml` 文件，分别添加 `ListBox控件`(命名：lstPeople)、 `TextBox控件`(命名：txtName) 和 `Button控件`(命名：btnAdd)，`Button控件` 添加单击事件。
{% asset_img untitled01.png MainWindow %}

MainWindow.xaml.cs 文件的代码如以下所示：

```C#
using Abp.Dependency;
using Acme.ContosoUniversity.People;
using System.Threading.Tasks;
using System.Windows;

namespace Acme.ContosoUniversity.WPF
{
    /// <summary>
    /// MainWindow.xaml 的交互逻辑
    /// </summary>
    public partial class MainWindow : Window, ISingletonDependency
    {
        private readonly IPersonAppService _personAppService;
        public MainWindow(IPersonAppService personAppService)
        {
            _personAppService = personAppService;
            InitializeComponent();            
        }

        // 添加按钮单击事件
        private async void BtnAdd_Click(object sender, RoutedEventArgs e)
        {
            try
            {
               await _personAppService.AddNewPerson(new People.Dto.AddNewPersonInput
                {
                    Name = txtName.Text
                }) ;
                await LoadAllPeopleAsync();
            }
            catch (System.Exception ex)
            {

                MessageBox.Show(ex.ToString());
            }
        }

        // 载入所有数据到 ListBox 控件
        private async Task LoadAllPeopleAsync()
        {
            lstPeople.Items.Clear();
            var result = await _personAppService.GetAllPeopleAsync();
            foreach (var person in result.People)
            {
                lstPeople.Items.Add(person.Name);
            }
        }
    }
}

```