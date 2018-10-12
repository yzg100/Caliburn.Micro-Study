---
layout: page
title: The Event Aggregator
---

Caliburn.Micro comes pre-bundled with an Event Aggregator, conveniently called EventAggregator. For those unfamiliar, an Event Aggregator is a service that provides the ability to publish an object from one entity to another in a loosely based fashion. Event Aggregator is actually a pattern and it's implementation can vary from framework to framework. For Caliburn.Micro we focused on making our Event Aggregator implementation simple to use without sacrificing features or flexibility.

---
><font color="#63aebb" face="微软雅黑">Caliburn.Micro 预先绑定了一个事件聚合器，方便地称为事件 EventAggregator。对于那些不熟悉的人，事件聚合器是一种服务，它提供了以一种基于松散的方式将对象从一个实体发布到另一个实体的能力。事件聚合器实际上是一种模式，它的实现可能因框架而异。对于Caliburn.Micro，我们专注于使我们的事件聚合器实现易于使用而不牺牲功能或灵活性。</font>

### Getting Started - 入门

As previously mentioned we provide an implementation of Event Aggregator for you. This implementation implements the IEventAggregator interface, however, you can provide your own implementation if required. Please take a moment to familiarize yourself with the IEventAggregator signature.

---
><font color="#63aebb" face="微软雅黑">如上所述，我们为你提供了一个事件聚合器的实现。这个实现实现了 IEventAggregator 接口，如果需要，你可以提供自己的实现。请花点时间熟悉 IEventAggregator 签名。</font>

``` csharp
public interface IEventAggregator {
    bool HandlerExistsFor(Type messageType);
    void Subscribe(object subscriber);
    void Unsubscribe(object subscriber);
    void Publish(object message, Action<Action> marshal);
}
```

### Creation and Lifecycle - 创建和生命周期

To use the EventAggregator correctly it must exist as an application level service. This is usually achieved by creating an instance of EventAggregator as a Singleton. We recommend that you use Dependency Injection to obtain a reference to the instance although we do not enforce this. The sample below details how to create an instance of EventAggregator, add it to the IoC container included with Caliburn.Micro (although you are free to use any container you wish) and request it in a ViewModel.

---
><font color="#63aebb" face="微软雅黑">要正确使用 EventAggregator，它必须作为应用程序级别的服务存在。这通常是通过创建一个 EventAggregator 实例作为单例来实现的。我们建议你使用依赖注入来获取对实例的引用，尽管我们不强制执行此操作。下面的示例详细说明了如何创建 EventAggregator 实例，将其添加到 Caliburn.Micro 附带的 IoC 容器中（尽管你可以自由使用任何容器）并在 ViewModel 中请求它。</font>

``` csharp
// Creating the EventAggregator as a singleton.
    public class Bootstrapper : BootstrapperBase {
        private readonly SimpleContainer _container =
            new SimpleContainer();

         // ... Other Bootstrapper Config

        protected override void Configure(){
            _container.Singleton<IEventAggregator, EventAggregator>();
        }

        // ... Other Bootstrapper Config
    }

    // Acquiring the EventAggregator in a viewModel.
    public class FooViewModel {
        private readonly IEventAggregator _eventAggregator;

        public FooViewModel(IEventAggregator eventAggregator) {
            _eventAggregator = eventAggregator;
        }
    }
```

Note that we are utilizing the Bootstrapper, and specifically the Configure method, in the code above. There is no requirement to wire up the EventAggregator in a specific location, simply ensure it is created before it is first requested.

---
><font color="#63aebb" face="微软雅黑">注意，我们在上面的代码中使用了Bootstrapper，特别是Configure方法。不需要在特定位置连接EventAggregator，只需确保在首次请求之前创建它即可。</font>

### Publishing Events - 发布事件

Once you have obtained a reference to the EventAggregator instance you are free to begin publishing Events. An Event or message as we call it to distinguish between .Net events, can be any object you like. There is no requirement to build your Events in any specific fashion. As you can see in the sample below the Publish methods can accept any entity that derives from System.Object and will happily publish it to any interested subscribers.

---
><font color="#63aebb" face="微软雅黑">获得对EventAggregator实例的引用后，你就可以自由地发布事件了。Event 和 message 是 .NET 的一系列事件，它可以是任何对象，无需以任何特定方式构建你的事件。正如下面的示例中所看到的，Publish 方法可以接受从 System.Object 派生的任何实体，并将其发布给任何对它感兴趣的订阅者。</font>

``` csharp
public class FooViewModel {
    private readonly IEventAggregator _eventAggregator;

    public FooViewModel(IEventAggregator eventAggregator) {
        _eventAggregator = eventAggregator;

        _eventAggregator.PublishOnUIThread(new object());
        _eventAggregator.PublishOnUIThread("Hello World");
        _eventAggregator.PublishOnUIThread(22);
    }
}
```

### Publishing using a custom thread Marshal - 使用自定义线程 Marshal 发布

By convention, the EventAggregator publishes on the UI thread (using PublishOnUIThread() method). You can override this per publish. Consider the following code below which publishes the message supplied on a background thread.

---
><font color="#63aebb" face="微软雅黑">按照惯例，EventAggregator 在 UI 线程上发布(使用 PublishOnUIThread()方法)。你可以在每次发布时重写此命令。考虑下面的代码，它在后台线程上提供消息。</font>

``` csharp
_eventAggregator.Publish(new object(), action => {
    Task.Factory.StartNew(action);
});
```

### Subscribing To Events - 事件订阅

Any entity can subscribe to any Event by providing itself to the EventAggregator via the Subscribe method. To keep this functionality easy to use we provide a special interface (IHandle<T>) which marks a subscriber as interested in an Event of a given type.

---
><font color="#63aebb" face="微软雅黑">任何实体都可以通过 Subscribe 方法将自身提供给 EventAggregator 来订阅任何事件。为了使这个功能易于使用，我们提供了一个特殊的接口(IHandle<T>)，它将订阅者标记为对给定类型的事件感兴趣。</font>

``` csharp
IHandle<TMessage> : IHandle {
    void Handle<TMessage>(TMessage message);
}
```

Notice that by implementing the interface above you are forced to implement the method Handle(T message) were T is the type of message you have specified your interest in. This method is what will be called when a matching Event type is published. 

---
><font color="#63aebb" face="微软雅黑">请注意，通过实现上面的接口，你必须实现方法Handle（T message）T是你已指定自己感兴趣的消息类型。当发布匹配的事件类型时，将调用此方法。</font>

``` csharp
public class FooViewModel : IHandle<object> {
    private readonly IEventAggregator _eventAggregator;

    public FooViewModel(IEventAggregator eventAggregator) {
        _eventAggregator = eventAggregator;
        _eventAggregator.Subscribe(this);
    }

    public void Handle(object message) {
        // Handle the message here.
    }
}
```

### Subscribing To Many Events - 订阅更多事件

It is not uncommon for a single entity to want to listen for multiple event types. Because of our use of generics this is as simple as adding a second IHandle<T> interface to the subscriber. Notice that Handle method is now overloaded with the new Event type.

---
><font color="#63aebb" face="微软雅黑">单个实体希望侦听多个事件类型的情况并不少见。由于我们使用了泛型，这就像向订阅服务器添加第二个 IHandle<T> 接口一样简单。Handle 方法现在使用新的 Event 类型重载。</font>

``` csharp
public class FooViewModel : IHandle<string>, IHandle<bool> {
    private readonly IEventAggregator _eventAggregator;

    public FooViewModel(IEventAggregator eventAggregator) {
        _eventAggregator = eventAggregator;
        _eventAggregator.Subscribe(this);
    }

    public void Handle(string message) {
        // Handle the message here.
    }

    public void Handle(bool message) {
        // Handle the message here.
    }
}
```

### Polymorphic Subscribers - 多态订阅

Caliburn.Micro's EventAggregator honors polymorphism. When selecting handlers to call, the EventAggregator will fire any handler who's Event type is assignable from the Event being sent. This results in a lot of flexibility and helps reuse.

---
><font color="#63aebb" face="微软雅黑">Caliburn.Micro EventAggregator 支持多态。选择要调用的处理程序时，EventAggregator 将触发任何可从发送的事件分配事件类型的处理程序。这带来了很大的灵活性，并有助于重用。</font>

``` csharp
public class FooViewModel : IHandle<object>, IHandle<string> {
    private readonly IEventAggregator _eventAggregator;

    public FooViewModel(IEventAggregator eventAggregator) {
        _eventAggregator = eventAggregator;
        _eventAggregator.Subscribe(this);
        _eventAggregator.PublishOnUIThread("Hello");
    }

    public void Handle(object message) {
        // This will be called
    }

    public void Handle(string message) {
        // This also
    }
}
```

In the example above, because String is derived from System.Object both handlers will be called when a String message is published.

---
><font color="#63aebb" face="微软雅黑">在上面的示例中，因为 String 是从 System.Object 派生的，所以在发布 String 消息时将调用这两个处理程序。</font>

### Querying Handlers - 查询处理程序

When a subscriber is passed to the EventAggregator it is broken down into a special object called a Handler and a weak reference is maintained. We provide a mechanism to query the EventAggregator to see if a given Event type has any handlers, this can be useful in specific scenarios were at least one handler is assumed to exist.

---
><font color="#63aebb" face="微软雅黑">当订阅被传递给 EventAggregator 时，它被分解为一个名为 Handler 的特殊对象，并保持弱引用。我们提供了一种查询 EventAggregator 的机制，以查看给定 Event 类型是否有任何处理程序，这在假设至少存在一个处理程序的特定场景中非常有用。</font>

``` csharp
public class FooViewModel : IHandle<object> {
    private readonly IEventAggregator _eventAggregator;

    public FooViewModel(IEventAggregator eventAggregator) {
        _eventAggregator = eventAggregator;
        _eventAggregator.Subscribe(this);
    }

    public void Handle(object message){
        if (_eventAggregator.HandlerExistsFor(typeof(SpecialMessageEvent))){
            _eventAggregator.PublishOnUIThread(new SpecialEventMessage(message));
        }
    }
}
```

### Coroutine Aware Subscribers - Coroutine Aware 订阅

If you are using the EventAggregator with Caliburn.Micro as opposed to on it's own via Nuget, access to Coroutine support within the Event Aggregator becomes available. Coroutines are supported via the IHandleWithCoroutine<T> Interface.

---
><font color="#63aebb" face="微软雅黑">如果你使用的是带有 Caliburn.Micro 的 EventAggregator，而不是通过 Nuget 单独使用，则可以访问Event Aggregator中的Coroutine支持。协同程序通过IHandleWithCoroutine<T>接口支持。</font>

``` csharp
public interface IHandleWithCoroutine<TMessage> : IHandle {
    IEnumerable<IResult> Handle(TMessage message);
}
```

The code below utilizes Coroutines with the EventAggregator. In this instance Activate will be fired asynchronously, Do work however, will not be called until after Activate has completed.

---
><font color="#63aebb" face="微软雅黑">下面的代码使用 Coroutines 和 EventAggregator。在这种情况下，Activate将以异步方式触发，但是，在 Activate 完成之后才会调用。</font>

``` csharp
public class FooViewModel : Screen, IHandleWithCoroutine<EventWithCoroutine> {
    private readonly IEventAggregator _eventAggregator;

    public FooViewModel(IEventAggregator eventAggregator) {
        _eventAggregator = eventAggregator;
        _eventAggregator.Subscribe(this);
    }

    public IEnumerable<IResult> Handle(EventWithCoroutine message) {
        yield return message.Activate();
        yield return message.DoWork();
    }
}
```

``` csharp
public class EventWithCoroutine {
    public IResult Activate() {
        return new TaskResult(Task.Factory.StartNew(() => {
                // Activate logic
        }));
    }

    public IResult DoWork() {
        return new TaskResult(Task.Factory.StartNew(() => {
            // Do work logic
        }));
    }
}
```

### Task Aware Subscribers - 任务感知订阅

Caliburn.Micro also provides support for task based subscribers where the asynchronous functionality of Coroutines is desired but in a more light weight fashion. To utilize this functionality implement the IHandleWithTask<T> Interface, seen below.

---
><font color="#63aebb" face="微软雅黑">Caliburn.Micro 还提供了对基于任务的订阅支持,在这些订阅者中，协同程序的异步功能是必须的，它是一种更轻量级的方式。要使用此功能，请实现 IHandleWithTask<T> 接口，如下所示。</font>

``` csharp
public interface IHandleWithTask<TMessage> : IHandle {
    Task Handle(TMessage message);
}
```

Any subscribers that implement the above interface can then handle events in Task based manner.

---
><font color="#63aebb" face="微软雅黑">实现上述接口的任何订阅者都可以以基于任务的方式处理事件。</font>

``` csharp
public class FooViewModel : Screen, IHandleWithTask<object> {
    private readonly IEventAggregator _eventAggregator;

    public FooViewModel(IEventAggregator eventAggregator) {
        _eventAggregator = eventAggregator;
        _eventAggregator.Subscribe(this);
    }

    public Task Handle(object message) {
        return Task.Factory.StartNew(() => message);
    }
}
```

### Unsubscribing and Leaks - 取消订阅和泄漏

The problem with standard .Net events is that they are prone to memory leaks. We avoid this situation by maintaining a weak reference to subscribers. If the only thing that references a subscriber is the EventAggregator then it will be allowed to go out of scope and ultimately be garbage collected. However, we still provide an explicit way to unsubscribe to allow for conditional handling as seen below.

---
><font color="#63aebb" face="微软雅黑">标准 .Net 事件的问题在于它们容易发生内存泄漏。我们通过维持对订阅者的弱引用来避免这种情况。如果引用订阅者的唯一事物是 EventAggregator，那么它将被允许超出范围并最终被垃圾回收。但是，我们仍然提供了一种明确的取消订阅方式，根据条件来处理，如下所示。</font>

``` csharp
public class FooViewModel : Screen, IHandle<object> {
    private readonly IEventAggregator _eventAggregator;

    public FooViewModel(IEventAggregator eventAggregator) {
        _eventAggregator = eventAggregator;
        _eventAggregator.Subscribe(this);
    }

    public void Handle(object message) {
        // Handle the message here.
    }

    protected override void OnActivate() {
        _eventAggregator.Subscribe(this);
        base.OnActivate();
    }

    protected override void OnDeactivate(bool close) {
        _eventAggregator.Unsubscribe(this);
        base.OnDeactivate(close);
    }
}
```

In the code above Screen is used to expose lifecycle events on the ViewModel. More on this can be found on this in the Screens, Conductors and Composition article on this wiki.

---
><font color="#63aebb" face="微软雅黑">在上面的代码中，Screen 用于公开 ViewModel 上的生命周期事件。有关此内容的更多信息，请参阅此维基上的 Screens, Conductors，Composition 的文章。</font>

### Custom Result Handling - 自定义结果处理

In more complex scenarios it may be required to override the default handling of Handlers which have results. In this instance it is possible to swap out the existing implementation in favour of your own. First we create a new handler type.

---
><font color="#63aebb" face="微软雅黑">在更复杂的场景中，可能需要覆盖具有结果的处理程序的默认处理。在这种情况下，可以替换现有的实现，以支持你的实现。首先，我们创建一个新的处理程序类。</font>

``` csharp
IHandleAndReturnString<T> : IHandle {
    string Handle<T>(T message);
}
```

Next we create our new result processor. This can be configured in the bootstrapper.

---
><font color="#63aebb" face="微软雅黑">接下来，我们创建新的结果处理器。这可以在引导程序中配置。</font>

``` csharp
var standardResultProcesser = EventAggregator.HandlerResultProcessing;
EventAggregator.HandlerResultProcessing = (target, result) =>
{
    var stringResult = result as string;
    if (stringResult != null)
        MessageBox.Show(stringResult);
    else
        standardResultProcesser(target, result);
};
```

Now, any time an event is processed returns a string, it will be captured and displayed in a MessageBox. The new handler falls back to the default implementations in cases were the result is not assignable from string. It is extremely important to note however, this feature was not designed for request / response usage, treating it as such will most definitely create bottle necks on publish.

---
><font color="#63aebb" face="微软雅黑">现在，每当处理一个事件返回一个字符串时，它都会被捕获并显示在一个 MessageBox 中。在无法从 String 分配结果的情况下，新处理程序将返回到默认实现。需要特别注意的是，这个特性并不是为请求/响应的使用而设计的，因此对其进行处理肯定会在发布时产生瓶颈。</font>


[目录](./index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[Simple IoC Container - 简单的IoC容器](./simple-container.md)