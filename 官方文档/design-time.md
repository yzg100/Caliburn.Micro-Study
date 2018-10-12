---
layout: page
title: Design Time support
---

Enabling Caliburn.Micro inside the Visual Studio designer (or Blend) is quite easy.

You have to set a Desinger-DataContext and tell CM to enable its magic in your view XAML:

---
><font color="#63aebb" face="微软雅黑">在Visual Studio设计器（或Blend）中启用Caliburn.Micro非常简单。

>你必须设置Desinger-DataContext并告诉CM在你的视图XAML中启用它：</font>

``` csharp
<Window 
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    xmlns:vm="clr-namespace:CaliburnDesignTimeData.ViewModels"
    xmlns:cal="clr-namespace:Caliburn.Micro;assembly=Caliburn.Micro.Platform"
    mc:Ignorable="d" 
    d:DataContext="{d:DesignInstance Type=vm:MainPageViewModel, IsDesignTimeCreatable=True}"
    cal:Bind.AtDesignTime="True">
```

For this to work, the ViewModel must have a default constructor. If this isn't suitable, you can also use a ViewModelLocator for your design-time ViewModel creation.

---
><font color="#63aebb" face="微软雅黑">为此，ViewModel必须具有默认构造函数。如果这不合适，你还可以使用ViewModelLocator进行设计时ViewModel创建。</font>

### Issues - 问题

It seems that VS2010 has an issue in the WP7 designer and an exception in CM ConventionManager is thrown. You can workaround this by overriding ApplyValidation in your bootstrapper:

---
><font color="#63aebb" face="微软雅黑">VS2010 在 WP7 设计器中存在问题，并且抛出了CM ConventionManager 中的异常。你可以通过在引导程序中覆盖 ApplyValidation 来解决此问题：</font>

``` csharp
ConventionManager.ApplyValidation = (binding, viewModelType, property) => {
        if (typeof(INotifyDataErrorInfo).IsAssignableFrom(viewModelType)) {
            binding.ValidatesOnNotifyDataErrors = true;
            binding.ValidatesOnExceptions = true;
        }
    };
```

[目录](./index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[Async (Task Support) - 异步(任务支持)](./async.md)