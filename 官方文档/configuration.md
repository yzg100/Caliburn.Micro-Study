---
layout: page
title: Basic Configuration, Actions and Conventions
---

Open Visual Studio and create a new Silverlight 4 Application called “Caliburn.Micro.Hello”. You don’t need a web site or test project. Add a reference to System.Windows.Interactivity.dll and Caliburn.Micro.dll. You can find them both in the \src\Caliburn.Micro.Silverlight\Bin\Release (or Debug) folder. Delete “MainPage.xaml” and clean up your “App.xaml.cs” so that it looks like this:

---
><font color="#63aebb" face="微软雅黑">打开Visual Studio，创建一个名为 “Caliburn.Micro.Hello” 的 Silverlight 4 新应用程序。你不需要 web 站点或测试项目。添加对 System.Windows.Interactivity.dll 的引用 和 Caliburn.Micro.dll。你可以在\src\Caliburn.Micro中找到它们。Silverlight\Bin\Release(或Debug)文件夹。删除 "MainPage.xaml",然后清理你的"App.xaml.cs" 它看起来是这样的:</font>
 
``` csharp
namespace Caliburn.Micro.Hello {
    using System.Windows;

    public partial class App : Application {
        public App() {
            InitializeComponent();
        }
    }
}
```

Since Caliburn.Micro prefers a View-Model-First approach, let’s start there. Create your first VM and call it ShellViewModel. Use the following code for the implementation:

---
><font color="#63aebb" face="微软雅黑">Caliburn.Micro 的 View-Model-First 方法，我们从这里开始。创建第一个 VM 并将其称为 ShellViewModel。使用以下代码实现:</font>

``` csharp
namespace Caliburn.Micro.Hello {
    using System.Windows;

    public class ShellViewModel : PropertyChangedBase {
        string name;

        public string Name {
            get { return name; }
            set {
                name = value;
                NotifyOfPropertyChange(() => Name);
                NotifyOfPropertyChange(() => CanSayHello);
            }
        }

        public bool CanSayHello {
            get { return !string.IsNullOrWhiteSpace(Name); }
        }

        public void SayHello() {
            MessageBox.Show(string.Format("Hello {0}!", Name)); //Don't do this in real life :)
        }
    }
}
```

Notice that the ShellViewModel inherits from PropertyChangedBase. This is a base class that implements the infrastructure for property change notification and automatically performs UI thread marshalling. It will come in handy :)

Now that we have our VM, let’s create the bootstrapper that will configure the framework and tell it what to do. Create a new class named HelloBootstrapper. You can use this tiny bit of code:

---
><font color="#63aebb" face="微软雅黑">注意，ShellViewModel 继承自 PropertyChangedBase ，这个基类实现了属性更改通知的基础，并自动执行UI线程编组。
现在我们有了 VM，让我们创建一个引导程序来配置框架并告诉它要做什么。创建一个名为 HelloBootstrapper 的新类。你可以使用这一小段代码:</font>

``` csharp
namespace Caliburn.Micro.Hello {
    public class HelloBootstrapper : BootstrapperBase {
        public HelloBootstrapper() {
            Initialize();
        }

        protected override void OnStartup(object sender, StartupEventArgs e) {
            DisplayRootViewFor<ShellViewModel>();
        }
    }
}
```

The Bootsrapper allows you to specify the type of “root view model” via the generic method. The “root view model” is a VM that Caliburn.Micro will instantiate and use to show your application. Next, we need to place the HelloBootstrapper somewhere where it will be run at startup. To do that, change your App.xaml to match this:

---
><font color="#63aebb" face="微软雅黑">Bootsrapper 允许你通过通用方法指定 “根视图模型”的类型。“根视图模型” 是 Caliburn.Micro 将实例化并用于显示你的应用程序的VM。接下来，我们需要将 HelloBootstrapper 放在启动时运行它的地方。要做到这一点，请更改 App.xaml 以匹配以下内容:</font>

#### Silverlight / Windows Phone
``` xml
<Application xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:local="clr-namespace:Caliburn.Micro.Hello"
             x:Class="Caliburn.Micro.Hello.App">
    <Application.Resources>
        <local:HelloBootstrapper x:Key="bootstrapper" />
    </Application.Resources>
</Application>
```

#### WPF
``` xml
<Application xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:local="clr-namespace:Caliburn.Micro.Hello"
             x:Class="Caliburn.Micro.Hello.App">
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <ResourceDictionary>
                    <local:HelloBootstrapper x:Key="bootstrapper" />
                </ResourceDictionary>
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>
</Application>
```

All we have to do is place a Caliburn.Micro bootstrapper in the Application.Resources and it will do the rest of the work. Now, run the application. You should see something like this:

---
><font color="#63aebb" face="微软雅黑">我们要做的就是在 Application.Resources 中放置 Caliburn.Micro 引导程序，它会做剩下的工作。现在，运行应用程序，你应该看到的是这样:</font>

![View not found](/public/images/documentation/view-not-found.jpg)

Caliburn.Micro creates the ShellViewModel, but doesn’t know how to render it. So, let’s create a view. Create a new Silverlight User Control named ShellView. Use the following xaml:

---
><font color="#63aebb" face="微软雅黑">Caliburn.Micro 创建了 ShellViewModel，但不知道如何渲染。我们来创建一个视图。创建一个名为ShellView的Silverlight用户控件。使用以下xaml:</font>

``` xml
<UserControl x:Class="Caliburn.Micro.Hello.ShellView"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <StackPanel>
        <TextBox x:Name="Name" />
        <Button x:Name="SayHello"
                Content="Click Me" />
    </StackPanel>
</UserControl>
```

再次运行应用程序。现在应该看到UI了:

![View found](/public/images/documentation/view-found.jpg)

在文本框中键入内容将启用该按钮，单击它将显示一条消息:

![View with data](/public/images/documentation/view-with-data.jpg)

Caliburn.Micro uses a simple naming convention to locate Views for ViewModels. Essentially, it takes the FullName and removes “Model” from it. So, given MyApp.ViewModels.MyViewModel, it would look for MyApp.Views.MyView. Looking at the View and ViewModel side-by-side, you can see that the TextBox with x:Name=”Name” is bound to the “Name” property on the VM. You can also see that the Button with x:Name=”SayHello” is bound to the method with the same name on the VM. The “CanSayHello” property is guarding access to the “SayHello” action by disabling the Button. These are the basics of Caliburn.Micro’s ActionMessage and Conventions functionality. There’s much more to show. But, next time I want to show how we can integrate an IoC container such as MEF.

---
><font color="#63aebb" face="微软雅黑">Caliburn.Micro 使用简单的命名约定来查找 ViewModel 的 Views。它获取全名并从其中删除“Model”。所以 MyApp.ViewModels.MyViewModel ，它会查找 MyApp.Views.MyView 。查看 View 和 ViewModel，你可以看到带有 x:Name="Name" 的文本框绑定到VM上的 "Name"属性。你还可以看到，带有 x:Name="SayHello" 的按钮绑定到 VM 上同名的方法。“CanSayHello” 属性通过禁用按钮来保护对 “SayHello” 操作的访问。这是 Caliburn.Micro ActionMessage 和 Conventions 功能的基础。还有很多东西要展示，下次我想展示如何集成一个 IoC 容器，比如 MEF。</font>

[目录](./index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[Customizing the Bootstrapper - 自定义启动加载器](./bootstrapper.md)