Before you get started with StrucutureMap it's recommended that you first get comfortable with some of the [design concepts]() and the basic patterns of [Dependency Injection]() and [Inversion Of Control](). It's important that you structure and build your application with this concepts in mind.

Assuming that you're already familiar with those concepts, or you'd really rather skip the pedantry and jump right into concrete code, the first thing to do is, go [get StructureMap]() and jump into usage.

### Table of Contents

1. Main Usage.
2. Configuring the container.
3. Resolving instances of a service or dependency.
4. An even simpler usage.
5. Integrating StructureMap into your application or framework.
6. What to do when things go wrong?
7. Need Help?

### Main Usage

The main usage of StructureMap consist of the following two steps:

1. Configuring the container by registering the **what** and **how** StructureMap should build or find requested services.

2. Resolving instances of a service or dependency.

### Configuring the container

Configuring also known as bootstrapping the container is possible in various and mixed ways. The recommended way to do this, is the usage of the [Registry DSL](), which provide you a way to define code as configuration in a stronly typed fashion. While the [Registry DSL]() is the strongly recommended means of configuring StructureMap, there will be times when reverting to the other supported [XML configuration]() is appropriate. The configuration is highly modular and you can mix and match all of the configuration choices within the same `Container` instance.

The first step in using StructureMap is configuring an `Container` object or `ObjectFactory`. The following examples are based on the usage of the [Registry DSL]().

```csharp
public interface IBar { }
public class Bar : IBar { }

public interface IFoo { }
public class Foo : IFoo
{
    public IBar Bar { get; private set; }

    public Foo(IBar bar)
    {
		if (bar == null) throw new ArgumentNullException("bar");
        Bar = bar;
    }
}
```
```csharp 
// Example #1 - Create an container instance and directly pass in the configuration.
var container1 = new Container(c =>
{
    c.For<IFoo>().Use<Foo>();
    c.For<IBar>().Use<Bar>();
});

// Example #2 - Create an container instance but add configuration later.
var container2 = new Container();

container2.Configure(c =>
{
    c.For<IFoo>().Use<Foo>();
    c.For<IBar>().Use<Bar>();
});

// Example #3 - Initialize the static ObjectFactory container.
ObjectFactory.Initialize(i =>
{
    i.For<IFoo>().Use<Foo>();
    i.For<IBar>().Use<Bar>();
});
```

Initializing or configuring the container is usually done at application startup and is located as close as possible to the application's entry point. This place is sometimes referred to as the composition root of the application. Like in our example we are composing our application's object graph by connecting abstractions to concrete types.

We are using the fluent API `For<PLUGIN_TYPE>().Use<CONCRETE_TYPE>()` which registers a default instance for a given plugin type. In our example we want an new instance of `Foo` every time we request the abstraction `IFoo`.

The recommended way of using the [Registry DSL]() is by defining one or more `Registry` classes. Typically, you would subclass the `Registry` class, then use the Fluent API methods exposed by the `Registry` class to create `Container` configuration. Here's a sample `Registry` class below used to configure the same types as in our previous example.


```csharp
public class FooBarRegistry : Registry
{
    public FooBarRegistry()
    {
        For<IFoo>().Use<Foo>();
        For<IBar>().Use<Bar>();
    }
}
```
When you set up a `Container` or `ObjectFactory`, you need to simply direct the `Container` to use the configuration in that `Registry` class.

```csharp
// Example #1
var container1 = new Container(new FooBarRegistry());

// Example #2
var container2 = new Container(c =>
{
    c.AddRegistry<FooBarRegistry>();
});

// Example #3
ObjectFactory.Initialize(c =>
{
    c.AddRegistry<FooBarRegistry>();
});
```

In real world applications you also have to deal with repetitive similar registrations. Such registrations are tedious, easy to forget and can be a weak spot in your application. StructureMap provides [auto-registrations]() policies which mitigates this pain and eases the maintenance burden. StructureMap exposes this feature through the [Registry DSL]() by the `Scan` method.

In our example there is an reoccuring pattern, we are connecting the plugin type `ISomething` to a concrete type `Something`, meaning `IFoo` to `Foo` and `IBar` to `Bar`. Wouldn't it be cool if we could write a convention for exactly doing that? Fortunatly StructureMap has already one build in. Let's see how we can create an container with the same configuration as in the above examples.

```csharp
// Example #1
var container1 = new Container(c =>
    c.Scan(scanner =>
    {
        scanner.TheCallingAssembly();
        scanner.WithDefaultConventions();
    }));

// Example #2
var container2 = new Container();

container2.Configure(c =>
    c.Scan(scanner =>
    {
        scanner.TheCallingAssembly();
        scanner.WithDefaultConventions();
    }));

// Example #3
ObjectFactory.Initialize(i => 
    i.Scan(scanner =>
	{
	    scanner.TheCallingAssembly();
	    scanner.WithDefaultConventions();
	}));
```

We instruct the scanner to scan through the calling assembly with default conventions on. This wil find and registers the default instance for `IFoo` and `IBar` which are obviously the concrete types `Foo` and `Bar`. Now whenever you add an additional interface `IMoreFoo` and a class `MoreFoo` to your application's code base, it's automatically picked up by the scanner. 

Sometimes classes need to be suplied with some primitive value in its constructor. For example the `System.Data.SqlClient.SqlConnection` needs to be supplied with the connection string in its constructor. No problem, just set up the value of the constructor argument in the bootstrapping:

```csharp
//just for demostration purposes, normally you don't want to embed the connection string directly into code.
For<IDbConnection>().Use<SqlConnection>().Ctor<string>().Is("YOUR_CONNECTION_STRING");

//a better solution is to pull the connection string from your AppSettings
For<IDbConnection>().Use<SqlConnection>().Ctor<string>().EqualToAppSetting("CONNECTION-STRING");
```
StructureMap will look up the "CONNECTION-STRING" value in the AppSettings portion of the App.config or Web.config file, and use that string value to invoke the constructor function of the `SqlConnection` class, then hand back that new `SqlConnection` object. 

So far you have seen an couple of ways to work with the registry DSL and configure an `Container` object or `ObjectFactory`. We have seen examples of configuration that allows us to build objects that doesn't depend on anything like the `Bar` class, or do depend on other types like the `Foo` class needs an instance of `IBar`. In our last example we have seen configuration for objects that needs some primitive types like strings in its constructor function.

Now that we have done some basic configuration, it's time to move on to the next step of the main usage: resolving instances of a service or dependency.

### Resolving instances of a service or dependency

This will be the easy part of interacting with StructureMap. During application execution, you will be needing the services you registered in the container. When you resolve a service, depending on the [lifecycle](), a new instance of the service will be created by StructureMap or StructureMap is delegating the responsibility of resolving the instance to external code. This all depends on your configuration for that specific service. All of our examples so far let's StructureMap create the instance for the service.

In our bootstrapping of the container we didn't specified any lifecycle. In short, a lifecycle let's you specify how long an instance is tracked by StructureMap. More information on this subject can be found in the [lifestyle]() chapter. The default lifecycle for all services is [`PerRequest`](). This means that a new instance will be created for each request to the container. These instances aren't tracked by the container. Lifecycles for services can be defined through code or XML.

Before we can resolve services from the container, we need an reference to a `Container` object or `ObjectFactory`. For our examples we are going to use the static `ObjectFactory` class, which is StructureMap's implementation of a [Service Locator]().

```csharp
var fooInstance = ObjectFactory.GetInstance<IFoo>();
```

This will create an instance of the class `Foo`. Before creating this instance, StructureMap will inspect the constructor and see that it needs to supply an instance of `IBar`. Because we also provided configuration for `IBar`, StructureMap wil be able to create an instance of the `Bar` class and injects that into the constructor of the `Foo` class. The resulting instance of `Foo` will be returned to the caller.

StructureMap is also able to resolve weakly typed services. When you only have a `Type` object that you need to  get resolved, you can use the non generic `GetInstance` method.

```csharp
Type foo = typeof (IFoo);

var fooInstance = ObjectFactory.GetInstance(foo);
```

Both `GetInstance` methods will provide you the default instance of the `IFoo`. That is, StructureMap could have multiple configurations for the `IFoo` plugin type, but can only have one default instance. Let's alter the configuration so that there will be two `IFoo` registrations.

```csharp
ObjectFactory.Initialize(i =>
{
    i.For<IFoo>().Use<Foo>();
	i.For<IFoo>().Use<SomeOtherFoo>();
    i.For<IBar>().Use<Bar>();
});
```
The last registration for `IFoo` will be the default instance. So whenever you want to resolve the service `IFoo` you will get an instance of `SomeOtherFoo`. But what if you want all foo's? That's easy.


```csharp
IList<IFoo> fooInstances = ObjectFactory.GetAllInstances<IFoo>();
```

When you request a service that's unknown to StructureMap you will be presented with a StructureMap exception, complaining that there is no default instance defined for the requested service type. If you don't want this exception to occur you can use the `TryGetInstance` method. 

```csharp
var blahInstance = ObjectFactory.TryGetInstance<IBlah>();

Debug.Assert(blahInstance != null, String.Format("no default instance for {0}", typeof(IBlah).FullName));
```

This will try to create or find the default instance of the requested service type. It will returns the default value (i.e. `null`) of the requested service if it's not known to the container. The assertion condition will be false because `blahInstance` will be `null`.

StructureMap's `ObjectFactory` class should be used with caution because a [Service Locator]() is largely considered an anti-pattern. Meaning the use of the `ObjectFactory` class through your code is not recommended. Typically, you must try to minimize the number of service locator usages in your system to a bare minimum. Most of the value of an IoC tool is in automatically doing Dependency Injection. This is possible by a feature called [auto wiring]() and can most effectivly be utilized in your application's [integration]() setup. This ensures  that needed services get resolved from a central, top level location in your application, making the need for manual resolution rare.

For supporting different use cases, StructureMap supports a wide variety of methods that you can use for resolving services. These will be adressed in the [resolving services]() chapter.

### An even simpler usage

There is an even a simpler usage than the main usage because StructureMap allows you to resolve instances without configuring the container. StructureMap is able to resolve **concrete types** without configuration as long as it has a default constructor or a constructor where the arguments are concrete types too.  It's important that the entire tree of constructor arguments are concrete types where eventually all leave types have a default constructor.

Let's say we have the following object model, which represents the weather condition for a certain location.

```csharp
public class Weather
{
    public Location Location { get; set; }
    public Atmosphere Atmosphere { get; set; }
    public Wind Wind { get; set; }
    public Condition Condition { get; set; }

    public Weather(Location location, Atmosphere atmosphere, Wind wind, Condition condition)
    {
        Location = location;
        Atmosphere = atmosphere;
        Wind = wind;
        Condition = condition;
    }
}

public class Location
{
   //some properties
}

public class Atmosphere
{
   //some properties
}

public class Wind
{
    //some properties        
}

public class Condition
{
    //some properties        
}
```

Before we can resolve the concrete `Weather` type, we need an instance of an `Container` object or `ObjectFactory`. As mentioned earlier, these objects defines a generic `GetInstance` method which can build us an instance of the `Weather` type.

You can create a container yourself or use the statically accessed container.

```csharp
var container = new Container();
var weather = container.GetInstance<Weather>();
```

```csharp
var weather = ObjectFactory.Container.GetInstance<Weather>();
    weather = ObjectFactory.GetInstance<Weather>(); //short version for above.
```

The reason why we don't need to supply any configuration is because StructureMap supports a concept called [auto wiring](). It's basically a smart way of building instances of types by looking to the constructors of the requested and all the needed underlaying types. During this inspection StructureMap also uses any provided configuration to help building the requested service or dependency.

In our example, where there isn't any configuration available, StructureMap looks at the constructor of the requested `Weather` type. It sees that it depends on four concrete types which all have a default constructor. StructureMap is therefor able to create an instance for all of them and inject them into the `Weather` constructor. After that the `Weather` instance is returned to the caller.

Most of the time you will be mapping abstractions to concrete types, but as you have seen StructureMap supports other use cases as well.

### Integrating StructureMap within your application

At some point you will want to integrate StructureMap into your application. Wether you are using Windows Presentation Foundation (WPF), FubuMVC, ASP.NET WebForms, ASP.NET MVC or any other framework or technology, you will have to do some sort of plumbing and bootstrapping. Depending on the used technology or framework there can be important integration points that you will have to use to fully enable the power of StructureMap.

While StructureMap doesn't provide integration support out of the box for all of the frameworks and technologies out there, we do find it important to help you get started with integrating StructureMap into your application. That said, StructureMap does provide integration support for FubuMVC (a web framework, which is part of the same family as StructureMap).

We have created some starter integration guides for the most common frameworks and technologies. Those guides aren't the one and true way, most of the time, integrating StructureMap can be done in various ways.

- [Integrations Guides]()

	- [FubuMVC]()
	- [ASP.NET WebForms]()
	- [ASP.NET MVC]()
	- [ASP.NET WebApi]()
	- [ASP.NET SignalR]()
	- [AutoMapper]()
	- [Nancy]()
	- [WCF]()
	- [WPF]()
	- [Console]()
	- [Windows Services]()

### What to do when things go wrong?

StructureMap, and any other IoC tool for that matter, is configuration intensive, which means that their will be problems in that configuration. We're all moving to more convention based type registration, so more stuff is happening off stage and out of your sight, making debugging the configuration even trickier. Not to worry (too much), StructureMap has some diagnostic abilities to help you solve configuration problems:

- [Interpreting Exceptions]()
- [Diagnostics]()
	- [What's inside]()
	- [Asserting the configuration]()
	- [The Model (ObjectFactory.Model)]()
	- [Build Plans]()

#### Need Help?

- There is a google group for StructureMap support at http://groups.google.com/group/structuremap-users?hl=en
- You can ask questions on [StackOverflow](http://stackoverflow.com/).
