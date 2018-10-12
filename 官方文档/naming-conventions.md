---
layout: page
title: View / View Model Naming Conventions
---

After receiving feedback about View and ViewModel resolution in Caliburn Micro, we have added new capabilities to make type resolution simpler while maintaining the robust regular expression-based name transformation mechanism that drives it. To get a better understanding about these new capabilities and how type resolution, in general, works in the framework, it's an appropriate time to describe in detail the naming conventions that the framework supports out-of-the-box. As you should already know by now, the framework depends considerably on naming conventions, and in type resolution specifically there are two different naming conventions to consider: the convention for naming the type itself and the convention for naming the namespace of the type.

---
> <font color="#63aebb" face="微软雅黑">在 Caliburn Micro 中收到有关 View 和 ViewModel 分辨率的反馈后，我们添加了新的功能，使类型解析更简单，同时保持驱动它的强大的基于正则表达式的名称转换机制。为了更好地理解这些新功能以及类型解析通常如何在框架中工作，现在是详细描述框架支持开箱即用的命名约定的适当时机。现在你应该已经知道，框架很大程度上依赖于命名约定，在类型解析中，有两种不同的命名约定要考虑:命名类型本身的约定和命名类型名称空间的约定。</font>

### Naming Conventions for Name of a Type - 类型名称的命名约定

As mentioned briefly in other areas of the documentation, the most common naming convention for a View and its companion ViewModel can be described as follows:

---
> <font color="#63aebb" face="微软雅黑">正如在文档的其他方面简要提到的，View 及其对应 ViewModel 的最常见命名约定可以描述如下：</font>

| &nbsp; | View Model | View |
|-|------------|------|
| Convention | &lt;EntityName&gt;ViewModel | &lt;EntityName&gt;View |
| Example 1 | ShellViewModel | ShellView |
| Example 2 | TabViewModel | TabView |

Because we recognize that "View" is an abstract term and that the primary "View" of most applications is actually some sort "Page", we believe that it's important for the framework to consider "Page" as a synonym for "View". Therefore, the framework has built-in support for this use case:

---
> <font color="#63aebb" face="微软雅黑">我们知道 “View” 是一个抽象术语，并且大多数应用程序的主要 “View” 实际上是某种 “Page”，我们认为框架将 “Page” 视为 “View” 的同义词非常重要。因此，该框架内置了对此用例的支持：</font>

| &nbsp; | View Model | View |
|-|------------|------|
| Convention | &lt;EntityName&gt;PageViewModel | &lt;EntityName&gt;Page |
| Example 1 | MainPageViewModel | MainPage |
| Example 2 | OrderPageViewModel | OrderPage |

If you examine closely, you'll see that there is a subtle difference between the two conventions above. "ViewModel" is simply added to a "Page"-suffixed name to yield the name of its ViewModel. However, only "Model" is added to a "View"-suffixed name to yield the name of its companion ViewModel. This difference primarily stems from the semantic awkwardness of naming something "MainViewViewModel" as opposed to "MainPageViewModel". Therefore, the naming convention for ViewModels derived from "View"-suffixed View names avoids the redundancy by naming the ViewModel as "MainViewModel".

One limitation of the standard naming conventions supported by the framework is that there isn't consideration for different languages or even different terminologies within English. Although "View" and "ViewModel" can be assumed to be universally understood, as they are both important aspects of the MVVM design pattern that Caliburn Micro is dedicated to, a word like "Page" is not. Therefore, a robust framework would at least allow for supporting additional "View name suffixes" (e.g. "Pagina", "Seite", "Form", "Screen") through customization.

---
> <font color="#63aebb" face="微软雅黑">如果仔细研究，你会发现上述两个约定之间存在细微差别。“ViewModel” 只是添加到一个以 “Page” 为后缀的名称中。但是，只有 “Model” 被添加到“View” 后缀的名称中，以生成其相应的 ViewModel 名称。这种差异主要源于命名 “MainViewViewModel” 而不是 “MainPageViewModel” 的语义尴尬。从“View”-后缀 View名称派生的 ViewModel 的命名约定通过将 ViewModel 命名为 “MainViewModel” 来避免冗余。</font>

> <font color="#63aebb" face="微软雅黑">框架支持的标准命名约定的一个限制是在英语中不考虑不同的语言甚至不同的术语。尽管可以假设“View”和“ViewModel”被普遍理解，因为它们都是Caliburn Micro专用的MVVM设计模式的重要方面，但“Page”这样的词却不是。因此，一个健壮的框架应该可以通过定制支持额外的“View名称后缀”（例如“Pagina”，“Seite”，“Form”，“Screen”）。
</font>

### Naming Convention for Multi-View Support - 多视图支持的命名约定

As mentioned in the Conventions section of the documentation, the framework was designed to handle a one-to-many relationship between ViewModel and View. The standard convention supported by the framework is as follows:

---
> <font color="#63aebb" face="微软雅黑">如文档的“约定”部分所述，框架的设计是为了处理ViewModel和View之间的一对多关系。框架支持的标准约定如下:</font>

| &nbsp; | View Model | View |
|-|------------|------|
| Convention | &lt;EntityName&gt;[&lt;ViewSuffix&gt;]ViewModel | &lt;EntityName&gt;.&lt;Context&gt; |
| Example 1 | TabViewModel | Tab.First |
| Example 2 | CustomerViewModel | Customer.Master |

As explained in the previous section, the ViewModel's name may or may not include a "View" suffix. That is why the \<ViewSuffix> is indicated as optional.

---
> <font color="#63aebb" face="微软雅黑">如前一节所述，ViewModel的名称可能包括也可能不包括“View”后缀。这就是 \<ViewSuffix> 是可选的原因。</font>

### Naming Conventions for Namespace of a Type - 类型命名空间的命名约定

In .NET development, all assemblies must have a single default namespace. So the most basic use case has both the View and ViewModel component layers residing together in the same one. This convention can be described as follows:

---
> <font color="#63aebb" face="微软雅黑">在 .NET 开发中，所有程序集都必须具有一个默认命名空间。因此，最基本的用例包含 View 和 ViewModel 组件层，它们位于同一层中。此约定描述如下:</font>

| &nbsp; | View Model | View |
|-|------------|------|
| Convention | &lt;RootNS&gt;.&lt;ViewModelTypeName&gt; | &lt;RootNS&gt;.&lt;ViewTypeName&gt; |
| Example 1 | MyProject.ShellViewModel | MyProject.ShellView |
| Example 2 | MyProject.MainPageViewModel | MyProject.MainPage |

While all the Views and ViewModels of many applications may reside in a single assembly, it's common practice to organize Views and ViewModels in separate folders within a project. As a consequence, Visual Studio will, by default, place the components into separate namespaces corresponding to those folders. Since project folders are similar to operating system folders, project subfolders may be nested several layers deep as well. The namespace naming convention for this common use case can be described as follows:

---
> <font color="#63aebb" face="微软雅黑">虽然许多应用程序的所有Views 和 ViewModel 都可以驻留在单个程序集中，但通常的做法是在项目的单独文件夹中组织 Views 和 ViewModel。因此，默认情况下，Visual Studio 会将组件放入与这些文件夹对应的命名空间中。由于项目文件夹类似于操作系统文件夹，因此项目子文件夹也可以嵌套多个层。此常见用例的命名空间命名约定可以描述如下：</font>

| &nbsp; | View Model | View |
|-|------------|------|
| Convention | &lt;RootNS&gt;.ViewModels.&lt;ChildNS&gt;.&lt;ViewModelTypeName&gt; | &lt;RootNS&gt;.Views.&lt;ChildNS&gt;.&lt;ViewTypeName&gt; |
| Example 1 | MyProject.ViewModels.ShellViewModel | MyProject.Views.ShellView |
| Example 2 | MyProject.ViewModels.Utilities.SettingsViewModel | MyPoject.Views.Utitlities.SettingsView |

While the convention above covers many possibilities in terms of how deeply-nested namespaces may be, it does, however, assume a parallel structure in the organizational scheme of both the Views and ViewModels. Furthermore, it is also quite common to place Views and ViewModels into separate assemblies, which makes the likelihood of parallel organization across different assemblies even less likely.

---
> <font color="#63aebb" face="微软雅黑">虽然上面的约定涵盖了关于命名空间嵌套深度的许多可能性，但是它在 Views 和 ViewModels 的组织方法中假设了一个并行结构。此外，将 Views 和 ViewModel 放入单独的程序集中也很常见，这使得跨不同程序集的并行组织的可能性更小。</font>


[目录](./index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[Handling Custom Conventions - 自定义处理约定](./custom-conventions.md)