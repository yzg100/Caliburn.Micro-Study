---
layout: page
title: All about Conventions
---

One of the main features of Caliburn.Micro is manifest in its ability to remove the need for boiler plate code by acting on a series of conventions. Some people love conventions and some hate them. That’s why CM’s conventions are fully customizable and can even be turned off completely if not desired. If you are going to use conventions, and since they are ON by default, it’s good to know what those conventions are and how they work. That’s the subject of this article.

---
> <font color="#63aebb" face="微软雅黑">Caliburn.Micro 的主要特征之一就是能够通过一系列约定来消除对样板代码的需求。有些人喜欢约定俗成，有些人讨厌它。这就是为什么 CM 的约定是完全可定制的，甚至可以在不需要时完全关闭。如果你打算使用约定，并且由于它们默认为 ON，那么最好知道这些约定是什么以及它们如何工作。这是本文的主题。</font>

### View Resolution (ViewModel-First)

##### Basics - 基础

The first convention you are likely to encounter when using CM is related to view resolution. This convention affects any ViewModel-First areas of your application. In ViewModel-First, we have an existing ViewModel that we need to render to the screen. To do this, CM uses a simple naming pattern to find a UserControl1 that it should bind to the ViewModel and display. So, what is that pattern? Let’s just take a look at ViewLocator.LocateForModelType to find out:

---
> <font color="#63aebb" face="微软雅黑">在使用CM时可能遇到的第一个约定与视图解析有关。此约定会影响应用程序的任何 ViewModel-First 区域。在 ViewModel-First 中，我们需要一个现有的 ViewModel 呈现到屏幕上。为此，CM 使用一个简单的命名模式来查找它应绑定到 ViewModel 并显示 UserControl1。那么，这种模式是什么？我们来看看 ViewLocator.LocateForModelType 以找出：</font>

``` csharp
public static Func<Type, DependencyObject, object, UIElement> LocateForModelType = (modelType, displayLocation, context) =>{
    var viewTypeName = modelType.FullName.Replace("Model", string.Empty);
    if(context != null)
    {
        viewTypeName = viewTypeName.Remove(viewTypeName.Length - 4, 4);
        viewTypeName = viewTypeName + "." + context;
    }

    var viewType = (from assmebly in AssemblySource.Instance
                    from type in assmebly.GetExportedTypes()
                    where type.FullName == viewTypeName
                    select type).FirstOrDefault();

    return viewType == null
        ? new TextBlock { Text = string.Format("{0} not found.", viewTypeName) }
        : GetOrCreateViewType(viewType);
};
```

Let’s ignore the “context” variable at first. To derive the view, we make an assumption that you are using the text “ViewModel” in the naming of your VMs, so we just change that to “View” everywhere that we find it by removing the word “Model”. This has the effect of changing both type names and namespaces. So ViewModels.CustomerViewModel would become Views.CustomerView. Or if you are organizing your application by feature: CustomerManagement.CustomerViewModel becomes CustomerManagement.CustomerView. Hopefully, that’s pretty straight forward. Once we have the name, we then search for types with that name. We search any assembly you have exposed to CM as searchable via AssemblySource.Instance.2 If we find the type, we create an instance (or get one from the IoC container if it’s registered) and return it to the caller. If we don’t find the type, we generate a view with an appropriate “not found” message.

---
> <font color="#63aebb" face="微软雅黑">我们首先忽略“context”变量。为了得到视图，我们假设你在 VM 的命名中使用了文本“ViewModel”，因此我们只需删除单词 “Model” 将其更改为 “View” 我们就能找到它。这可以改变类型名称和名称空间。所以视图模型。因此 ViewModels.CustomerViewModel 将成为 Views.CustomerView。或者，如果你按功能组织应用程序：CustomerManagement.CustomerViewModel 将成为 CustomerManagement.CustomerView。希望这很简单，一旦我们有了名称，我们就会搜索具有该名称的类型。我们可以通过 AssemblySource.Instance.2 搜索你已公开给 CM 的任何程序集。如果我们找到了类型，我们就创建一个实例（如果已经注册，则从IoC容器中获取一个实例）并将其返回给调用者，如果找不到，我们会生成一个带有 “not found” 消息。</font>

Now, back to that “context” value. This is how CM supports multiple Views over the same ViewModel. If a context (typically a string or an enum) is provided, we do a further transformation of the name, based on that value. This transformation effectively assumes you have a folder (namespace) for the different views by removing the word “View” from the end and appending the context instead. So, given a context of “Master” our ViewModels.CustomerViewModel would become Views.Customer.Master.

---
> <font color="#63aebb" face="微软雅黑">现在，回到那个“context” 的值。这就是 CM 在同一 ViewModel 上支持多个视图的方式。如果提供了上下文（通常是字符串或枚举），我们将根据该值对名称进行进一步转换。通过从末尾删除单词 “View” 并附加上下文，此转换有效地假定你具有不同视图的文件夹（命名空间）。因此，给定 “Master” 的上下文，我们的ViewModels.CustomerViewModel 将成为 Views.Customer.Master。</font>

##### Other Things To Know - 其他要知道的

Besides instantiation of your View, GetOrCreateViewType will call InitializeComponent on your View (if it exists). This means that for Views created by the ViewLocator, you don’t have to have code-behinds at all. You can delete them if that makes you happy :) You should also know that ViewLocator.LocateForModelType is never called directly. It is always called indirectly through ViewLocator.LocateForModel. LocateForModel takes an instance of your ViewModel and returns an instance of your View. One of the functions of LocateForModel is to inspect your ViewModel to see if it implements IViewAware. If so, it will call it’s GetView method to see if you have a cached view or if you are handling View creation explicitly. If not, then it passes your ViewModel’s type to LocateForModelType.

---
> <font color="#63aebb" face="微软雅黑">除了实例化 View 之外，GetOrCreateViewType 将在 View 上调用 InitializeComponent（如果存在）。这意味着对于 ViewLocator 创建的视图，你根本不必拥有代码隐藏。你可以删除它们，希望这能让你开心:)。你也应该知道 ViewLocator.LocateForModelType 永远不会被直接调用。它总是通过 ViewLocator.LocateForModel 间接调用。LocateForModel 获取 ViewModel 的实例并返回 View 的实例。LocateForModel 的一个功能是检查 ViewModel 以查看它是否实现了 IViewAware。如果是，它将调用 GetView 方法来查看你是否有缓存视图，或者是否显式处理 View 创建。如果没有，那么它将你的 ViewModel 类型传递给 LocateForModelType。</font>

##### Customization - 自定义

The out-of-the-box convention is pretty simple and based on a number of patterns we’ve used and seen others use in the real world. However, by no means are you limited to these simple patterns. You’ll notice that all the methods discussed above are implemented as Funcs rather than actual methods. This means that you can customize them by simply replacing them with your own implementations. If you just want to add to the existing behavior, simply store the existing Func in a variable, create a new Func that calls the old and and assign the new Func to ViewLocator.LocateForModelType.

---
> <font color="#63aebb" face="微软雅黑">开箱即用的约定非常简单，它基于我们在现实世界中使用和看到的许多模式。但是，绝不仅限于这些简单的模式。你会注意到上面讨论的所有方法都是以 Funcs 而不是实际方法实现的。这意味着你可以通过简单地将它们替换为你自己的实现来自定义它们。如果你只想添加到现有行为，只需将现有 Func 存储在变量中，创建一个调用旧函数的新 Func，并将新 Func 分配给 ViewLocator.LocateForModelType。</font>

**v1.1 Changes** In v1.1 we've completely changed the implementation of the LocateForModelType func. We now use an instance of our new NameTransformer class with pre-configured RexEx-based rules to do the name mapping. We support the same conventions out of the box as before, but you can now more easily add custom transformation rules.

---
> **v1.1 的改变** <font color="#63aebb" face="微软雅黑">在 v1.1 中的变化我们完全改变了 LocateForModelType func 的实现。现在，我们使用新 NameTransformer 类的一个实例和预配置的基于 RexEx-based 的规则来进行名称映射。我们支持与以前一样的开箱即用的相同约定，但你现在可以更轻松地添加自定义转换规则。</font>

##### Framework Usage - 框架用法

There are three places that the framework uses the ViewLocator; three places where you can expect the view location conventions to be applied. The first place is in Bootstrapper<T>. Here, your root ViewModel is passed to the locator in order to determine how your application’s shell should be rendered. In Silverlight this results in the setting or your RootVisual. In WPF, this creates your MainWindow. In fact, in WPF the bootstrapper delegates this to the WindowManager, which brings me to… The second place the ViewLocator is used is the WindowManager, which calls it to determine how any dialog ViewModels should be rendered. The third and final place that leverages these conventions is the View.Model attached property. Whenever you do ViewModel-First composition rendering by using the View.Model attached property on a UIElement, the locator is invoked to see how that composed ViewModel should be rendered at that location in the UI. You can use the View.Model attached property explicitly in your UI (optionally combining it with the View.Context attached property for contextual rendering), or it can be added by convention, thus causing conventional composition of views to occur. See the section below on property binding conventions.

---
> <font color="#63aebb" face="微软雅黑">框架使用 ViewLocator 有三个地方; 你可以期望应用视图位置约定的三个位置。第一个是 Bootstrapper。在这里，你的根 ViewModel 将传递给 Bootstrapper<T>，以确定应该如何呈现应用程序的 shell。在 Silverlight 中，会设置RootVisual。在 WPF 中，这将创建你的 MainWindow。实际上，在WPF中，引导程序将这个委托给WindowManager，这让我想到…使用ViewLocator的第二个地方是WindowManager，它调用它来确定应该如何呈现任何对话框 ViewModel。利用这些约定的第三个也是最后一个是 View.Model 附加属性。每当你使用 UIElement 上的 View.Model  附加属性执行ViewModel-First 组合渲染时，调用定位器以查看应如何在 UI 中的该位置呈现组合的 ViewModel。你可以在UI中显式使用 View.Model 附加属性（可选地将其与用于上下文呈现的 View.Context 附加属性组合），或者可以按惯例添加它，从而导致产生传统的视图组合。请参阅下面有关属性绑定约定的部分。</font>

### ViewModel Resolution (View-First) - ViewModel 解析 (View-First)

##### Basics - 基础

Though Caliburn.Micro prefers ViewModel-First development, there are times when you may want to take a View-First approach, especially when working with WP7. In the case where you start with a view, you will likely then need to resolve a ViewModel. We use a similar naming convention for this scenario as we did with view location. This is handled by ViewModelLocator.LocateForViewType. While with View location we change instances of “ViewModel” to “View”, with ViewModel location we change “View” to “ViewModel.” The other interesting difference is in how we get the instance of the ViewModel itself. Because your ViewModels may be registered by an interface or a concrete class we attempt to generate possible interface names as well. If we find a match, we resolve it from the IoC container.

---
> <font color="#63aebb" face="微软雅黑">虽然Caliburn.Micro更喜欢ViewModel-First开发，但有时你可能想采用View-First方法，特别是在使用WP7时。如果你从视图开始，则可能需要解析ViewModel。我们使用与此视图位置类似的命名约定。这由ViewModelLocator.LocateForViewType处理。使用 View 定位时，我们将 “ViewModel” 的实例更改为 “View”，使用 ViewModel 定位我们将 “View” 更改为 “ViewModel”。另一个有趣的区别在于我们如何获取ViewModel本身的实例。因为ViewModel可能是由接口或具体类注册的，所以我们也尝试生成可能的接口名称。如果我们找到匹配项，我们将从IoC容器中解析它。</font>

##### Other Things To Know - 其他需要知道的

ViewModelLocator.LocateForViewType is actually never called directly by the framework. It’s called internally by ViewModelLocator.LocateForView. LocateForView first checks your View instance’s DataContext to see if you’ve previous cached or custom created your ViewModel. If the DataContext is null, only then will it call into LocateForViewType. A final thing to note is that automatic InitializeComponent calls are not supported by view first, by its nature.

---
> <font color="#63aebb" face="微软雅黑">ViewModelLocator.LocateForViewType 实际上永远不会被框架直接调用。它由 ViewModelLocator.LocateForView 在内部调用。LocateForView 首先检查 View 实例的 DataContext，看你是否已缓存或自定义 ViewModel。如果 DataContext 为 null，则只会调用 LocateForViewType。最后要注意的是，View First 不支持自动 InitializeComponent 调用。</font>

##### Customization - 自定义

In v1.1 we've completely changed the implementation of the LocateForViewType func. We now use an instance of our new NameTransformer class with pre-configured RexEx-based rules to do the name mapping. We support the same conventions out of the box as before, but you can now more easily add custom transformation rules.

---
> <font color="#63aebb" face="微软雅黑">在v1.1中，我们完全改变了 LocateForViewType func 的实现。现在，我们使用新 NameTransformer 类的实例和预配置的基于 RexEx-based 规则来进行名称映射。我们支持与以前一样的开箱即用约定，但是现在你可以更轻松地添加定制转换规则。</font>

##### Framework Usage - 框架使用

The ViewModelLocator is only used by the WP7 version of the framework. It’s used by the FrameAdapter which insures that every time you navigate to a page, it is supplied with the correct ViewModel. It could be easily adapted for use by the Silverlight Navigation Framework if desired.

---
> <font color="#63aebb" face="微软雅黑">ViewModelLocator 仅由WP7 框架版本的使用。由 FrameAdapter 确保每次导航到页面时，都会提供正确的 ViewModel。它可以很容易地适用于 Silverlight 导航框架。</font>

### ViewModelBinder

##### Basics - 基础

When we bind together your View and ViewModel, regardless of whether you use a ViewModel-First or a View-First approach, the ViewModelBinder.Bind method is called. This method sets the Action.Target of the View to the ViewModel and correspondingly sets the DataContext to the same value.4 It also checks to see if your ViewModel implements IViewAware, and if so, passes the View to your ViewModel. This allows for a more SupervisingController style design, if that fits your scenario better. The final important thing the ViewModelBinder does is determine if it needs to create any conventional property bindings or actions. To do this, it searches the UI for a list of element candidates for bindings/actions and compares that against the properties and methods of your ViewModel. When a match is found, it creates the binding or the action on your behalf.

---
> <font color="#63aebb" face="微软雅黑">当我们将View和ViewModel绑定在一起时，无论你使用的是ViewModel-First还是View-First方法，都会调用ViewModelBinder.Bind方法。此方法将View的Action.Target设置为ViewModel并相应地将DataContext设置为相同的值。它还会检查ViewModel是否实现IViewAware，如果是，则将View传递给ViewModel。为了更适合你的场景，允许更多的 SupervisingController 样式设计。ViewModelBinder 最重要的事情是确定它是否需要创建任何传统的属性绑定或操作。为此，它在 UI 中搜索绑定/操作的候选元素列表，并将其与 ViewModel 的属性和方法进行比较。找到匹配项后，它会为你创建绑定或操作。</font>

##### Other Things To Know - 其他需要知道的

On all platforms, conventions cannot by applied to the contents of a DataTemplate. This is a current limitation of the Xaml templating system. I have asked Microsoft to fix this, but I doubt they will respond. As a result, in order to have Binding and Action conventions applied to your DataTemplate, you must add a Bind.Model="{Binding}" attached property to the root element inside the DataTemplate. This provides the necessary hook for Caliburn.Micro to apply its conventions each time a UI is instantiated from a DataTemplate.

On the WP7 platform, if the View you are binding is a PhoneApplicationPage, this service is responsible for wiring up actions to the ApplicationBar’s Buttons and Menus. See the WP7 specific docs for more information on that.

---
> <font color="#63aebb" face="微软雅黑">在所有平台上，约定不能应用于 DataTemplate 的内容。这是当前 Xaml 模板系统的限制。我已经要求微软解决这个问题，但我怀疑他们会做出回应。因此，为了将 Binding 和 Action 约定应用于 DataTemplate，必须将 Bind.Model = “{Binding}”附加属性添加到 DataTemplate 内的根元素。这为 Caliburn.Micro 提供了必要的钩子，以便在每次从 DataTemplate 实例化UI时应用它的约定。</font>

---
> <font color="#63aebb" face="微软雅黑">在WP7平台上，如果你要绑定的View是PhoneApplicationPage，则此服务负责将操作连接到ApplicationBar的按钮和菜单。有关详细信息，请参阅WP7特定文档。</font>

##### Customization - 自定义

Should you decide that you don’t like the behavior of the ViewModelBinder (more details below), it follows the same patterns as the above framework services. It has several Funcs you can replace with your own implementations, such as Bind, BindActions and BindProperties. Probably the most important aspect of customization though, is the ability to turn off the binder’s convention features. To do this, set ViewModelBinder.ApplyConventionsByDefault to false. If you want to enable it on a view-by-view basis, you can set the View.ApplyConventions attached property to true on your View. This attached property works both ways. So, if you have conventions on by default, but need to turn them off on a view-by-view basis, you just set this property to false.

---
> <font color="#63aebb" face="微软雅黑">如果你决定不喜欢ViewModelBinder的行为（下面有更多详细信息），它将遵循与上述框架服务相同的模式。它有几个Func可以替换为你自己的实现，例如Bind，BindActions和BindProperties。自定义最重要的方面可能是关闭绑定器的约定功能。为此，请将 ViewModelBinder.ApplyConventionsByDefault 设置为 false。如果要在逐个视图的基础上启用它，可以在 View 上将 View.ApplyConventions  附加属性设置为true。这个附加的属性是双向的。因此，如果你默认启用约定，但需要在逐个视图的基础上关闭它们，则只需将此属性设置为 false 即可。</font>

##### Framework Usage - 框架使用

The ViewModelBinder is used in three places inside of Caliburn.Micro. The first place is inside the implementation of the View.Model attached property. This property takes your ViewModel, locates a view using the ViewLocator and then passes both of them along to the ViewModelBinder. After binding is complete, the View is injected inside the element on which the property is defined. That’s the ViewModel-First usage pattern. The second place that uses the ViewModelBinder is inside the implementation of the Bind.Model attached property. This property takes a ViewModel and passes it along with the element on which the property is defined to the ViewModelBinder. In other words, this is View-First, since you have already instantiated the View inline in your Xaml and are then just invoking the binding against a ViewModel. The final place that the ViewModelBinder is used is in the WP7 version of the framework. Inside of the FrameAdapter, when a page is navigated to, the ViewModelLocator is first used to obtain the ViewModel for that page. Then, the ViewModelBinder is uses to connect the ViewModel to the page.

---
> <font color="#63aebb" face="微软雅黑">ViewModelBinder 用于 Caliburn.Micro 内的三个地方。首先是 View.Model 附加属性的实现。此属性使用 ViewModel，使用 ViewLocator 定位视图，然后将它们传递给 ViewModelBinder。绑定完成后，View 将在定义属性的元素内注入。这是 ViewModel-First 使用模式。第二个地方是 Bind.Model 附加属性的实现。此属性采用 ViewModel 并将其与定义属性的元素一起传递给 ViewModelBinder。换句话说，这是View-First，因为你已经在 Xaml 中实例化了 View inline，然后只是调用 ViewModel 的绑定。最后是 WP7 框架的版本。在 FrameAdapter 内部，当页面导航到时，ViewModelLocator 首先用于获取该页面的ViewModel。然后，ViewModelBinder 用于将 ViewModel 连接到页面。</font>

### Element Location - 元素定位

##### Basics - 基础

Now that you understand the basic role of the ViewModelBinder and where it is used by the framework, I want to dig into the details of how it applies conventions. As mentioned above, the ViewModelBinder “searches the UI for a list of element candidates for bindings/actions and compares that against the properties and methods of your ViewModel.” The first step in understanding how this works is knowing how the framework determines which elements in your UI may be candidates for conventions. It does this by using a func on the static ExtensionMethods class called GetNamedElementsInScope.5 Basically this method does two things. First, it identifies a scope to search for elements in. This means it walks the tree until it finds a suitable root node, such as a Window, UserControl or element without a parent (indicating that we are inside a DataTemplate). Once it defines the “outer” borders of the scope, it begins it’s second task: locating all elements in that scope that have names. The search is careful to respect an “inner” scope boundary by not traversing inside of child user controls. The elements that are returned by this function are then used by the ViewModelBinder to apply conventions.

---
> <font color="#63aebb" face="微软雅黑">现在你已经了解了 ViewModelBinder 的基本角色以及框架在哪里使用它，接下来我将深入了解它如何应用约定。如上所述，ViewModelBinder 在UI中搜索绑定/操作的候选元素列表，并将其与ViewModel的属性和方法进行比较。”了解其工作原理的第一步是了解框架如何确定哪些元素在你的UI可能是约定的候选者。它通过在名为GetNamedElementsInScope的静态ExtensionMethods类上使用func来实现这一点。基本上这个方法做了两件事，首先，它确定了搜索元素的范围。这意味着它将遍历树，直到找到合适的根节点，例如没有父节点的 Window、UserControl 或元素(表明我们在 DataTemplate 中)。一旦定义了作用域的“外部”边界，它就开始了它的第二个任务：找到该作用域中具有名称的所有元素。搜索不遍历子用户控件内部来遵守“内部”范围边界。然后，ViewModelBinder 使用此函数返回的元素来应用约定。</font>

##### Other Things To Know - 其他需要知道的

There are a few limitations to what the GetNamedElementsInScope method can accomplish out-of-the-box. It can only search the visual tree, ContentControl.Content and ItemsControl.Items. In WPF, it also searches HeaderContentControl.Header and HeaderedItemsControl.Header. What this means is that things like ContextMenus, Tooltips or anything else that isn’t in the visual tree or one of these special locations won’t get found when trying to apply conventions.

---
> <font color="#63aebb" face="微软雅黑">GetNamedElementsInScope 方法可以实现开箱即用的功能有一些限制。它只能搜索可视树，ContentControl.Content 和 ItemsControl.Items。在 WPF 中，它还搜索 HeaderContentControl.Header 和 HeaderedItemsControl.Header。这意味着在尝试应用约定时，将无法找到 ContextMenus，Tooltips 或其他任何不在可视树或这些特殊位置中的内容。</font>

##### Customization - 自定义

You may not encounter issues related to the above limitations of element location. But if you do, you can easily replace the default implementation with your own. Here’s an interesting technique you might choose to use: If the view is a UserControl or Window, instead of walking the tree for elements, use some reflection to discover all private fields that inherit from FrameworkElement. We know that when Xaml files are compiled, a private field is created for everything with an x:Name. Use this to your advantage. You will have to fall back to the existing implementation for DataTemplate UI though. I don’t provide this implementation out-of-the-box, because it is not guaranteed to succeed in Silverlight. The reason is due to the fact that Silverlight doesn’t allow you to get the value of a private field unless the calling code is the code that defines the field. However, if all of your views are defined in a single assembly, you can easily make the modification I just described by creating your new implementation in the same assembly as the views. Furthermore, if you have a multi-assembly project, you could write a little bit of plumbing code that would allow the GetNamedElementsInScope Func to find the assembly-specific implementation which could actually perform the reflection.

---
> <font color="#63aebb" face="微软雅黑">你可能不会遇到与上述元素位置限制相关的问题。但是如果你这样做了，你可以很容易地用自己的实现替换默认的实现。这里有一个你可以选择使用的有趣技术:如果视图是 UserControl 或 Window，而不是遍历树中的元素，那么使用一些反射来发现从 FrameworkElement 继承的所有私有字段。我们知道，在编译 Xaml 文件时，会为所有 x:Name 创建一个私有字段。利用这个优势，不过，你必须返回 DataTemplate UI的现有实现。我不提供开箱即用的实现，因为它不能保证在 Silverlight 中成功。原因是 Silverlight 不允许获取私有字段的值，除非调用代码是定义字段的代码。但是，如果你的所有视图都在一个程序集中定义，则可以通过在与视图相同的程序集中创建新实现来轻松地进行我刚刚描述的修改。此外，如果你有一个多组件项目，你可以编写一些管道代码，以允许 GetNamedElementsInScope Func 查找可以实际执行反射的特定于程序集的实现。</font>

##### Framework Usage - 框架使用

I’ve already mentioned that element location occurs when the ViewModelBinder attempts to bind properties or methods by convention. But, there is a second place that uses this functionality: the Parser. Whenever you use Message.Attach and your action contains parameters, the message parser has to find the elements that you are using as parameter inputs. It would seem that we could just do a simple FindName, but FindName is case-sensitive. As a result, we have to use our custom implementation which does a case-insensitive search. This ensures that the same semantics for binding are used in both places.

---
> <font color="#63aebb" face="微软雅黑">我已经提到，当 ViewModelBinder 尝试按约定绑定属性或方法时，会定位元素位置。但是，还有第二个使用此功能的地方：Parser。每当你使用 Message.Attach 并且你的操作包含参数时，消息解析器必须找到你用作参数输入的元素。看起来我们可以只做一个简单的 FindName，但 FindName 是区分大小写的。因此，我们必须使用我们的自定义实现，它执行不区分大小写的搜索。这确保了在两个地方都使用相同的绑定语义。</font>

### Action Matching - 活动匹配

##### Basics - 基础

The next thing the ViewModelBinder does after locating the elements for convention bindings is inspect them for matches to methods on the ViewModel. It does this by using a bit of reflection to get the public methods of the ViewModel. It then loops over them looking for a case-insensitive name match with an element. If a match is found, and there aren’t any pre-existing Interaction.Triggers on the element, an action is attached. The check for pre-existing triggers is used to prevent the convention system from creating duplicate actions to what the developer may have explicitly declared in the markup. To be on the safe side, if you have declared any triggers on the matched element, it is skipped.

---
> <font color="#63aebb" face="微软雅黑">在定位了约定绑定的元素之后，ViewModelBinder 接下来会检查它们是否与 ViewModel 上的方法匹配。它通过使用一些反射来获取 ViewModel 的公共方法。然后它循环遍历它们，寻找与元素不区分大小写的名称。如果找到匹配项，并且元素上没有任何预先存在的 Interaction.Triggers，则会附加一个操作。检查预先存在的触发器用于防止约定系统对开发人员可能在标记中明确声明的内容创建重复操作。为了安全起见，如果你在匹配的元素上声明了触发器，则会跳过它。</font>

##### Other Things To Know - 其他需要知道的

常规操作是通过在元素上设置 Message.Attach 附加属性创建的。让我们看看它是如何建立起来的:

``` csharp
var message = method.Name;
var parameters = method.GetParameters();

if(parameters.Length > 0)
{
    message += "(";

    foreach(var parameter in parameters)
    {
        var paramName = parameter.Name;
        var specialValue = "$" + paramName.ToLower();

        if(MessageBinder.SpecialValues.Contains(specialValue))
            paramName = specialValue;

        message += paramName + ",";
    }

    message = message.Remove(message.Length - 1, 1);
    message += ")";
}

Log.Info("Added convention action for {0} as {1}.", method.Name, message);
Message.SetAttach(foundControl, message);
```

As you can see, we build a string representing the message. This string contains only the action part of the message; no event is declared. You can also see that it loops through the parameters of the method so that they are included in the action. If the parameter name is the same as a special parameter value, we make sure to append the “$” to it so that it will be recognized correctly by the Parser and later by the MessageBinder when the action is invoked.

---
> <font color="#63aebb" face="微软雅黑">如你所见，我们构建了一个表示消息的字符串。此字符串仅包含消息的操作部分;没有声明任何事件。你还可以看到它循环遍历方法的参数，以便它们包含在操作中。如果参数名称与特殊参数值相同，我们确保将 “$” 附加到它，以便解析器能够正确地识别它，由 MessageBinder 调用。</font>

When the Message.Attach property is set, the Parser immediately kicks in to convert the string message into some sort of TriggerBase with an associated ActionMessage. Because we don’t declare an event as part of the message, the Parser looks up the default Trigger for the type of element that the message is being attached to. For example, if the message was being attached to a Button, then we would get an EventTrigger with it’s Event set to Click. This information is configured through the ConventionManager with reasonable defaults out-of-the-box. See the sections on ConventionManager and ElementConventions below for more information on that. The ElementConvention is used to create the Trigger and then the parser converts the action information into an ActionMessage. The two are connected together and then added to the Interaction.Triggers collection of the element.

---
> <font color="#63aebb" face="微软雅黑">设置 Message.Attach 属性后，解析器立即开始将字符串消息转换为具有关联 ActionMessage 的 TriggerBase。因为我们没有将事件声明为消息的一部分，因此解析器将查找附加消息的元素类型的默认触发器。例如，如果消息被附加到 Button ，那么我们将得到一个 EventTrigger，其 Event 设置为 Click。此信息通过 ConventionManager 配置，具有合理的默认开箱即用功能。有关详细信息，请参阅下面有关 ConventionManager 和 ElementConventions。ElementConvention 用于创建 Trigger，然后解析器将操作信息转换为 ActionMessage。两者连接在一起，然后添加到交互中，触发元素的集合。</font>

##### Customization - 自定义

ViewModelBinder.BindActions is a Func and thus can be entirely replaced if desired. Adding to or changing the ElementConventions via the ConventionManager will also effect how actions are put together. More on that below.

---
> <font color="#63aebb" face="微软雅黑">ViewModelBinder.BindActions 是一个 Func，因此可以根据需要完全替换。通过 ConventionManager 添加或更改 ElementConventions 也会影响操作的组合方式。下面详细介绍。</font>

##### Framework Usage - 框架使用

BindActions 只被 ViewModelBinder 使用。

### Property Matching - 属性匹配

##### Basics - 基础

Once action binding is complete, we move on to property binding. It follows a similar process by looping through the named elements and looking for case-insensitive name matches on properties. Once a match is found, we then get the ElementConventions from the ConventionManager so we can determine just how databinding on that element should occur. The ElementConvention defines an ApplyBinding Func that takes the view model type, property path, property info, element instance, and the convention itself. This Func is responsible for creating the binding on the element using all the contextual information provided. The neat thing is that we can have custom binding behaviors for each element if we want. CM defines a basic implementation of ApplyBinding for most elements, which lives on the ConventionManager. It’s called SetBinding and looks like this:

---
> <font color="#63aebb" face="微软雅黑">当 Action 绑定完成，我们将继续属性绑定。它通过循环遍历已命名的元素并在属性上查找不区分大小写的名称匹配。找到匹配后，我们从 ConventionManager 获取 ElementConventions，这样我们就可以确定该元素的数据绑定应该如何发生。ElementConvention 定义了一个 ApplyBinding Func，它接受视图模型类型，属性路径，属性信息，元素实例和约定本身。此 Func 负责使用提供的所有上下文信息在元素上创建绑定。有趣的是，我们可以为每个元素提供自定义绑定行为。CM 为大多数元素定义了 ApplyBinding 的基本实现，它们存在于  ConventionManager中。它是  SetBinding，代码如下：</font>

``` csharp
public static Func<Type, string, PropertyInfo, FrameworkElement, ElementConvention, bool> SetBinding =
    (viewModelType, path, property, element, convention) => {
        var bindableProperty = convention.GetBindableProperty(element);
        if(HasBinding(element, bindableProperty))
            return false;

        var binding = new Binding(path);

        ApplyBindingMode(binding, property);
        ApplyValueConverter(binding, bindableProperty, property);
        ApplyStringFormat(binding, convention, property);
        ApplyValidation(binding, viewModelType, property);
        ApplyUpdateSourceTrigger(bindableProperty, element, binding);

        BindingOperations.SetBinding(element, bindableProperty, binding);

        return true;
    };
```

The first thing this method does is get the dependency property that should be bound by calling GetBindableProperty on the ElementConvention. Next we check to see if there is already a binding set for that property. If there is, we don’t want to overwrite it. The developer is probably doing something special here, so we return false indicating that a binding has not been added. Assuming no binding exists, this method then basically delegates to other methods on the ConventionManager for the details of binding application. Hopefully that part makes sense. Once the binding is fully constructed we add it to the element and return true indicating that the convention was applied.

---
> <font color="#63aebb" face="微软雅黑">该方法的第一件事就是通过在 ElementConvention 上调用 GetBindableProperty 来获取应该绑定的依赖项属性。接下来，我们检查是否已经存在该属性的绑定集。如果有，我们不覆盖它。开发人员可能在这里做了一些特别的事情，所以我们返回 false 表示没有添加绑定。假设不存在绑定，则此方法基本上委托给 ConventionManager 上的其他方法以获取绑定应用程序的详细信息，希望这部分是有道理的。一旦完成绑定初始化，我们将它添加到元素并返回 true 表示已应用约定。</font>

There’s another important aspect to property matching that I haven’t yet mentioned. We can match on deep property paths by convention as well. So, let’s say that you have a Customer property on your ViewModel that has a FirstName property you want to bind a Textbox to. Simply give the TextBox an x:Name of “Customer_FirstName” The ViewModelBinder will do all the work to make sure that that is a valid property and will pass the correct view model type, property info and property path along to the ElementConvention’s ApplyBinding func.

---
> <font color="#63aebb" face="微软雅黑">属性匹配还有另一个重要方面我还没有提到。我们也可以按照惯例匹配深层属性路径。假设你的 ViewModel 上有一个 Customer 属性，该属性具有要将 Textbox 绑定到的 FirstName 属性。只需TextBox 的 x:Name="Customer_FirstName" ViewModelBinder将完成所有工作，以确保它是一个有效的属性，并将正确的视图模型类型，属性信息和属性路径传递给 ElementConvention 的 ApplyBinding 函数。</font>

##### Other Things To Know - 其他需要知道的

I mentioned above that “CM defines a basic implementation of ApplyBinding for most elements.” It also defines several custom implementations of the ApplyBinding Func for elements that are typically associated with specific usage patterns or composition. For WPF and Silverlight, there are custom binding behaviors for ItemsControl and Selector. In addition to binding the ItemsSource on an ItemsControl, the ApplyBinding func also inspects the ItemTemplate, DisplayMemberPath and ItemTemplateSelector (WPF) properties. If none of these are set, then the framework knows that since you haven’t specified a renderer for the items, it should add one conventionally.7 So, we set the ItemTemplate to a default DataTemplate. Here’s what it looks like:

---
> <font color="#63aebb" face="微软雅黑">我在上面提到“ CM 为大多数元素定义了 ApplyBinding 的基本实现”。它还为通常与特定使用模式或组合相关联的元素定义了 ApplyBinding Func 的几个自定义实现。对于 WPF 和 Silverlight，ItemsControl 和 Selector 有自定义绑定行为。除了在 ItemsControl 上绑定 ItemsSource 之外，ApplyBinding func 还检查 ItemTemplate，DisplayMemberPath 和 ItemTemplateSelector（WPF）属性。如果没有设置这些，那么框架知道由于你没有为项目指定渲染器，它应该按常规添加一个。因此，我们将 ItemTemplate设置为默认的 DataTemplate。如下：</font>

``` xml
<DataTemplate xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
              xmlns:cal="clr-namespace:Caliburn.Micro;assembly=Caliburn.Micro">
    <ContentControl cal:View.Model="{Binding}" 
                    VerticalContentAlignment="Stretch"
                    HorizontalContentAlignment="Stretch" />
</DataTemplate>
```

Since this template creates a ContentControl with a View.Model attached property, we create the possibility of rich composition for ItemsControls. So, whatever the Item is, the View.Model attached property allows us to invoke the ViewModel-First workflow: locate the view for the item, pass the item and the view to the ViewModelBinder (which in turn sets it’s own conventions, possibly invoking more composition) and then take the view and inject it into the ContentControl. Selectors have the same behavior as ItemsControls, but with an additional convention around the SelectedItem property. Let’s say that your Selector is called Items. We follow the above conventions first by binding ItemsSource to Items and detecting whether or not we need to add a default DataTemplate. Then, we check to see if the SelectedItem property has been bound. If not, we look for three candidate properties on the ViewModel that could be bound to SelectedItem: ActiveItem, SelectedItem and CurrentItem. If we find one of these, we add the binding. So, the pattern here is that we first call ConventionManager.Singularize on the collection property’s name. In this case “Items” becomes “Item” Then we call ConventionManager.DerivePotentialSelectionNames which prepends “Active” “Selected” and “Current” to “Item” to make the three candidates above. Then, we create a binding if we find one of these on the ViewModel. For WPF, we have a special ApplyBinding behavior for the TabControl.8 It takes on all the conventions of Selector (setting it’s ContentTemplate instead of ItemTemplate to the DefaultDataTemplate), plus an additional convention for the tab header’s content. If the TabControl’s DisplayMemberPath is not set and the ViewModel implements IHaveDisplayName, then we set it’s ItemTemplate to the DefaultHeaderTemplate, which looks like this:

---
> <font color="#63aebb" face="微软雅黑">由于此模板创建了一个带 有 View.Model 附加属性的 ContentControl，因此我们为 ItemsControls 创建了丰富组合的可能性。因此，无论 Item 是什么，View.Model 附加属性允许我们调用 ViewModel-First 工作流：找到项的视图，将项和视图传递给 ViewModelBinder（它依次设置自己的约定，可能调用更多的组合）然后获取视图并将其注入 ContentControl。选择器具有与 ItemsControls 相同的行为，但是有一个关于 SelectedItem 属性的附加约定。假设你的选择器名为 Items。我们首先遵循上述约定，将 ItemsSource 绑定到项，并检测是否需要添加默认 DataTemplate。然后，我们检查 SelectedItem 属性是否已绑定。如果没有，我们在 ViewModel 上查找三个可能绑定到 SelectedItem 的候选属性:ActiveItem，SelectedItem和CurrentItem。如果找到其中一个，就添加绑定。所以，这里的模式是我们首先在集合属性的名称上调用 ConventionManager.Singularize。在这种情况下， “Items” 变为 “Item” 然后我们调用 ConventionManager.DerivePotentialSelectionNames，它将 “Active”、“Selected” 和 “Current” 添加到 “Item” 以构成上述三个候选项。然后，如果我们在 ViewModel 上找到其中一个，我们就会创建一个绑定。对于 WPF，我们对 TabControl 具有特殊的 ApplyBinding 行为。它采用 Selector 的所有约定（将它的 ContentTemplate 而不是 ItemTemplate 设置为 DefaultDataTemplate），并为 tab Header 的内容添加一个额外的约定。如果未设置 TabControl 的 DisplayMemberPath，而 ViewModel 实现了IHaveDisplayName，那么我们将它的 ItemTemplate 设置为 DefaultHeaderTemplate，如下所示:</font>

``` xml
<DataTemplate xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation">
    <TextBlock Text="{Binding DisplayName, Mode=TwoWay}" />
</DataTemplate>
```

So, for a named WPF TabControl we can conventionally bind in the list of tabs (ItemsSource), the tab item’s name (ItemTemplate), the content for each tab (ContentTemplate) and keep the selected tab synchronized with the model (SelectedItem). That’s not bad for one line of Xaml like this:

---
> <font color="#63aebb" face="微软雅黑">因此，对于命名的WPF TabControl，我们通常可以在选项卡列表（ItemsSource），选项卡项的名称（ItemTemplate），每个选项卡的内容（ContentTemplate）中绑定，并使选定的选项卡与模型（SelectedItem）保持同步。对于像这样的一行Xaml来说，这不错：</font>

``` xml
<TabControl x:Name="Items" />
```

In addition to the special cases listed above, we have one more that is important: ContentControl. In this case, we don’t supply a custom ApplyBinding Func, but we do supply a custom GetBindableProperty Func. For ContentControl, when we go to determine which property to bind to, we inspect the ContentTemplate and ContentTemplateSelector (WPF). If they are both null, you haven’t specified a renderer for your model. Therefore, we assume that you want to use the ViewModel-First workflow. We implement this by having the GetBindableProperty Func return the View.Model attached property as the property to be bound. In all other cases ContentControl would be bound on the Content property. By selecting the View.Model property in the absence of a ContentTemplate, we enable rich composition.

I hope you will find that these special cases make sense when you think about them. As always, if you don’t like them, you can change them…

---
> <font color="#63aebb" face="微软雅黑">除了上面列出的特殊情况之外，我们还有一个非常重要的内容：ContentControl。在本例中，我们不提供自定义的 ApplyBinding Func，但我们提供了自定义的 GetBindableProperty Func。对于 ContentControl，当我们决定绑定到哪个属性时，我们检查 ContentTemplate 和 ContentTemplateSelector (WPF)。如果它们都为 null，则没有为模型指定渲染器。因此，我们假设你要使用 ViewModel-First 工作流程。我们通过让 GetBindableProperty Func 返回 View.Model 作为要绑定的属性。其它情况下，ContentControl 将绑定在 Content 属性上。在没有 ContentTemplate 的情况下选择 View.Model 属性，我们启用了丰富的组合。<br>

> 我希望你会发现这些特殊的情况在你们思考的时候是有意义的。像往常一样，如果你不喜欢它们，你可以改变它们……</font>

##### Customization - 自定义

As you might imagine, the BindProperties functionality is completely customizable by replacing the Func on the ViewModelBinder. For example, if you like the idea of Action conventions but not Property conventions, you could just replace this Func with one that doesn’t do anything. However, it’s likely that you will want more fine grained control. Fortunately, nearly every aspect of the ConventionManager or of a particular ElementConvention is customizable. More details about the ConventionManager are below.

One of the common ways you will configure conventions is by adding new conventions to the system. Most commonly this will be in adding the Silverlight toolkit controls or the WP7 toolkit controls. Here’s an example of how you would set up an advanced convention for the WP7 Pivot control which would make it work like the WPF TabControl:

---
> <font color="#63aebb" face="微软雅黑">如你所想，通过替换 ViewModelBinder 上的 Func，可以完全自定义 BindProperties 功能。如果你喜欢 Action 约定而不是 Property 约定，那么你可以将 Func 替换为不执行任何操作的 Func。你可能需要更细粒度的控制。幸运的是，ConventionManager 或特定 ElementConvention 的几乎每个方面都是可定制的。有关 ConventionManager 的更多详细信息如下。<br>

> 配置约定的常用方法之一是向系统添加新约定。最常见的是添加Silverlight工具包控件或WP7工具包控件。下面是一个如何为WP7 Pivot控件设置高级约定的示例，它将使其像WPF TabControl一样工作：</font>

``` csharp
ConventionManager.AddElementConvention<Pivot>(Pivot.ItemsSourceProperty, "SelectedItem", "SelectionChanged").ApplyBinding =
    (viewModelType, path, property, element, convention) => {
        ConventionManager
            .GetElementConvention(typeof(ItemsControl))
            .ApplyBinding(viewModelType, path, property, element, convention);
        ConventionManager
            .ConfigureSelectedItem(element, Pivot.SelectedItemProperty, viewModelType, path);
        ConventionManager
            .ApplyHeaderTemplate(element, Pivot.HeaderTemplateProperty, viewModelType);
    };
```

酷吗？

##### Framework Usage - 框架使用

BindProperties 仅由 ViewModelBinder 使用。

### Convention Manager - 约定管理

If you’ve read this far, you know that the ConventionManager is leveraged heavily by the Action and Property binding mechanisms. It is the gateway to fine-tuning the majority of the convention behavior in the framework. What follows is a list of the replaceable Funcs and Properties which you can use to customize the conventions of the framework:

---
> <font color="#63aebb" face="微软雅黑">如果你已经阅读过这篇文章，那么你就会知道，ActionManager和Property绑定机制会大量使用ConventionManager。它是微调框架中大多数约定行为的门户。以下是可替换的Func和Properties列表，你可以使用它们来自定义框架的约定：</font>

##### Properties - 属性

 - BooleanToVisibilityConverter – The default IValueConverter used for converting Boolean to Visibility and back. Used by ApplyValueConverter.
 - IncludeStaticProperties - Indicates whether or not static properties should be included during convention name matching. False by default.
 - DefaultItemTemplate – Used when an ItemsControl or ContentControl needs a DataTemplate.
 - DefaultHeaderTemplate – Used by ApplyHeaderTemplate when the TabControl needs a header template.

> - <font color="#63aebb" face="微软雅黑">BooleanToVisibilityConverter – 用于将 Boolean 转换为 Visibility 并返回的默认 IValueConverter。由 ApplyValueConverter 使用。</font>

> - <font color="#63aebb" face="微软雅黑">IncludeStaticProperties - 指示在约定名称匹配期间是否应包含静态属性。默认为False。</font>

> - <font color="#63aebb" face="微软雅黑">DefaultItemTemplate –  在 ItemsControl 或 ContentControl 需要 DataTemplate 时使用。</font>

> - <font color="#63aebb" face="微软雅黑">DefaultHeaderTemplate – 当TabControl需要标题模板时由ApplyHeaderTemplate使用。</font>

##### Funcs

 - Singularize – Turns a word from its plural form to its singular form. The default implementation is really basic and just strips the trailing ‘s’.
 - DerivePotentialSelectionNames – Given a base collection name, returns a list of possible property names representing the selection. Uses Singularize.
 - SetBinding – The default implementation of ApplyBinding used by ElementConventions (more info below). Changing this will change how all conventional bindings are applied. Uses the following Funcs internally:
	 - HasBinding - Determines whether a particular dependency property already has a binding on the provided element. If a binding already exists, SetBinding is aborted.
	 - ApplyBindingMode - Applies the appropriate binding mode to the binding.
	 - ApplyValidation - Determines whether or not and what type of validation to enable on the binding.
	 - ApplyValueConverter - Determines whether a value converter is is needed and applies one to the binding. It only checks for BooleanToVisibility conversion by default.
	 - ApplyStringFormat - Determines whether a custom string format is needed and applies it to the binding. By default, if binding to a DateTime, uses the format "{0:MM/dd/yyyy}".
	 - ApplyUpdateSourceTrigger - Determines whether a custom update source trigger should be applied to the binding. For WPF, always sets to UpdateSourceTrigger=PropertyChanged. For Silverlight, calls ApplySilverlightTriggers.

 As an example lets change `ConventionManager.Singularize` to use the awesome library [Humanizer](https://github.com/Humanizr/Humanizer).

---
> - <font color="#63aebb" face="微软雅黑">Singularize – 将一个单词从复数形式转变为单数形式。默认实现是非常基本的，只是删除尾随的's'。</font>

> - <font color="#63aebb" face="微软雅黑">DerivePotentialSelectionNames – 给定基本集合名称，返回表示选择的可能属性名称列表。使用Singularize。</font>

> - <font color="#63aebb" face="微软雅黑">SetBinding – ElementConventions使用的ApplyBinding的默认实现（更多信息如下）。更改此项将更改所有常规绑定的应用方式。在内部使用以下Func：<br>
>     - HasBinding - 确定特定依赖项属性是否已对提供的元素具有绑定。如果绑定已存在，则中止SetBinding。
>     - ApplyBindingMode - 将适当的绑定模式应用于绑定。
>     - ApplyValidation - 确定是否在绑定上启用哪种验证。
>     - ApplyValueConverter - 确定是否需要值转换器，并将值转换器应用于绑定。默认只检查 BooleanToVisibility。
>     - ApplyStringFormat - 确定是否需要自定义字符串格式，并将其应用于绑定。默认如果绑定到日期时间，则使用“{0:MM/dd/yyyy}” 的格式。
>     - ApplyUpdateSourceTrigger - 确定是否应将自定义更新源触发器应用于绑定。对于 WPF，始终设置为 UpdateSourceTrigger = PropertyChanged。对于 Silverlight，请调用 ApplySilverlightTriggers。</font>

> <font color="#63aebb" face="微软雅黑">一个例子，让我们改变ConventionManager.Singularize使用库 [Humanizer](https://github.com/Humanizr/Humanizer)。</font>

``` csharp
ConventionManager.Singularize = original => original.Singularize(inputIsKnownToBePlural: false);
```

##### Methods - 方法

 - AddElementConvention – Adds or replaces an ElementConvention.
 - GetElementConvention – Gets the convention for a particular element type. If not found, searches the type hierarchy for a match.
 - ApplyHeaderTemplate – Applies the header template convention to an element.
 - ApplySilverlightTriggers – For TextBox and PasswordBox, wires the appropriate events to binding updates in order to simulate WPF’s UpdateSourceTrigger=PropertyChanged.

---
> - <font color="#63aebb" face="微软雅黑">AddElementConvention – 添加或替换ElementConvention。</font>
> - <font color="#63aebb" face="微软雅黑">GetElementConvention – 获取特定元素类型的约定。如果没有找到，则在类型层次结构中搜索匹配。</font>
> - <font color="#63aebb" face="微软雅黑">ApplyHeaderTemplate – 将标题模板约定应用于元素。</font>
> - <font color="#63aebb" face="微软雅黑">ApplySilverlightTriggers – 对于 TextBox 和 PasswordBox，将相应的事件连接到绑定更新，以模拟 WPF 的 UpdateSourceTrigger = PropertyChanged。</font>

### ElementConvention

ElementConventions can be added or replaced through ConventionManager.AddElementConvention. But, it’s important to know what these conventions are and how they are used throughout the framework. At the very bottom of this article is a code listing showing how all the elements are configured out-of-the-box. Here are the Properties and Funcs of the ElementConvention class with brief explanations:

---
> <font color="#63aebb" face="微软雅黑">可以通过 ConventionManager.AddElementConvention 添加或替换 ElementConventions。但是，了解这些约定是什么以及如何在整个框架中使用它们非常重要。在本文的最底部的代码，展示了如何配置所有元素的开箱即用。以下是 ElementConvention 类的 Properties 和 Funcs，说明如下：</font>

##### Properties - 属性

 - ElementType – The type of element to which the convention applies.
 - ParameterProperty – When using Message.Attach to declare an action, if a parameter that refers to an element is specified, but the property of that element is not, the ElementConvention will be looked up and the ParameterProperty will be used. For example, if we have this markup:

---
> - <font color="#63aebb" face="微软雅黑">ElementType – 约定适用的元素的类型。</font>
> - <font color="#63aebb" face="微软雅黑">ParameterProperty – 使用 Message.Attach 声明操作时，如果指定了引用元素的参数，但该元素的属性不是，则将查找 ElementConvention 并使用 ParameterProperty。例如，如果我们有这个标记：</font>

``` csharp
<TextBox x:Name="something" />
<Button cal:Message.Attach="MyMethod(something)" />
```

 - When the Button’s ActionMessage is created, we look up “something”, which is a TextBox. We get the ElementConvention for TextBox which has its ParameterProperty set to “Text.” Therefore, we create the parameter for MyMethod from something.Text.

---
> - <font color="#63aebb" face="微软雅黑">当 Button 的 ActionMessage 被创建时，我们会查找 “something”，这是一个 TextBox。我们得到 TextBox 的 ElementConvention，它的 ParameterProperty 设置为 “Text”。因此，我们从 something.Text 为 MyMethod 的参数。</font>

##### Funcs

 - GetBindableProperty – Gets the property for the element which should be used in convention binding.
 - CreateTrigger – When Message.Attach is used to declare an Action, and the specific event is not specified, the ElementConvention will be looked up and the CreateTrigger Func will be called to create the Interaction.Trigger. For example in the Xaml above, when the ActionMessage is created for the Button, the Button’s ElementConvention will be looked up and its CreateTrigger Func will be called. In this case, the ElementConvention returns an EventTrigger configured to use the Click event.
 - ApplyBinding – As described above, when conventional databinding occurs, the element we are binding has its ElementConvention looked up and it’s ApplyBinding func is called. By default this just passes through to ConventionManager.SetBinding. But certain elements (see above…or below) customize this in order to enable more powerful composition scenarios.

There is a helper method on the ConventionManager called AddElementConvention which can be used like this:

---
> - <font color="#63aebb" face="微软雅黑">GetBindableProperty – 获取在约定绑定中应该使用的元素的属性。</font>

> - <font color="#63aebb" face="微软雅黑">CreateTrigger – 当 Message.Attach 用于声明 Action，并且未指定特定事件时，将查找 ElementConvention 并调用 CreateTrigger Func 以创建 Interaction.Trigger。例如，在上面的 Xaml 中，为 Button 创建 ActionMessage 时，将查找 Button 的 ElementConvention 并调用其 CreateTrigger Func。在这种情况下， ElementConvention 返回一个配置为使用 Click 事件的 EventTrigger。</font>

> - <font color="#63aebb" face="微软雅黑">ApplyBinding – 如上所述，当传统的数据绑定发生时，我们绑定的元素会查找其 ElementConvention，并调用它的 ApplyBinding 函数。默认情况下，这只是传递给 ConventionManager.SetBinding。但是某些元素（参见上文或下文）会对此进行自定义，以支持更强大的组合场景。</font>

> <font color="#63aebb" face="微软雅黑">在ConventionManager上有一个名为AddElementConvention的辅助方法，可以这样使用：</font>

``` csharp
ConventionManager.AddElementConvention<Rating>(Rating.ValueProperty, "Value", "ValueChanged");
```

In the case mentioned above, the first parameter value of Rating.ValueProperty tells the convention system what the default bindable property is for the element. So, if we have a convention match on a Rating control, we set up the binding against the ValueProperty. The second parameter represents the default property to be used in Action bindings. So, if you create an action binding with an ElementName that points to a Rating control, but do not specify the property, we fall back to the "Value" property. Finally, the third parameter represents the default event for the control. So, if we attach an action to a rating control, but don't specify the event to trigger that action, the system will fall back to the "ValueChanged" event. These element conventions allow the developer to supply as much or as little information in a variety of situations, allowing the framework to fill in the missing details as approptiate.

---
> <font color="#63aebb" face="微软雅黑">在上面提到的情况中，Rating.ValueProperty 的第一个参数值告诉约定系统该元素的默认可绑定属性是什么。因此，如果我们在 Rating 控件上有一个约定匹配，我们就设置了对 ValueProperty 的绑定。第二个参数表示要在 Action 绑定中使用的默认属性。因此，如果使用指向 Rating 控件的 ElementName 创建动作绑定，但未指定属性，则返回 “Value” 属性。最后，第三个参数表示控件的默认事件。因此，如果我们将操作附加到评级控件，但未指定事件来触发该操作，则系统将回退到 “ValueChanged” 事件。</font>

[目录](./index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[View / ViewModel Naming Conventions - 视图/视图模型命名约定](./naming-conventions.md)