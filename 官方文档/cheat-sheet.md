---
layout: page
title: Cheat Sheet
---
这是对 Caliburn.Micro 项目中最常用的约定和特性的快速指南。

### 连接事件
自动连接控件上的事件以调用 ViewModel 上的方法。

#### 约定

``` xml
<Button x:Name="Save">
```
按钮的单击事件调用 ViewModel 上的“Save”方法

#### 短语法

``` xml
<Button cal:Message.Attach="Save">
```

这样同样可以调用 ViewModel 中的 "Save" 方法。
可以这样使用不同的事件:

``` xml
<Button cal:Message.Attach="[Event MouseEnter] = [Action Save]">
```

不同的参数可以这样传递给方法:(参数中的 this 参数传递的是对应的 Model )

``` xml
<Button cal:Message.Attach="[Event MouseEnter] = [Action Save($this)]"> 
``` 

<dl>
	<dt>$eventArgs</dt>
	<dd>将 EventArgs 或输入参数传递给你的操作。注意:对于保护方法，这将为null，因为触发器实际上并没有发生.</dd>
	<dt>$dataContext</dt>
	<dd>传递 ActionMessage 附加到的元素的 DataContext 。这在主/明细场景中非常有用，在这些场景中，ActionMessage 可能会冒泡到父 VM，但需要携带要执行操作的子实例.</dd>
	<dt>$source</dt>
	<dd>触发要发送的 ActionMessage 的实际 FrameworkElement.</dd>
	<dt>$view</dt>
	<dd>绑定到 ViewModel 的视图(通常是 UserControl 或 Window).</dd>
	<dt>$executionContext</dt>
	<dd>执行 Action 的上下文，包含上述所有信息和更多信息。这在高级场景中非常有用.</dd>
	<dt>$this</dt>
	<dd>操作附加的实际 UI 元素。在本例中，元素本身不会作为参数传递，而是作为默认属性传递.</dd>
</dl>


#### 长语法

``` xml
<UserControl x:Class="Caliburn.Micro.CheatSheet.ShellView"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation" 
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml" 
    xmlns:i="clr-namespace:System.Windows.Interactivity;assembly=System.Windows.Interactivity" 
    xmlns:cal="http://www.caliburnproject.org"> 
    <StackPanel> 
        <TextBox x:Name="Name" />
        <Button Content="Save"> 
            <i:Interaction.Triggers> 
                <i:EventTrigger EventName="Click"> 
                    <cal:ActionMessage MethodName="Save"> 
                       <cal:Parameter Value="{Binding ElementName=Name, Path=Text}" /> 
                    </cal:ActionMessage> 
                </i:EventTrigger> 
            </i:Interaction.Triggers> 
        </Button> 
    </StackPanel> 
</UserControl>
```

这种语法是 Expression Blend 支持的.

### 数据绑定

这会自动将控件上的依赖项属性绑定到 ViewModel 上的属性。

#### 约定

``` xml
<TextBox x:Name="FirstName" />
```

将使 TextBox 的 “Text” 属性绑定到 ViewModel 上的“FirstName” 属性.

#### Explicit - 明确的

``` xml
<TextBox Text="{Binding Path=FirstName, Mode=TwoWay}" />
```

这是绑定属性的常规方式.

#### 事件聚合器

Event Aggregator 的三种不同方法:

``` csharp
public interface IEventAggregator {  
    void Subscribe(object instance);  //订阅
    void Unsubscribe(object instance);  //退订
    void Publish(object message, Action<System.Action> marshal);  //发布
}
```

事件可以是简单的类，例如:

``` csharp
public class MyEvent {
    public MyEvent(string myData) {
        this.MyData = myData;
    }

    public string MyData { get; private set; }
}
```

[目录](./index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[Migrating to 2.0.0 - 迁移到2.0.0](./migrating-to-2.0.0.md)