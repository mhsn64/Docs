---
title: "Dependency Injection in ASP.NET Web API 2 | Microsoft Docs"
author: MikeWasson
description: "This tutorial shows how to inject dependencies into your ASP.NET Web API controller. Software versions used in the tutorial Web API 2 Unity Application Block..."
ms.author: riande
manager: wpickett
ms.date: 01/20/2014
ms.topic: article
ms.assetid: 
ms.technology: dotnet-webapi
ms.prod: .net-framework
msc.legacyurl: /web-api/overview/advanced/dependency-injection
---
Dependency Injection in ASP.NET Web API 2
====================
by [Mike Wasson](https://github.com/MikeWasson)

[Download Completed Project](http://code.msdn.microsoft.com/ASP-NET-Web-API-Tutorial-468ee148)

> This tutorial shows how to inject dependencies into your ASP.NET Web API controller.
> 
> ## Software versions used in the tutorial
> 
> 
> - Web API 2
> - [Unity Application Block](https://www.nuget.org/packages/Unity/)
> - Entity Framework 6 (version 5 also works)


## What is Dependency Injection?

A *dependency* is any object that another object requires. For example, it's common to define a [repository](http://martinfowler.com/eaaCatalog/repository.html) that handles data access. Let's illustrate with an example. First, we'll define a domain model:

    public class Product
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public decimal Price { get; set; }
    }

Here is a simple repository class that stores items in a database, using Entity Framework.

    public class ProductsContext : DbContext
    {
        public ProductsContext()
            : base("name=ProductsContext")
        {
        }
        public DbSet<Product> Products { get; set; }
    }
    
    public class ProductRepository : IDisposable
    {
        private ProductsContext db = new ProductsContext();
    
        public IEnumerable<Product> GetAll()
        {
            return db.Products;
        }
        public Product GetByID(int id)
        {
            return db.Products.FirstOrDefault(p => p.Id == id);
        }
        public void Add(Product product)
        {
            db.Products.Add(product);
            db.SaveChanges();
        }
    
        protected void Dispose(bool disposing)
        {
            if (disposing)
            {
                if (db != null)
                {
                    db.Dispose();
                    db = null;
                }
            }
        }
    
        public void Dispose()
        {
            Dispose(true);
            GC.SuppressFinalize(this);
        }
    }

Now let's define a Web API controller that supports GET requests for `Product` entities. (I'm leaving out POST and other methods for simplicity.) Here is a first attempt:

    public class ProductsController : ApiController
    {
        // This line of code is a problem!
        ProductRepository _repository = new ProductRepository();
    
        public IEnumerable<Product> Get()
        {
            return _repository.GetAll();
        }
    
        public IHttpActionResult Get(int id)
        {
            var product = _repository.GetByID(id);
            if (product == null)
            {
                return NotFound();
            }
            return Ok(product);
        }
    }

Notice that the controller class depends on `ProductRepository`, and we are letting the controller create the `ProductRepository` instance. However, it's a bad idea to hard code the dependency in this way, for several reasons.

- If you want to replace `ProductRepository` with a different implementation, you also need to modify the controller class.
- If the `ProductRepository` has dependencies, you must configure these inside the controller. For a large project with multiple controllers, your configuration code becomes scattered across your project.
- It is hard to unit test, because the controller is hard-coded to query the database. For a unit test, you should use a mock or stub repository, which is not possible with the currect design.

We can address these problems by *injecting* the repository into the controller. First, refactor the `ProductRepository` class into an interface:

    public interface IProductRepository
    {
        IEnumerable<Product> GetAll();
        Product GetById(int id);
        void Add(Product product);
    }
    
    public class ProductRepository : IProductRepository
    {
        // Implementation not shown.
    }

Then provide the `IProductRepository` as a constructor parameter:

    public class ProductsController : ApiController
    {
        private IProductRepository _repository;
    
        public ProductsController(IProductRepository repository)  
        {
            _repository = repository;
        }
    
        // Other controller methods not shown.
    }

This example uses [constructor injection](http://www.martinfowler.com/articles/injection.html#FormsOfDependencyInjection). You can also use *setter injection*, where you set the dependency through a setter method or property.

But now there is a problem, because your application doesn't create the controller directly. Web API creates the controller when it routes the request, and Web API doesn't know anything about `IProductRepository`. This is where the Web API dependency resolver comes in.

## The Web API Dependency Resolver

Web API defines the **IDependencyResolver** interface for resolving dependencies. Here is the definition of the interface:

    public interface IDependencyResolver : IDependencyScope, IDisposable
    {
        IDependencyScope BeginScope();
    }
    
    public interface IDependencyScope : IDisposable
    {
        object GetService(Type serviceType);
        IEnumerable<object> GetServices(Type serviceType);
    }

The **IDependencyScope** interface has two methods:

- **GetService** creates one instance of a type.
- **GetServices** creates a collection of objects of a specified type.

The **IDependencyResolver** method inherits **IDependencyScope** and adds the **BeginScope** method. I'll talk about scopes later in this tutorial.

When Web API creates a controller instance, it first calls **IDependencyResolver.GetService**, passing in the controller type. You can use this extensibility hook to create the controller, resolving any dependencies. If **GetService** returns null, Web API looks for a parameterless constructor on the controller class.

## Dependency Resolution with the Unity Container

Although you could write a complete **IDependencyResolver** implementation from scratch, the interface is really designed to act as bridge between Web API and existing IoC containers.

An IoC container is a software component that is responsible for managing dependencies. You register types with the container, and then use the container to create objects. The container automatically figures out the dependency relations. Many IoC containers also allow you to control things like object lifetime and scope.

> [!NOTE] "IoC" stands for "inversion of control", which is a general pattern where a framework calls into application code. An IoC container constructs your objects for you, which "inverts" the usual flow of control.


For this tutorial, we'll use [Unity](https://msdn.microsoft.com/en-us/library/ff647202.aspx) from Microsoft Patterns &amp; Practices. (Other popular libraries include [Castle Windsor](http://www.castleproject.org/), [Spring.Net](http://www.springframework.net/), [Autofac](https://code.google.com/p/autofac/), [Ninject](http://www.ninject.org/), and [StructureMap](http://docs.structuremap.net/).) You can use NuGet Package Manager to install Unity. From the **Tools** menu in Visual Studio, select **Library Package Manager**, then select **Package Manager Console**. In the Package Manager Console window, type the following command:

    Install-Package Unity

Here is an implementation of **IDependencyResolver** that wraps a Unity container.

    using Microsoft.Practices.Unity;
    using System;
    using System.Collections.Generic;
    using System.Web.Http.Dependencies;
    
    public class UnityResolver : IDependencyResolver
    {
        protected IUnityContainer container;
    
        public UnityResolver(IUnityContainer container)
        {
            if (container == null)
            {
                throw new ArgumentNullException("container");
            }
            this.container = container;
        }
    
        public object GetService(Type serviceType)
        {
            try
            {
                return container.Resolve(serviceType);
            }
            catch (ResolutionFailedException)
            {
                return null;
            }
        }
    
        public IEnumerable<object> GetServices(Type serviceType)
        {
            try
            {
                return container.ResolveAll(serviceType);
            }
            catch (ResolutionFailedException)
            {
                return new List<object>();
            }
        }
    
        public IDependencyScope BeginScope()
        {
            var child = container.CreateChildContainer();
            return new UnityResolver(child);
        }
    
        public void Dispose()
        {
            container.Dispose();
        }
    }

> [!NOTE] If the **GetService** method cannot resolve a type, it should return **null**. If the **GetServices** method cannot resolve a type, it should return an empty collection object. Don't throw exceptions for unknown types.


## Configuring the Dependency Resolver

Set the dependency resolver on the **DependencyResolver** property of the global **HttpConfiguration** object.

The following code registers the `IProductRepository` interface with Unity and then creates a `UnityResolver`.

    public static void Register(HttpConfiguration config)
    {
        var container = new UnityContainer();
        container.RegisterType<IProductRepository, ProductRepository>(new HierarchicalLifetimeManager());
        config.DependencyResolver = new UnityResolver(container);
    
        // Other Web API configuration not shown.
    }

## Dependenecy Scope and Controller Lifetime

Controllers are created per request. To manage object lifetimes, **IDependencyResolver** uses the concept of a *scope*.

The dependency resolver attached to the **HttpConfiguration** object has global scope. When Web API creates a controller, it calls **BeginScope**. This method returns an **IDependencyScope** that represents a child scope.

Web API then calls **GetService** on the child scope to create the controller. When request is complete, Web API calls **Dispose** on the child scope. Use the **Dispose** method to dispose of the controller's dependencies.

How you implement **BeginScope** depends on the IoC container. For Unity, scope corresponds to a child container:

    public IDependencyScope BeginScope()
    {
        var child = container.CreateChildContainer();
        return new UnityResolver(child);
    }

Most IoC containers have similar equivalents.