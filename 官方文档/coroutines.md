---
layout: page
title: IResult and Coroutines
---

Previously, I mentioned that there was one more compelling feature of the Actions concept called Coroutines. If you haven’t heard that term before, here’s what wikipedia has to say:

---
><font color="#63aebb" face="微软雅黑">之前，我提到过 Action 概念中有一个更引人注目的特性，叫做协同程序。如果你以前没听说过这个词，维基百科是这样说的:</font>

> In computer science, coroutines are program components that generalize subroutines to allow multiple entry points for suspending and resuming execution at certain locations. Coroutines are well-suited for implementing more familiar program components such as cooperative tasks, iterators,infinite lists and pipes.

---
><font color="#63aebb" face="微软雅黑">在计算机科学中，协程是程序组件，它泛化子程序，允许多个入口点在特定位置暂停和恢复执行。协同程序非常适合实现更熟悉的程序组件，比如协作任务、迭代器、无限列表和管道。</font>

Here’s one way you can thing about it: Imagine being able to execute a method, then pause it’s execution on some statement, go do something else, then come back and resume execution where you left off. This technique is extremely powerful in task-based programming, especially when those tasks need to run asynchronously. For example, let’s say we have a ViewModel that needs to call a web service asynchronously, then it needs to take the results of that, do some work on it and call another web service asynchronously. Finally, it must then display the result in a modal dialog and respond to the user’s dialog selection with another asynchronous task. Accomplishing this with the standard event-driven async model is not a pleasant experience. However, this is a simple task to accomplish by using coroutines. The problem…C# doesn’t implement coroutines natively. Fortunately, we can (sort of) build them on top of iterators.

---
><font color="#63aebb" face="微软雅黑">这里有一种方法:想象一下执行一个方法，然后暂停它在某个语句上的执行，执行其他操作，然后返回并恢复你停止的执行。这种技术在基于任务的编程中非常强大，特别是当这些任务需要异步运行时。假设我们有一个 ViewModel，它需要异步调用 Web 服务，然后它需要获取它的结果，对它做一些工作，然后异步调用另一个 Web 服务。然后在模态对话框中显示结果，并用另一个异步任务响应用户的对话框选择。用标准事件驱动异步模型完成这一过程不是一个令人愉快的体验。这是一个通过使用协同程序来完成的简单任务。问题是，C# 本身并没有实现协程。幸运的是，我们可以(某种程度上)在迭代器之上构建它们。</font>


There are two things necessary to take advantage of this feature in Caliburn.Micro: First, implement the IResult interface on some class, representing the task you wish to execute; Second, yield instances of IResult from an Action2. Let’s make this more concrete. Say we had a Silverlight application where we wanted to dynamically download and show screens not part of the main package. First we would probably want to show a “Loading” indicator, then asynchronously download the external package, next hide the “Loading” indicator and finally navigate to a particular screen inside the dynamic module. Here’s what the code would look like if your first screen wanted to use coroutines to navigate to a dynamically loaded second screen:

---
><font color="#63aebb" face="微软雅黑">要利用 Caliburn.Micro 中的这个特性，有两件事是必要的:首先，在某些类上实现IResult接口，表示你希望执行的任务;其次，生成来自 Action2 的 IResult 实例。让我们更具体一点，假设我们有一个 Silverlight 应用程序，我们想动态下载并显示屏幕，而不是主包的一部分。首先，我们可能希望显示一个 “加载” 提示，然后异步下载外部包，接下来隐藏 “加载” 提示，最后导航到动态模块中的特定屏幕。如果你的第一个屏幕想要使用协程导航到动态加载的第二个屏幕，那么下面是代码的样子:</font>

``` csharp
using System.Collections.Generic;
using System.ComponentModel.Composition;

[Export(typeof(ScreenOneViewModel))]
public class ScreenOneViewModel
{
    public IEnumerable<IResult> GoForward()
    {
        yield return Loader.Show("Downloading...");
        yield return new LoadCatalog("Caliburn.Micro.Coroutines.External.xap");
        yield return Loader.Hide();
        yield return new ShowScreen("ExternalScreen");
    }
}
```

First, notice that the Action “GoForward” has a return type of IEnumerable<IResult>. This is critical for using coroutines. The body of the method has four yield statements. Each of these yields is returning an instance of IResult. The first is a result to show the “Downloading” indicator, the second to download the xap asynchronously, the third to hide the “Downloading” message and the fourth to show a new screen from the downloaded xap. After each yield statement, the compiler will “pause” the execution of this method until that particular task completes. The first, third and fourth tasks are synchronous, while the second is asynchronous. But the yield syntax allows you to write all the code in a sequential fashion, preserving the original workflow as a much more readable and declarative structure. To understand a bit more how this works, have a look at the IResult interface:

---
><font color="#63aebb" face="微软雅黑">首先，注意动作“GoForward”有一个返回类型为IEnumerable。这是使用协同程序的关键。该方法的主体有四个yield语句。这些结果中的每一个都返回一个IResult实例。第一个结果显示“下载”指示符，第二个结果显示异步下载xap，第三个结果隐藏“下载”消息，第四个结果显示下载xap的新屏幕。在每个yield语句之后，编译器将“暂停”此方法的执行，直到该特定任务完成为止。第一、第三和第四任务是同步的，而第二任务是异步的。但是yield语法允许你以顺序的方式编写所有代码，并将原始工作流保留为更具可读性和声明性的结构。要进一步了解它的工作原理，请查看IResult接口:</font>

``` csharp
public interface IResult
{
    void Execute(CoroutineExecutionContext context);
    event EventHandler<ResultCompletionEventArgs> Completed;
}
```

It’s a fairly simple interface to implement. Simply write your code in the “Execute” method and be sure to raise the “Completed” event when you are done, whether it be a synchronous or an asynchronous task. Because coroutines occur inside of an Action, we provide you with an ActionExecutionContext useful in building UI-related IResult implementations. This allows the ViewModel a way to declaratively state its intentions in controlling the view without having any reference to a View or the need for interaction-based unit testing. Here’s what the ActionExecutionContext looks like:

---
><font color="#63aebb" face="微软雅黑">这是一个简单的接口实现。只需在 “Execute” 方法中编写代码，并确保在完成时引发 “Completed” 事件，无论是同步任务还是异步任务。因为协同作用发生在操作内部，所以我们为你提供了一个 ActionExecutionContext，它在构建与 UI 相关的 IResult 实现时非常有用。这使得 ViewModel 可以声明其控制视图的意图，而不需要引用视图或基于交互的单元测试。下面是 ActionExecutionContext 的样子:</font>

``` csharp
public class ActionExecutionContext
{
    public ActionMessage Message;
    public FrameworkElement Source;
    public object EventArgs;
    public object Target;
    public DependencyObject View;
    public MethodInfo Method;
    public Func<bool> CanExecute;
    public object this[string key];
}
```

下面是对这些属性的解释:

<dl>
	<dt>Message</dt>
	<dd>The original ActionMessage that caused the invocation of this IResult.<br>
    <font color="#63aebb" face="微软雅黑">引发 IResult 调用的原始 ActionMessage</font></dd>
	<dt>Source</dt>
	<dd>The FrameworkElement that triggered the execution of the Action.<br>
    <font color="#63aebb" face="微软雅黑">触发执行 FrameworkElement 的活动</font></dd>
	<dt>EventArgs</dt>
	<dd>Any event arguments associated with the trigger of the Action.<br>
    <font color="#63aebb" face="微软雅黑">与活动的触发器关联的事件参数</font></dd>
	<dt>Target</dt>
	<dd>The class instance on which the actual Action method exists.<br>
    <font color="#63aebb" face="微软雅黑">实际活动方法所在的类实例</font></dd>
	<dt>View</dt>
	<dd>The view associated with the Target.<br>
    <font color="#63aebb" face="微软雅黑">与目标相关联的视图</font></dd>
	<dt>Method</dt>
	<dd>The MethodInfo specifying which method to invoke on the Target instance.<br>
    <font color="#63aebb" face="微软雅黑">MethodInfo指定在目标实例上调用哪个方法</font></dd>
	<dt>CanExecute</dt>
	<dd>A function that returns true if the Action can be invoked, false otherwise.<br>
    <font color="#63aebb" face="微软雅黑">如果可以调用操作，返回true的函数，否则返回false</font></dd>
	<dt>Key Index</dt>
	<dd>A place to store/retrieve any additional metadata which may be used by extensions to the framework.<br>
    <font color="#63aebb" face="微软雅黑">存储/检索框架扩展可能使用的任何额外元数据的地方</font></dd>
</dl>

Bearing that in mind, I wrote a naive Loader IResult that searches the VisualTree looking for the first instance of a BusyIndicator to use to display a loading message. Here’s the implementation:

---
><font color="#63aebb" face="微软雅黑">记住这一点，我编写了一个简单的加载器 IResult，它搜索 VisualTree 查找用于显示加载消息的 BusyIndicator 的第实例。这是实现:</font>

``` csharp
using System;
using System.Windows;
using System.Windows.Controls;

public class Loader : IResult
{
    readonly string message;
    readonly bool hide;

    public Loader(string message)
    {
        this.message = message;
    }

    public Loader(bool hide)
    {
        this.hide = hide;
    }

    public void Execute(CoroutineExecutionContext context)
    {
        var view = context.View as FrameworkElement;
        while(view != null)
        {
            var busyIndicator = view as BusyIndicator;
            if(busyIndicator != null)
            {
                if(!string.IsNullOrEmpty(message))
                    busyIndicator.BusyContent = message;
                busyIndicator.IsBusy = !hide;
                break;
            }

            view = view.Parent as FrameworkElement;
        }

        Completed(this, new ResultCompletionEventArgs());
    }

    public event EventHandler<ResultCompletionEventArgs> Completed = delegate { };

    public static IResult Show(string message = null)
    {
        return new Loader(message);
    }

    public static IResult Hide()
    {
        return new Loader(true);
    }
}
```

See how I took advantage of context.View? This opens up a lot of possibilities while maintaining separation between the view and the view model. Just to list a few interesting things you could do with IResult implementations: show a message box, show a VM-based modal dialog, show a VM-based Popup at the user’s mouse position, play an animation, show File Save/Load dialogs, place focus on a particular UI element based on VM properties rather than controls, etc. Of course, one of the biggest opportunities is calling web services. Let’s look at how you might do that, but by using a slightly different scenario, dynamically downloading a xap:

---
><font color="#63aebb" face="微软雅黑">看到我如何利用 context.View 了吗？在保持视图和视图模型之间的分离的同时，这展现了许多可能性。只列出一些有趣的事情你可以做 IResult 实现:显示一个消息框,显示基于 VM 模式对话框，显示一个基于 VM 的弹出用户的鼠标位置，播放动画，显示文件保存/加载对话框，专注于一个特定的UI元素基于VM的属性而不是控件，等等。当然，最主要的是调用 web 服务。让我们看看如何做到这一点，通过不同的场景，动态下载xap:</font>

``` csharp
using System;
using System.Collections.Generic;
using System.ComponentModel.Composition;
using System.ComponentModel.Composition.Hosting;
using System.ComponentModel.Composition.ReflectionModel;
using System.Linq;

public class LoadCatalog : IResult
{
    static readonly Dictionary<string, DeploymentCatalog> Catalogs = new Dictionary<string, DeploymentCatalog>();
    readonly string uri;

    [Import]
    public AggregateCatalog Catalog { get; set; }

    public LoadCatalog(string relativeUri)
    {
        uri = relativeUri;
    }

    public void Execute(CoroutineExecutionContext context)
    {
        DeploymentCatalog catalog;

        if(Catalogs.TryGetValue(uri, out catalog))
            Completed(this, new ResultCompletionEventArgs());
        else
        {
            catalog = new DeploymentCatalog(uri);
            catalog.DownloadCompleted += (s, e) =>{
                if(e.Error == null)
                {
                    Catalogs[uri] = catalog;
                    Catalog.Catalogs.Add(catalog);
                    catalog.Parts
                        .Select(part => ReflectionModelServices.GetPartType(part).Value.Assembly)
                        .Where(assembly => !AssemblySource.Instance.Contains(assembly))
                        .Apply(x => AssemblySource.Instance.Add(x));
                }
                else Loader.Hide().Execute(context);

                Completed(this, new ResultCompletionEventArgs {
                    Error = e.Error,
                    WasCancelled = false
                });
            };

            catalog.DownloadAsync();
        }
    }

    public event EventHandler<ResultCompletionEventArgs> Completed = delegate { };
}
```

In case it wasn’t clear, this sample is using MEF. Furthermore, we are taking advantage of the DeploymentCatalog created for Silverlight 4. You don’t really need to know a lot about MEF or DeploymentCatalog to get the takeaway. Just take note of the fact that we wire for the DownloadCompleted event and make sure to fire the IResult.Completed event in its handler. This is what enables the async pattern to work. We also make sure to check the error and pass that along in the ResultCompletionEventArgs. Speaking of that, here’s what that class looks like:

---
><font color="#63aebb" face="微软雅黑">这个示例使用MEF，我们还利用了为 Silverlight 4 创建的 DeploymentCatalog。你实际上不需要了解很多关于 MEF 或 DeploymentCatalog 的知识。请注意，我们为 DownloadCompleted 事件进行了连接，并确保触发 IResult.Completed 事件。这就是使异步模式能够工作的原因。我们还确保检查错误并将其传递到 ResultCompletionEventArgs 中。这就是这个类的样子：</font>

``` csharp
public class ResultCompletionEventArgs : EventArgs
{
    public Exception Error;
    public bool WasCancelled;
}
```

Caliburn.Micro’s enumerator checks these properties after it get’s called back from each IResult. If there is either an error or WasCancelled is set to true, we stop execution. You can use this to your advantage. Let’s say you create an IResult for the OpenFileDialog. You could check the result of that dialog, and if the user canceled it, set WasCancelled on the event args. By doing this, you can write an action that assumes that if the code following the Dialog.Show executes, the user must have selected a file. This sort of technique can simplify the logic in such situations. Obviously, you could use the same technique for the SaveFileDialog or any confirmation style message box if you so desired. My favorite part of the LoadCatalog implementation shown above, is that the original implementation was written by a CM user! Thanks janoveh for this awesome submission! As a side note, one of the things we added to the CM project site is a “Recipes” section. We are going to be adding more common solutions such as this to that area in the coming months. So, it will be a great place to check for cool plugins and customizations to the framework.

Another thing you can do is create a series of IResult implementations built around your application’s shell. That is what the ShowScreen result used above does. Here is its implementation:

---
><font color="#63aebb" face="微软雅黑">Caliburn.Micro 枚举器从每个 IResult 调用后获取属性进行检查。如果存在错误或 WasCancelled = true，则停止执行。你可以利用这一点。假设你为 OpenFileDialog 创建了一个 IResult，你可以检查该对话框的结果，如果用户取消了对话框，那么将EventArgs WasCancelled 设置为取消。你可以编写一个活动，该活动假定如果 Dialog.Show 后面的代码执行，用户必须选择了一个文件。这种技术可以简化这种情况下的逻辑。显然，你可以对 SaveFileDialog 或任何确认样式消息框使用相同的技术。上面显示的 LoadCatalog 实现中我最喜欢的部分是最初的实现是由 CM 用户编写的!感谢janoveh，感谢你给了我这么棒的作品!作为补充说明，我们向 CM 项目站点添加的内容之一是 “Recipes” 部分。在接下来的几个月里，我们将在这一领域增加更多常见的解决方案。因此，这将是一个很酷的插件和框架定制的好地方。

>你可以做的另一件事是围绕应用程序的外壳创建一系列 IResult 实现。这就是上面使用的ShowScreen结果所做的。下面是它的实现:</font>

``` csharp
using System;
using System.ComponentModel.Composition;

public class ShowScreen : IResult
{
    readonly Type screenType;
    readonly string name;

    [Import]
    public IShell Shell { get; set; }

    public ShowScreen(string name)
    {
        this.name = name;
    }

    public ShowScreen(Type screenType)
    {
        this.screenType = screenType;
    }

    public void Execute(CoroutineExecutionContext context)
    {
        var screen = !string.IsNullOrEmpty(name)
            ? IoC.Get<object>(name)
            : IoC.GetInstance(screenType, null);

        Shell.ActivateItem(screen);
        Completed(this, new ResultCompletionEventArgs());
    }

    public event EventHandler<ResultCompletionEventArgs> Completed = delegate { };

    public static ShowScreen Of<T>()
    {
        return new ShowScreen(typeof(T));
    }
}
```

This bring up another important feature of IResult. Before CM executes a result, it passes it through the IoC.BuildUp method allowing your container the opportunity to push dependencies in through the properties. This allows you to create them normally within your view models, while still allowing them to take dependencies on application services. In this case, we depend on IShell. You could also have your container injected, but in this case I chose to use the IoC static class internally. As a general rule, you should avoid pulling things from the container directly. However, I think it is acceptable when done inside of infrastructure code such as a ShowScreen IResult.

---
><font color="#63aebb" face="微软雅黑">这引出了 IResult 的另一个重要特性。在CM执行一个结果之前，它通过 IoC.BuildUp 方法传递，允许容器有机会通过属性将依赖项注入其中。这允许你正常地在视图模型中创建它们，同时仍然允许它们依赖于应用程序服务。在这种情况下，我们依赖于 IShell。你也可以将容器注入，但在本例中，我选择在内部使用IoC静态类。一般来说，你应该避免直接从容器中取出东西。但是，我认为在基础架构代码(如ShowScreen IResult)内部执行是可以接受的。</font>

### Other Usages - 其他用法
Out-of-the-box Caliburn.Micro can execute coroutines automatically for any action invoked via an ActionMessage. However, there are times where you may wish to take advantage of the coroutine feature directly. To execute a coroutine, you can use the static Coroutine.BeginExecute method.

I hope this gives some explanation and creative ideas for what can be accomplished with IResult. Be sure to check out the sample application attached. There’s a few other interesting things in there as well.

---
><font color="#63aebb" face="微软雅黑">开箱即用的 Caliburn.Micro 可以为通过 ActionMessage 调用的任何操作自动执行协程。然而，有时你可能希望直接利用协同工作特性。要执行协同程序，可以使用静态 Coroutine.BeginExecute 方法。</font>

---
><font color="#63aebb" face="微软雅黑">我希望这能为IResult所能做到的提供一些解释和创造性的想法。请务必查看附件中的示例应用程序。还有一些其他有趣的东西在那里。</font>

[目录](./index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[Screens, Conductors and Composition - 屏幕,指挥和组件](./composition.md)