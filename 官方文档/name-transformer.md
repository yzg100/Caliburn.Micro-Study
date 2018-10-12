---
layout: page
title: Using the Name Transformer
---

The NameTransformer was introduced in Caliburn.Micro v1.1 and is an integral part of how both the ViewLocator and ViewModelLocator map class names to their companion role. While you can override various funcs on these services to replace the underlying behavior, most of your needs should be met by simply configuring rules with the appropriate NameTransformer which describe your unique mapping strategies.

Name transformation is based on rules that use regular expression pattern-matching. When a transformation is executed, all registered rules are evaluated in sequence. By default, the resultant names yielded by all matching rules are returned by the NameTransformer. The ViewLocator and ViewModelLocator classes will use the resultant name list to check, in order, the existence of a matching type in the AssemblySource.Instance collection. Once a type is found, the remaining names in the list are ignored. While the locator classes will always return at most one type regardless of the number of names returned by the NameTransformer, it's important to be able to specify how the NameTransformer constructs that list of names to provide better control over which type will be located. The primary means of control is through sequence. Because the locator classes need to support some type naming conventions out-of-the-box, some default name transformation rules are automatically added. However, to be able to support custom rules and allow them to take precedence over the default rules, the NameTransformer evaluates the rules in reverse order from which they are added (LIFO). In general, you will want more-general rules to be evaluated after more-specific rules. Therefore, when adding rules to the NameTransfomer it is necessary to add the more-general rules first and the more-specific rules last. To limit the names returned by the NameTransformer to those yielded by the first matching rule, the UseEagerRuleSelection property on the NameTransformer can be set to false. By default, UseEagerRuleSelection is set to true.

Custom rules are added by calling the AddRule() method of the NameTransformer object maintained by the ViewLocator and ViewModelLocator classes. Both classes reference their own separate static instance of NameTransformer, so each class maintains its own set of rules.

The calling convention is as follows:

---
><font color="#63aebb" face="微软雅黑">NameTransformer 是在 Caliburn.Micro v1.1中引入，是 ViewLocator 和 ViewModelLocator 将类名称映射到其伴随角色的不可或缺的一部分。虽然你可以覆盖这些服务上的各种 func 来替换基础行为，但只需使用描述你的唯一映射策略的相应 NameTransformer 配置规则即可满足你的大多数需求。

>名称转换基于使用正则表达式模式匹配的规则。执行转换时，将按顺序评估所有已注册的规则。默认情况下，NameTransformer 返回由所有匹配规则生成的结果名称。ViewLocator 和 ViewModelLocator 类将使用结果名称列表按顺序检查 AssemblySource.Instance 集合中是否存在匹配类型。找到类型后，将忽略列表中的其余名称。虽然无论 NameTransformer 返回的名称数量如何，locator 类总是最多返回一种类型，但重要的是能够指定 NameTransformer 如何构造该名称列表以更好地控制将定位的类型。控制的主要手段是顺序。因为 locator 类需要支持一些开箱即用的类型命名约定，所以会自动添加一些缺省名称转换规则。但为了能够支持自定义规则并允许它们优先于默认规则，NameTransformer以相反的顺序计算添加规则(LIFO)。通常，你需要在更具体的规则之后评估更通用的规则。因此，在向 NameTransfomer 添加规则时，必须先添加更通用的规则，然后再添加更具体的规则。为了将 NameTransformer 返回的名称限制为第一个匹配规则生成的名称，可以将 NameTransformer 的 UseEagerRuleSelection 属性设置为false，默认为true。

>通过调用由 ViewLocator 和 ViewModelLocator 类维护的 NameTransformer 对象的AddRule() 方法来添加自定义规则。这两个类都引用了自己独立的 NameTransformer 静态实例，因此每个类都维护着自己的一组规则。

>调用约定如下：</font>

``` csharp
public void AddRule(string replacePattern,
    IEnumerable<string> replaceValueList,
    string globalFilterPattern = null)
```

 - replacePattern: a regular expression pattern for replacing all or parts of the input string
 - replaceValueList: a collection of strings to apply to the replacePattern
 - globalFilterPattern: a regular expression pattern used for determining if rule should be evaluated. Optional.
  
---
> - <font color="#63aebb" face="微软雅黑">replacePattern - 用于替换输入字符串的全部或部分的正则表达式。</font>
> - <font color="#63aebb" face="微软雅黑">replaceValueList - 应用于 replacePattern 的字符串集合。</font>
> - <font color="#63aebb" face="微软雅黑">globalFilterPattern - 确定是否应评估规则的正则表达式模式。可选</font>

``` csharp
public void AddRule(string replacePattern, string replaceValue, string globalFilterPattern = null)
```

 - replacePattern: a regular expression pattern for replacing all or parts of the input string
 - replaceValue: a string to apply to the replacePattern
 - globalFilterPattern: a regular expression pattern used for determining if rule should be evaluated. Optional.

To show how this method is used, we can take a look at one of the built-in rules added by the ViewLocator class:

---
> - <font color="#63aebb" face="微软雅黑">replacePattern - 用于替换输入字符串的全部或部分的正则表达式。</font>
> - <font color="#63aebb" face="微软雅黑">replaceValue - 应用于replacePattern的字符串。</font>
> - <font color="#63aebb" face="微软雅黑">globalFilterPattern - 确定是否应评估规则的正则表达式模式。可选</font>

> <font color="#63aebb" face="微软雅黑">为了说明如何使用此方法，我们可以查看ViewLocator类添加的一个内置规则：</font>

``` csharp
NameTransformer.AddRule("Model$", string.Empty);
```

This transformation rule looks for the substring "Model" terminating the ViewModel name and strips out that substring (i.e. replace with string.Empty or "null string"). The "$" in the first argument indicates that the pattern must match at the end of the source string. If "Model" exists anywhere else, the pattern is not matched. Because this call didn't include the optional "globalFilterPattern" argument, this rule applies to all ViewModel names.

This rule yields the following results:

---
><font color="#63aebb" face="微软雅黑">此转换规则查找终止ViewModel名称的子字符串“Model”并删除该子字符串（即替换为 string.Empty 或“null string”）。第一个参数中的“$”表示模式必须在源字符串的末尾匹配。如果“Model”存在于其他任何位置，则模式不匹配。由于此调用不包含可选的“globalFilterPattern”参数，因此此规则适用于所有ViewModel名称。

> 此规则产生以下结果：</font>

| Input | Result |
|------------|------|
| MainViewModel | MainView |
| ModelAirplaneViewModel | ModelAirplaneView |
| CustomerViewModelBase | CustomerViewModelBase |

ViewModelLocator添加的相应内置规则是：

``` csharp
NameTransformer.AddRule(@"(?<fullname>^.*$)", @"${fullname}Model");
```

This rule takes whatever the input is and adds "Model" to the end. This rule uses a regular expression capture group, which is extremely useful in complex transformations. The "replacePattern" assigns the full name of the View to a capture group called "fullname" and the "replaceValue" transforms it into \<fullname> "Model".

To demonstrate how the "globalFilterPattern" applies, we can take a look at two other built-in rules for the ViewModelLocator:

---
><font color="#63aebb" face="微软雅黑">此规则接受任何输入，并在末尾添加 “Model”。此规则使用正则表达式捕获组，这在复杂的转换中非常有用。“replacePattern” 将视图的全名分配给一个名为 “fullname” 的捕获组，而 “replaceValue” 将其转换为\<fullname> "Model"。

>为了演示 “globalFilterPattern” 如何使用，请看 ViewModelLocator 的另外两个内置规则：</font>

``` csharp
//Check for <Namespace>.<BaseName>View construct
NameTransformer.AddRule
(
    @"(?<namespace>(.*\.)*)(?<basename>[A-Za-z_]\w*)(?<suffix>View$)",
    new[] {
        @"${namespace}${basename}ViewModel",
        @"${namespace}${basename}",
        @"${namespace}I${basename}ViewModel",
        @"${namespace}I${basename}"
    },
    @"(.*\.)*[A-Za-z_]\w*View$"
);

//Check for <Namespace>.Views.<BaseName>View construct
NameTransformer.AddRule
(
    @"(?<namespace>(.*\.)*)Views\.(?<basename>[A-Za-z_]\w*)(?<suffix>View$)",
    new[] {
        @"${namespace}ViewModels.${basename}ViewModel",
        @"${namespace}ViewModels.${basename}",
        @"${namespace}ViewModels.I${basename}ViewModel",
        @"${namespace}ViewModels.I${basename}"
    },
    @"(.*\.)*Views\.[A-Za-z_]\w*View$"
);
```

The "globalFilterPattern" arguments for the two calls are identical with the exception of the addition of "Views\." to the argument in the second method call. This indicates that the rule should be applied only if the namespace name terminates with "Views." (dot inclusive). If the pattern is matched, the result is an array of ViewModel names with namespace terminating with "ViewModels.".

The first rule, which echoes back the original namespace unchanged, would cover all other cases. As mentioned earlier, the least specific rule is added first. It covers the fall-through case when the namespace doesn't end with "Views.".

When adding custom application-specific transformation rules, the following replace pattern should prove quite useful. The replace pattern takes a fully-qualified ViewModel name and breaks it into capture groups that should cover almost any transformation:

---
><font color="#63aebb" face="微软雅黑">除了添加 “Views”，两个 “globalFilterPattern” 参数相同。这表示仅当命名空间名称以“Views”终止时才应用该规则。（dot inclusive）。如果模式匹配，则结果是一个 ViewModel 名称数组，其名称空间以“ViewModels” 结尾。

>第一个规则将覆盖所有其他情况，它将原样返回命名空间。正如前面提到的，首先添加最不具体的规则。它覆盖了命名空间不以 “Views” 结束时的失败情况。

>在添加特定于应用程序的自定义转换规则时，下面的替换模式应该非常有用。替换模式采用完全限定的 ViewModel 名称，并将其分解为捕捉组，这些组应该涵盖几乎所有的转换:</font>

```
(?<nsfull>((?<nsroot>[A-Za-z_]\w*\.)(?<nsstem>([A-Za-z_]\w*\.)*))?(?<nsleaf>[A-Za-z_]\w*\.))?(?<basename>[A-Za-z_]\w*)(?<suffix>ViewModel$)
```

例如，添加以下规则：

``` csharp
ViewLocator.NameTransformer.AddRule
(
    @"(?<nsfull>((?<nsroot>[A-Za-z_]\w*\.)(?<nsstem>([A-Za-z_]\w*\.)*))?(?<nsleaf>[A-Za-z_]\w*\.))?(?<basename>[A-Za-z_]\w*)(?<suffix>ViewModel$)",
    @"nsfull=${nsfull}, nsroot=${nsroot}, nsstem=${nsstem}, nsleaf=${nsleaf}, basename=${basename}, suffix=${suffix}"
);
```

将产生以下结果：

| Input | Output |
|------------|------|
| MainViewModel | nsfull=, nsroot=, nsstem=, nsleaf=, basename=Main, suffix=ViewModel |
| ViewModels.MainViewModel | nsfull=ViewModels., nsroot=, nsstem=, nsleaf=ViewModels., basename=Main, suffix=ViewModel |
| Root.ViewModels.MainViewModel | nsfull=Root.ViewModels., nsroot=Root., nsstem=, nsleaf=ViewModels., basename=Main, suffix=ViewModel |
| Root.Stem.ViewModels.MainViewModel | nsfull=Root.Stem.ViewModels., nsroot=Root., nsstem=Stem., nsleaf=ViewModels., basename=Main, suffix=ViewModel |

##### Notes - 说明:

The same replace pattern above can be used for the ViewModelLocator by changing the "ViewModel$" to "View$".

You wouldn't ever construct a replace value like in the example above since it would yield an illegal type name. It's merely a replace value that will echo back all of the capture groups for demonstration purposes.

You might notice that the capture groups are not mutually exclusive. Capture groups can be nested as in the example such that "nsfull" captures a full namespace while "nsroot", "nsstem", and "nsleaf" capture individual components of that namespace. You would use the individual components if you needed to "swap out" any one of the individual components.

The capture group "suffix" in the example above does a pattern match on names that end with "ViewModels". The main purpose of this capture group isn't so that it could be used as part of the transformation, since the purpose of the ViewLocator is to resolve a View name. The main reason to have this capture group is to prevent the substring "ViewModels" from being captured in the "basename" group, which in most cases would be part of the string transformation.

---
><font color="#63aebb" face="微软雅黑">通过将 “ViewModel$” 更改为 “View$”，可以将上述相同的替换模式用于ViewModelLocator。

>你不会像上面的例子那样构造一个替换值，因为它会产生一个非法的类型名称。它只是一个替换值，它将返回所有捕获组以用于演示目的。

>你可能会注意到，捕获组并不相互排斥。捕获组可以嵌套在示例中，例如 “nsfull” 捕获一个完整的命名空间，而 “nsroot”、“nsstem” 和 “nsleaf” 捕获该命名空间的各个组件。如果你需要“交换”任何单个组件，你将使用单个组件。

>上面示例中的捕获组“后缀”对以 “ViewModels” 结尾的名称执行模式匹配。这个捕获组的主要目的不是让它可以用作转换的一部分，因为 ViewLocator 的目的是解析视图名称。这个捕获组的主要原因是防止子字符串 “ViewModels” 在 “basename” 组中被捕获，大多数情况下，这是字符串转换的一部分。</font>

[目录](./index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[The Event Aggregator - 事件聚合器](./event-aggregator.md)