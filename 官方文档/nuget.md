---
layout: page
title: Nuget Package Installation
---

[NuGet][nuget] 是一个Visual Studio扩展，它使得在使用.NET Framework的Visual Studio项目中添加、删除和更新库和工具变得容易。Caliburn.Micro 支持 NuGet 包管理器。


### Installing the packages - 安装包

With the latest version of Nuget installed, open the Package Manager Console and type:

```
PM> Install-Package Caliburn.Micro.Start
```

### After installation

#### 清理你的App.xaml.cs文件，它应该是这样的:


``` csharp
namespace YourNamespace
{
    using System.Windows;

    public partial class App : Application
    {
        public App()
        {
            InitializeComponent();
        }
    }
}
```

#### 将 AppBoostrapper 添加到 App.xaml 资源定义部分.

##### Silverlight

``` xml
<Application xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:local="clr-namespace:YourNamespace"
             x:Class="YourNamespace.App">
    <Application.Resources>
        <local:AppBootstrapper x:Key="bootstrapper" />
    </Application.Resources>
</Application>
```

**提示**: 你不再需要默认的 MainPage.xaml.


##### WPF

``` xml
<Application xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:local="clr-namespace:YourNamespace"
             x:Class="YourNamespace.App">
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <ResourceDictionary>
                    <local:AppBootstrapper x:Key="bootstrapper" />
                </ResourceDictionary>
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>
</Application>
```

**提示**: 确保删除 StartupUri 值。Caliburn.Micro 将为你处理主窗口的创建。因此，你不再需要主窗口 MainWindow.xaml.

##### Windows Phone

``` xml
<Application xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:local="clr-namespace:YourNamespace"
             x:Class="YourNamespace.App">
    <Application.Resources>
        <local:AppBootstrapper x:Key="bootstrapper" />
    </Application.Resources>
</Application>
```

**Note**: If you move your MainPage.xaml into a Views folder, don't forget to update your WMAppManfiest.xml to point to the new URI, as follows:


**提示**:如果你移动你的 MainPage.xaml 到一个视图文件夹，不要忘记更新你的 WMAppManfiest.xml 指向新的URI，如下所示:

``` xml
<Tasks>
   <DefaultTask Name="_default" NavigationPage="/Views/MainPage.xaml" />
</Tasks>
```

##### WinRT
For WinRT, the process of getting started is unfortunately quite different from the other platforms, due to significant design differences in the Windows Xaml APIs. For detailed instructions please see [Working with WinRT](./windows-runtime.md).

[nuget]: http://www.nuget.org/

[目录](./index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[Cheat Sheet - 备忘录](./cheat-sheet.md)
