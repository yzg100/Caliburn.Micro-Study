---
layout: page
title: Screens, Conductors and Composition
---

Actions, Coroutines and Conventions tend to draw the most attention to Caliburn.Micro, but the Screens and Conductors piece is probably most important to understand if you want your UI to be engineered well. It’s particularly important if you want to leverage composition. The terms Screen, Screen Conductor and Screen Collection have more recently been codified by Jeremy Miller during his work on the book "Presentation Patterns" for Addison Wesley. While these patterns are primarily used in CM by inheriting ViewModels from particular base classes, it’s important to think of them as roles rather than as View-Models. In fact, depending on your architecture, a Screen could be a UserControl, Presenter or ViewModel. That’s getting a little ahead of ourselves though. First, let’s talk about what these things are in general.

---
><font color="#63aebb" face="微软雅黑">活动、协同程序和约定往往会把最多的注意力吸引到 Caliburn.Micro 上，但是如果你想让你的UI得到很好的设计，那么 Screen 和 conductor 部分可能是最重要的。如果你想利用组件，这一点尤为重要。最近，杰里米·米勒( Jeremy Miller )在为艾迪生·韦斯利( Addison Wesley )撰写《表现模式》( Presentation Patterns )一书时，将 “Screen”、 “Screen Conductor” 和 “Screen Collection” 等术语进行了编纂。虽然这些模式在 CM 中主要是通过继承特定基类的 ViewModel 来使用的，但是将它们看作任务而不是 View-Models 是很重要的。实际上，根据您的架构， Screen 可以是 UserControl、Presenter 或ViewModel。这有点超前了。首先，我们来谈谈这些内容的意义。</font>

### Theory - 理论

##### Screen 
This is the simplest construct to understand. You might think of it as a stateful unit of work existing within the presentation tier of an application. It’s independent from the application shell. The shell may display many different screens, some even at the same time. The shell may display lots of widgets as well, but these are not part of any screen. Some screen examples might be a modal dialog for application settings, a code editor window in Visual Studio or a page in a browser. You probably have a pretty good intuitive sense about this.



Often times a screen has a lifecycle associated with it which allows the screen to perform custom activation and deactivation logic. This is what Jeremy calls the ScreenActivator. For example, take the Visual Studio code editor window. If you are editing a C# code file in one tab, then you switch to a tab containing an XML document, you will notice that the toolbar icons change. Each one of those screens has custom activation/deactivation logic that enables it to setup/teardown the application toolbars such that they provide the appropriate icons based on the active screen. In simple scenarios, the ScreenActivator is often the same class as the Screen. However, you should remember that these are two separate roles. If a particular screen has complex activation logic, it may be necessary to factor the ScreenActivator into its own class in order to reduce the complexity of the Screen. This is particularly important if you have an application with many different screens, but all with the same activation/deactivation logic.

---
><font color="#63aebb" face="微软雅黑">这是最容易理解的结构。您可能认为它是存在于应用程序表示层中的有状态的工作单元。它独立于应用程序 shell。shell 可以显示许多不同的 Screen ，有些甚至同时显示。shell 还可以显示许多小部件，但这些小部件不属于任何 Screen。一些 Screen 示例可能是应用程序设置的模态对话框、Visual Studio 中的代码编辑器窗口或浏览器中的页面。你们可能对这个有直观的感受。

>通常， Screen 具有与其相关联的生命周期，这允许 Screen 执行自定义激活和停用逻辑。这就是 Jeremy 所说的 ScreenActivator。例如，以 Visual Studio 代码编辑器窗口为例。如果在一个选项卡中编辑 C# 代码文件，然后切换到包含 XML 文档的选项卡，您将注意到工具栏图标发生了变化。这些 Screen 每个都具有自定义激活/停用逻辑，使其能够装配/拆卸工具栏，以便根据活动 Screen 提供适当的图标。在简单的场景中，ScreenActivator 通常与 Screen 相同。但是，您应该记住这是两个独立的角色。如果某个 Screen 具有复杂的激活逻辑，则可能需要将 ScreenActivator 分解为其自身的类，以降低 Screen 的复杂性。如果您的应用程序具有许多不同的 Screen ，但所有 Screen 都具有相同的激活/停用逻辑，那么这一点就显得尤为重要。</font>

##### Screen Conductor
Once you introduce the notion of a Screen Activation Lifecycle into your application, you need some way to enforce it. This is the role of the ScreenConductor. When you show a screen, the conductor makes sure it is properly activated. If you are transitioning away from a screen, it makes sure it gets deactivated. There’s another scenario that’s important as well. Suppose that you have a screen which contains unsaved data and someone tries to close that screen or even the application. The ScreenConductor, which is already enforcing deactivation, can help out by implementing Graceful Shutdown. In the same way that your Screen might implement an interface for activation/deactivation, it may also implement some interface which allows the conductor to ask it “Can you close?” This brings up an important point: in some scenarios deactivating a screen is the same as closing a screen and in others, it is different. For example, in Visual Studio, it doesn’t close documents when you switch from tab to tab. It just activates/deactivates them. You have to explicitly close the tab. That is what triggers the graceful shutdown logic. However, in a navigation based application, navigating away from a page would definitely cause deactivation, but it might also cause that page to close. It all depends on your specific application’s architecture and it’s something you should think carefully about.

---
><font color="#63aebb" face="微软雅黑">一旦将 Screen 激活生命周期的概念引入到应用程序中，您就需要一些方法来实现它。这就是 ScreenConductor 的作用。当你显示一个 Screen 时，conductor 会确保它被正确激活。如果你离开 Screen ，它会确保它被停用。还有另一种情况也很重要。假设您有一个包含未保存数据的 Screen ，有人试图关闭该 Screen 甚至应用程序，已经强制停用的 ScreenConductor 可以通过实现优雅的帮助来关机。就像你的 Screen 可能实现一个激活/停用的接口一样，它也可能实现一些接口，允许 conductor 问它“你要关闭吗?”这就引出了一个重要的问题:在某些情况下，停用和关闭 Screen 是一样的，而在另一些情况下则不同。例如，在 Visual Studio 中，当您从一个选项卡切换到另一个选项卡时，它不会关闭文档。它只是激活/停用它们。您必须显式地关闭选项卡。这就是触发优雅关闭逻辑的原因。然而，在基于导航的应用程序中，导航离开页面肯定会导致停用，但也可能导致该页面关闭。这完全取决于您的特定应用程序的体系结构，这是您应该仔细考虑的事情。</font>

##### Screen Collection
In an application like Visual Studio, you would not only have a ScreenConductor managing activation, deactivation, etc., but would also have a ScreenCollection maintaining the list of currently opened screens or documents. By adding this piece of the puzzle, we can also solve the issue of deactivation vs. close. Anything that is in the ScreenCollection remains open, but only one of those items is active at a time. In an MDI-style application like VS, the conductor would manage switching the active screen between members of the ScreenCollection. Opening a new document would add it to the ScreenCollection and switch it to the active screen. Closing a document would not only deactivate it, but would remove it from the ScreenCollection. All that would be dependent on whether or not it answers the question “Can you close?” positively. Of course, after the document is closed, the conductor needs to decide which of the other items in the ScreenCollection should become the next active document.

---
><font color="#63aebb" face="微软雅黑">在 Visual Studio 中，您不仅可以使用 ScreenConductor 来管理激活、停用等操作，还可以使用 ScreenCollection 来维护当前打开的 Screen 或文档列表。通过这一难题，我们还可以解决停用与关闭的问题。ScreenCollection 中的任何内容都保持打开状态，但一次只有其中一个项处于活动状态。在像 VS 这样的 MDI 风格的应用程序中，conductor 管理 ScreenCollection 的成员之间激活切换。打开一个新文档会将其添加到 ScreenCollection 中，并将其切换到激活状态。关闭文档不仅会使其停用，还会将其从 ScreenCollection 中移除。所有这些都取决于是否回答了“你要关闭吗?”。当然，在关闭文档之后， conductor 需要决定 ScreenCollection 中的其他项目中哪个应该成为下一个活动文档。</font>

### Implementations - 实现
There are lots of different ways to implement these ideas. You could inherit from a TabControl and implement an IScreenConductor interface and build all the logic directly in the control. Add that to your IoC container and you’re off and running. You could implement an IScreen interface on a custom UserControl or you could implement it as a POCO used as a base for Supervising Controllers. ScreenCollection could be a custom collection with special logic for maintaining the active screen, or it could just be a simple IList<IScreen>.

---
><font color="#63aebb" face="微软雅黑">有很多不同的方法来实现这些想法。您可以从 TabControl 继承并实现 IScreenConductor 接口，并直接在控件中构建所有逻辑。将其添加到 IoC 容器中，就可以关闭和运行了。您可以在自定义 UserControl 上实现 IScreen 接口，也可以将其实现为作为监控控制器基础的 POCO。ScreenCollection 可以是自定义集合，具有用于维护活动 Screen 的特殊逻辑，也可以是简单的 IList<IScreen>。</font>

#### Caliburn.Micro Implementations - Caliburn.Micro实现
These concepts are implemented in CM through various interfaces and base classes which can be used mostly to build ViewModels. Let’s take a look at them:

---
><font color="#63aebb" face="微软雅黑">这些概念是通过各种接口和基类在 CM 中实现的，这些基类主要用于构建 ViewModel。让我们来看看它们:</font>

##### Screens
In Caliburn.Micro we have broken down the notion of screen activation into several interfaces:

 - IActivate – Indicates that the implementer requires activation. This interface provides an Activate method, an IsActive property and an Activated event which should be raised when activation occurs.
 - IDeactivate – Indicates that the implementer requires deactivation. This interface has a Deactivate method which takes a bool property indicating whether to close the screen in addition to deactivating it. It also has two events: AttemptingDeactivation, which should be raised before deactivation and Deactivated which should be raised after deactivation.
 - IGuardClose – Indicates that the implementer may need to cancel a close operation. It has one method: CanClose. This method is designed with an async pattern, allowing complex logic such as async user interaction to take place while making the close decision. The caller will pass an Action<bool> to the CanClose method. The implementer should call the action when guard logic is complete. Pass true to indicate that the implementer can close, false otherwise.

---
><font color="#63aebb" face="微软雅黑">在 Caliburn.Micro 中，我们将 Screen 激活的概念分解为几个接口:</font>


> - <font color="#63aebb" face="微软雅黑">IActivate – 表明实现者需要激活。此接口提供了 Activate 方法、IsActive 属性和 Activated 事件，在激活时触发。</font>

> - <font color="#63aebb" face="微软雅黑">IDeactivate – 指示实现者需要停用。此接口具有 Deactivate 方法，该方法除了停用之外还采用指示是否关闭 screen 的 bool 属性。它还有两个事件：AttemptingDeactivation，在停用之前触发，Deactivated 在停用后触发。</font>

> - <font color="#63aebb" face="微软雅黑">IGuardClose – 指示实现者可能需要取消关闭操作。它有一个方法: CanClose 。此方法采用异步模式设计的，在关闭时允许复杂的逻辑如异步用户交互。调用者将向 CanClose 方法传递 Action<bool>。当监视逻辑完成时，实现者应该调用操作。传递 true 表示实现者可以关闭，否则为false。</font>


In addition to these core lifecycle interfaces, we have a few others to help in creating consistency between presentation layer classes:

 - IHaveDisplayName – Has a single property called DisplayName
 - INotifyPropertyChangedEx – This interface inherits from the standard INotifyPropertyChanged and augments it with additional behaviors. It adds an IsNotifying property which can be used to turn off/on all change notification, a NotifyOfPropertyChange method which can be called to raise a property change and a Refresh method which can be used to refresh all bindings on the object.
 - IObservableCollection<T> – Composes the following interfaces: IList<T>, INotifyPropertyChangedEx and INotifyCollectionChanged
 - IChild<T> – Implemented by elements that are part of a hierarchy or that need a reference to an owner. It has one property named Parent.
 - IViewAware – Implemented by classes which need to be made aware of the view that they are bound to. It has an AttachView method which is called by the framework when it binds the view to the instance. It has a GetView method which the framework calls before creating a view for the instance. This enables caching of complex views or even complex view resolution logic. Finally, it has an event which should be raised when a view is attached to the instance called ViewAttached.

---
><font color="#63aebb" face="微软雅黑">除了这些核心生命周期接口之外，我们还有一些其他的接口来帮助创建表示层类之间的一致性:</font>

> - <font color="#63aebb" face="微软雅黑">IHaveDisplayName – 具有一个名为 DisplayName 的属性。</font>

> - <font color="#63aebb" face="微软雅黑">INotifyPropertyChangedEx – 此接口继承自标准的 INotifyPropertyChanged 并使用其他行为对其进行了扩充。它添加了一个 IsNotifying 属性，可以用来关闭/打开所有更改通知，NotifyOfPropertyChange 方法可以调用来引发属性更改，刷新方法可以用来刷新对象上的所有绑定。</font>

> - <font color="#63aebb" face="微软雅黑">IObservableCollection<T> – 编写以下接口，IList<T>,INotifyPropertyChangedEx 和 INotifyCollectionChanged。</font>

> - <font color="#63aebb" face="微软雅黑">IChild<T> – 由层次结构的元素或需要引用所有者的元素实现。它有一个名为 Parent 的属性。</font>

> - <font color="#63aebb" face="微软雅黑">IViewAware – 由需要知道它们绑定到的视图的类实现。它有一个 AttachView 方法，当它将视图绑定到实例时由框架调用。它有一个 GetView 方法，框架在为实例创建视图之前调用该方法。这支持缓存复杂视图，甚至复杂视图解析逻辑。最后，它有一个事件，当视图被附加到名为 ViewAttached 的实例时，这个事件应该被触发。</font>


Because certain combinations are so common, we have some convenience interfaces and base classes:

 - PropertyChangedBase – Implements INotifyPropertyChangedEx (and thus INotifyPropertyChanged). It provides a lambda-based NotifyOfPropertyChange method in addition to the standard string mechanism, enabling strongly-typed change notification. Also, all property change events are automatically marshaled to the UI thread.
 - BindableCollection – Implements IObservableCollection<T> by inheriting from the standard ObservableCollection<T> and adding the additional behavior specified by INotifyPropertyChangedEx. Also, this class ensures that all property change and collection change events occur on the UI thread.
 - IScreen – This interface composes several other interfaces: IHaveDisplayName, IActivate, IDeactivate, IGuardClose and INotifyPropertyChangedEx.
 - Screen – Inherits from PropertyChangedBase and implements the IScreen interface. Additionally, IChild and IViewAware are implemented.

---
><font color="#63aebb" face="微软雅黑">由于某些组合非常常见，我们有一些方便的接口和基类:</font>

> - <font color="#63aebb" face="微软雅黑">PropertyChangedBase – 实现 INotifyPropertyChangedEx (从 INotifyPropertyChanged)。除了标准的字符串外，它还提供了基于 lambda 的 NotifyOfPropertyChange 方法，支持强类型的更改通知。此外，所有属性更改事件都会自动封送到UI线程。</font>

> - <font color="#63aebb" face="微软雅黑">BindableCollection – 通过继承 ObservableCollection<T>，并添加 INotifyPropertyChangedEx 指定的附加行为，实现 IObservableCollection<T>。该类确保所有属性更改和集合更改事件在UI线程上发生。</font>

> - <font color="#63aebb" face="微软雅黑">IScreen – 这个接口由其他几个接口组成:IHaveDisplayName、IActivate、IDeactivate、IGuardClose和INotifyPropertyChangedEx。</font>

> - <font color="#63aebb" face="微软雅黑">Screen – 从PropertyChangedBase 继承并实现 IScreen 接口。此外，还实现了 IChild 和 IViewAware。</font>


What this all means is that you will probably inherit most of your view models from either PropertyChangedBase or Screen. Generally speaking, you would use Screen if you need any of the activation features and PropertyChangedBase for everything else. CM’s default Screen implementation has a few additional features as well and makes it easy to hook into the appropriate parts of the lifecycle:

 - OnInitialize – Override this method to add logic which should execute only the first time that the screen is activated. After initialization is complete, IsInitialized will be true.
 - OnActivate – Override this method to add logic which should execute every time the screen is activated. After activation is complete, IsActive will be true.
 - OnDeactivate – Override this method to add custom logic which should be executed whenever the screen is deactivated or closed. The bool property will indicated if the deactivation is actually a close. After deactivation is complete, IsActive will be false.
 - CanClose – The default implementation always allows closing. Override this method to add custom guard logic.
 - OnViewLoaded – Since Screen implements IViewAware, it takes this as an opportunity to let you know when your view’s Loaded event is fired. Use this if you are following a SupervisingController or PassiveView style and you need to work with the view. This is also a place to put view model logic which may be dependent on the presence of a view even though you may not be working with the view directly.
 - TryClose – Call this method to close the screen. If the screen is being controlled by a Conductor, it asks the Conductor to initiate the shutdown process for the Screen. If the Screen is not controlled by a Conductor, but exists independently (perhaps because it was shown using the WindowManager), this method attempts to close the view. In both scenarios the CanClose logic will be invoked and if allowed, OnDeactivate will be called with a value of true.

So, just to re-iterate: if you need a lifecycle, inherit from Screen; otherwise inherit from PropertyChangedBase.

---
><font color="#63aebb" face="微软雅黑">这一切意味着您可能会从 PropertyChangedBase 或 Screen 继承大部分视图模型。一般来说，如果您需要任何激活特性和 PropertyChangedBase，您都可以使用Screen。CM 的默认 Screen 实现也有一些额外的特性，使得它很容易连接到生命周期的合适部分:</font>

> - <font color="#63aebb" face="微软雅黑">OnInitialize – 重写此方法以添加逻辑，该逻辑只应在 Screen 第一次激活时执行。初始化完成后，IsInitialized = true。</font>

> - <font color="#63aebb" face="微软雅黑">OnActivate – 重写此方法以添加逻辑，该逻辑应在每次 Screen 被激活时执行。激活完成后，IsActive = true。</font>

> - <font color="#63aebb" face="微软雅黑">OnDeactivate – 重写此方法以添加自定义逻辑，该逻辑应在 Screen 停用或关闭时执行。如果停用实际上是关闭的，则使用 bool 属性表示。停用后， IsActive = false。</font>

> - <font color="#63aebb" face="微软雅黑">CanClose – 默认的实现是允许关闭。重写此方法以添加自定义保护逻辑。</font>

> - <font color="#63aebb" face="微软雅黑">OnViewLoaded – 由于 Screen 实现了 IViewAware，所以它将此作为一个条件，让您知道何时触发了视图的加载事件。如果您使用的是 SupervisingController 或 PassiveView 样式，并且需要使用该视图，请使用此选项。这也是放置视图模型逻辑的地方，它可能依赖于视图的存在，即使您可能没有直接使用视图。</font>

> - <font color="#63aebb" face="微软雅黑">TryClose – 调用此方法关闭 Screen。如果 Screen 由 Conductor 控制，它要求 Conductor 启动 Screen 的关闭过程。如 Screen 不是由 Conductor 控制的，而是独立存在的(它可能是使用 WindowManager 显示的)，这个方法尝试关闭视图。在这两种情况下，都将调用 CanClose ，如果允许，将调用 OnDeactivate = true。</font>

---
><font color="#63aebb" face="微软雅黑">所以，为了重新迭代:如果你需要一个生命周期，从 Screen 继承;否则从 PropertyChangedBase 继承。</font>

##### Conductors

As I mentioned above, once you introduce lifecycle, you need something to enforce it. In Caliburn.Micro, this role is represented by the IConductor interface which has the following members:

 - ActivateItem – Call this method to activate a particular item. It will also add it to the currently conducted items if the conductor is using a “screen collection.”
 - DeactivateItem – Call this method to deactivate a particular item. The second parameter indicates whether the item should also be closed. If so, it will also remove it from the currently conducted items if the conductor is using a “screen collection.”
 - ActivationProcessed – Raised when the conductor has processed the activation of an item. It indicates whether or not the activation was successful.
 - GetChildren – Call this method to return a list of all the items that the conductor is tracking. If the conductor is using a “screen collection,” this returns all the “screens,” otherwise this only returns ActiveItem. (From the IParent interface)
 - INotifyPropertyChangedEx – This interface is composed into IConductor.

---
><font color="#63aebb" face="微软雅黑">正如我上面提到的，一旦引入了生命周期，就需要一些东西来实现它。在 Caliburn.Micro 中，此角色由 IConductor 接口表示，该接口具有以下成员:</font>

> - <font color="#63aebb" face="微软雅黑">ActivateItem – 调用此方法激活特定项。如果 conductor 正在使用 “screen collection”，它还将把它添加到当前执行的 items 中。</font>

> - <font color="#63aebb" face="微软雅黑">DeactivateItem – 调用此方法停用特定项。第二个参数表示项是否应该关闭。如关闭且 conductor 使用的是 “screen collection”，它会将其从当前执行的  items 中删除。</font>

> - <font color="#63aebb" face="微软雅黑">ActivationProcessed – 当 conductor 处理了某一项激活。它表示激活是否成功。</font>

> - <font color="#63aebb" face="微软雅黑">GetChildren – 调用此方法以返回 conductor 正在跟踪的所有 items 的列表。如果 conductor 使用的是 “screen collection”，则返回所有 “screen”，否则只返回 ActiveItem。(来自IParent接口)</font>

> - <font color="#63aebb" face="微软雅黑">INotifyPropertyChangedEx – 此接口由 IConductor 组成。</font>



We also have an interface called IConductActiveItem which composes IConductor and IHaveActiveItem to add the following member:
 
 - ActiveItem – A property that indicates what item the conductor is currently tracking as active.

---
><font color="#63aebb" face="微软雅黑">还有一个接口 IConductActiveItem ，它混合 IConductor 和 IHaveActiveItem 来添加以下成员：</font>

> - <font color="#63aebb" face="微软雅黑">ActiveItem – conductor 当前活动 item。</font>


You may have noticed that CM’s IConductor interface uses the term “item” rather than “screen” and that I was putting the term “screen collection” in quotes. The reason for this is that CM’s conductor implementations do not require the item being conducted to implement IScreen or any particular interface. Conducted items can be POCOs. Rather than enforcing the use of IScreen, each of the conductor implementations is generic, with no constraints on the type. As the conductor is asked to activate/deactivate/close/etc each of the items it is conducting, it checks them individually for the following fine-grained interfaces: IActivate, IDeactive, IGuardClose and IChild. In practice, I usually inherit conducted items from Screen, but this gives you the flexibility to use your own base class, or to only implement the interfaces for the lifecycle events you care about on a per-class basis. You can even have a conductor tracking heterogeneous items, some of which inherit from Screen and others that implement specific interfaces or none at all.

Out of the box CM has three implementations of IConductor, two that work with a “screen collection” and one that does not. We’ll look at the conductor without the collection first. 

---
><font color="#63aebb" face="微软雅黑">您可能已经注意到 CM 的 IConductor 接口使用了术语  “item”而不是 “screen”，并且我将术语 “screen collection” 放在引号中。原因是 CM 的 conductor 实现不需要执行某个项来实现 IScreen 或任何特定的接口。已引导的项目可以是 POCOs。不是强制使用 IScreen，每个 conductor 实现都是通用的，对类型没有约束。当 conductor 被要求激活/停用/关闭/等的每个项时，它会分别检查它们，以确定下面的细粒度接口: IActivate、IDeactive、IGuardClose 和 IChild。在实践中，我通常从 Screen 上继承已执行的项，但这使您能够灵活地使用自己的基类，或者仅为每个类所关心的生命周期事件实现接口。您甚至可以让一个 conductor 跟踪异类项，其中一些项继承自 Screen ，而另一些则实现了特定的接口或根本没有接口。</font>

><font color="#63aebb" face="微软雅黑">开箱即用的 CM 有三种 IConductor 的实现，两种使用 “screen collection”，另一种不使用。我们先来看看没有 collection 的 conductor。</font>

##### Conductor<T>

This simple conductor implements the majority of IConductor’s members through explicit interface mechanisms and adds strongly typed versions of the same methods which are available publicly. This allows working with conductors generically through the interface as well as in a strongly typed fashion based on the items they are conducting. The Conductor<T> treats deactivation and closing synonymously. Since the Conductor<T> does not maintain a “screen collection,” the activation of each new item causes both the deactivation and close of the previously active item. The actual logic for determining whether or not the conducted item can close can be complex due to the async nature of IGuardClose and the fact that the conducted item may or may not implement this interface. Therefore, the conductor delegates this to an ICloseStrategy<T> which handles this and tells the conductor the results of the inquiry. Most of the time you’ll be fine with the DefaultCloseStrategy<T> that is provided automatically, but should you need to change things (perhaps IGuardClose is not sufficient for your purposes) you can set the CloseStrategy property on Conductor<T> to your own custom strategy.

---
><font color="#63aebb" face="微软雅黑">这个简单的 conductor 通过显式接口机制实现了 IConductor 的大多数成员，并添加了相同方法的强类型版本，这些方法可以公开使用。允许通过接口以通用的方式与 conductor 一起工作，以及基于正在 conductor 的项强类型的方式。Conductor\<T> 处理停用和关闭。由于 Conductor\<T> 不维护 “Screen Collection”，每个新项激活都会导致之前的活动项停用和关闭。IGuardClose的异步特性以及所执行的项可能实现或不实现此接口，因此决定所执行项是否可以关闭的实际逻辑可能很复杂。因此，Conductor 将这个委托给 ICloseStrategy<T>，它处理并告诉 Conductor 询问的结果。大多数情况下，默认的 DefaultCloseStrategy\<T> 是自动提供的，但是如果您需要修改(也许 IGuardClose 对您的目的来说还不够)，您可以将 Conductor<T> 上的 CloseStrategy 属性设置为您自己的定制策略。</font>

##### Conductor<T>.Collection.OneActive

This implementation has all the features of Conductor<T> but also adds the notion of a “screen collection.” Since conductors in CM can conduct any type of class, this collection is exposed through an IObservableCollection<T> called Items rather than Screens. As a result of the presence of the Items collection, deactivation and closing of conducted items are not treated synonymously. When a new item is activated, the previous active item is deactivated only and it remains in the Items collection. To close an item with this conductor, you must explicitly call its CloseItem method. When an item is closed and that item was the active item, the conductor must then determine which item should be activated next. By default, this is the item before the previous active item in the list. If you need to change this behavior, you can override DetermineNextItemToActivate.

---
><font color="#63aebb" face="微软雅黑">此实现具有 Conductor<T> 的所有功能，也添加了 “screen collection” 的概念。由于 CM 中的 conductor 可以执行任何类型的类，所以这个集合是通过 IObservableCollection<T> 来公开的，名为 Items 而不是 Screens。由于 Items 集合的存在，对已执行 Items 的停用和关闭不会相同。激活 Item 时，上一个激活的 Item 仅被停用，并保留在 Items 集合中。要使用此 conductor 关闭 item，必须显式调用 CloseItem 方法。当 item 关闭且该项目是激活的 Item 时，conductor 必须确定接下来应激活哪个 Item。默认情况下，这是列表中上一个激活的 Item。如果需要更改此行为，可以覆盖 DetermineNextItemToActivate。</font>

##### Conductor<T>.Collection.AllActive

Similarly, this implementation also has the features of Conductor<T> and adds the notion of a "screen collection." The main difference is that rather than a single item being active at one time, many items can be active. Closing an item deactivates it and removes it from the collection.

There are two very important details about CMs IConductor implementations that I have not mentioned yet. First, they both inherit from Screen. This is a key feature of these implementations because it creates a composite pattern between screens and conductors. So, let’s say you are building a basic navigation-style application. Your shell would be an instance of Conductor<IScreen> because it shows one Screen at a time and doesn’t maintain a collection. But, let’s say that one of those screens was very complicated and needed to have a multi-tabbed interface requiring lifecycle events for each tab. Well, that particular screen could inherit from Conductor<T>.Collection.OneActive. The shell need not be concerned with the complexity of the individual screen. One of those screens could even be a UserControl that implemented IScreen instead of a ViewModel if that’s what was required. The second important detail is a consequence of the first. Since all OOTB implementations of IConductor inherit from Screen it means that they too have a lifecycle and that lifecycle cascades to whatever items they are conducting. So, if a conductor is deactivated, its ActiveItem will be deactivated as well. If you try to close a conductor, it’s going to only be able to close if all of the items it conducts can close. This turns out to be a very powerful feature. There’s one aspect about this that I’ve noticed frequently trips up developers. **If you activate an item in a conductor that is itself not active, that item won’t actually be activated until the conductor gets activated.** This makes sense when you think about it, but can occasionally cause hair pulling.

---
><font color="#63aebb" face="微软雅黑">同样，这个实现还具有 Conductor<T> 的特性，并添加了 “screen collection” 的概念。主要的不同之处在于，不是单个 Item 在同一时间处于激活状态，而是多个 Item 同时处于活动状态。关闭项将使其停用并从集合中删除。</font>

---
><font color="#63aebb" face="微软雅黑">关于 CMs IConductor实现有两个非常重要的细节我还没有提到。首先，它们都是从 Screen 继承的。这是这些实现的一个关键特性，因为它在 screens 和 conductors 之间创建了一个复合模式。假设您正在构建一个基本的导航风格的应用程序。您的 shell 将是 Conductor<IScreen> 的实例，因为它每次只显示一个 Screen，不维护集合。假设其中一个屏幕非常复杂，需要一个多选项卡接口，每个选项卡都需要生命周期事件。这个特定的屏幕可以从 Conductor<T>.Collection.OneActive 继承。shell不必关心单个 Screen 的复杂性。如果需要的话，其中一个 Screen 甚至可以是实现IScreen的用户控件，而不是ViewModel。第二个重要的细节是第一个的结果。因为 IConductor 的所有 OOTB 实现都是从 Screen 上继承的，这意味着它们也有一个生命周期，生命周期级联到它们正在执行的任何项目。所以，如果一个 conductor 处于停用状态，它的 ActiveItem 也会停用。如果你试图关闭一个 conductor，它只能在它运行的的所有 Item 都可以关闭的情况下关闭。这是一个非常强大的特性。关于这一点，我经常注意到开发人员会犯一个错误。 </font> <font color="#0881a3" face="微软雅黑"> __如果您激活 conductor 中未激活的 Item，则该 Item 在 conductor 被激活之前不会被激活。__ </font> <font color="#63aebb" face="微软雅黑">当你思考这个问题时，这是有道理的，但偶尔也会让你抓头。</font>

##### Quasi-Conductors

Not everything in CM that can be a screen is rooted inside of a Conductor. For example, what about the your root view model? If it’s a conductor, who is activating it? Well, that’s one of the jobs that the Bootstrapper performs. The Bootstrapper itself is not a conductor, but it understands the fine-grained lifecycle interfaces discussed above and ensures that your root view model is treated with the respect it deserves. The WindowManager works in a similar way by acting a little like a conductor for the purpose of enforcing the lifecycle of your modal (and modeless - WPF only) windows. So, lifecycle isn’t magical. All your screens/conductors must be either rooted in a conductor or managed by the Bootstrapper or WindowManager to work properly; otherwise you are going to need to manage the lifecycle yourself.

---
><font color="#63aebb" face="微软雅黑">并非 CM 中可以作为 screen 的所有内容都置于 conductor 内部。例如，根视图模型?如果是 conductor ，谁在激活它?这是 Bootstrapper 执行的任务之一。Bootstrapper 本身不是一个 conductor ，但它理解上面讨论的细粒度生命周期接口，并确保根视图模型得到遵守。WindowManager 的工作方式与此类似，它的工作方式有点像一个 conductor 以强制执行模态（和无模式 - 仅WPF）窗口的生命周期。所以，生命周期并不神奇。您的所有 screens/conductors 必须置于 conductor 中或由 Bootstrapper 或 WindowManager 管理才能正常工作; 否则你将需要自己管理生命周期。</font>

##### View-First

If you are working with WP7 or using the Silverlight Navigation Framework, you may be wondering if/how you can leverage screens and conductors. So far, I’ve been assuming a primarily ViewModel-First approach to shell engineering. But the WP7 platform enforces a View-First approach by controlling page navigation. The same is true of the SL Nav framework. In these cases, the Phone/Nav Framework acts like a conductor. In order to make this play well with ViewModels, the WP7 version of CM has a FrameAdapter which hooks into the NavigationService. This adapter, which is set up by the PhoneBootstrapper, understands the same fine-grained lifecycle interfaces that conductors do and ensures that they are called on your ViewModels at the appropriate points during navigation. You can even cancel the phone’s page navigation by implementing IGuardClose on your ViewModel. While the FrameAdapter is only part of the WP7 version of CM, it should be easily portable to Silverlight should you wish to use it in conjunction with the Silverlight Navigation Framework.

---
><font color="#63aebb" face="微软雅黑">如果您正在使用 WP7 或使用 Silverlight 导航框架，您可能想知道是否/如何利用 screens 和 conductors。到目前为止，我一直在假设一种主要的 ViewModel-First 方法来进行 shell 工程。但是 WP7 平台通过控制页面导航来实施 View-First 方法。SL Nav框架也是如此。在这些情况下，电话/导航框架就像一个 conductor。为了在 ViewModels 中很好地发挥作用，WP7 版本的 CM 有一个 FrameAdapter，它连接到NavigationService。这个适配器是由PhoneBootstrapper设置的，它理解导体所做的相同的细粒度生命周期接口，并确保在导航过程中在适当的点调用视图模型。你甚至可以通过在视图模型上实现 IGuardClose 来取消手机的页面导航。虽然 FrameAdapter 只是 WP7 版本 CM 的一部分，但是如果您希望将其与 Silverlight 导航框架一起使用，那么它应该可以轻松地移植到 Silverlight。</font>

### Simple Navigation - 示例导航

![Simple Navigation](/public/images/documentation/simple-navigation.jpg)

Previously, we discussed the theory and basic APIs for Screens and Conductors in Caliburn.Micro. Now I would like to walk through the first of several samples. This particular sample demonstrates how to set up a simple navigation-style shell using Conductor<T> and two “Page” view models. As you can see from the project structure, we have the typical pattern of Bootstrapper and ShellViewModel. In order to keep this sample as simple as possible, I’m not even using an IoC container with the Bootstrapper. Let’s look at the ShellViewModel first. It inherits from Conductor<object> and is implemented as follows:

---
><font color="#63aebb" face="微软雅黑">之前，我们讨论了在 Caliburn.Micro 中 Screens 和 Conductors的理论和基本 API。现在，我想介绍几个示例中的第一个，这个示例演示了如何使用 Conductor<T> 和两个 “Page” 视图模型来设置一个简单的导航样式 shell。从项目结构可以看出，我们有 Bootstrapper 和 ShellViewModel 的典型模式。为了使这个示例尽可能简单，我甚至没有使用带引导程序的 IoC 容器。让我们先看一下 ShellViewModel。它继承了 Conductor\<object\>，实现方式如下:</font>

``` csharp
public class ShellViewModel : Conductor<object> {
    public ShellViewModel() {
        ShowPageOne();
    }

    public void ShowPageOne() {
        ActivateItem(new PageOneViewModel());
    }

    public void ShowPageTwo() {
        ActivateItem(new PageTwoViewModel());
    }
}
```

下面是对应的 ShellView:

``` xml
<UserControl x:Class="Caliburn.Micro.SimpleNavigation.ShellView"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:tc="clr-namespace:System.Windows.Controls;assembly=System.Windows.Controls.Toolkit">
    <tc:DockPanel>
        <StackPanel Orientation="Horizontal"
                    HorizontalAlignment="Center"
                    tc:DockPanel.Dock="Top">
            <Button x:Name="ShowPageOne"
                    Content="Show Page One" />
            <Button x:Name="ShowPageTwo"
                    Content="Show Page Two" />
        </StackPanel>

        <ContentControl x:Name="ActiveItem" />
    </tc:DockPanel>
</UserControl>
```

Notice that the ShellViewModel has two methods, each of which passes a view model instance to the ActivateItem method. Recall from our earlier discussion that ActivateItem is a method on Conductor<T> which will switch the ActiveItem property of the conductor to this instance and push the instance through the activation stage of the screen lifecycle (if it supports it by implementing IActivate). Recall also, that if ActiveItem is already set to an instance, then before the new instance is set, the previous instance will be checked for an implementation of IGuardClose which may or may not cancel switching of the ActiveItem. Assuming the current ActiveItem can close, the conductor will then push it through the deactivation stage of the lifecycle, passing true to the Deactivate method to indicate that the view model should also be closed. This is all it takes to create a navigation application in Caliburn.Micro. The ActiveItem of the conductor represents the “current page” and the conductor manages the transitions from one page to the other. This is all done in a ViewModel-First fashion since it’s the conductor and it’s child view models that are driving the navigation and not the “views.”

Once the basic Conductor structure is in place, it’s quite easy to get it rendering. The ShellView demonstrates this. All we have to do is place a ContentControl in the view. By naming it “ActiveItem” our data binding conventions kick in. The convention for ContentControl is a bit interesting. If the item we are binding to is not a value type and not a string, then we assume that the Content is a ViewModel. So, instead of binding to the Content property as we would in the other cases, we actually set up a binding with CM’s custom attached property: View.Model. This property causes CM’s ViewLocator to look up the appropriate view for the view model and CM’s ViewModelBinder to bind the two together. Once that is complete, we pop the view into the ContentControl’s Content property. This single convention is what enables the powerful, yet simple ViewModel-First composition in the framework.

For completeness, let’s take a look at the PageOneViewModel and PageTwoViewModel:

---
><font color="#63aebb" face="微软雅黑">注意，ShellViewModel有两个方法，每个方法都将一个视图模型实例传递给 ActivateItem 方法。回想一下我们之前的讨论，ActivateItem 是 Conductor<T> 上的一个方法，它将把 Conductor 的 ActiveItem 属性切换到这个实例，并将实例推送到 screen 生命周期的激活阶段(如果它通过实现 IActivate 来支持它)。还请记住，如果 ActiveItem 已经被设置为一个实例，那么在设置新实例之前，将检查前一个实例的 IGuardClose 实现，该实现可能取消，也可能不取消 ActiveItem 的切换。假设当前 ActiveItem 可以关闭，那么 conductor 会将其置于生命周期的停用阶段，将true传递给 Deactivate 方法，表示视图模型也应该关闭。这就是在 Caliburn.Micro 中创建导航应用程序所需的全部内容。conductor 的 ActiveItem 表示 “current page”，conductor 管理从一个页面到另一个页面的转换。这一切都是以 ViewModel-First 方式完成的，因为它是 conductor，它的子视图模型正在驱动导航，而不是 “views”。</font>

---
><font color="#63aebb" face="微软雅黑">一旦基本导体结构就位，就很容易得到渲染结果。ShellView 演示了这一点。我们要做的就是在视图中放置一个 ContentControl。通过将其命名为 “ActiveItem”，我们的数据绑定约定开始发挥作用。ContentControl 的约定非常有趣。如果我们绑定到的项不是值类型也不是字符串，那么我们假设内容是一个 ViewModel。因此，我们没有像在其他情况下那样绑定到 Content，而是使用 CM 的自定义附加属性 View.Model 建立了绑定。该属性使 CM 的 ViewLocator 查找视图模型的适当视图，并使 CM 的 ViewModelBinder 将两者绑定在一起。完成后，我们将视图取出到 ContentControl 的内容属性中。这种单一的约定使框架中具有强大而简单的 ViewModel-First组合成为可能。</font>

---
><font color="#63aebb" face="微软雅黑">为了完整起见，我们来看看 PageOneViewModel 和 PageTwoViewModel:</font>

``` csharp
public class PageOneViewModel {}

public class PageTwoViewModel : Screen {
    protected override void OnActivate() {
        MessageBox.Show("Page Two Activated"); //Don't do this in a real VM.
        base.OnActivate();
    }
}
```

与他们的观点一致：

``` xml
<UserControl x:Class="Caliburn.Micro.SimpleNavigation.PageOneView"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <TextBlock FontSize="32">Page One</TextBlock>
</UserControl>

<UserControl x:Class="Caliburn.Micro.SimpleNavigation.PageTwoView"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <TextBlock FontSize="32">Page Two</TextBlock>
</UserControl>
```

---
><font color="#63aebb" face="微软雅黑">我想指出一些最后的事情。注意，PageOneViewModel 只是一个 POCO，但 PageTwoViewModel 继承自 Screen。请记住，CM 中的导体不会对可以进行的操作施加任何限制。相反，他们会在必要的时间检查每个实例是否支持各种细粒度生命周期实例。因此，当为 PageTwoViewModel 调用 ActivateItem 时，它将首先检查 PageOneViewModel 以查看它是否实现了 IGuardClose，由于它没有这样做，它将试图关闭它。然后它将检查是否实现了 IDeactivate。由于它没有这样做，它将继续激活新 Item。首先，它检查新项是否实现IChild。从Screen开始，它就连接了层次关系。接下来，它将检查PageTwoViewModel以查看它是否实现了IActivate。从 Screen 开始，然后 OnActivate 方法中的代码将运行。最后，它将在 conductor 上设置 ActiveItem 属性并引发相应的事件。以下是应该记住的要点：激活是特定于 ViewModel 的生命周期过程，并不保证 View 的状态。很多时候，即使您的 ViewModel 已被激活，其视图可能还不可见。运行示例时，您将看到这一点。MessageBox 将在激活发生时显示，但第二页的视图将不可见。请记住，如果您有任何依赖于已加载视图的激活逻辑，则应覆盖 Screen.OnViewLoaded 而不是与 OnActivate 结合使用。</font>

### Simple MDI - MDI 示例

Let’s look at another example: this time a simple MDI shell that uses “Screen Collections.” As you can see, once again, I have kept things pretty small and simple:

---
><font color="#63aebb" face="微软雅黑">让我们看一下另一个例子：这次是一个使用 “Screen Collections” 的简单 MDI shell。正如你所看到的，我再一次让事情变得非常简单:</font>

![Simple MDI](/public/images/documentation/simple-mdi.jpg)

下面是应用程序运行时的屏幕截图:

![Simple MDI Example](/public/images/documentation/simple-mdi-screenshot.jpg)

Here we have a simple WPF application with a series of tabs. Clicking the “Open Tab” button does the obvious. Clicking the “X” inside the tab will close that particular tab (also, probably obvious). Let’s dig into the code by looking at our ShellViewModel: 

---
><font color="#63aebb" face="微软雅黑">这里我们有一个简单的带有一系列选项卡的WPF应用程序。单击“打开选项卡”按钮可以明显地看到这一点。单击选项卡内的“X”将关闭该特定选项卡(也可能很明显)。让我们通过查看我们的ShellViewModel来深入研究代码:</font>


``` csharp
public class ShellViewModel : Conductor<IScreen>.Collection.OneActive {
    int count = 1;

    public void OpenTab() {
        ActivateItem(new TabViewModel {
            DisplayName = "Tab " + count++
        });
    }
}
```

Since we want to maintain a list of open items, but only keep one item active at a time, we use Conductor<T>.Collection.OneActive as our base class. Note that, different from our previous example, I am actually constraining the type of the conducted item to IScreen. There’s not really a technical reason for this in this sample, but this more closely mirrors what I would actually do in a real application. The OpenTab method simply creates an instance of a TabViewModel and sets its DisplayName property (from IScreen) so that it has a human-readable, unique name. Let’s think through the logic for the interaction between the conductor and its screens in several key scenarios:

---
><font color="#63aebb" face="微软雅黑">由于我们想要维护一个打开项的列表，但是每次只保留一个活动项，所以我们使用 Conductor<T>.Collection.OneActive 作为基类。注意，与前面的示例不同的是，我实际上将所引导的项的类型限制为 IScreen。在这个示例中没有真正的技术原因，但这更接近于我在实际应用中所做的事情。OpenTab 方法简单地创建了 TabViewModel 的一个实例，并设置了它的 DisplayName 属性(来自IScreen)，使其具有可读的惟一名称。让我们通过几个关键场景中思考 conductor 与 screens 之间的交互逻辑：</font>

##### Opening the First Item - 打开第一项
 1. Adds the item to the Items collection. 
 2. Checks the item for IActivate and invokes it if present.
 3. Sets the item as the ActiveItem.

---
> <font color="#63aebb" face="微软雅黑"> 
> &nbsp;&nbsp;1. 将项添加到 Item 集合。<br>
> &nbsp;&nbsp;2. 检查 IActivate Item 并调用它(如果存在)。<br>
> &nbsp;&nbsp;3. 将 Item设置为 ActiveItem。
</font>

##### Closing an Existing Item - 关闭现有项
 1. Passes the item to the CloseStrategy to determine if it can be closed (by default it looks for IGuardClose). If not, the action is cancelled.
 2. Checks to see if the closing item is the current ActiveItem. If so, determine which item to activate next and follow steps from “Opening an Additional Item.”
 3. Checks to see if the closing item is IDeactivate. If so, invoke with true to indicate that it should be deactivated and closed.
 4. Remove the item from the Items collection.

Those are the main scenarios. Hopefully you can see some of the differences from a Conductor without a collection and understand why those differences are there. Let’s see how the ShellView renders:

---
> <font color="#63aebb" face="微软雅黑">
> &nbsp;&nbsp;1. 将项目传递给CloseStrategy以确定它是否可以关闭（默认情况下它会查找IGuardClose）。如果不是，则取消操作。<br>
> &nbsp;&nbsp;2. 检查结束项是否是当前的ActiveItem。如果是，请确定下一个要激活的项目，然后按照“打开其他项目”中的步骤操作。<br>
> &nbsp;&nbsp;3. 检查结束项是否为IDeactivate。如果是，则调用true表示应该停用并关闭它。<br>
> &nbsp;&nbsp;4. 从Items集合中删除该项目。
</font>

---
><font color="#63aebb" face="微软雅黑">这些是主要情景。希望你能看到一个没有集合的 Conductor 的一些差异，并理解为什么存在这些差异。让我们看看 ShellView 如何呈现：</font>

``` xml
<Window x:Class="Caliburn.Micro.SimpleMDI.ShellView"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:cal="http://www.caliburnproject.org"
        Width="640" Height="480">
    <DockPanel>
        <Button x:Name="OpenTab"
                Content="Open Tab" 
                DockPanel.Dock="Top" />
        <TabControl x:Name="Items">
            <TabControl.ItemTemplate>
                <DataTemplate>
                    <StackPanel Orientation="Horizontal">
                        <TextBlock Text="{Binding DisplayName}" />
                        <Button Content="X" cal:Message.Attach="CloseItem($dataContext)" />
                    </StackPanel>
                </DataTemplate>
            </TabControl.ItemTemplate>
        </TabControl>
    </DockPanel>
</Window>
```

As you can see we are using a WPF TabControl. CM’s conventions will bind its ItemsSource to the Items collection and its SelectedItem to the ActiveItem. It will also add a default ContentTemplate which will be used to compose in the ViewModel/View pair for the ActiveItem. Conventions can also supply an ItemTemplate since our tabs all implement IHaveDisplayName (through Screen), but I’ve opted to override that by supplying my own to enable closing the tabs. We’ll talk more in depth about conventions in a later article. For completeness, here are the trivial implementations of TabViewModel along with its view:

---
><font color="#63aebb" face="微软雅黑">如您所见，我们正在使用 WPF TabControl。CM 的约定将其 ItemsSource 绑定到 Items 集合，将其 SelectedItem 绑定到 ActiveItem。它还将添加一个默认的 ContentTemplate，用于在 ActiveItem 的 ViewModel / View对中进行组合。约定也可以提供 ItemTemplate，因为我们的选项卡都实现了 IHaveDisplayName（通过Screen），但是我选择通过提供我自己的选项来启用关闭选项卡。我们将在后面的文章中更深入地讨论约定。为了完整起见，这里有 TabViewModel 的简单实现及其视图：</font>

``` csharp
namespace Caliburn.Micro.SimpleMDI {
    public class TabViewModel : Screen {}
}
```

``` xml
<UserControl x:Class="Caliburn.Micro.SimpleMDI.TabView"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <StackPanel Orientation="Horizontal">
        <TextBlock Text="This is the view for "/>
        <TextBlock x:Name="DisplayName" />
        <TextBlock Text="." />
    </StackPanel>
</UserControl>
```

I’ve tried to keep it simple so far, but that’s not the case for our next sample. In preparation, you might want to at least think through or try to do these things:

 - Get rid of the generic TabViewModel. You wouldn’t really do something like this in a real application. Create a couple of custom view models and views. Wire things up so that you can open different view models in the conductor. Confirm that you see the correct view in the tab control when each view model is activated.
 - Rebuild this sample in Silverlight. Unfortunately, Silverlight’s TabControl is utterly broken and cannot fully leverage databinding. Instead, try using a horizontal ListBox as the tabs and a ContentControl as the tab content. Put these in a DockPanel and use some naming conventions and you will have the same effect as a TabControl.
 - Create a toolbar view model. Add an IoC container and register the ToolBarViewModel as a singleton. Add it to the ShellViewModel and ensure that it is rendered in the ShellView (remember you can use a named ContentControl for this). Next, have the ToolBarViewModel injected into each of the TabViewModels. Write code in the TabViewModel OnActivate and OnDeactivate to add/remove contextual items from the toolbar when the particular TabViewModel is activated. BONUS: Create a DSL for doing this which doesn’t require explicit code in the OnDeactivate override. HINT: Use the events.
 - Take the SimpleMDI sample and the SimpleNavigation sample and compose them together. Either add the MDI Shell as a PageViewModel in the Navigation Sample or add the Navigation Shell as a Tab in the MDI Sample.

---
><font color="#63aebb" face="微软雅黑">到目前为止，我一直试图让它保持简单，但这不是我们下一个示例的情况。在准备过程中，你可能想至少要仔细考虑一下，或者试着去做这些事情:</font>

> - <font color="#63aebb" face="微软雅黑">摆脱通用的 TabViewModel。你真的不会在真正的应用程序中做这样的事情。创建几个自定义视图模型和视图。连接起来，以便您可以在导体中打开不同的视图模型。激活每个视图模型时，确认您在选项卡控件中看到了正确的视图。</font>

> - <font color="#63aebb" face="微软雅黑">在 Silverlight 中重建此示例。不幸的是，Silverlight 的 TabControl 完全被破坏，无法充分利用数据绑定。相反，尝试使用水平 ListBox 作为选项卡，使用 ContentControl 作为选项卡内容。将它们放在 DockPanel 中并使用一些命名约定，您将获得与 TabControl 相同的效果。</font>

> - <font color="#63aebb" face="微软雅黑">创建工具栏视图模型。添加一个 IoC 容器并将 ToolBarViewModel 注册为单例。将它添加到 ShellViewModel 中，并确保它在 ShellView 中呈现(记住，您可以为此使用 ContentControl 命名它)。接下来，将 ToolBarViewModel 注入每个 TabViewModel。在 TabViewModel OnActivate 和 OnDeactivate 中编写代码，以在激活特定 TabViewModel 时从工具栏添加/删除上下文项。好处：为此创建一个 DSL，它不需要 OnDeactivate 覆盖中的显式代码。提示：使用事件。</font>

> - <font color="#63aebb" face="微软雅黑">将 SimpleMDI 示例和 SimpleNavigation 示例组合在一起。在导航示例中将 MDI Shell 添加为 PageViewModel，或者将导航 Shell添加为 MDI示例中的选项卡。</font>

### Hybrid - 混合

This sample is based loosely on the ideas demonstrated by Billy Hollis in this well-known DNR TV episode. Rather than take the time to explain what the UI does, [have a look at this short video for a brief visual explanation](http://vimeo.com/16975621).

> - <font color="#63aebb" face="微软雅黑">这个例子是基于比利·霍利斯在这个著名的DNR电视剧集里的观点。与其花时间解释UI的功能，[不如看看这段简短的视频，获得一个简短的视觉解释](http://vimeo.com/16975621)。</font>

![Hybrid](/public/images/documentation/hybird.jpg)

Ok, now that you’ve seen what it does, let’s look at how it’s put together. As you can see from the screenshot, I’ve chosen to organize the project by features: Customers, Orders, Settings, etc. In most projects I prefer to do something like this rather than organizing by “technical” groupings, such as Views and ViewModels. If I have a complex feature, then I might break that down into those areas.

I’m not going to go line-by-line through this sample. It’s better if you take the time to look through it and figure out how things work yourself. But, I do want to point out a few interesting implementation details.

> - <font color="#63aebb" face="微软雅黑">好了，现在您已经了解了它的作用，让我们看看它是如何组合起来的。正如您从屏幕截图中看到的，我选择了通过特性来组织项目:客户、订单、设置等等。在大多数项目中，我更喜欢这样做，而不是通过“技术”分组来组织，比如 Views 和 ViewModels。如果我有一个复杂的功能，那么我可能会把它分解成这些区域。</font>

> - <font color="#63aebb" face="微软雅黑">我不会逐行分析这个示例。如果你花点时间去看一看，弄清楚是如何进行的，那就更好了。我想指出一些有趣的实现细节。</font>

##### ViewModel Composition - ViewModel 组合

One of the most important features of Screens and Conductors in Caliburn.Micro is that they are an implementation of the Composite Pattern, making them easy to compose together in different configurations. Generally speaking, composition is one of the most important aspects of object oriented programming and learning how to use it in your presentation tier can yield great benefits. To see how composition plays a role in this particular sample, let's look at two screenshots. The first shows the application with the CustomersWorkspace in view, editing a specific Customer’s Address. The second screen is the same, but with its View/ViewModel pairs rotated three-dimensionally, so you can see how the UI is composed.

> - <font color="#63aebb" face="微软雅黑">Caliburn.Micro 中 Screens 和 Conductors 最重要的特性之一是它们是复合模式的实现，使得它们在不同的配置下很容易组合在一起。一般来说，组合是面向对象编程中最重要的方面之一，学习如何在表示层中使用它会带来很大的好处。了解合成在这个示例中是如何发挥作用的，让我们来看两个截图。第一个视图显示了带有 CustomersWorkspace 的应用程序，它编辑客户的地址。第二个屏幕是相同的，但是它的 View  /ViewModel 对是三维旋转的，所以您可以看到UI是如何组成的。</font>

*Editing a Customer’s Address*

![Composition](/public/images/documentation/composition.png)

*Editing a Customer’s Address (3D Breakout)*

![Breakout](/public/images/documentation/breakout.png)

In this application, the ShellViewModel is a Conductor<IWorkspace>.Collection.OneActive. It is visually represented by the Window Chrome, Header and bottom Dock. The Dock has buttons, one for each IWorkspace that is being conducted. Clicking on a particular button makes the Shell activate that particular workspace. Since the ShellView has a TransitioningContentControl bound to the ActiveItem, the activated workspace is injected and its view is shown at that location. In this case, it’s the CustomerWorkspaceViewModel that is active. It so happens that the CustomerWorkspaceViewModel inherits from Conductor<CustomerViewModel>.Collection.OneActive. There are two contextual views for this ViewModel (see below). In the screenshot above, we are showing the details view. The details view also has a TransitioningContentControl bound to the CustomerWorkspaceViewModel’s ActiveItem, thus causing the current CustomerViewModel to be composed in along with its view. The CustomerViewModel has the ability to show local modal dialogs (they are only modal to that specific custom record, not anything else). This is managed by an instance of DialogConductor, which is a property on CustomerViewModel. The view for the DialogConductor overlays the CustomerView, but is only visible (via a value converter) if the DialogConductor’s ActiveItem is not null. In the state depicted above, the DialogConductor’s ActiveItem is set to an instance of AddressViewModel, thus the modal dialog is displayed with the AddressView and the underlying CustomerView is disabled. The entire shell framework used in this sample works in this fashion and is entirely extensible simply by implementing IWorkspace. CustomerViewModel and SettingsViewModel are two different implementations of this interface you can dig into.

> - <font color="#63aebb" face="微软雅黑">在此应用程序中，ShellViewModel 是 Conductor.Collection.OneActive。它由 Window Chrome，Header 和底部 Dock 可视化合成。Dock 有按钮，每个 IWorkspace 都有一个按钮。单击特定按钮可使 Shell 激活该特定工作区。由于 ShellView 具有绑定到 ActiveItem 的 TransitioningContentControl，因此将注入激活的工作空间，并在该位置显示其视图。在这种情况下，它是活动的 CustomerWorkspaceViewModel。碰巧 CustomerWorkspaceViewModel 继承自 Conductor.Collection.OneActive。这个 ViewModel 有两个上下文视图（见下文）。在上面的屏幕截图中，我们显示了详细信息视图。详细信息视图还有一个 TransitioningContentControl 绑定到 CustomerWorkspaceViewModel 的 ActiveItem，从而导致当前的 CustomerViewModel 与其视图一起组成。CustomerViewModel 能够显示本地模态对话框（它们仅对特定的自定义记录是模态的，而不是其他任何内容）。这由 DialogConductor 实例管理，该实例是 CustomerViewModel 上的属性。DialogConductor 的视图覆盖 CustomerView，但只有在 DialogConductor 的 ActiveItem 不为 null 时才可见（通过值转换器）。在上面描述的状态中， DialogConductor 的 ActiveItem 被设置为 AddressViewModel 的实例，因此模式对话框与 AddressView 一起显示，并且底层 CustomerView 被禁用。此示例中使用的整个 shell 框架以这种方式工作，并且只需通过实现 IWorkspace 即可完全扩展。</font>

##### Multiple Views over the Same ViewModel - 同一视图模型上的多个视图

You may not be aware of this, but Caliburn.Micro can display multiple Views over the same ViewModel. This is supported by setting the View.Context attached property on the View/ViewModel’s injection site. Here’s an example from the default CustomerWorkspaceView:

> - <font color="#63aebb" face="微软雅黑">您可能没有注意到，Caliburn.Micro 可以在同一个 ViewModel 上显示多个视图。通过在 View / ViewModel 的注入站点上设置 View.Context 附加属性来支持此功能。以下是默认 CustomerWorkspaceView 的示例：</font>

``` xml
<clt:TransitioningContentControl cal:View.Context="{Binding State, Mode=TwoWay}"
                                 cal:View.Model="{Binding}" 
                                 Style="{StaticResource specialTransition}"/>
```

There is a lot of other Xaml surrounding this to form the chrome of the CustomerWorkspaceView, but the content region is the most noteworthy part of the view. Notice that we are binding the View.Context attached property to the State property on the CustomerWorkspaceViewModel. This allows us to dynamically change out views based on the value of that property. Because this is all hosted in the TransitioningContentControl, we get a nice transition whenever the view changes. This technique is used to switch the CustomerWorkspaceViewModel from a “Master” view, where it displays all open CustomerViewModels, a search UI and a New button, to a “Detail” view, where it displays the currently activated CustomerViewModel along with its specific view (composed in). In order for CM to find these contextual views, you need a namespace based on the ViewModel name, minus the words “View” and “Model”, with some Views named corresponding to the Context. For example, when the framework looks for the Detail view of Caliburn.Micro.HelloScreens.Customers.CustomersWorkspaceViewModel, it’s going to look for Caliburn.Micro.HelloScreens.Customers.CustomersWorkspace.Detail That’s the out-of-the-box naming convention. If that doesn’t work for you, you can simply customize the ViewLocator.LocateForModelType func.

> - <font color="#63aebb" face="微软雅黑">围绕它的很多其他Xaml构成了 CustomerWorkspaceView 的 chrome，但内容区域是视图中最值得注意的部分。请注意，我们将 View.Context 附加属性绑定到 CustomerWorkspaceViewModel 上的 State 属性。这允许我们根据该属性的值动态更改视图。因为这都是在 TransitioningContentControl 中托管的，所以只要视图发生变化，我们就会得到一个很好的转换。此技术用于将 CustomerWorkspaceViewModel 从“主”视图切换到“详细信息”视图，其中显示所有打开的 CustomerViewModel，搜索 UI 和 “新建” 按钮，其中显示当前激活的 CustomerViewModel 及其特定视图（组成）。为了让 CM 找到这些上下文视图，您需要一个基于 ViewModel 名称的命名空间，减去单词 “View” 和 “Model”，其中一些 Views 对应于 Context。例如，当框架查找 Caliburn.Micro.HelloScreens.Customers.CustomersWorkspaceViewModel 的详细信息视图时，它将查找 Caliburn.Micro.HelloScreens.Customers.CustomersWorkspace.Detail 这是开箱即用的命名约定。如果这对您不起作用，您只需自定义 ViewLocator.LocateForModelType func即可。</font>

##### Custom IConductor Implementation - 自定义IConductor实现

Although Caliburn.Micro provides the developer with default implementations of IScreen and IConductor. It’s easy to implement your own. In the case of this sample, I needed a dialog manager that could be modal to a specific part of the application without affecting other parts. Normally, the default Conductor<T> would work, but I discovered I needed to fine-tune shutdown sequence, so I implemented my own. Let’s take a look at that:

> - <font color="#63aebb" face="微软雅黑">尽管 Caliburn.Micro 为开发人员提供了IScreen 和 IConductor 的默认实现。很容易实现自己的功能。示例中，我需要一个对话管理器，它可以在不影响其他部分的情况下对应用程序的特定部分进行模态化。通常情况下，默认的 Conductor<T> 可以工作，但是我发现我需要对关闭顺序进行调整，所以我进行了自定义。让我们来看看:</font>

``` csharp
[Export(typeof(IDialogManager)), PartCreationPolicy(CreationPolicy.NonShared)]
public class DialogConductorViewModel : PropertyChangedBase, IDialogManager, IConductActiveItem {
    readonly Func<IMessageBox> createMessageBox;

    [ImportingConstructor]
    public DialogConductorViewModel(Func<IMessageBox> messageBoxFactory) {
        createMessageBox = messageBoxFactory;
    }

    public IScreen ActiveItem { get; private set; }

    public IEnumerable GetChildren() {
        return ActiveItem != null ? new[] { ActiveItem } : new object[0];
    }

    public void ActivateItem(object item) {
        ActiveItem = item as IScreen;

        var child = ActiveItem as IChild;
        if(child != null)
            child.Parent = this;

        if(ActiveItem != null)
            ActiveItem.Activate();

        NotifyOfPropertyChange(() => ActiveItem);
        ActivationProcessed(this, new ActivationProcessedEventArgs { Item = ActiveItem, Success = true });
    }

    public void DeactivateItem(object item, bool close) {
        var guard = item as IGuardClose;
        if(guard != null) {
            guard.CanClose(result => {
                if(result)
                    CloseActiveItemCore();
            });
        }
        else CloseActiveItemCore();
    }

    object IHaveActiveItem.ActiveItem
    {
        get { return ActiveItem; }
        set { ActivateItem(value); }
    }

    public event EventHandler<ActivationProcessedEventArgs> ActivationProcessed = delegate { };

    public void ShowDialog(IScreen dialogModel) {
        ActivateItem(dialogModel);
    }

    public void ShowMessageBox(string message, string title = "Hello Screens", MessageBoxOptions options = MessageBoxOptions.Ok, Action<IMessageBox> callback = null) {
        var box = createMessageBox();

        box.DisplayName = title;
        box.Options = options;
        box.Message = message;

        if(callback != null)
            box.Deactivated += delegate { callback(box); };

        ActivateItem(box);
    }

    void CloseActiveItemCore() {
        var oldItem = ActiveItem;
        ActivateItem(null);
        oldItem.Deactivate(true);
    }
}
```

Strictly speaking, I didn’t actually need to implement IConductor to make this work (since I’m not composing it into anything). But I chose to do this in order to represent the role this class was playing in the system and keep things as architecturally consistent as possible. The implementation itself is pretty straight forward. Mainly, a conductor needs to make sure to Activate/Deactivate its items correctly and to properly update the ActiveItem property. I also created a couple of simple methods for showing dialogs and message boxes which are exposed through the IDialogManager interface. This class is registered as NonShared with MEF so that each portion of the application that wants to display local modals will get its own instance and be able to maintain its own state, as demonstrated with the CustomerViewModel discussed above.

> - <font color="#63aebb" face="微软雅黑">严格地说，我实际上不需要实现 IConductor 来实现这个功能(因为我没有将它组合成任何东西)。但是我选择这样做是为了表示这个类在系统中扮演的角色，并尽可能保持体系结构上的一致性。实现本身非常简单。主要来说，一个 conductor 需要确保正确地激活/关闭它的 items，并正确地更新 ActiveItem 属性。我还创建了两个简单的方法，用于显示通过 IDialogManager 接口公开的对话框和消息框。此类与 MEF 注册为 NonShared，以便应用程序的每个想要显示本地模态的部分都将获得自己的实例并能够维护自己的状态，如上面讨论的 CustomerViewModel 所示。</font>

##### Custom ICloseStrategy - 自定义 ICloseStrategy

Possibly one of the coolest features of this sample is how we control application shutdown. Since IShell inherits IGuardClose, in the Bootstrapper we just override OnStartup and wire Silverlight’s MainWindow.Closing event to call IShell.CanClose:

> - <font color="#63aebb" face="微软雅黑">这个示例中最酷的特性之一可能是我们如何控制应用程序关闭。由于 IShell 继承了 IGuardClose，在 Bootstrapper 中我们只是重写 OnStartup 并连接 Silverlight 的 MainWindow.Closing 事件来调用 IShell.CanClose：</font>

``` csharp
protected override void OnStartup(object sender, StartupEventArgs e) {
    base.OnStartup(sender, e);

    if(Application.IsRunningOutOfBrowser) {
        mainWindow = Application.MainWindow;
        mainWindow.Closing += MainWindowClosing;
    }
}

void MainWindowClosing(object sender, ClosingEventArgs e) {
    if (actuallyClosing)
        return;

    e.Cancel = true;

    Execute.OnUIThread(() => {
        var shell = IoC.Get<IShell>();

        shell.CanClose(result => {
            if(result) {
                actuallyClosing = true;
                mainWindow.Close();
            }
        });
    });
}
```

The ShellViewModel inherits this functionality through its base class Conductor<IWorkspace>.Collection.OneActive. Since all the built-in conductors have a CloseStrategy, we can create conductor specific mechanisms for shutdown and plug them in easily. Here’s how we plug in our custom strategy:

> - <font color="#63aebb" face="微软雅黑">ShellViewModel 通过其基类 Conductor.Collection.OneActive 继承此功能。由于所有内置 conductors 都具有 CloseStrategy，我们可以创建导体特定的关闭机制并轻松插入。以下是我们如何插入自定义策略：</font>

``` csharp
[Export(typeof(IShell))]
public class ShellViewModel : Conductor<IWorkspace>.Collection.OneActive, IShell
{
    readonly IDialogManager dialogs;

    [ImportingConstructor]
    public ShellViewModel(IDialogManager dialogs, [ImportMany]IEnumerable<IWorkspace> workspaces) {
        this.dialogs = dialogs;
        Items.AddRange(workspaces);
        CloseStrategy = new ApplicationCloseStrategy();
    }

    public IDialogManager Dialogs {
        get { return dialogs; }
    }
}
```

以下是该策略的实现:

``` csharp
public class ApplicationCloseStrategy : ICloseStrategy<IWorkspace> {
    IEnumerator<IWorkspace> enumerator;
    bool finalResult;
    Action<bool, IEnumerable<IWorkspace>> callback;

    public void Execute(IEnumerable<IWorkspace> toClose, Action<bool, IEnumerable<IWorkspace>> callback) {
        enumerator = toClose.GetEnumerator();
        this.callback = callback;
        finalResult = true;

        Evaluate(finalResult);
    }

    void Evaluate(bool result)
    {
        finalResult = finalResult && result;

        if (!enumerator.MoveNext() || !result)
            callback(finalResult, new List<IWorkspace>());
        else
        {
            var current = enumerator.Current;
            var conductor = current as IConductor;
            if (conductor != null)
            {
                var tasks = conductor.GetChildren()
                    .OfType<IHaveShutdownTask>()
                    .Select(x => x.GetShutdownTask())
                    .Where(x => x != null);

                var sequential = new SequentialResult(tasks.GetEnumerator());
                sequential.Completed += (s, e) => {
                    if(!e.WasCancelled)
                    Evaluate(!e.WasCancelled);
                };
                sequential.Execute(new ActionExecutionContext());
            }
            else Evaluate(true);
        }
    }
}
```

The interesting thing I did here was to reuse the IResult functionality for async shutdown of the application. Here’s how the custom strategy uses it:

 1. Check each IWorkspace to see if it is an IConductor.
 2. If true, grab all the conducted items which implement the application-specific interface IHaveShutdownTask.
 3. Retrieve the shutdown task by calling GetShutdownTask. It will return null if there is no task, so filter those out.
 4. Since the shutdown task is an IResult, pass all of these to a SequentialResult and begin enumeration.
 5. The IResult can set ResultCompletionEventArgs.WasCanceled to true to cancel the application shutdown.
 6. Continue through all workspaces until finished or cancellation occurs.
 7. If all IResults complete successfully, the application will be allowed to close.

The CustomerViewModel and OrderViewModel use this mechanism to display a modal dialog if there is dirty data. But, you could also use this for any number of async tasks. For example, suppose you had some long running process that you wanted to prevent shutdown of the application. This would work quite nicely for that too.

---
> <font color="#63aebb" face="微软雅黑">我在这里做的一件有趣的事情是重用 IResult 功能来异步关闭应用程序。下面是定制策略如何使用它:</font>

---
> <font color="#63aebb" face="微软雅黑"> 
> &nbsp;&nbsp;1. 检查每个IWorkspace以查看它是否是IConductor。<br>
> &nbsp;&nbsp;2. 如果为 true，则获取实现特定于应用程序的接口 IHaveShutdownTask 的所有已执行项。<br>
> &nbsp;&nbsp;3. 通过调用GetShutdownTask来检索关闭任务。如果没有任务，它将返回 null，因此将其过滤掉。<br>
> &nbsp;&nbsp;4. 由于关闭任务是 IResult，因此将所有这些传递给 SequentialResult 并开始列举。<br>
> &nbsp;&nbsp;5. IResult 可以设置 ResultCompletionEventArgs.WasCanceled = true 来取消应用程序关闭。<br>
> &nbsp;&nbsp;6. 继续遍历所有工作空间，直到完成或取消。<br>
> &nbsp;&nbsp;7. 如果所有IResults成功完成，应用程序将被关闭。
</font>

---
> <font color="#63aebb" face="微软雅黑">如果存在脏数据，CustomerViewModel和OrderViewModel使用此机制显示模式对话框。但是，您也可以将此用于任何数量的异步任务。例如，假设您有一些长时间运行的进程，希望阻止应用程序的关闭。这也可以很好地工作。</font>

[目录](index)&nbsp;&nbsp;|&nbsp;&nbsp;[All About Conventions - 约定](./conventions)