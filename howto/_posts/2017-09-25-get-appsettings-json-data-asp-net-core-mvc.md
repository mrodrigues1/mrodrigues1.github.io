---
layout: post
title: 'How To: Get appsettings.json Data In Asp.Net Core Mvc'
date: 2017-09-25 20:17:48.000000000 -03:00
categories: howto
tags: .net aspnetcore dependencyinjection
permalink: "/2017/09/25/get-appsettings-json-data-asp-net-core-mvc/"
image: 
  path: /assets/img/blog/How-To-Get-appsettings.json-Data-In-Asp.Net-Core-Mvc.png
description: >
  A clean way to information from appsetting.json file.
---
> [You can download this sample from this git repository.](https://github.com/mrodrigues1/AppSettingsDI)

In this tutorial I'll show you how to get the data from `appsettings.json` using the built-in dependency injection in Asp.Net Core MVC.

Let’s start right away.

First, you need to create a project in visual studio. I’m using visual studio 2017 community with .net core version 1.1.

![]({{ site.baseurl }}/assets/2017/09/1-4Rx8EdKFP6Q3Ut5SK1itNg.png){:.lead}

New ASP.NET Core Web App Project
{:.figcaption}

I’m creating an ASP.NET Core Web Application, that’s the new version of the mvc framework.

![]({{ site.baseurl }}/assets/2017/09/1-al1lJTfD17VM3DLeFzAmNQ.png){:.lead}

Web Application Template
{:.figcaption}

For the template, I’m choosing the Web Application template without authentication, we don’t need fancy stuff for this tutorial.

![]({{ site.baseurl }}/assets/2017/09/1-dqV44D1cH2kQNHAGMyVldA.png){:.lead}

Solution Explorer
{:.figcaption}

This is the basic structure of our solution, as we can see we have the appsettings.json file under the root of the solution. We don’t have the Web.config file anymore in the ASP.NET Core MVC, instead we have this file, that has a similar function. In the `appsettings.json` file, we can put all configurations needed for our project.

~~~json
{ 
    "Logging": { 
        "IncludeScopes": false, 
        "LogLevel": { 
            "Default": "Warning" 
        } 
    } 
}
~~~

This is the default appsettings.json, with the default logging configuration of the .net core MVC framework.

For this tutorial I’ll add another configuration to the file:

~~~json
{
  "Logging": {
    "IncludeScopes": false,
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "HelloWorldObject" : {
    "Text" :  "Hello World! \o/" 
  } 
}
~~~

I added the `“HelloWorldObject”` configuration to the file, and our goal here is to take the `Text` property and show it on the screen.

To be able to do that, we need to create a model that will store this data.

![]({{ site.baseurl }}/assets/2017/09/1-QopYl4UEZygJzL_D-3dPYA.png){:.lead}

Project Structure
{:.figcaption}

~~~csharp
public class HelloWorldObject
{
    public string Text { get; set; }
}
~~~

Done that, we have to configure the dependencies of this object, to able to inject it in our controller. The built-in dependency injection is all configured in the `Startup` class.

~~~csharp
public class Startup
{
    public Startup(IHostingEnvironment env)
    {
        var builder = new ConfigurationBuilder()
            .SetBasePath(env.ContentRootPath)
            .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
            .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
            .AddEnvironmentVariables();
        Configuration = builder.Build();
    }
    
    public IConfigurationRoot Configuration { get; }
    
    //Other Stuff...   
}
~~~

In the `Startup` class constructor, we have a set of configurations, one of them is the `appsettings.json` file. Using `AddJsonFile()` method, you are saying that a json file is a configuration file for the system, also you can have multiple configuration files.

~~~csharp
public class Startup
{
    //Other Stuff...

    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services)
    {
        // Add framework services.
        services.AddMvc();
        
        // Adds services required for using options.
        services.AddOptions();

        // Register the ConfigurationBuilder instance which HelloWordObject binds against.
        services.Configure<HelloWorldObject>(Configuration.GetSection("HelloWorldObject"));
    }
    
    //Other Stuff...    
}
~~~

Here, we’re adding two important configurations to the service container, `AddOptions()`, which allow us to use options, and with the extension method `Configure<T>`, we add an `IConfigureOptions<T>` service to the service container, in this case I’m saying that our `HelloWordObject` will the data from the `“HelloWordObject”` section in the `appsettings.json` file.

Now we can finally inject the `IOption<T>` object that we register in our controller.

~~~csharp
public class HomeController : Controller
{
    private readonly HelloWorldObject _helloWorldObject;

    public HomeController(IOptions<HelloWorldObject> helloWorldObject)
    {
        _helloWorldObject = helloWorldObject.Value;
    }

    public IActionResult Index()
    {
        return View(_helloWorldObject);
    }
}
~~~

In the `HomeController` constructor, we put our `IOptions<HelloWordObject>` and the dependency injection framework will resolve it for us.

I made some changes to the `index.cshtml` page to just show the `Text` property from the `HelloWordObject`.

~~~csharp
@model AppSettingsDI.Model.HelloWorldObject

@{
    ViewData["Title"] = "Home Page";
}

<div class="row">
    <div class="col-md-12 text-center">
        <h1>@Model.Text</h1>
    </div>
</div>
~~~

We can finally run the application and see the message in our browser.

![]({{ site.baseurl }}/assets/2017/09/1-Emtm4lOlkJlCWMuq_COZvg-1.png){:.lead}

Hello World
{:.figcaption}

As you can see, that’s our message on the screen.

Just one more thing before I finish, this is just a simple example of how you can get data from the appsettings.json, you can have complex objects too. You [download the code](https://github.com/mrodrigues1/AppSettingsDI) and try to expand this sample for yourself!

