---
layout: post
title: 'How To: Create a Simple Alert System Using Tag Helpers In Asp.Net Core Mvc'
date: 2017-12-18 09:00:45.000000000 -03:00
categories: howto
tags: .net aspnetcore
permalink: "/2017/12/18/how-to-create-a-simple-alert-system-using-tag-helpers-in-asp-net-core-mvc/"
image: 
  path: /assets/img/blog/How-To_-Create-a-Simple-Alert-System-Using-Tag-Helpers-In-Asp.Net-Core-Mvc.png
description: >
  Create a simple alert system to enhance user experience.
---
In this post I'm gonna show you how to create an alert system using the tag helpers feature from the asp.net core mvc.

During this tutorial, I'll be using a simple to do list project that I build, in order to experiment new stuff, [you can download or clone this git repository](https://github.com/mrodrigues1/SimpleToDo).

## **The Problem**

Feedback is important for the user, usually, is in form of messages on the screen. We could follow the client-side approach, by, creating an ajax request and showing a message after the response, or instead, follow the server-side approach, storing a message on a view bag, or even return a model with the message. 

The asp.net core mvc has a new feature called tag helper, enabling us to create an alert system on the server-side.

## **Setting Up The Alert System**

First, we need to configure the project to use session. With this in mind, we add the line `services.AddSession();`in the _ConfigureServices_ method from the _Startup_ class.

~~~csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<ToDoDbContext>(
        options =>
            options.UseSqlServer(Configuration.GetConnectionString("SimpleToDo")));

    services.AddTransient<IToDoListService, ToDoListService>();
    services.AddTransient<IToDoListRepository, ToDoListRepository>();
    services.AddTransient<ITaskService, TaskService>();
    services.AddTransient<ITaskRepository, TaskRepository>();
            
    services.AddSession();
    services.AddMvc();
}
~~~

This line enables your asp.net core mvc project to use the _TempData_ feature. The _TempData _uses a session variable internally to store data between requests. We are going to use _TempData_ to store our alerts.

Now we're going to create a class which will contain the message and the message type:

~~~csharp
public class Alert
{
    public string Message;
    public string Type;

    public Alert(string message, string type)
    {
        Message = message;
        Type = type;
    }
}
~~~

To be able to store data in the _TempData_, we're going to create a class that will contain extension methods, to add the alerts in the _TempData_.

~~~csharp
public static class AlertExtensions
{
    private const string AlertKey = "SimpleToDo.Alert";

    public static void AddAlertSuccess(this Controller controller, string message)
    {
        var alerts = GetAlerts(controller);

        alerts.Add(new Alert(message, "alert-success"));

        controller.TempData[AlertKey] = JsonConvert.SerializeObject(alerts);
    }

    public static void AddAlertInfo(this Controller controller, string message)
    {
        var alerts = GetAlerts(controller);

        alerts.Add(new Alert(message, "alert-info"));

        controller.TempData[AlertKey] = JsonConvert.SerializeObject(alerts);
    }

    public static void AddAlertWarning(this Controller controller, string message)
    {
        var alerts = GetAlerts(controller);

        alerts.Add(new Alert(message, "alert-warning"));

        controller.TempData[AlertKey] = JsonConvert.SerializeObject(alerts);
    }

    public static void AddAlertDanger(this Controller controller, string message)
    {
        var alerts = GetAlerts(controller);

        alerts.Add(new Alert(message, "alert-danger"));

        controller.TempData[AlertKey] = JsonConvert.SerializeObject(alerts);
    }

    private static ICollection<Alert> GetAlerts(Controller controller)
    {
        if (controller.TempData[AlertKey] == null)
            controller.TempData[AlertKey] = JsonConvert.SerializeObject(new HashSet<Alert>());

        ICollection<Alert> alerts = JsonConvert.DeserializeObject<ICollection<Alert>>(controller.TempData[AlertKey].ToString());

        if (alerts == null)
        {
            alerts = new HashSet<Alert>();
        }
        return alerts;
    }
}
~~~

With this class we can add many alert types in the _TempData_.

First, we get all alerts stored in the _TempData_ by a key, like a dictionary. They will be stored as a serialized _JSON_.

After that, we add the alert on the list. The type a bootstrap class that we're using to apply style on the client-side. Finally, we serialize the list and store it again in the _TempData_.

One side note, in the asp.net core mvc the _TempData_ has some trouble to store complex data like we're doing, so the solution was to serialize the _HashSet_, and deserialize it when we need it.

## **The Alert Tag Helper**

Tag Helpers enable the server-side to create and render HTML in razor files, enabling custom html tags creation. You can read more about tag helpers [here](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/intro).

To create our own Alert tag helper, we need to create a new class extending the _TagHelper_ class.

~~~csharp
public class AlertsTagHelper : TagHelper
{
    private const string AlertKey = "SimpleToDo.Alert";

    [ViewContext]
    public ViewContext ViewContext { get; set; }

    protected ITempDataDictionary TempData => ViewContext.TempData;

    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        output.TagName = "div";

        if (TempData[AlertKey] == null)
            TempData[AlertKey] = JsonConvert.SerializeObject(new HashSet<Alert>());

        var alerts = JsonConvert.DeserializeObject<ICollection<Alert>>(TempData[AlertKey].ToString());

        var html = string.Empty;

        foreach (var alert in alerts)
        {
            html += $"<div class='alert {alert.Type}' id='inner-alert' role='alert'>" +
                        $"<button type='button' class='close' data-dismiss='alert' aria-label='Close'>" +
                            $"<span aria-hidden='true'>&times;</span>" +
                        $"</button>" +
                        $"{alert.Message}" +
                    $"</div>";
        }

        output.Content.SetHtmlContent(html);
    }
}
~~~

The class' name is _AlertsTagHelper_, is important to use this kind of nomenclature in tag helper, like, "_TagName"_ + "_TagHelper"_. The _"TagName"_ will be our custom tag, as I'm going to show later.

We can use dependency injection in the tag helper, for this reason, we create public properties and these properties will be injected. In our case we have one property, the _ViewContext_, which is responsible to get the current view's context. The _TempData_ property is just a way to access the _ViewContext's TempData_ more easily.

To create the html output, we have to override the _Process_ method, which is the only thing that we have to do to set the output. The _Process_ method has two arguments, the context and output, we're going to use only  the output in this case.

In the _Process_ method we set the output's _TagName_ as "div", meaning that a div will embrace the html produce in the output. We get all the alerts stored in the _TempData_ and create a string which will contain all the html produced. We iterate through all alerts, concatenating each alert with the html alert template from bootstrap 3, you can check it [here](https://getbootstrap.com/docs/3.3/components/#alerts). The _Type_ is used to set the bootstrap alert class and the _Message_ will be showed on the screen. After that, we set the html output as our variable containing all the alerts.

The last thing that we need to do is to make the project recognize our tag helper. To do this, we have register the tag helper dependency in the _\_ViewImports.cshtml_ file:

![]({{ site.baseurl }}/assets/2017/12/ViewImport.png){:.lead}

\_ViewImport.cshtml
{:.figcaption}

By adding the `@addTagHelper *, SimpleToDo.Web`line in the file, we say to the asp.net core mvc to look for tag helpers on the web project. Now our custom html tag, alert, will be recognized as a html tag and also we'll have intellisense.

## **The Usage**

Use the tag helper is pretty simple, basically we need to put the tag `<alerts>` in a view. My choice was to put the tag on the layout page, by doing this, the alerts will be available for every page.

![]({{ site.baseurl }}/assets/2017/12/alerts-tag-usage.png){:.lead}

Alerts tag usage in the \_Layout.cshtml
{:.figcaption}

The only thing left is to add the alerts on the _TempData_, for this reason, we're going to use those extensions methods that we created before.

For example, we could add an alert when we successfully add a task in a list:

~~~csharp
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Create([Bind("TaskId,ListId,Name,Description,Done")] Task task)
{
    if (ModelState.IsValid)
    {
        await _taskService.CreateTask(task);

        this.AddAlertSuccess($"{task.Name} created successfully.");

        return RedirectToAction(nameof(Index), new { listId = task.ListId });
    }

    return View(new TaskCreateEditViewModel
    {
        ListId = task.ListId,
        Name = task.Name,
        Description = task.Description
    });
}
~~~

The line `this.AddAlertSuccess($"{task.Name} created successfully.");`do the trick. Now when we run the project and add a task, as a result, the alert will show up on the screen:

![]({{ site.baseurl }}/assets/2017/12/Screenshot_1.png){:.lead}

As you can see, the alert is showing up on the screen after the postback and the redirect to the list page. Every time that a postback/redirect/refresh happens on the page, the alerts tag helper will be called and check if there's alerts to show.

That's all what I have for this tutorial, there's some details in the project's implementation which I omit, because I wanted to show only the tag helper details, you can check all the project's code [here](https://github.com/mrodrigues1/SimpleToDo).

This was an example of the tag helper usage. There's some much more room to explore in this new asp.net core mvc functionality.

