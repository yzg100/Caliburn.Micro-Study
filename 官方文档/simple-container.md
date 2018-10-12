---
layout: page
title: Simple IoC Container
---

Caliburn.Micro comes pre-bundled with a Dependency Injection container called SimpleContainer. For those unfamiliar, a dependency injection container is an object that is used to hold dependency mappings for use later in an app via Dependency Injection. Dependency Injection is actually a pattern typically using the container element instead of manual service mapping.

---
><font color="#63aebb" face="微软雅黑">Caliburn.Micro 预先绑定了一个名为 SimpleContainer 的依赖注入容器。对于那些不熟悉依赖注入容器的人来说，依赖注入容器是一个对象，用于保存依赖映射，以便以后通过依赖注入在应用程序中使用。依赖注入实际上是一种常用容器元素而不是手动服务映射的模式。</font>

### Getting Started - 入门

SimpleContainer is the main class used for Dependency Injection in Caliburn.Micro. There are other classes which act as supporting infrastructure in the form of extension methods, which are discussed later. For now, the public definition of SimpleContainer is shown below.

---
><font color="#63aebb" face="微软雅黑">SimpleContainer 是 Caliburn.Micro 中用于依赖注入的主要类。还有其他类以扩展方法的形式充当支持基础设施，稍后将对此进行讨论。目前，SimpleContainer的公共定义如下所示。</font>

``` csharp
public SimpleContainer()

void RegisterInstance(Type service, string key, object implementation)
void RegisterPerRequest(Type service, string key, Type implementation)
void RegisterSingleton(Type service, string key, Type implementation)
void RegisterHandler(Type service, string key Func<SimpleContainer, object> handler)
void UnregisterHandler(Type service, string key)

object GetInstance(Type service, string key)
IEnumerable<object> GetAllInstances(Type service)
void BuildUp(object instance)

SimpleContainer CreateChildContainer()
bool HasHandler(Type service, string key)

event Action<object> Activated
```

As you can see above, the api is broken into service registration and retrieval with a couple of support methods and a single event.

---
><font color="#63aebb" face="微软雅黑">正如您在上面看到的，这个api被分解为服务注册和检索，有两个支持方法和一个事件。</font>

### Creation and Configuration - 创建和配置

Before adding any service bindings to SimpleContainer, it is important to remember that the container itself must be registered with Caliburn.Micro for the framework to make use of the aforementioned bindings. This process will inject SimpleContainer into IoC which is Caliburn.Micro's built in Service Locator. The code below details how to register SimpleContainer with Caliburn.Micro.

---
><font color="#63aebb" face="微软雅黑">在向 SimpleContainer 添加任何服务绑定之前，必须记住容器本身必须注册到 Caliburn.Micro，以便框架使用上述绑定。这个过程会将SimpleContainer注入到IoC中，这是 Caliburn.Micro 内置的 Service Locator。下面的代码详细说明了如何使用 Caliburn.Micro 注册 SimpleContainer。</font>

``` csharp
public class CustomBootstrapper : BootstrapperBase {
	private SimpleContainer _container = new SimpleContainer();

	...

	protected override object GetInstance(Type serviceType, string key) {
		return _container.GetInstance(serviceType, key);
	}

	protected override IEnumerable<object> GetAllInstances(Type serviceType) {
		return _container.GetAllInstances(serviceType);
	}

	protected override void BuildUp(object instance) {
		_container.BuildUp(instance);
	}

	...
}
```

As you can see above there are 3 methods which need to be overriden to correctly register SimpleContainer with Caliburn.Micro. You may refer to the Bootstrapper documentation for more information on the methods above.

---
><font color="#63aebb" face="微软雅黑">如您所见，有三种方法需要重写才能正确注册 SimpleContainer 和 Caliburn.Micro。有关上述方法的更多信息，请参阅 Bootstrapper 文档。</font>

### Registering Service Bindings - 注册服务绑定

SimpleContainer provides many different ways to create service bindings based on lifecycle needs. On top of this, there are a number of extension methods included with Caliburn.Micro that provide even more flexibility when choosing to register services.

---
><font color="#63aebb" face="微软雅黑">SimpleContainer 提供了许多不同的方法来根据生命周期需求创建服务绑定。除此之外，Caliburn.Micro 还包含许多扩展方法，在选择注册服务时提供更大的灵活性。</font>


##### Register Instance - 注册实体

The RegisterInstance method allows a pre-constructed instance to be registered with the container against a type, key or both. Instance on the other hand registers a pre-constructed instance to be registered against a type only.

---
><font color="#63aebb" face="微软雅黑">RegisterInstance 方法允许根据Type，Key 或 Type 与 Key 向容器注册预构造的实例。另一方面，实例注册仅针对类型注册的预构造实例。</font>

``` csharp
void RegisterInstance(Type serviceType, string key, object instance);
SimpleContainer Instance<TService>(TService instance)
```

Registering a pre-constructed instance is useful when an entity is required to be in a specific state before the first request for it is made.

---
><font color="#63aebb" face="微软雅黑">当实体在第一次请求之前需要处于特定状态时，注册预构造实例很有用。</font>

##### Register Per Request - 请求注册

SimpleContainer contains 2 methods to handle per request registration. RegisterPerRequest registers an implementation to be registered against a type, key or both. PerRequest is overloaded to enable registration against the implementation's type or another type it implements or inherits from. PerRequest calls can be chained.

---
><font color="#63aebb" face="微软雅黑">SimpleContainer 包含两个方法来处理每个注册请求。RegisterPerRequest 注册要针对Type，Key 或 Type 与 Key 的注册实现。重载 PerRequest 以启用对实现类型或其实现或继承的其他类型的注册。可以链接 PerRequest 调用。</font>

``` csharp
void RegisterPerRequest(Type serviceType, string key, Type implementation);
SimpleContainer PerRequest<TService, TImplementation>()
SimpleContainer PerRequest<TImplementation>()
```

Per Request registration causes the creation of the returned entity once per request. This means that two different requests for the same entity will result in two distinct instances being created.

---
><font color="#63aebb" face="微软雅黑">每个请求注册会导致每个请求创建一次返回的实体。这意味着对同一实体的两个不同请求将导致创建两个不同的实例。</font>

##### Register Singleton - 注册Singleton

Singleton registration, like PerRequest, has 2 different methods for registration. RegisterSingleton registers an implementation against a type, key or both while Singleton is overloaded to enable registration against the implementation's type or another type it implements or inherits from. Singleton calls can be chained.

---
><font color="#63aebb" face="微软雅黑">Singleton 注册与 PerRequest 一样，有两种不同的注册方法。RegisterSingleton 针对Type，Key 或 Type 与 Key 注册实现，而 Singleton 被重载以启用对实现类型或继承的其他类型的注册。单例调用可以被链接。</font>

``` csharp
void RegisterSingleton(Type serviceType, string key, Type implementation);
SimpleContainer Singleton<TImplementation>()
SimpleContainer Singleton<TService, TImplementation>()
```

Registering Singletons may look the same as registering instances but there is an important difference in lifecycle. Instances are pre constructed before registration while singleton registrations are only constructed when first requested.

---
><font color="#63aebb" face="微软雅黑">注册单例可能看起来与注册实例相同，但生命周期中存在重要差异。实例在注册前预先构建，而单例注册仅在首次请求时构建。</font>

##### Register Handler - 注册处理程序

Factories, or more specifically, factory methods can be registered with the Handler method. The Handler method takes a Func<SimpleContainer, object> as its parameter; This allows the factory method to take advantage of the container itself which is useful in complex construction scenarios.

---
><font color="#63aebb" face="微软雅黑">可以使用 Handler 方法注册工厂，或更具体地说，工厂方法可以用处理程序方法注册。Handler方法将 Func <SimpleContainer，object> 作为参数; 这允许工厂方法利用容器本身，这在复杂的构造场景中非常有用。</font>

``` csharp
SimpleContainer Handler<TService>(Func<SimpleContainer, object> handler)
```

Some registrations may require multiple context sensitive implementations to be registered. Handles provides a convenient way to wrap this logic up and make use of it at the time of requesting.

Note: All of the above registration methods actually use Handles under the covers.

---
><font color="#63aebb" face="微软雅黑">某些注册可能需要注册多个上下文敏感的实现。Handles提供了一种方便的方法来包装此逻辑并在请求时使用它。

>注意：以上所有注册方法实际上都使用了Handles。</font>

##### Register All Types Of - 注册所有类型

SimpleContainer provides basic assembly inspection. AllTypesOf allows an assembly inspected for any Implementation that implements or inherits the service type being registered. Optionally a filter can be provided to narrow the collection of implementations registered.

---
><font color="#63aebb" face="微软雅黑">SimpleContainer提供基本的装配检查。AllTypesOf允许检查任何实现或继承正在注册的服务类型的实现的程序集。可以选择提供一个过滤器来缩小已注册实现的集合。</font>


``` csharp
SimpleContainer AllTypesOf<TService>(Assembly assembly, Func<Type, bool> filter = null)
```

Assembly inspection is useful when dealing with modular systems where assemblies may not be available or known until run time.

---
><font color="#63aebb" face="微软雅黑">在处理模块化系统时，装配检查非常有用，在模块化系统中，装配可能无法在运行时使用或知道。</font>

### Injecting Services - 注入服务

The main benefit of Dependency injection is that any service requested will have it's dependencies resolved before it is returned to the caller. This is recursive so dependencies are satisfied for the whole object graph returned. This process can also be utilized on instances that did not originate from the dependency container in the form of property injection.

---
><font color="#63aebb" face="微软雅黑">依赖注入的主要好处是，任何被请求的服务在返回给调用者之前都会解决它的依赖关系。这是递归的，因此对返回的整个对象图的依赖性都得到了满足。这个过程也可以用于不是以属性注入的形式从依赖容器中产生的实例。</font>

##### Constructor Injection - 构造函数注入

Constructor injection is the most widely used form of dependency injection and it denotes a required dependency between services and the class into which they are injected. Constructor injection should be used when you require the non optional use of a given service.

---
><font color="#63aebb" face="微软雅黑">构造函数注入是最广泛使用的依赖注入形式，它表示服务与注入它们的类之间所需的依赖关系。当您需要非可选地使用给定服务时，应该使用构造函数注入。</font>

``` csharp
public class ShellViewModel {
	private readonly IWindowManager _windowManager;
	
	public ShellViewModel(IWindowManager windowManager) {
		_windowManager = windowManager;
	}
}
```

By specifying IWindowManager as a constructor parameter we are explicitly requesting it as a non optional service. If ShellViewModel gets constructed by the dependency container it will have an implementation of IWindowManager injected into it.

---
><font color="#63aebb" face="微软雅黑">通过将IWindowManager指定为构造函数参数，我们明确地将其作为非可选服务请求它。如果ShellViewModel由依赖容器构造，它将注入IWindowManager的实现。</font>

##### Property Injection - 属性注入

Property Injection provides the ability to inject services into an entity created outside of the dependency container. When an entity is passed into the BuildUp method its properties will be inspected and any available matching services will be injected using the same recursive logic as above.

---
><font color="#63aebb" face="微软雅黑">Property Injection提供了将服务注入到依赖容器外部创建的实体的功能。当实体传递到BuildUp方法时，将检查其属性，并使用与上面相同的递归逻辑注入任何可用的匹配服务。</font>

``` csharp
...
	var shellViewModel = new ShellViewModel();
		_container.BuildUp(shellViewModel);
	}
}


public class ShellViewModel {
	public IEventAggregator EventAggregator { get; set; }
}
```

In most cases constructor injection is the best option because it makes service requirements explicit, however property injection has many use cases. It is important to note that property injection only works for Interface types. 

---
><font color="#63aebb" face="微软雅黑">在大多数情况下，构造函数注入是最好的选择，因为它使服务需求显式化，然而属性注入有许多用例。需要注意的是，属性注入只适用于接口类型。</font>

### Advanced Features - 高级功能

The techniques discussed in the Getting Started section are more than enough for most applications but SimpleContainer also provides some advanced features that can help with complex registration or retrieval scenarios.

---
><font color="#63aebb" face="微软雅黑">入门部分中讨论的技术对于大多数应用程序来说已经足够了，但SimpleContainer还提供了一些可以帮助进行复杂注册或检索方案的高级功能。</font>

##### Handling Instance Activation - 处理实例激活

SimpleContainer provides the Activate event that is raised when a service is requested from the container and its corresponding implementation is created thus allowing you to perform any custom initialization or operation you wish on the newly created instance. An Example of this can be seen below.

---
><font color="#63aebb" face="微软雅黑">SimpleContainer 提供从容器请求服务时引发的 Activate 事件，并创建其相应的实现，从而允许您在新创建的实例上执行任何自定义初始化或操作。下面是例子:</font>

``` csharp
class CustomBootstrapper : BootstrapperBase {

	private SimpleContainer _container

	...
			
	protected override void Configure() {
		...
		_container.Activate += _OnInstanceActivation;
	}

	private void OnInstanceActivation(object instance) {
		// Perform any custom operations
	}
}
```

##### Utilizing Child Containers - 利用子容器

Child Containers are useful in complex modular application scenarios. Requesting a child container will create a new instance of SimpleContainer with all of the currently registered services copied over. Any new instances registered with the parent will not be registered in the child container implicitly

---
><font color="#63aebb" face="微软雅黑">子容器在复杂的模块化应用程序场景中非常有用。请求子容器将创建SimpleContainer的新实例，该实例包含复制过来的所有当前已注册的服务。在父容器中注册的任何新实例都不会在子容器中隐式注册。</font>

``` csharp
var childContainer = _simpleContainer.CreateChildContainer();
```

Do remember that due to the nature of singleton and instance registrations that both containers will effectively be accessing the same instance. This allows complex registration scenarios to be achieved.

---
><font color="#63aebb" face="微软雅黑">请记住，由于单例和实例注册的性质，两个容器将有效地访问同一个实例。这允许实现复杂的注册场景。</font>

[目录](./index.md)&nbsp;&nbsp;|&nbsp;&nbsp;[Service Locator - 服务定位](./service-locator.md)