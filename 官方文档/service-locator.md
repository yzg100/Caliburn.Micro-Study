---
layout: page
title: Service Locator
---

Caliburn.Micro comes pre-bundled with a static Service Locator called IoC. For those unfamiliar, a Service Locator is an entity that can provide another entity with service instances, usually based on some type or key. Service Locator is actually a pattern and is related to Inversion of Control. Many consider Service Locator to be an anti-pattern but like all patterns it has its use cases. 

---
><font color="#63aebb" face="微软雅黑">Caliburn.Micro 预先捆绑了一个名为 IoC 的静态服务定位器。对于那些不熟悉的人，服务定位器是一个可以为另一个实体提供服务实例的实体，通常基于某种类型或密钥。服务定位器实际上是一种模式，与控制反转有关。许多人认为 Service Locator 是一种反模式，但就像所有模式一样，它也有自己的用例。</font>

### Getting Started - 入门

IoC is the static entity used for Service Location in Caliburn.Micro, this enables IoC to work with static entites such as dependency properties with ease. The public definition of IoC is shown below.

---
><font color="#63aebb" face="微软雅黑">IoC 是 Caliburn.Micro 中用于服务定位的静态实体，这使IoC能够轻松地处理依赖属性等静态实体。IoC的公开定义如下所示。</font>

``` csharp
public static class IoC {
	public static Func<Type, string, object> GetInstance;
        public static Func<Type, IEnumerable<object>> GetAllInstances;
        public static Action<object> BuildUp;

        public static T Get<T>(string key = null);
	public static IEnumerable<T> GetAll<T>();
}
```

As you can see above much of the functionality of IoC is dependant on the consumer providing it. In most cases the relevant methods required map directly to methods provided by all Dependency Injection containers (although the name and functionality may differ).

---
><font color="#63aebb" face="微软雅黑">如您所见，IoC的许多功能取决于提供它的消费者。在大多数情况下，所需的相关方法直接映射到所有依赖注入容器提供的方法（尽管名称和功能可能不同）。</font>

### Injecting IoC with functionality - 向 IoC 注入功能

Caliburn.Micro requires IoC to work correctly because it takes advantage of it at a framework level. As a non optional service we provide an extensibility point directly on the Bootstrapper for the purposes of injecting functionality into IoC. Below, the sample uses Caliburn.Micro's own SimpleContainer to inject functionality into IoC.

---
><font color="#63aebb" face="微软雅黑">Caliburn.Micro 要求 IoC 正常工作，因为它在框架级别利用。作为非可选服务，我们直接在 Bootstrapper 上提供可扩展点，以便将功能注入 IoC。下面，该示例使用 Caliburn.Micro 自己的 SimpleContainer 为 IoC 注入功能。</font>

``` csharp
public class CustomBootstrapper : Bootstrapper {
	private SimpleContainer _container = new SimpleContainer();
		
	//...

	protected override object GetInstance(Type service, string key) {
		return _container.GetInstance(service, key);
	}

	protected override IEnumerable<object> GetAllInstances(Type service) {
		return _container.GetAllInstances(service);
	}

	protected override void BuildUp(object instance) {
		_container.BuildUp(instance);
	}

	//...
}
```

By mapping your chosen dependency container to IoC, Calburn.Micro can take advantage of any service bindings made on the container via Service Location. 

---
><font color="#63aebb" face="微软雅黑">通过将您选择的依赖容器映射到IoC，Calburn.Micro可以利用通过服务位置对容器进行的任何服务绑定。</font>

### Using IoC in your application - 在您的应用程序中使用 IoC

As stated at the outset Service Location, apart from a few specific areas, is considered by many to be an anti pattern; in most cases you will want to make use of your dependency injection container. Many problems that Service Locator solves can be fixed without it by planning out your applications composition; refer to Screens, Conductors & Composition for more information on composition.

However, if you still require Service Location, IoC makes it easy. The code below shows how to use the service locator to retrieve or inject instances with services.

---
><font color="#63aebb" face="微软雅黑">如开头服务地点所述，除了一些特定的领域，许多人认为这是一种反模式; 在大多数情况下，您将需要使用依赖注入容器。通过规划应用程序组合，可以在没有它的情况下修复 Service Locator 解决的许多问题; 有关成分的更多信息，请参阅Screens, Conductors 和 Composition。

>如果您仍需要服务定位，IoC 可以轻松实现。下面的代码显示了如何使用服务定位器来检索或注入带有服务的实例。</font>

##### Getting a single service - 获得单一服务

IoC supports the retrieval of a single type by type or type and key. Key-based retrieval is not supported by all dependency injection containers, because of this the key param is optional.

---
><font color="#63aebb" face="微软雅黑">IoC 支持按 Type 或 Type 和 Key 检索单个类型。所有依赖注入容器都不支持基于 Key 的检索，因此 Key 参数是可选的。</font>

``` csharp
var windowManager = IoC.Get<IWindowManager>();
var windowManager = IoC.Get<IWindowManager>("windowManager");
```

##### Getting a collection of services - 获得服务集合

Requesting a collection of services is also supported by IoC. The return type is IEnumerable T where T is the type of service requested. LINQ can be used to filter the final collection but be aware that at this point any entity in the collection will have already been instantiated.

---
><font color="#63aebb" face="微软雅黑">IoC 也支持请求服务集合。返回类型是 IEnumerable T，其中 T 是请求的服务类型。LINQ 可以用来过滤最终的集合，但是要注意，此时集合中的任何实体都已经实例化了。</font>

``` csharp
var viewModelCollection = IoC.GetAll<IViewModel>();
var viewModel = IoC.GetAll<IViewModel>().FirstOrDefault(vm => vm.GetType() == typeof(ShellViewModel));
```

##### Injecting an instance with services - 使用服务注入实例

IoC supports the injection of services into a given instance. The mechanics of how this is done is left to the implementation injected into the Action<Instance> BuildUp field. There are various places in the framework were this is used to inject functionality into entities that were created externally to the dependency container mapped to IoC.

---
><font color="#63aebb" face="微软雅黑">IoC 支持将服务注入到给定实例中。如何完成这项工作的机制留给了注入 Action<Instance> BuildUp字段。框架中有许多地方用于将功能注入到映射到 IoC 的依赖容器外部创建的实体中。</font>

``` csharp
var viewModel = new LocallyCreatedViewModel();
IoC.BuildUp(viewModel);
```

[目录](index)&nbsp;&nbsp;|&nbsp;&nbsp;[The Window Manager - 窗口管理器](./window-manager)