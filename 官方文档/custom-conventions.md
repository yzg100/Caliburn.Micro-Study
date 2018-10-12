---
layout: page
title: Handling Custom Conventions
---

While the ViewLocator and ViewModelLocator classes were designed to support non-standard conventions by giving public access to each class' instance of NameTransformer, adding new name transformation rules based on regular expressions can be quite a daunting task for those unfamiliar with regular expression syntax. Furthermore, because the NameTransformer is designed to perform generic name transformations, it doesn't allow for the customization of name and namespace transformation independently of each other. In other words, there's no simple way of adding support for a custom View name suffix while maintaining the standard transformations for namespaces, and there's no simply way of changing namespace transformations while maintaining the standard transformations for type names.
In recognition of these limitations, we have added configurability and several high-level methods to each of the locator classes. These new features allow for building custom transformation rules for common use cases without requiring knowledge of regular expressions. In addition, the methods are domain-aware (i.e. they contemplate the concepts of both namespaces and type names) rather than being geared toward generic name transformation.

---
> <font color="#63aebb" face="微软雅黑">虽然 ViewLocator 和 ViewModelLocator 类旨在通过为每个类的 NameTransformer 实例提供公共访问来支持非标准约定，但是对于那些不熟悉正则表达式语法的人来说，添加基于正则表达式的新名称转换规则可能是一项非常艰巨的任务。此外，因为 NameTransformer 被设计为执行通用名称转换，因此它不允许独立地定制名称和命名空间转换。换句话说，在维护命名空间的标准转换的同时，没有简单的方法来添加对自定义视图名称后缀的支持，并且没有简单的方法来更改命名空间转换，同时保持类型名称的标准转换。认识到这些局限性，我们为每个 locator 类添加了可配置性和几个高级方法。这些新功能允许为常见用例构建自定义转换规则，而无需了解正则表达式。此外，这些方法是域感知的（即它们考虑了名称空间和类型名称的概念），而不是面向通用名称转换。</font>

### Terminology - 术语

Before introducing these new methods of the locator classes, it would be appropriate to discuss terminology.

---
> <font color="#63aebb" face="微软雅黑">在介绍 locator 类的这些新方法之前，先讨论一下术语。</font>

**Name transformation** is a general term used to describe how type resolution is done. A type's fully qualified name is taken as the source and then "transformed" into the name of the output type. At the lowest level, the NameTransfomer class is responsible for this action and performs the transformation using regular expression-based "transformation rules".

---
> <font color="#63aebb" face="微软雅黑">名称转换 - 是描述如何进行类型解析的通用术语。将类型的完全限定名称作为源，然后 “transformed” 为输出类型的名称。NameTransfomer类负责此操作，并使用基于正则表达式的 “transformation rules” 执行转换。</font>

**Type mapping** is a term to describe the new capabilities that were added to the locator classes. Creating a type mapping is considered a higher-level operation because type mappings contemplate the two facets of type resolution: resolving the type's name and resolving the type's namespace. Although a type mapping is ultimately expressed as a transformation rule for the NameTransformer, the methods for creating type mappings were intended to relieve you of having to understand regular expressions in addition to being more domain-specific.

---
> <font color="#63aebb" face="微软雅黑">类型映射 - 是用于描述添加到定位器类的新功能的术语。创建类型映射被认为是更高级别的操作，因为类型映射考虑了类型解析的两个方面：解析类型的名称并解析类型的命名空间。虽然类型映射最终表示为 NameTransformer 的转换规则，但创建类型映射的方法旨在减轻您必须理解正则表达式以及更具域特性。</font>

### Configuration of Type Mappings - 类型映射配置

Both of the locator classes can be configured by calling the new ConfigureTypeMappings() method, which takes an instance of the TypeMappingConfiguration class as an argument.

---
> <font color="#63aebb" face="微软雅黑">可以通过调用新的 ConfigureTypeMappings() 方法来配置这两个 locator 类，它将TypeMappingConfiguration 类的实例作为参数。</font>

#### TypeMappingConfiguration Class

This class has various properties whose values are used as global settings required by the locator classes for configuring how the various high-level type mapping methods behave.

---
> <font color="#63aebb" face="微软雅黑">此类具有各种属性，其值用作 locator 类所需的全局设置，用于配置各种高级类型映射方法的行为方式。</font>

##### Properties - 属性

 - DefaultSubNamespaceForViews: The subnamespace comprising the application's Views (i.e. "Views" in the namespace"MyProject.Views"). This value is used to create a default mapping with subnamespace for ViewModels. The default value is "Views".
 - DefaultSubNamespaceForViewModels: The subnamespace comprising the application's ViewModels (i.e. "ViewModels" in the namespace"MyProject.ViewModels"). This value is used to create a default mapping with subnamespace for Views. The default value is "ViewModels".
 - UseNameSuffixesInMappings: Flag indicating if mappings should account for name suffixes in the type's name to distinguish between Views and ViewModels. This value can be set false if Views and ViewModels can be distinguished by namespace or subnamespace.The default value is true.
 - NameFormat: Format string used to construct a type's name using a base name (or entity name) and the View or ViewModel suffix. The format items are as follows:
	 - {0}: base name
	 - {1}: name suffix
	 - As only two arguments will be used with the specified format string, NameFormat can contain any combination of the format items listed above but must not contain any more (i.e. {2}, {3}, etc.). The default value is "{0}{1}" (i.e. &lt;basename&gt;&lt;suffix&gt;).
 - IncludeViewSuffixInViewModelNames: Flag indicating if mappings should account for name suffixes like "Page" or "Form" as part of the name of the companion ViewModel (e.g. CustomerPageViewModel vs. CustomerViewModel or CustomerFormViewModel vs. CustomerViewModel). Note that by convention, regardless of this property's value, if the View suffix is part of the ViewModel suffix, the View suffix is assumed to be omitted (i.e. CustomerViewModel rather than CustomerViewViewModel). The default value is true.
 - ViewSuffixList: List of View suffixes for which to create default type mappings during configuration. The default values are "View" and "Page".
 - ViewModelSuffix: The ViewSuffix to use for all type mappings, both the default type mappings added during configuration as well as custom type mappings added after configuration. The default value is "ViewModel".

---
> - <font color="#63aebb" face="微软雅黑">DefaultSubNamespaceForViews - 包含应用程序视图的子命名空间（即命名空间 “MyProject.Views” 中的 “Views”）。此值用于为 ViewModel 创建具有子名称空间的默认映射，默认值为 “Views” 。</font>

> - <font color="#63aebb" face="微软雅黑">DefaultSubNamespaceForViewModels - 包含应用程序的 ViewModels 的子命名空间（即命名空间 “MyProject.ViewModels” 中的 “ViewModels”）。此值用于为视图的子名命空间创建默认映射，默认值为 “ViewModels”。</font>

> - <font color="#63aebb" face="微软雅黑">UseNameSuffixesInMappings - 指示映射是否应考虑类型名称中的名称后缀以区分 Views 和 ViewModel 的标志。如果可以通过命名空间或子命名空间区分 Views 和 ViewModel，则可以将此值设置为 false，默认值为 true。</font>

> - <font color="#63aebb" face="微软雅黑">NameFormat - 用于使用基本名称（或实体名称）和View或ViewModel后缀构造类型名称的格式字符串。格式项如下：<br>
>   - {0} - 基本名称。
>   - {1} - 名称后缀。
>   - 由于只有两个参数将与指定的格式字符串一起使用，因此NameFormat可以包含上面列出的格式项的任意组合，但不能再包含（即{2}，{3}等）。默认值为“{0} {1}”（即\<basename>\<suffix>）。</font>

> - <font color="#63aebb" face="微软雅黑">IncludeViewSuffixInViewModelNames - 指示映射是否应考虑名称后缀（如 “Page” 或 “Form”）作为伴随 ViewModel 名称的一部分的标志（例如，CustomerPageViewModel 与 CustomerViewModel 或 CustomerFormViewModel 与 CustomerViewModel）。请注意，按照惯例，无论此属性的值如何，如果 View 后缀是 ViewModel 后缀的一部分，则假定视图后缀被省略（即 CustomerViewModel 而不是 CustomerViewViewModel）。默认值是true。</font>

> - <font color="#63aebb" face="微软雅黑">ViewSuffixList - ViewSuffixList：在配置期间为其创建默认类型映射的View后缀列表。默认值为 “View” 和 “Page”。</font>

> - <font color="#63aebb" face="微软雅黑">ViewModelSuffix - ViewSuffix 用于所有类型映射，包括配置期间添加的默认类型映射以及配置后添加的自定义类型映射，默认值为“ViewModel”。</font>

##### ViewLocator.ConfigureTypeMapping(), ViewModelLocator.ConfigureTypeMapping() Methods

This method configures or reconfigures how type mappings are added by the locator class.

---
> - <font color="#63aebb" face="微软雅黑">此方法配置或重新配置locator类如何添加类型映射。</font>

``` csharp
public static void ConfigureTypeMapping(TypeMappingConfiguration config)
```

The config parameter is the configuration object whose property values are used as settings to configure the locator class for type mapping.

This method is called internally by the locator class using the default property values of the TypeMappingConfiguration class.

Each time this method is called, the existing name transformation rules are cleared and new default type mappings are added automatically.

The settings of the configuration object are applied globally to the default type mappings that are automatically added at configuration and to any type mappings added after configuration. For instance, if the NameFormat is customized to specify a naming convention that places name suffixes before the base name, thereby turning the name suffixes to prefixes (e.g. ViewCustomer and ViewModelCustomer), this convention will be used as the standard type naming convention for any subsequent calls to methods that add a type mapping.

---
> - <font color="#63aebb" face="微软雅黑">配置参数是配置对象，其属性值用作配置类型映射的locator类。
> - Locator类使用TypeMappingConfiguration类的默认属性值在内部调用此方法。
> - 每次调用此方法时，都会清除现有的名称转换规则，并自动添加新的默认类型映射。
> - 配置对象的全局设置应用于在配置时自动添加的默认类型映射，以及在配置后添加的任何类型映射。例如，如果自定义NameFormat以指定将名称后缀放在基本名称之前的命名约定，从而将名称后缀转换为前缀（例如ViewCustomer和ViewModelCustomer），则此约定将用作任何后续的标准类型命名约定调用添加类型映射的方法。</font>

``` csharp
//Override the default subnamespaces
var config = new TypeMappingConfiguration
{
    DefaultSubNamespaceForViewModels = "MyViewModels",
    DefaultSubNamespaceForViews = "MyViews"
};

ViewLocator.ConfigureTypeMappings(config);
ViewModelLocator.ConfigureTypeMappings(config);

//Resolves:
//MyProject.MyViewModels.CustomerViewModel -> MyProject.MyViews.CustomerView
//MyProject.MyViewModels.CustomerPageViewModel -> MyProject.MyViews.CustomerPage
//MyProject.MyViews.CustomerView -> MyProject.MyViewModels.CustomerViewModel
//MyProject.MyViews.CustomerPage -> MyProject.MyViewModels.CustomerPageViewModel

//Change ViewModel naming convention to always exclude View suffix
var config = new TypeMappingConfiguration
{
    DefaultSubNamespaceForViewModels = "MyViewModels",
    DefaultSubNamespaceForViews = "MyViews",
    IncludeViewSuffixInViewModelNames = false
};

ViewLocator.ConfigureTypeMappings(config);
ViewModelLocator.ConfigureTypeMappings(config);

//Resolves:
//MyProject.MyViewModels.CustomerViewModel -> MyProject.MyViews.CustomerPage, MyProject.MyViews.CustomerView
//MyProject.MyViews.CustomerView -> MyProject.MyViewModels.CustomerViewModel
//MyProject.MyViews.CustomerPage -> MyProject.MyViewModels.CustomerViewModel

//Change naming conventions to place name suffixes before the base name (i.e. name prefix)
var config = new TypeMappingConfiguration
{
    NameFormat = "{1}{0}",
    IncludeViewSuffixInViewModelNames = false
};

ViewLocator.ConfigureTypeMappings(config);
ViewModelLocator.ConfigureTypeMappings(config);

//Resolves:
//MyProject.ViewModels.ViewModelCustomer -> MyProject.Views.PageCustomer, MyProject.Views.ViewCustomer
//MyProject.Views.ViewCustomer -> MyProject.ViewModels.ViewModelCustomer
//MyProject.Views.PageCustomer -> MyProject.ViewModels.ViewModelCustomer

//Change naming conventions to omit name suffixes altogether (i.e. distinguish View and ViewModel types by namespace alone)
var config = new TypeMappingConfiguration
{
    UseNameSuffixesInMappings = false
};

ViewLocator.ConfigureTypeMappings(config);
ViewModelLocator.ConfigureTypeMappings(config);

//Resolves:
//MyProject.ViewModels.Customer -> MyProject.Views.Customer
//MyProject.Views.Customer -> MyProject.ViewModels.Customer

//Configure for Spanish language and semantics
var config = new TypeMappingConfiguration()
{
    DefaultSubNamespaceForViewModels = "ModelosDeVistas",
    DefaultSubNamespaceForViews = "Vistas",
    ViewModelSuffix = "ModeloDeVista",
    ViewSuffixList = new List<string>(new string[] { "Vista", "Pagina" }),
    NameFormat = "{1}{0}",
    IncludeViewSuffixInViewModelNames = false
};

ViewLocator.ConfigureTypeMappings(config);
ViewModelLocator.ConfigureTypeMappings(config);

//Resolves:
//MiProyecto.ModelosDeVistas.ModeloDeVistaCliente -> MiProyecto.Vistas.VistaCliente, MiProyecto.Vistas.PaginaCliente
//MiProyecto.Vistas.VistaCliente -> MiProyecto.ModelosDeVistas.ModeloDeVistaCliente
//MiProyecto.Vistas.PaginaCliente -> MiProyecto.ModelosDeVistas.ModeloDeVistaCliente
```

### New Type Mapping Methods - 新类型映射方法

##### ViewLocator.AddDefaultTypeMapping(), ViewModelLocator.AddDefaultTypeMapping()

This method is used to add a type mapping that supports the standard type and namespace naming conventions for a given View name suffix.

---
> - <font color="#63aebb" face="微软雅黑">此方法用于添加支持给定 View 名称后缀的标准类型和命名空间命名约定的类型映射。</font>

``` csharp
public static void AddDefaultTypeMapping(string viewSuffix = "View")
```

The viewSuffix parameter is the suffix for type name. Should be "View" or synonym of "View". (Optional)

This method is primarily used to add support of types that have custom synonyms (e.g. "Form", "Screen", "Tab") but otherwise use the standard naming conventions.

While the viewSuffix argument is optional, defaulting to "View", it is unnecessary to call this method in such a way since the locator classes already add type mappings for both the "View" and "Page" View name suffixes, although this might not be the case if the locator classes were configured differently using the ConfigureTypeMappings() method and modifying ViewSuffixList property of the TypeMappingConfiguration object. However, modifying the ViewSuffixList property of the configuration object and reconfiguring the locator class would obviate the need to call this method after the fact.

Keep in mind that this method does not add any type mappings if the UseNameSuffixesInMappings property of the configuration object is set to false. When this is the case, there is no default type naming convention for which to add mappings.

---
> - <font color="#63aebb" face="微软雅黑">viewSuffix参数是类型名称的后缀。应该是“View”或“View”的同义词。(可选)

> - 此方法主要用于添加对具有自定义同义词的类型的支持（例如 “Form”，“Screen”，“Tab”），但使用标准命名约定。

> - 虽然 viewSuffix 参数是可选的，默认为 “View”，但是没有必要以这种方式调用此方法，因为定位器类已经为 “View” 和 “Page” 视图名称后缀添加了类型映射，如果使用 ConfigureTypeMappings（）方法以及修改 TypeMappingConfiguration 对象的 ViewSuffixList 属性来配置定位器类，则会出现这种情况。但是，修改配置对象的 ViewSuffixList 属性并重新配置定位器类将避免在事后调用此方法。

> - 请记住，如果配置对象的UseNameSuffixesInMappings属性设置为false，则此方法不会添加任何类型映射。在这种情况下，没有添加映射的默认类型命名约定。</font>

``` csharp
//Add support for "Form" as a synonym of "View" using the standard type and namespace naming conventions
ViewLocator.AddDefaultTypeMapping("Form");
//Resolves: MyProject.ViewModels.MainFormViewModel -> MyProject.Views.MainForm


ViewModelLocator.AddDefaultTypeMapping("Form");
//Resolves: MyProject.Views.MainForm -> MyProject.ViewModels.MainFormViewModel
```

##### ViewLocator.RegisterViewSuffix()

This method is used to indicate to the ViewLocator that a transformation rule to support a custom View suffix was added using the NameTransformer.AddRule(). This method is automatically called internally by the ViewLocator class when the AddNamespaceMapping(), AddTypeMapping() and the AddDefaultTypeMapping() type mapping methods are called.

---
> - <font color="#63aebb" face="微软雅黑">此方法用于向 ViewLocator 指示使用 NameTransformer.AddRule() 添加了支持自定义 View 后缀的转换规则。当调用AddNamespaceMapping()，AddTypeMapping()和AddDefaultTypeMapping()类型映射方法时，ViewLocator 类会自动调用此方法。</font>

``` csharp
public static void RegisterViewSuffix(string viewSuffix)
```

The viewSuffix parameter is the suffix for type name. Should be "View" or synonym of "View". (Optional)

In order for multi-View support to work properly, the ViewLocator needs to track all of the View suffixes that an application may use. Although this is managed automatically when name transformation rules are added using the new type mapping methods, transformation rules that are added directly through the NameTransformer instance of the ViewLocator class will bypass this registration step. Therefore, the ViewLocator.RegisterViewSuffix() will need to be called in such cases. The method will perform a check for existing entries prior to adding new ones.

---
> - <font color="#63aebb" face="微软雅黑">viewSuffix参数是类型名称的后缀。是 “View” 或 “View” 的同义词。(可选)

> - 为了使多视图能正常工作，ViewLocator 需要跟踪应用程序可能使用的所有 View后缀。虽然使用新类型映射方法添加名称转换规则时会自动管理，但直接通过 ViewLocator 类的 NameTransformer 实例添加的转换规则将绕过此注册步骤。因此，在这种情况下需要调用 ViewLocator.RegisterViewSuffix()。该方法将在添加新条目之前检查现有实体。</font>

``` csharp
//Manually add a rule to the NameTransformer to do simple text replacement independent of namespace
ViewLocator.NameTransformer.AddRule("ScreenViewModel$", "Screen");
//However, we need to treat "Screen" as a synonym of "View" somehow in order to
//enable multi-view support for View types with names ending in "Screen"
ViewLocator.RegisterViewSuffix("Screen");
//Resolves: MyProject.ViewModels.CustomerScreenViewModel -> MyProject.Views.Customer.Master
//when the context is "Master"
```

##### ViewLocator.AddNamespaceMapping(), ViewModelLocator.AddNamespaceMapping()

This method is used to add a type mapping between a source namespace and one or more target namespaces . The resultant type mapping creates a transformation rule supporting the standard type naming conventions but with a custom namespace naming convention. Optionally, a custom View suffix may be specified for this mapping.

---
> - <font color="#63aebb" face="微软雅黑">此方法用于在源命名空间和一个或多个目标命名空间之间添加类型映射。结果类型映射创建支持标准类型命名约定但具有自定义命名空间命名约定的转换规则。（可选）可以为此映射指定自定义视图后缀。</font>

``` csharp
public static void AddNamespaceMapping(string nsSource, string[] nsTargets, string viewSuffix = "View")
```

 - nsSource: Namespace of source type
 - nsTargets: Namespaces of target type as an array
 - viewSuffix: Suffix for type name. Should be "View" or synonym of "View". (Optional)

---
> - <font color="#63aebb" face="微软雅黑">nsSource - 源类型的命名空间。</font>
> - <font color="#63aebb" face="微软雅黑">nsTargets - 作为数组的目标类型的名称空间。</font>
> - <font color="#63aebb" face="微软雅黑">viewSuffix - 类型名称的后缀。应该是 “View” 或 “View” 的同义词。(可选)</font>

``` csharp
public static void AddNamespaceMapping(string nsSource, string nsTarget, string viewSuffix = "View")
```

 - nsSource: Namespace of source type
 - nsTarget: Namespace of target type
 - viewSuffix: Suffix for type name. Should be "View" or synonym of "View". (Optional)

This method supports the use of wildcards (indicated with *) in the nsSource argument. When a null string (or String.Empty) is used for the nsSource argument, the namespace(s) passed as the nsTarget/nsTargets argument will be appended to the namespace of the source type. Refer to the example for more details.

An array can be passed as an argument for the target namespaces to indicate that the target type could exist in multiple namespaces ("one-to-many" mapping). Because the locator classes are designed to pick up the first occurrence of a type that matches the name transformation rule, it does not matter if a type doesn't actually exist in one of the target namespaces or if there exist several types in different namespaces that share the same name. One possible use case for this mechanism would be to map a ViewModel namespace to an assembly for custom Views and to another assembly for standard Views. If the assembly for custom Views doesn't exist or if the particular View doesn't exist in the custom View assembly, the ViewLocator would pick up the View from the standard View assembly.

---
> - <font color="#63aebb" face="微软雅黑">nsSource - 源类型的命名空间。</font>
> - <font color="#63aebb" face="微软雅黑">nsTarget - 目标类型的名称空间。</font>
> - <font color="#63aebb" face="微软雅黑">viewSuffix - 类型名称的后缀。应该是 “View” 或 “View” 的同义词。(可选)

> 此方法支持在nsSource参数中使用通配符（用*表示）。当空字符串（或String.Empty）用于nsSource参数时，作为nsTarget / nsTargets参数传递的名称空间将附加到源类型的名称空间。有关更多详细信息，请参阅示例。

> 可以将数组作为目标名称空间的参数传递，以指示目标类型可以存在于多个名称空间中（“一对多”映射）。因为 locator 类被设计为在第一次出现时获取与名称转换规则匹配的类型，所以如果某个类型实际上不存在于其中一个目标命名空间中，或者如果在不同的命名空间中存在多个类型，则无关紧要共享相同的名称。此机制的用例是将 ViewModel 命名空间映射到自定义视图的程序集和标准视图的另一个程序集。如果自定义视图的程序集不存在或者自定义 View 程序集中不存在特定 View，则 ViewLocator 将从标准 View 程序集中选取 View。
</font>

``` csharp
//"Append target to source" mapping
//Null string or String.Empty passed as source namespace is special case to allowe this
//Note the need to prepend target namespace with "." to make this work
ViewLocator.AddNamespaceMapping("", ".Views");
//Resolves: MyProject.Customers.CustomerViewModel -> MyProject.Customers.Views.CustomerView
 
//Standard explicit namespace mapping
ViewLocator.AddNamespaceMapping("MyProject.ViewModels.Customers", "MyClient1.Views");
//Resolves: MyProject.ViewModels.CustomerViewModel -> MyClient1.Views.CustomerView
 
//One to many explicit namespace mapping
ViewLocator.AddNamespaceMapping("MyProject.ViewModels.Customers", new string[] { "MyClient1.Views", "MyProject.Views" } );
//Resolves: MyProject.ViewModels.CustomerViewModel -> {MyClient1.Views.CustomerView, MyProject.Views.CustomerView }
 
//Wildcard mapping
ViewLocator.AddNamespaceMapping("*.ViewModels.Customers.*", "MyClient1.Customers.Views");
//Resolves: MyProject.ViewModels.Customers.CustomerViewModel -> MyClient1.Customers.Views.CustomerView
//          MyProject.More.ViewModels.Customers.MasterViewModel -> MyClient1.Customers.Views.MasterView
//          MyProject.ViewModels.Customers.More.OrderHistoryViewModel -> MyClient1.Customers.Views.OrderHistoryView 
```

##### ViewLocator.AddSubNamespaceMapping(), ViewModelLocator.AddSubNamespaceMapping()

---
><font color="#63aebb" face="微软雅黑">This method is used to add a type mapping between a source namespace and one or more target namespaces by substituting a given subnamespace for another. The resultant type mapping creates a transformation rule supporting the standard type naming conventions but with a custom namespace naming convention. Optionally, a custom View suffix may be specified for this mapping.</font>

该方法通过用给定的子命名空间替换另一个子命名空间来添加源命名空间和一个或多个目标命名空间之间的类型映射。结果类型映射创建支持标准类型命名约定但具有自定义命名空间命名约定的转换规则。（可选）可以为此映射指定自定义视图后缀。

``` csharp
public static void AddSubNamespaceMapping(string nsSource, string[] nsTargets, string viewSuffix = "View")
```

 - nsSource: Subnamespace of source type
 - nsTargets: Subnamespaces of target type as an array
 - viewSuffix: Suffix for type name. Should be "View" or synonym of "View". (Optional)

---
> - <font color="#63aebb" face="微软雅黑">nsSource - 源类型的子命名空间。</font>
> - <font color="#63aebb" face="微软雅黑">nsTargets - 作为数组的目标类型的子命名空间。</font>
> - <font color="#63aebb" face="微软雅黑">viewSuffix - 类型名称的后缀。应该是“View”或“View”的同义词。(可选)</font>

``` csharp
public static void AddSubNamespaceMapping(string nsSource, string nsTarget, string viewSuffix = "View")
```

 - nsSource: Subnamespace of source type
 - nsTarget: Subnamespace of target type
 - viewSuffix: Suffix for type name. Should be "View" or synonym of "View". (Optional)

This method supports the use of a wildcards (indicated with *) in the nsSource argument. When a null string (or String.Empty) is used for the nsSource argument, the namespace(s) passed as the nsTarget/nsTargets argument will be appended to the namespace of the source type. When nsSource is the null string or if it begins and terminates with a wild-card, the behavior is identical to that of AddNamespaceMapping(). As with AddNamespaceMapping(), the AddSubNamespaceMapping() method supports one-to-many mapping. Refer to the notes on AddNamespaceMapping() for an explanation.

This method is called internally at configuration for each View suffix in the ViewSuffixList of the configuration object. The subnames specified by DefaultSubNamespaceForViews and DefaultSubNamespaceForViewModels are used for the mapping. If the default mapping between the "Views" and "ViewModels" subnamespaces is not required, the appropriate configuration settings can be used to eliminate the need to call AddSubNamespaceMapping() directly.

---
> - <font color="#63aebb" face="微软雅黑">nsSource - 源类型的子命名空间。</font>
> - <font color="#63aebb" face="微软雅黑">nsTarget - 目标类型的名称空间。</font>
> - <font color="#63aebb" face="微软雅黑">viewSuffix - 类型名称的后缀。应该是“View”或“View”的同义词。(可选)</font>

> <font color="#63aebb" face="微软雅黑">此方法支持在 nsSource 参数中使用通配符（用*表示）。当空字符串（或 String.Empty）用于 nsSource 参数时，作为 nsTarget / nsTargets 参数传递的名称空间将附加到源类型的名称空间。当 nsSource 是空字符串或者它以一个通配符开始和结尾时，其行为与 AddNamespaceMapping()的行为相同。与 AddNamespaceMapping()一样，AddSubNamespaceMapping()方法支持一对多映射。请参阅 AddNamespaceMapping()注释。</font>

> <font color="#63aebb" face="微软雅黑">此方法在配置时内部调用，用于配置对象的 ViewSuffixList 中的每个 View 后缀。映射使用 DefaultSubNamespaceForViews 和 DefaultSubNamespaceForViewModels 指定的子名称。如果不需要 “Views” 和 “ViewModels” 子名称空间之间的默认映射，那么可以使用适当的配置来直接调用 AddSubNamespaceMapping()。</font>

``` csharp
//Add support for Spanish namespaces
ViewLocator.AddSubNamespaceMapping("ModelosDeVistas", "Vistas");
//Resolves: MiProyecto.ModelosDeVistas.Clientes.ClienteViewModel -> MiProyecto.Vistas.Clientes.ClienteView
 
ViewModelLocator.AddSubNamespaceMapping("Vistas", "ModelosDeVistas");
//Resolves: MiProyecto.Vistas.Clientes.ClienteView -> MiProyecto.ModelosDeVistas.Clientes.ClienteViewModel
 
//Wildcard subnamespace mapping
ViewLocator.AddSubNamespaceMapping("*.ViewModels", "ExtLib.Views");
//Resolves: MyCompany.MyApp.SomeNamespace.ViewModels.CustomerViewModel -> ExtLib.Views.CustomerView
 
ViewLocator.AddSubNamespaceMapping("ViewModels.*", "Views");
//Resolves: MyApp.ViewModels.Some.Name.Space.CustomerViewModel -> MyApp.Views.CustomerView
 
ViewLocator.AddSubNamespaceMapping("MyApp.*.ViewModels", "ExtLib.Views");
//Resolves: MyCompany.MyApp.SomeNamespace.ViewModels.CustomerViewModel -> MyCompany.ExtLib.Views.CustomerView
```

##### ViewLocator.AddTypeMapping(), ViewModelLocator.AddTypeMapping()

This method is used to add a type mapping expressed as regular expression-based transformations. The resultant type mapping creates a transformation rule supporting the standard type naming conventions but with a custom namespace naming convention. Optionally, a custom View suffix may be specified for this mapping.

---
><font color="#63aebb" face="微软雅黑">此方法用于添加表示为基于正则表达式的转换的类型映射。结果类型映射创建支持标准类型命名约定但具有自定义命名空间命名约定的转换规则。（可选）可以为此映射指定自定义视图后缀。</font>

``` csharp
public static void AddTypeMapping(string nsSourceReplaceRegEx,
    string nsSourceFilterRegEx,
    string[] nsTargetsRegEx,
    string viewSuffix = "View")
```

 - nsSourceReplaceRegEx: RegEx replace pattern for source namespace
 - nsSourceFilterRegEx: RegEx filter pattern for source namespace
 - nsTargetsRegEx: Array of RegEx replace values for target namespaces
 - viewSuffix: Suffix for type name. Should be "View" or synonym of "View". (Optional)

---
> - <font color="#63aebb" face="微软雅黑">nsSourceReplaceRegEx - RegEx替换源名称空间的模式。</font>
> - <font color="#63aebb" face="微软雅黑">nsSourceFilterRegEx - 用于源名称空间的正则表达式过滤模式。</font>
> - <font color="#63aebb" face="微软雅黑">nsTargetsRegEx - 目标命名空间的正则表达式替换值数组。</font>
> - <font color="#63aebb" face="微软雅黑">viewSuffix - 类型名称的后缀。应该是“View”或“View”的同义词。(可选)</font>

``` csharp
public static void AddTypeMapping(string nsSourceReplaceRegEx, string nsSourceFilterRegEx, string nsTargetRegEx, string viewSuffix = "View")
```

 - nsSourceReplaceRegEx: RegEx replace pattern for source namespace
 - nsSourceFilterRegEx: RegEx filter pattern for source namespace
 - nsTargetsRegEx: RegEx replace value for target namespace
 - viewSuffix: Suffix for type name. Should be "View" or synonym of "View". (Optional)

Refer to the documentation on the NameTransformer for details on creating regular expression-based transformation rules. Unlike adding a transformation rule through the NameTransformer class, this method separates out the namespace transformations from the type name transformation. In addition, it supports one-to-many namespace mapping. Refer to the notes on AddNamespaceMapping() for an explanation.

---
> - <font color="#63aebb" face="微软雅黑">nsSourceReplaceRegEx - RegEx替换源名称空间的模式。</font>
> - <font color="#63aebb" face="微软雅黑">nsSourceFilterRegEx - 用于源名称空间的正则表达式过滤模式。</font>
> - <font color="#63aebb" face="微软雅黑">nsTargetsRegEx - 目标命名空间的正则表达式替换值数组。</font>
> - <font color="#63aebb" face="微软雅黑">viewSuffix - 类型名称的后缀。应该是“View”或“View”的同义词。(可选)</font>

> <font color="#63aebb" face="微软雅黑">有关创建基于正则表达式的转换规则的详细信息，请参阅 NameTransformer 的文档。与通过 NameTransformer 类添加转换规则不同，此方法将命名空间转换与类型名称转换分开。此外，它还支持一对多命名空间映射。有关说明，请参阅 AddNamespaceMapping() 的注释。</font>

``` csharp
//Capture namespace fragment preceding "ViewModels." as "nsbefore"
string subns = RegExHelper.NamespaceToRegEx("ViewModels.");
string rxrep = RegExHelper.GetNamespaceCaptureGroup("nsbefore") + subns;
            
//Output the namespace fragment after "Views." in the target namespace
string rxtgt = @"Views.${nsbefore}";
ViewLocator.AddTypeMapping(rxrep, null, rxtgt);
//Resolves: MyApp.Some.Name.Space.ViewModels.TestViewModel -> Views.MyApp.Some.Name.Space.TestView
```

[目录](index)&nbsp;&nbsp;|&nbsp;&nbsp;[Using the NameTransformer - 使用NameTransformer](./name-transformer)