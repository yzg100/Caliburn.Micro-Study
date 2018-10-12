---
layout: page
title: All About Actions
---

We briefly introduced actions in [Pt. 1](./configuration), but there is so much more to know. To begin our investigation, we’ll take our simple “Hello” example and see what it looks like when we explicitly create the actions rather than use conventions. Here’s the Xaml:

---
><font color="#63aebb" face="微软雅黑">我们在 [Basic Configuration - 基本配置](./configuration) 中简要介绍了 Action，但要知道的还有很多。为了开始我们的研究，我们将以一个简单的 “Hello” 示例为例，看看当我们显式地创建操作而不是使用约定时是什么样子的。这里的Xaml:</font>

``` xml
<UserControl x:Class="Caliburn.Micro.Hello.ShellView"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:i="clr-namespace:System.Windows.Interactivity;assembly=System.Windows.Interactivity"
             xmlns:cal="http://www.caliburnproject.org">
    <StackPanel>
        <TextBox x:Name="Name" />
        <Button Content="Click Me">
            <i:Interaction.Triggers>
                <i:EventTrigger EventName="Click">
                    <cal:ActionMessage MethodName="SayHello" />
                </i:EventTrigger>
            </i:Interaction.Triggers>
        </Button>
    </StackPanel>
</UserControl> 
```

As you can see, the Actions feature leverages System.Windows.Interactivity for it’s trigger mechanism. This means that you can use anything that inherits from System.Windows.Interactivity.TriggerBase to trigger the sending of an ActionMessage.1 Perhaps the most common trigger is an EventTrigger, but you can create almost any kind of trigger imaginable or leverage some common triggers already created by the community. ActionMessage is, of course, the Caliburn.Micro-specific part of this markup. It indicates that when the trigger occurs, we should send a message of “SayHello.” So, why do I use the language “send a message” instead of “execute a method” when describing this functionality? That’s the interesting and powerful part. ActionMessage bubbles through the Visual Tree searching for a target instance that can handle it. If a target is found, but does not have a “SayHello” method, the framework will continue to bubble until it finds one, throwing an exception if no “handler” is found.2 This bubbling nature of ActionMessage comes in handy in a number of interesting scenarios, Master/Details being a key use case. Another important feature to note is Action guards. When a handler is found for the “SayHello” message, it will check to see if that class also has either a property or a method named “CanSayHello.” If you have a guard property and your class implements INotifyPropertyChanged, then the framework will observe changes in that property and re-evaluate the guard accordingly. We’ll discuss method guards in further detail below.

---
><font color="#63aebb" face="微软雅黑">Action 特性利用 System.Windows.Interactivity 作为它的触发器机制。你可以使用任何继承自 System.Windows.Interactivity.TriggerBase 的内容，触发一个 ActionMessage 的发送。也许最常见的触发器是 EventTrigger，你可以创建任何可以想到的触发器，或者利用社区已经创建的一些常见触发器。当然 ActionMessage 是 Caliburn.Micro 特定部分。当触发器触发时，发送 “SayHello” 消息。为什么我在描述这个功能时使用 “发送消息” 而不是 “执行方法” 这种语言呢？这是有趣而强大的部分。ActionMessage 通过 Visual Tree 搜索能够处理它的目标实例，如果找到了目标，但没有“SayHello”方法，框架将继续冒泡查找，直到找到为止，如果没有找到 “handler”，就抛出异常。ActionMessage 的冒泡特性在很多场景中都很有用，Master/Details是一个关键的用例。另一个需要注意的重要特性是动作保护。当找到 “SayHello” 消息的处理程序时，它将检查这个类是否拥有名为 “CanSayHello” 的属性或方法。如果有这个保护属性，而你的类实现了 INotifyPropertyChanged，那么框架会观察到该属性的变化，并重新评估该保护。下面我们将更详细地讨论方法保护。</font>

### Action Targets - 活动目标

Now you’re probably wondering how to specify the target of an ActionMessage. Looking at the markup above, there’s no visible indication of what that target will be. So, where does that come from? Since we used a Model-First approach, when Caliburn.Micro (hereafter CM) created the view and bound it to the ViewModel using the ViewModelBinder, it set this up for us. Anything that goes through the ViewModelBinder will have its action target set automatically. But, you can set it yourself as well, using the attached property Action.Target. Setting this property positions an ActionMessage “handler” in the Visual Tree attached to the node on with you declare the property. It also sets the DataContext to the same value, since you often want these two things to be the same. However, you can vary the Action.Target from the DataContext if you like. Simply use the Action.TargetWithoutContext attached property instead. One nice thing about Action.Target is that you can set it to a System.String and CM will use that string to resolve an instance from the IoC container using the provided value as its key. This gives you a nice way of doing View-First MVVM if you so desire. If you want Action.Target set and you want Action/Binding Conventions applied as well, you can use the Bind.Model attached property in the same way.

---
><font color="#63aebb" face="微软雅黑">现在您可能想知道如何指定 ActionMessage 的目标。看看上面的标记，没有明显的迹象表明目标是什么。那么，这从何而来呢?由于我们使用 Model-First 方法，当 Caliburn.Micro 创建视图并使用 ViewModelBinder 将其绑定到 ViewModel 时，它为我们设置它。通过 ViewModelBinder 的内容都会自动设置其操作目标。您也可以使用附加属性 Action.Target 设置它。设置此属性将 ActionMessage “handler” 放置在定位到节点的可视树中，并声明该属性。它还将 DataContext 设置为相同的值，这两者是相同的。如果您愿意，可以从 DataContext 更改 Action.Target。只需使用 Action.TargetWithoutContext 附加属性即可。Action.Target的一个好处是你可以将它设置为 System.String，CM 将使用该字符串来解析来自 IoC 容器的实例，使用提供的值作为 Key。这提供了一种很好的方式来执行 View-First MVVM。如果您想要设置 Action.Target，并且您也希望应用 Action / Binding Conventions，则可以以相同的方式使用Bind.Model附加属性。</font>

#### View First - 视图优先

让我们看看如何使用视图优先技术来实现 MVVM。下面是我们如何改变引导程序:

``` csharp
public class MefBootstrapper : BootstrapperBase
{
    //same as before

    protected override void OnStartup(object sender, StartupEventArgs e)
    {
        Application.RootVisual = new ShellView();
    }

    //same as before
} 
```

Because we are using View-First, we’ve inherited from the non-generic Bootstrapper. The MEF configuration is the same as seen previously, so I have left that out for brevity’s sake. The only other thing that is changed is how the view gets created. In this scenario, we simply override OnStartup, instantiate the view ourselves and set it as the RootVisual (or call Show in the case of WPF). Next, we’ll slightly alter how we are exporting our ShellViewModel, by adding an explicitly named contract:

---
><font color="#63aebb" face="微软雅黑">因为我们使用的是 View-First，所以我们继承了非泛型 Bootstrapper。MEF 配置与前面看到的相同，因此为了简洁起见，我省略了这一点。唯一改变的是视图的创建方式。在这个场景中，我们简单地覆盖 OnStartup ，自己实例化视图并将其设置为 RootVisual (或者WPF中的调用显示)。接下来，通过添加一个明确命名的约定，我们将略微改变如何输出 ShellViewModel 方式:</font>

``` csharp
[Export("Shell", typeof(IShell))]
public class ShellViewModel : PropertyChangedBase, IShell
{
    //same as before
} 
```

最后，我们将修改我们的视图来引入 VM 并执行所有绑定::

``` xml
<UserControl x:Class="Caliburn.Micro.ViewFirst.ShellView"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:cal="http://www.caliburnproject.org"
             cal:Bind.Model="Shell">
    <StackPanel>
        <TextBox x:Name="Name" />
        <Button x:Name="SayHello"
                Content="Click Me" />
    </StackPanel>
</UserControl> 
```

Notice the use of the Bind.Model attached property. This resolves our VM by key from the IoC container, sets the Action.Target and DataContext and applies all conventions. I thought it would be nice to show how View-First development is fully supported with CM, but mainly I want to make clear the various ways that you can set targets for actions and the implications of using each technique. Here’s a summary of the available attached properties:

---
><font color="#63aebb" face="微软雅黑">注意 Bind.Model 附加属性的使用。这将从 IoC 容器通过 Key 解析 VM，设置 Action.Target 和 DataContext 应用所有约定。我认为展示 CM 如何完全支持 View-First 开发是很好的，但主要是我想明确各种方法，你可以设置行动的目标和使用每种技术的含义。以下是可用附加属性的摘要：</font>

<dl>
	<dt>Action.Target</dt>
	<dd><font color="#63aebb" face="微软雅黑">将 Action.Target 属性和 DataContext 属性都设置为指定的实例。字符串值用于从 IoC 容器解析实例。</font></dd>
	<dt>Action.TargetWithoutContext</dt>
	<dd><font color="#63aebb" face="微软雅黑">将 Action.Target 属性设置为指定的实例。字符串值用于从 IoC 容器解析实例。</font></dd>
	<dt>Bind.Model</dt>
	<dd>View-First - Set’s the Action.Target and DataContext properties to the specified instance. Applies conventions to the view. String values are used to resolve an instance from the IoC container. (Use on root nodes like Window/UserControl/Page.)<br>
    <font color="#63aebb" face="微软雅黑">View-First - 将Action.Target和DataContext属性设置为指定的实例。将约定应用于视图。字符串值用于从 IoC 容器解析实例。(在Window / UserControl / Page 等根节点上使用)</font></dd>
	<dt>Bind.ModelWithoutContext</dt>
	<dd>View-First - Set’s the Action.Target to the specified instance. Applies conventions to the view. (Use inside of DataTemplate.)<br>
    <font color="#63aebb" face="微软雅黑">View-First 将 Action.Target 设置为指定的实例，约定应用于视图。(在 DataTemplate 中使用。)</font>
    </dd>
	<dt>View.Model</dt>
	<dd>ViewModel-First – Locates the view for the specified VM instance and injects it at the content site. Sets the VM to the Action.Target and the DataContext. Applies conventions to the view.<br>
    <font color="#63aebb" face="微软雅黑">ViewModel-First - 找到指定 VM 实例视图，并将其注入内容站点。将 VM 设置为 Action.Target 和 DataContext。将约定应用于视图。</font>
    </dd>
</dl>

### Action Parameters - 活动参数

Now, let’s take a look at another interesting aspect of ActionMessage: Parameters. To see this in action, let’s switch back to our original ViewModel-First bootstrapper, etc. and begin by changing our ShellViewModel to look like this: 

---
><font color="#63aebb" face="微软雅黑">现在，让我们来看看 ActionMessage 的另一个有趣的方面:参数。要看到实际效果，让我们切换回原始的 ViewModel-First bootstrapper ，然后开始修改 ShellViewModel ，让它看起来像这样:</font>


``` csharp
using System.ComponentModel.Composition;
using System.Windows;

[Export(typeof(IShell))]
public class ShellViewModel : IShell
{
    public bool CanSayHello(string name)
    {
        return !string.IsNullOrWhiteSpace(name);
    }

    public void SayHello(string name)
    {
        MessageBox.Show(string.Format("Hello {0}!", name));
    }
} 
```

There are a few things to note here. First, we are now working with a completely POCO class; no INPC goop here. Second, we have added an input parameter to our SayHello method. Finally, we changed our CanSayHello property into a method with the same inputs as the action, but with a bool return type. Now, let’s have a look at the Xaml:

---
><font color="#63aebb" face="微软雅黑">这里有几点需要注意，首先，使用的是完整的 POCO 类，没有 INPC 。向 SayHello 方法增加一个输入参数，并将 CanSayHello 属性更改为与操作输入相同但具有 bool 返回类型的方法。现在，让我们看看Xaml:</font>

``` xml
<UserControl x:Class="Caliburn.Micro.HelloParameters.ShellView"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:i="clr-namespace:System.Windows.Interactivity;assembly=System.Windows.Interactivity"
             xmlns:cal="http://www.caliburnproject.org">
    <StackPanel>
        <TextBox x:Name="Name" />
        <Button Content="Click Me">
            <i:Interaction.Triggers>
                <i:EventTrigger EventName="Click">
                    <cal:ActionMessage MethodName="SayHello">
                        <cal:Parameter Value="{Binding ElementName=Name, Path=Text}" />
                    </cal:ActionMessage>
                </i:EventTrigger>
            </i:Interaction.Triggers>
        </Button>
    </StackPanel>
</UserControl> 
```

Our markup now has one modification: We declared the parameter as part of the ActionMessage using an ElementName Binding. You can have any number of parameters you desire. Value is a DependencyProperty, so all the standard binding capabilities apply to parameters. Did I mention you can do all this in Blend?

---
><font color="#63aebb" face="微软雅黑">我们的标记现在有了一个修改:使用 ElementName 绑定将参数声明为 ActionMessage 的一部分。你可以拥有任意数量的参数。值是 DependencyProperty，因此所有的标准绑定功能都适用于参数。我说过你可以用 Blend 来做这些吗?</font>

![Actions in Blend](/public/images/documentation/actions-in-blend.jpg)

One thing that is nice about this is that every time the value of a parameter changes, we’ll call the guard method associated with the action(CanSayHello in this case) and use its result to update the UI that the ActionMessage is attached to. Go ahead and run the application. You’ll see that it behaves the same as in previous examples.

In addition to literal values and Binding Expressions, there are a number of helpful “special” values that you can use with parameters. These allow you a convenient way to access common contextual information:

---
><font color="#63aebb" face="微软雅黑">这样做的一个好处是，每次参数值发生变化时，我们都会调用与操作关联的方法(本例中是 CanSayHello)，并使用其结果更新 ActionMessage 所附加的 UI。运行应用程序。您将看到它的行为与前面的示例相同。<br><br>

除了字符串和绑定表达式之外，还有许多有用的“特殊”值可以与参数一起使用。这些可以让您方便地访问常见的上下文信息:</font>

<dl>
	<dt>$eventArgs</dt>
	<dd>Passes the EventArgs or input parameter to your Action. Note: This will be null for guard methods since the trigger hasn’t actually occurred.</dd>
	<dt>$dataContext</dt>
	<dd>Passes the DataContext of the element that the ActionMessage is attached to. This is very useful in Master/Detail scenarios where the ActionMessage may bubble to a parent VM but needs to carry with it the child instance to be acted upon.<br><font color="#63aebb" face="微软雅黑">传递ActionMessage附加到的元素的DataContext。这在主/细节场景中非常有用，在这些场景中，ActionMessage可能会冒泡到父VM，但需要携带要执行操作的子实例。</font>
    </dd>
	<dt>$source</dt>
	<dd>The actual FrameworkElement that triggered the ActionMessage to be sent.<br><font color="#63aebb" face="微软雅黑">触发 ActionMessage 发送的实际 FrameworkElement。</font></dd>
	<dt>$view</dt>
	<dd>The view (usually a UserControl or Window) that is bound to the ViewModel.<br><font color="#63aebb" face="微软雅黑">绑定到ViewModel的视图（通常是UserControl或Window）</font></dd>
	<dt>$executionContext</dt>
	<dd>The action's execution context, which contains all the above information and more. This is useful in advanced scenarios.<br><font color="#63aebb" face="微软雅黑">Action 的执行上下文，包含所有上述信息等。这在高级方案中很有用。</font></dd>
	<dt>$this</dt>
	<dd>The actual UI element to which the action is attached. In this case, the element itself won't be passed as a parameter, but rather its default property.<br><font color="#63aebb" face="微软雅黑">附加操作的实际UI元素。在这种情况下，元素本身不会作为参数传递，而是作为其默认属性传递。</font></dd>
</dl>

You must start the variable with a “$” but the name is treated in a case-insensitive way by CM. These can be extended through adding values to MessageBinder.SpecialValues.

---
><font color="#63aebb" face="微软雅黑">你必须以“$”开头，但是 CM 以不区分大小写的方式处理名称。可以通过向 MessageBinder.SpecialValues 添加值来扩展它们。</font>

##### Note: Using Special Values like $this or a Named Element
---
><font color="#63aebb" face="微软雅黑">注意:使用特殊值，如$this或命名元素</font>

When you don't specify a property, CM uses a default one, which is specified by the particular control convention. For button, that property happens to be "DataContext", while a TextBox defaults to Text, a Selector to SelectedItem, etc. The same happens when using a reference to another named control in the View instead of $this. The following: \<Button cal:Message.Attach="Click = MyAction(someTextBox)" /> causes CM to pass the Text contained in the TextBox named "someTextBox" to MyAction. The reason why the actual control is never passed to the action is that VMs should never directly deal with UI elements, so the convention discourages it. Note, however, that the control itself could easily be accessed anyway using the extended syntax (based on System.Windows.Interactivity) to populate the parameters, or customizing the Parser.

---
><font color="#63aebb" face="微软雅黑">当不指定属性时，CM 使用默认属性，这是由特定的控件约定指定的。对于按钮，该属性是 “DataContext”，而 TextBox 默认为 Text，Selector 默认为 SelectedItem 等等。在 View 中使用对另一个命名控件的引用而不是 $this 时，也会发生同样的情况。如下: \<Button cal:Message.Attach="Click = MyAction(someTextBox)" /> 使 CM 将包含在名为 “someTextBox” 的 TextBox 的 Text 传递给 MyAction 。实际的控件从未传递 Action 的原因是 VM 不应该直接处理 UI 元素，因为约定不允许这样做。但请注意，无论如何都可以使用扩展语法(基于System.Windows.Interactivity)轻松访问控件本身以填充参数或自定义解析。</font>


#### Enum values - 枚举值

If you want to pass Enum values as a parameteryou need to pass the value as an (uppercase) string:

---
><font color="#63aebb" face="微软雅黑">如果你想传递枚举值作为参数，你需要传递值转换为(大写)字符串:</font>

``` xml
...
<Fluent:Button Header="Go!" cal:Message.Attach="[Event Click] = [Action MethodWithEnum('MONKEY')]" />
```

``` csharp
public enum Animals
{
    Unicorn,
    Monkey,
    Dog
}

public class MyViewModel
{ 
    pubilc void MethodWithEnum(Animals a)
    {
         Animals myAnimal = a;
    }
}
```

##### Word to the Wise
Parameters are a convenience feature. They are very powerful and can help you out of some tricky spots, but they can be easily abused. Personally, I only use parameters in the simplest scenarios. One place where they have worked nicely for me is in login forms. Another scenario, as mentioned previously is Master/Detail operations.

Now, do you want to see something truly wicked? Change your Xaml back to this:

---
><font color="#63aebb" face="微软雅黑">参数是一种方便的特性。它们非常强大，可以帮助你摆脱一些棘手的问题，但它们很容易被滥用。就个人而言，我只在最简单的场景中使用参数。其中一个对我来说很好的地方是登录表单。如前面提到的另一种情况是主/细节操作。

>现在，你想看到真正邪恶的东西吗?将Xaml更改为以下内容:</font>

``` xml
<UserControl x:Class="Caliburn.Micro.HelloParameters.ShellView"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <StackPanel>
        <TextBox x:Name="Name" />
        <Button x:Name="SayHello" Content="Click Me" />
    </StackPanel>
</UserControl>
```

Running the application will confirm for you that CM’s conventions even understand ActionMessage parameters. We’ll discuss conventions a lot more in the future, but you should be happy to know that these conventions are case-insensitive and can even detect the before-mentioned “special” values.

---
><font color="#63aebb" face="微软雅黑">运行应用程序将确认 CM 的约定，甚至可以理解 ActionMessage 参数。我们将来会讨论更多的约定，但是您应该很高兴地知道这些约定是不区分大小写的，甚至可以检测到前面提到的“特殊”值。</font>

### Action Bubbling - 活动冒泡

Now, lets look at a simple Master/Detail scenario that demonstrates ActionMessage bubbling, but let’s do it with a shorthand syntax that is designed to be more developer friendly. We’ll start by adding a simple new class named Model:

---
><font color="#63aebb" face="微软雅黑">现在，让我们来看一个简单的主/细节场景，它演示了ActionMessage的冒泡，但是让我们用一种简化的语法来做，这种语法设计得更适合开发人员。我们将从添加一个简单的新类 Model 开始:</font>

``` csharp
using System;

public class Model
{
    public Guid Id { get; set; }
}
```

然后我们把 ShellViewModel 改成这样:

``` csharp
using System;
using System.ComponentModel.Composition;

[Export(typeof(IShell))]
public class ShellViewModel : IShell
{
    public BindableCollection<Model> Items { get; private set; }

    public ShellViewModel()
    {
        Items = new BindableCollection<Model>{
            new Model { Id = Guid.NewGuid() },
            new Model { Id = Guid.NewGuid() },
            new Model { Id = Guid.NewGuid() },
            new Model { Id = Guid.NewGuid() }
        };
    }

    public void Add()
    {
        Items.Add(new Model { Id = Guid.NewGuid() });
    }

    public void Remove(Model child)
    {
        Items.Remove(child);
    }
}
```

Now our shell has a collection of Model instances along with the ability to add or remove from the collection. Notice that the Remove method takes a single parameter of type Model. Now, let’s update the ShellView:

---
><font color="#63aebb" face="微软雅黑">现在我们的 shell 有一个 Model 实例的集合，以及从集合中添加或删除的功能。注意，Remove 方法只接受一个类型为 Model 的参数。现在，让我们更新ShellView:</font>

``` xml
<UserControl x:Class="Caliburn.Micro.BubblingAction.ShellView"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:cal="http://www.caliburnproject.org">
    <StackPanel>
        <ItemsControl x:Name="Items">
            <ItemsControl.ItemTemplate>
                <DataTemplate>,f
                    <StackPanel Orientation="Horizontal">
                        <Button Content="Remove" cal:Message.Attach="Remove($dataContext)" />
                        <TextBlock Text="{Binding Id}" />
                    </StackPanel>
                </DataTemplate>
            </ItemsControl.ItemTemplate>
        </ItemsControl>
        <Button Content="Add" cal:Message.Attach="Add" />
    </StackPanel>
</UserControl>
```

### Message.Attach
The first thing to notice is that we are using a more Xaml-developer-friendly mechanism for declaring our ActionMessages. The Message.Attach property is backed by a simple parser which takes its textual input and transforms it into the full Interaction.Trigger/ActionMessage that you’ve seen previously. If you work primarily in the Xaml editor and not in the designer, you’re going to like Message.Attach. Notice that neither Message.Attach declarations specify which event should send the message. If you leave off the event, the parser will use the ConventionManager to determine the default event to use for the trigger. In the case of Button, it’s Click. You can always be explicit of coarse. Here’s what the full syntax for our Remove message would look like if we were declaring everything:

---
><font color="#63aebb" face="微软雅黑">首先要注意的是，我们使用了一种对 xaml 开发人员更友好的机制来声明 ActionMessages。Message.Attach 属性由一个简单的解析器支持，解析器接受文本输入并将其转换为完整的交互。您以前见过的 Interaction.Trigger / ActionMessage。如果您主要在 Xaml 编辑器中工作，而不是在设计器中工作，那么您会喜欢 Message.Attach。请注意，Message.Attach声明都没有指定应该发送消息的事件。如果您不在事件中，解析器将使用 ConventionManager 确定用于触发器的默认事件。在按下按钮后解析器将使用常规管理器来确定用于触发器的默认事件。如果我们要声明所有内容，那么删除消息的完整语法如下:</font>

``` xml
<Button Content="Remove" cal:Message.Attach="[Event Click] = [Action Remove($dataContext)]" />
```

假设我们要用 Message.Attach 语法重写参数化的 SayHello 操作。它是这样的:

``` xml
<Button Content="Click Me" cal:Message.Attach="[Event Click] = [Action SayHello(Name.Text)]" />
```

但我们也可以利用解析器的一些智能默认值，这样做：

``` xml
<Button Content="Click Me" cal:Message.Attach="SayHello(Name)" />
```

你可以指定文字作为参数，甚至通过分号分隔它们来声明多个动作:

``` xml
<Button Content="Let's Talk" cal:Message.Attach="[Event MouseEnter] = [Action Talk('Hello', Name.Text)]; [Event MouseLeave] = [Action Talk('Goodbye', Name.Text)]" />
```

##### Warning - 警告
Those developers who ask me to expand this functionality into a full-blown expression parser will be taken out back and…dealt with. Message.Attach is not about cramming code into Xaml. It’s purpose is to provide a streamlined syntax for declaring when/what messages to send to the ViewModel. Please don’t abuse this.

---
><font color="#63aebb" face="微软雅黑">那些要求我将这个功能扩展为成熟的表达式解析器的开发人员将回收并处理。Message.Attach 不是将代码填入 Xaml。它的目的是提供简化的语法，用于声明何时/什么消息要发送到 ViewModel。。请不要滥用它。</font>

If you haven’t already, run the application. Any doubts you had will hopefully be put to rest when you see that the message bubbling works as advertised :) Something else I would like to point out is that CM automatically performs type-conversion on parameters. So, for example, you can pump TextBox.Text into a System.Double parameter without any fear of a casting issue.

---
><font color="#63aebb" face="微软雅黑">如果您还没有，请运行该应用程序。当你看到消息冒泡像宣传的那样工作时，你所怀疑的任何疑问都会被搁置。我想指出的其他事情是 CM 自动对参数进行类型转换。因此，例如，您可以将 TextBox.Text 放入 System.Double中，而不必担心出现转换问题。</font>

So, we’ve discussed using Interaction.Triggers with ActionMessage, including the use of Parameters with literals, element bindings3 and special values. We’ve discussed the various ways to set the action target depending on your needs/architectural style: Action.Target, Action.TargetWithoutContext, Bind.Model or View.Model. We also saw an example of the bubbling nature of ActionMessage and demoed it using the streamlined Message.Attach syntax. All along the way we’ve looked at various examples of conventions in action too. Now, there’s one final killer feature of ActionMessage we haven’t discussed yet…Coroutines. But, that will have to wait until next time.

---
><font color="#63aebb" face="微软雅黑">因此，我们讨论了将 Interaction.Triggers 与 ActionMessage 一起使用，包括使用带文字的参数、元素绑定和特殊值。我们已经讨论了根据您的需求/架构风格设置活动目标的各种方法: Action.Target, Action.TargetWithoutContext, Bind.Model 或 View.Model。我们还看到了 ActionMessage 冒泡特性的一个示例，并使用改进的 Message.Attach 语法演示了它。在整个过程中，我们也查看了各种 Action 约定的例子。现在，ActionMessage 还有一个最后的杀手级功能我们还没有讨论过......协同程序。但是，这将不得不等到下一次。</font>



[目录](index)&nbsp;&nbsp;|&nbsp;&nbsp;[Working with Windows Phone (Silverlight) - 可用于Windows Phone(Silverlight)](./windows-phone)