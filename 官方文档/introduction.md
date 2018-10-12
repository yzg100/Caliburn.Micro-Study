---
layout: page
title: Introduction
---

When my [“Build Your Own MVVM Framework”][mix] talk was chosen for Mix10, I was excited to have the opportunity to show others what we had been doing in [Caliburn][caliburn] in a simplified, but powerful way. After giving the talk, I received a ton of positive feedback on the techniques/framework that I had demonstrated. I was approached by several companies and individuals who expressed an interest in a more “official” version of what I had shown. That, coupled with the coming of Windows Phone 7, impressed upon me a need to have a more “lean and mean” framework. My vision was to take 90% of Caliburn’s features and squash them into 10% of the code. I started with the Mix sample framework, then used an iterative process of integrating Caliburn v2 features, simplifying and refactoring. I continued along those lines until I had what I felt was a complete solution that mirrored the full version of Caliburn v2, but on a smaller scale.

---
><font color="#63aebb" face="微软雅黑">当我为Mix10选择了“构建你自己的MVVM框架”演讲时，我很高兴有机会向其他人展示我们在[Caliburn]中以一种简化但强大的方式所做的事情。演讲结束后，我收到了大量关于我所演示的技术/框架的积极反馈。有几家公司和个人联系我，他们对我展示的更“正式”的版本表示了兴趣。再加上 Windows Phone 7 的到来，给我留下了需要有一个更“精简”的框架的印象。我的设想是将 Caliburn 90% 的特性压缩成 10% 的代码。我从 Mix 样例框架开始，然后使用集成 Caliburn v2功能，简化和重构的迭代过程。我继续沿着这些路线走，直到我觉得这是一个完整的解决方，它反映了Caliburn v2的完整版本，但规模更小。</font>

Caliburn.Micro consists of one ~75k assembly (~35k in XAP) that builds for WPF, Silverlight and Windows Phone. It has a single dependency, System.Windows.Interactivity, which you are probably already using regularly in development. Caliburn.Micro is about 2,700 LOC total, so you can easily go through the whole codebase in a short period and hold it in your head. That’s about 10% of the size of Caliburn v2, which is running around 27,000 LOC and is a lot harder to grasp in a short time. The best part is that I believe I was able to put something together that contains all of the features I consider most important in Caliburn. Here’s a brief list:

---
><font color="#63aebb" face="微软雅黑">Caliburn.Micro 由一个~75k的组件(XAP中~35k)组成，用于 WPF、Silverlight 和 Windows Phone 一个单独的依赖项 System.Windows.Interactivity，你可能已经在开发中经常使用它了。Caliburn.Micro 大约是2,700 行代码，所以你可以在短时间内轻松地浏览整个代码库，并将其记在大脑中。这大约是 Caliburn v2体积的10%，后者运行约2.7万代码，在短时间内很难掌握。最棒的是，我相信我能把一些我认为在 Caliburn 中最重要的特性整合到一起。这里有一个简短的清单:</font>

### Action Messages - 活动消息
The Action mechanism allows you to “bind” UI triggers, such as a Button’s “Click” event, to methods on your View-Model or Presenter. The mechanism allows for passing parameters to the method as well. Parameters can be databound to other FrameworkElements or can pass special values, such as the DataContext or EventArgs. All parameters are automatically type converted to the method’s signature. This mechanism also allows the “Action.Target” to vary independently of the DataContext and enables it to be declared at different points in the UI from the trigger. When a trigger occurs, the “message” bubbles through the element tree looking for an Action.Target (handler) that is capable of invoking the specified method. This is why we call them messages. The “bubbling” nature of Action Messages is extremely powerful and very helpful especially in master/detail scenarios. In addition to invocation, the mechanism supports a “CanExecute” guard. If the Action has a corresponding Property or Method with the same name, but preceded by the word “Can,” the invocation of the Action will be blocked and the UI will be disabled. Actions also support Coroutines (see below). That’s all fairly standard for existing Caliburn users, but we do have a few improvements in Caliburn.Micro that will be making their way into the larger framework. The Caliburn.Micro implementation of ActionMessages is built on System.Windows.Interactivity. This allows actions to be triggered by any TriggerBase developed by the community. Furthermore, Caliburn.Micro’s Actions have full design-time support in Blend. Code-centric developers will be happy to know that Caliburn.Micro supports a very terse syntax for declaring these ActionMessages through a special attached property called Message.Attach.

---
><font color="#63aebb" face="微软雅黑">Action 机制允许你将UI触发器（例如Button的“Click”事件）“绑定” 到 View-Model 或 Presenter 上的方法。该机制也允许将参数传递给方法。参数可以被数据绑定 FrameworkElements 元素，也可以传递特殊值，比如 DataContext 或 EventArgs，所有参数都是自动类型转换为方法的签名。允许 "Action.Target" 独立于 DataContext 而变化，并使其能够在 UI 中的不同点从触发器声明。当触发发生时，“message” 将在元素树中冒泡，寻找能够调用指定方法的 Action.Target （处理程序）。这就是我们称之为消息的原因。Action Messages 的“冒泡”特性非常强大，特别是在主/细节场景中非常有用。除了调用之外，该机制还支持 “CanExecute” 保护。如果 Action 具有以 “Can” 开头的相同名称的属性或方法，则将阻止对 Action 的调用，并禁用 UI。Action 也支持 Coroutines (见下文)。这对于现有的 Caliburn 用户来说是相当标准的，但是我们在 Caliburn.Micro 中有一些改进，它们将会融入更大的框架。ActionMessages 的 Caliburn.Micro 实现是基于 System.Windows.Interactivity 构建的。允许由社区开发的任何 TriggerBase 触发操作。Caliburn.Micro's Actions 在 Blend 中拥有完整的设计时支持。以代码为主的开发人员会很高兴地知道，Caliburn.Micro 支持一种非常简洁的语法，用于通过 Message.Attach 的特殊附加属性声明 ActionMessages。</font>

### Action Conventions - Action 约定
Out of the box, we support a set of binding conventions around the ActionMessage feature. These conventions are based on x:Name. So, if you have a method called “Save” on your ViewModel and a Button named “Save” in your UI, we will automatically create an EventTrigger for the “Click” event and assign an ActionMessage for the “Save” method. Furthermore, we will inspect the method’s signature and properly construct the ActionMessage parameters. This mechanism can be turned off or customized. You can even change or add conventions for different controls. For example, you could make the convention event for Button “MouseMove” instead of “Click” if you really wanted.

---
><font color="#63aebb" face="微软雅黑">开箱即用，支持 ActionMessage 特性的绑定约定,约定基于x:Name。因此，如果 ViewModel 上有一个名为 “Save” 的方法，UI中有一个名为 “Save” 的Button，我们将将自动为 “Click” 事件创建 EventTrigger，并为 “Save” 方法分配一个 ActionMessage。此外，我们将检查方法的签名并正确构造 ActionMessage 参数。这个机制可以关闭或自定义。你甚至可以为不同的控件添加或修改约定。例如:你可以将 Button 约定事件设置为 “MouseMove” 而不是 “Click”。</font>

### Binding Conventions - 绑定约定 
We also support convention-based databinding. This too works with x:Name. If you have a property on your ViewModel with the same name as an element, we will attempt to databind them. Whereas the framework understands convention events for Actions, it additionally understands convention binding properties (which you can customize or extend). When a binding name match occurs, we then proceed through several steps to build up the binding (all of which are customizable), configuring such details as BindingMode, StringFormat, ValueConverter, Validation and UpdateSourceTrigger (works for SL TextBox and PasswordBox too). Finally, we support the addition of custom behaviors for certain scenarios. This allows us to detect whether we need to auto-generate a DataTemplate or wire both the ItemsSource and the SelectedItem of a Selector based on naming patterns.

---
><font color="#63aebb" face="微软雅黑">支持基于约定的数据绑定。这也适用于x:Name。如果 ViewModel 上有一个与元素同名的属性，我们将尝试进行数据绑定。框架理解 Actions 的约定事件，也明白约定绑定属性（你可以自定义或扩展）。当发现绑定名称匹配时，我们将执行几个步骤来构建绑定(所有这些步骤都是可定制的)，BindingMode、StringFormat、ValueConverter、Validation 和 UpdateSourceTrigger 等详细信息(也适用于 SL 的 TextBox 和 PasswordBox)。支持为特定场景添加定制行为。允许我们检测是否需要根据命名模式自动生成 DataTemplate 或者连接 ItemsSource 和SelectedItem 的选择器。</font>

### Screens and Conductors - 屏幕和引导
The Screen, ScreenConductor and ScreenCollection patterns enable model-based tracking of the active or current item, enforcing of screen lifecycles and elegant shutdown or shutdown cancellation in an application. Caliburn.Micro’s implementation of these patterns is an evolution of the one found in Caliburn and supports conducting any type of class, not just implementations of IScreen. These improvements are being introduced back into Caliburn. You’ll find that Caliburn.Micro’s screen implementation is quite thorough and even handles asynchronous shutdown scenarios with ease.

---
><font color="#63aebb" face="微软雅黑">Screen、ScreenConductor 和 ScreenCollection 模式支持基于模型的活动或当前项目的跟踪，在应用程序中强制执行屏幕生命周期和关闭或关闭取消。Caliburn.Micro 对这些模式的实现是 Caliburn 发现模式的进化，它支持执行任何类型的类，而不仅仅是 IScreen 的实现。这些改进正在被引入 Caliburn 中。你会发现 Caliburn.Micro 的屏幕实现非常彻底，甚至可以轻松地处理异步关闭场景。</font>

### Event Aggregator - 事件聚合器
Caliburn.Micro’s EventAggregator is simple yet powerful. The aggregator follows a bus-style pub/sub model. You register a message handler with the aggregator, and it sends you any messages you are interested in. You declare your interest in a particular message type by implementing IHandle<TMessage>. References to handlers are held weakly and publication occurs on the UI thread. We even support polymorphic subscriptions.

---
><font color="#63aebb" face="微软雅黑">Caliburn.Micro 的 EventAggregator 既简单又强大。聚合器遵循一个总线式的 pub/sub 模型。向聚合器注册一个消息处理程序，它会向你发送你感兴趣的任何消息。通过实现 IHandle<TMessage>，可以声明你感兴趣的特定消息类型。对处理程序的引用很少，并且在UI线程上发布，它甚至支持多态订阅。</font>

### Coroutines - 协同程序
Any action can optionally choose to return IResult or IEnumerable<IResult>, opening the door to a powerful approach to handling asynchronous programming. Furthermore, implementations of IResult have access to an execution context which tells them what ActionMessage they are executing for, what FrameworkElement triggered the messsage to be sent, what instance the ActionMessage was handled by (invoked on) and what the View is for that instance. Such contextual information enables a loosely-coupled, declarative mechanism by which a Presenter or View-Model can communicate with its View without needing to hold a reference to it at any time.

---
><font color="#63aebb" face="微软雅黑">任何操作都可以选择返回 IResult 或 IEnumerable<IResult>，这为处理异步编程提供了一种强大的方法。此外，IResult 的实现可以访问执行上下文，告诉它们执行的是哪个 ActionMessage，哪个 FrameworkElement 触发了要发送的消息，哪个实例处理了 ActionMessage (调用 on)，以及 View 对于该实例是什么。这样的上下文信息实现了松散耦合的声明性机制，通过该机制 Presenter 或 View-Model 可以与 View 进行通信，而无需在任何时候保持对它的引用。。</font>

### View Locator - 视图定位器
For every ViewModel in your application, Caliburn.Micro has a basic strategy for locating the View that should render it. We do this based on naming conventions. For example, if your VM is called MyApplication.ViewModels.ShellViewModel, we will look for MyApplication.Views.ShellView. Additionally, we support multiple views over the same View-Model by attaching a View.Context in Xaml. So, given the same model as above, but with a View.Context=”Master” we would search for MyApplication.Views.Shell.Master. Of course, all this is customizable.

---
><font color="#63aebb" face="微软雅黑">对于应用程序中的每个 ViewModel， Caliburn.Micro 都有一个基本的策略来定位应该呈现它的 View。我们根据命名约定执行此操作。例如，如果你的VM名为 MyApplication.ViewModels.ShellViewModel，我们将查找 MyApplication.Views.ShellView。此外，我们通过在Xaml中附加View.Context来支持同一View-Model上的多个视图。因此，给定与上面相同的模型，但使用 View.Context =“Master”，我们将搜索 MyApplication.Views.Shell.Master。当然，所有这些都是可定制的。</font>

### View Model Locator - 视图模型定位器
Though Caliburn.Micro favors the ViewModel-First approach, we also support View-First by providing a ViewModelLocator with the same mapping semantics as the ViewLocator.

---
><font color="#63aebb" face="微软雅黑">虽然 Caliburn.Micro 支持 ViewModel-First方法，但我们通过提供具有与 ViewLocator 相同语法的 ViewModelLocator 来支持 View-First。</font>

### Window Manager - 窗口管理器
This service provides a View-Model-centric way of displaying Windows (ChildWindow in Silverlight, Window in WPF, a custom native-styled host in Windows Phone). Simply pass it an instance of the VM and it will locate the view, wrap it in a Window if necessary, apply all conventions you have configured and show the window.

---
><font color="#63aebb" face="微软雅黑">此服务提供以视图模型为中心的方式显示 Windows （ Silverlight 中的 ChildWindow、WPF 中的 Window 、 WindowPhone中的自定义native-styled host）。只要将 VM 的实例传递给它，它将定位视图，必要时将其包装在一个窗口中，应用你配置的所有约定并显示窗口。</font>

### PropertyChangedBase and BindableCollection
What self respecting WPF/SL framework could go without a base implementation of INotifyPropertyChanged? The Caliburn.Micro implementation enables string and lambda-based change notification. It also ensures that all events are raised on the UI thread. BindableCollection is a simple collection that inherits from ObservableCollection<T>, but that ensures that all its events are raised on the UI thread as well.

---
><font color="#63aebb" face="微软雅黑">如果没有 INotifyPropertyChanged 的实现，有什么值得 WPF/SL 框架可以实现呢? Caliburn.Micro 支持基于 string 和 lambda-based 的更改通知。它还确保在UI线程上引发所有事件。BindableCollection 是一个继承自ObservableCollection<T> 的简单集合，但这确保了它的所有事件能在 UI 线程上引发。</font>

### Bootstrapper - 引导程序
What’s required to configure this framework and get it up and running? Not much. Simply inherit from Bootstrapper and add an instance of your custom bootstrapper to the Application’s ResourceDictionary. Done. If you want, you can override a few methods to plug in your own IoC container, declare what assemblies should be inspected for Views, etc. It’s pretty simple.

---
><font color="#63aebb" face="微软雅黑">配置这个框架并让它运行起来需要什么？不多。简单地从 Bootstrapper 继承，并将自定义 Bootstrapper 的实例添加到应用程序的 ResourceDictionary 中就可以。如果你愿意，可以重写一些方法来插入你自己的 IoC 容器，声明应该检查哪些程序集来查看 Views，等等。这非常简单。</font>


### Logging - 日志
Caliburn.Micro implements a basic logging abstraction. This is important in any serious framework that encourages Convention over Configuration. All the most important parts of the framework are covered with logging. Want to know what conventions are or are not being applied? Turn on logging. Want to know what actions are being executed? Turn on logging. Want to know what events are being published? Turn on logging. You get the picture.

---
><font color="#63aebb" face="微软雅黑">Caliburn.Micro 实现一个基本的日志抽象。这在任何鼓励约定优于配置的重要框架中都很重要。框架中所有最重要的部分都包含日志记录。想知道哪些约定被使用或未使用?打开日志记录。想知道正在执行哪些操作?打开日志记录。想知道正在发布哪些事件?打开日志记录。你就明白了。</font>


### MVVM and MVP
In case it isn’t obvious, this framework enables MVVM. MVVM isn’t hard on its own, but Caliburn.Micro strives to go beyond simply getting it done. We want to write elegant, testable, maintainable and extensible presentation layer code…and we want it to be easy to do so. That’s what this is about. If you prefer using Supervising Controller and PassiveView to MVVM, go right ahead. You’ll find that Caliburn.Micro can help you a lot, particularly its Screen/ScreenConductor implementation. If you are not interested in any of the goals I just mentioned, you’d best move along. This framework isn’t for you.

Just to be clear, this isn’t a toy framework. As I said, I really focused on supporting the core and most commonly used features from Caliburn v2. In fact, Caliburn.Micro is going to be my default framework moving forward and I recommend that if you are starting a new project you begin with the Micro framework. I’ve been careful to keep the application developer API consistent with the full version of Caliburn. In fact, the improvements I made in Caliburn.Micro are being folded back into Caliburn v2. What’s the good part about that? You can start developing with Caliburn.Micro, then if you hit edge cases or have some other need to move to Caliburn, you will be able to do so with little or no changes in your application.

Keeping with the spirit of “Build Your Own…” I want developers to understand how this little framework works, inside and out. I’ve intentionally chosen Mercurial for source control, because I want developers to take ownership. While I’ve done some work to make the most important parts of the framework extensible, I’m hoping to see many Forks, and looking forward to seeing your Pull requests.

---
><font color="#63aebb" face="微软雅黑">如果不明显，这个框架支持 MVVM。MVVM本身并不难，但是 Caliburn.Micro 力求超越并简单地完成它。我们希望编写优雅、可测试、可维护和可扩展的表示层代码……我们希望希望它很容易实现。这就是它的意义所在。如果你喜欢使用 Supervising Controller 和 PassiveView 到 MVVM，请继续。Caliburn.Micro 可以为你提供很多帮助，特别是它的Screen / ScreenConductor 实现。如果你对我刚才提到的任何一个目标都不感兴趣，你最好离开，这个框架不适合你。

>明确一点，这不是一个玩具框架。正如我所说，我真正关注的是支持 Caliburn v2 的核心和最常用的功能。事实上 Caliburn.Micro 将会是我的默认框架，我建议如果你要开始一个新项目你应该 Caliburn.Micro 框架开始。我一直小心翼翼地使应用程序开发人员 API 与 Caliburn 的完整版本保持一致。事实上，我在 Caliburn.Micro 所做的改进正被返回 Caliburn v2。这有什么好处呢?你可以用 Caliburn.Micro 开发，如果你碰到边缘情况或者有其他需要移动到 Caliburn 的地方，你就可以在你的应用程序中做很少或不进行任何改变。

>本着“构建你自己的…”的精神，我希望开发人员理解这个小框架的内部和外部工作原理。我有意选择 Mercurial 作为源代码控制，因为我想让开发人员拥有所有权。虽然我已经做了一些工作使框架的最重要的部分可扩展，但是我希望看到许多 Forks，并期待看到你的 Pull 请求。</font>

[目录](./index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[Obtain and Build the Code - 获取并构建代码](./build.md)

[mix]: http://live.visitmix.com/MIX10/Sessions/EX15
[caliburn]: http://caliburn.codeplex.com/