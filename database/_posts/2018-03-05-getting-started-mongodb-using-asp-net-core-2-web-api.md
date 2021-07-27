---
layout: post
title: Getting Started With MongoDB Using ASP.NET Core Web API
date: 2018-03-05 10:00:47.000000000 -03:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories: database
tags: .net aspnetcore dependencyinjection mongodb nosql
permalink: "/2018/03/05/getting-started-mongodb-using-asp-net-core-2-web-api/"
image: 
  path: /assets/img/blog/Getting-Started-With-MongoDB-Using-ASP.NET-Core-Web-API.png
description: >
  Getting started with document databases in Asp.Net Core.
---
> [You can download the code here](https://github.com/mrodrigues1/MongoDB-WebApi)

Recently, I'm trying to get into NoSQL databases, more specific into document databases.

After some research, MongoDB was my choice to start with. MongoDB is a well know and popular document database, which store JSON-like documents. It's quite accessible, having drivers supporting major languages and frameworks. Have a great community support with a large number of  unofficial drivers. 

The goal is to show how to set up a MongoDB instance and how to consume it with a web API in asp.net core.

## **Softwares needed:**

- [MongoDB](https://www.mongodb.com/download-center#community)
- [Visual Studio 2017 Community Edition](https://www.visualstudio.com/downloads/)
- [Postman](https://www.getpostman.com/)

\*I used windows 10 to create this tutorial.

## **Configure MongoDB**

The first thing to do is download MongoDB and install it.

After installing, we need to configure MongoDB and set up where data will be located. So, to start, open the command prompt and use the command (please update these local paths with your own):

~~~
"C:\Program Files\MongoDB\Server\3.6\bin\mongod.exe" --dbpath "E:\Programming\Tools\Mongodb"
~~~

With this command we're starting the MongoDB Database Server and configuring the database location. Now, MongoDB is connect on the default port 27017.

Open another command prompt instance and type the following command (remember to use your own paths):

~~~
"C:\Program Files\MongoDB\Server\3.6\bin\mongo.exe"
~~~

This will connect to the default database. Don't close the other instance of the command prompt.

So, now we can configure our document database. For this tutorial I choose to store information from games. To create the database use the following command: 

~~~
use GamesDB
~~~

This will create the database if not exists and switch the context for this database. We can create a collection now, in which we will store the data. To create the collection run the following command:

~~~
db.createCollection('Games')
~~~

With the collection created, the only thing missing is to add some data to it. Given that MongoDB stores JSON-like objects, we can add data in the Games collections like this:

~~~
db.Games.insert({'Name':'Super Mario Odyssey','Developer':'Nintendo','Publisher':'Nintendo','Platforms':['Nintendo Switch']})
~~~

To check if the data was added correctly, run the following to return all data in the collection:

~~~
db.Games.find({})
~~~

If the data was added successfully, you should see the printed at the command prompt:

![]({{ site.baseurl }}/assets/2018/03/mongogamesdbfind.png){:.lead}

Games Collection Data
{:.figcaption}

As can be seen, the object has the property `"_id"`, that is a _ObjectId_ which is automatically generated and is unique.

Now that the database is configured, we can create the web API.

## **Creating a CRUD with Web API**

To use GameDB we're going to create a web API, in which we will implement a basic CRUD to work with the Game collection.

For this, we have to open Visual Studio and create a new Web API project, following these steps:

~~~
On Visual Studio >> New Project >> ASP.NET Core Web Application >> Web API with .Net Core and ASP.NET Core 2.0
~~~

Now, we have our project, I called it MongoDBGames. With that done, we need to be able to access our database within the application.

## Setup MongoDB settings

To access MongoDB we need two information, connection string and database name. Open `appsetting.json` file and add MongoDB information:

~~~json
{
  "MongoDB": {
    "ConnectionString": "mongodb://localhost:27017",
    "Database": "GamesDB"
  },
  "Logging": {
    "IncludeScopes": false,
    "Debug": {
      "LogLevel": {
        "Default": "Warning"
      }
    },
    "Console": {
      "LogLevel": {
        "Default": "Warning"
      }
    }
  }  
}
~~~

As the file name say, it stores json objects, that has information about application's settings. The `"MongoDB"` was added, containing the `"ConnectionString"` that will be used to connect to the MongoDB's default connection and `"Database"` which is the name of the database we created earlier.

To access this setting, we're going to use the built-in dependency injection from ASP.NET core. So, we have to create a class to store the information:

~~~csharp
public class Settings
{
    public string ConnectionString { get; set; }
    public string Database { get; set; }
}
~~~

Now, to configure the dependency injection, we have to add some code in the `ConfigureServices` method from the `Startup.cs` :

~~~csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc();

    services.Configure<Settings>(
        options =>
        {
            options.ConnectionString = Configuration.GetSection("MongoDb:ConnectionString").Value;
            options.Database = Configuration.GetSection("MongoDb:Database").Value;
        });
}
~~~

This is saying that when we pass the `Settings` class trough a constructor, the connection string and database name will have the values that we define on the `appsetting.json` file. I'll show you how to do this now, when creating the context class next.

## Context Class

The context class will expose our MongoDB collections, in our case, the Games collection. But, before creating out context class, we have to add the MongoDB C# driver to the project, because this is what will allow us to connect  to the MongoDB instance.

Follow the steps to install the driver:

~~~
On Visual Studio >> Tools >> NuGet Package Manager >> Package Manager Console >> Install-Package MongoDB.Driver
~~~

With the driver in place, we can go ahead with the context's creation: 

~~~csharp
public class GameContext : IGameContext
{
    private readonly IMongoDatabase _db;

    public GameContext(IOptions<Settings> options)
    {
        var client = new MongoClient(options.Value.ConnectionString);
        _db = client.GetDatabase(options.Value.Database);
    }

    public IMongoCollection<Game> Games => _db.GetCollection<Game>("Games");
}
~~~

~~~csharp
public interface IGameContext
{
    IMongoCollection<Game> Games { get; }
}
~~~

In the `GameContext` constructor we're injecting the `IOptions<Settings>` object, which has the information that we set on the  `Startup.cs`. Then, a `MongoClient`is created using the connection string. After that, we use the client to get the database instance and store it in a private field. The property `Games` returns the our mongo collection using the `GetCollection` method from the `IMongoDatabase`, this will enable us to query/insert/update/remove in the database. 

To instantiate a `IMongoCollection<>` we have to pass a TDocument, which represents a typed mongo collection. In our case, we have to pass a class that matches the collection we created earlier on the command line. It will be something like this:

~~~csharp
public class Game
{
    [BsonId]
    public ObjectId Id { get; set; }

    public string Name { get; set; }

    public string Developer { get; set; }

    public string Publisher { get; set; }

    public List<string> Platforms { get; set; }
}
~~~

The unusual thing in this class, is that the Id property is an `ObjectId`, MongoDB create this automatically when we add that register earlier. Another thing is the attribute `[BsonId]`, which indicates the property as an id. The other properties in the class just matches the object store in the database, we can use different names in our variable, for this, we need to use the attribute `[BsonElement("ElementName")]`.

To finish this section, we have to register the `GameContext` dependency in the `Startup.cs`:

~~~csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...
    // Other Stuff Registered
    // ...

    services.AddTransient<IGameContext, GameContext>();
}
~~~

## Game Repository Class

Now we are going to create a repository to make CRUD operations in the games collection.

~~~csharp
public class GameRepository : IGameRepository
{
    private readonly IGameContext _context;

    public GameRepository(IGameContext context)
    {
        _context = context;
    }

    public async Task<IEnumerable<Game>> GetAllGames()
    {
        return await _context
                        .Games
                        .Find(_ => true)
                        .ToListAsync();
    }

    public Task<Game> GetGame(string name)
    {
        FilterDefinition<Game> filter = Builders<Game>.Filter.Eq(m => m.Name, name);

        return _context
                .Games
                .Find(filter)
                .FirstOrDefaultAsync();
    }
       
    public async Task Create(Game game)
    {
        await _context.Games.InsertOneAsync(game);
    }

    public async Task<bool> Update(Game game)
    {
        ReplaceOneResult updateResult =
            await _context
                    .Games
                    .ReplaceOneAsync(
                        filter: g => g.Id == game.Id,
                        replacement: game);

        return updateResult.IsAcknowledged
                && updateResult.ModifiedCount > 0;
    }

    public async Task<bool> Delete(string name)
    {
        FilterDefinition<Game> filter = Builders<Game>.Filter.Eq(m => m.Name, name);

        DeleteResult deleteResult = await _context
                                            .Games
                                            .DeleteOneAsync(filter);

        return deleteResult.IsAcknowledged
            && deleteResult.DeletedCount > 0;
    }
}
~~~

The `GameRepository` class has all CRUD operations we need. I'm not going into much details, but I'm going to point out relevant things in this class.

As can be seen, all methods are async, since I want to implement an async API, I choose this route. The MongoDB's C# driver has a good support for async operations, all major actions has their async counterpart.

Another importante thing is the `Builders<Game>` which is being used to created filter for our queries. In most of the actions we need to pass a `FilterDefinition` obejct as parameter, the `Builders<T>`allow us to create the filter and them use the filter to search a game by id or name. There are some actions that we don't want a filter because we have to bring all elements from the database, like in the `GetAllGames()` method. There, we are using the `.Find(_ => true)`, this expression tells the collection to get all data it has.

The last point I want to talk about is the _IsAcknowledged_ and _ModifiedCount/DeletedCount_ properties, this is how MongoDB keep track of changes. When doing operations such as, `ReplaceOneAsync(...)` and `DeleteOneAsync(...)`, an object is returned, with this object we can know the database is acknowledge and the amount of elements modified or deleted. We can use this information to identify the success or fail of our operation.

The last thing is, of course, expose the `GameRepository` interface and register the dependency:

~~~csharp
public interface IGameRepository
{
    Task<IEnumerable<Game>> GetAllGames();
    Task<Game> GetGame(string name);
    Task Create(Game game);
    Task<bool> Update(Game game);
    Task<bool> Delete(string name);
}
~~~

~~~csharp
public void ConfigureServices(IServiceCollection services)
{
    // ... 
    // Other Stuff Registered 
    // ... 
    services.AddTransient<IGameRepository, GameRepository>();
}
~~~

## **Creating a REST API**

The last piece of this puzzle, is the web API. My choice was to create a rest API, because with this API type, we can create CRUD operations pretty easily. So, to being with we have to create a new controller in our project, following these steps:

~~~
In The Project Root >> Rigth Click In The Controllers Folder>> Add >> Controller... >> API Controller with read/write actions >> Choose a Name >> Add
~~~

The API implementation it's straight forward:

~~~csharp
[Produces("application/json")]
[Route("api/Game")]
public class GameController : Controller
{
    private readonly IGameRepository _gameRepository;

    public GameController(IGameRepository gameRepository)
    {
        _gameRepository = gameRepository;
    }

    // GET: api/Game
    [HttpGet]
    public async Task<IActionResult> Get()
    {
        return new ObjectResult(await _gameRepository.GetAllGames());
    }

    // GET: api/Game/name
    [HttpGet("{name}", Name = "Get")]
    public async Task<IActionResult> Get(string name)
    {
        var game = await _gameRepository.GetGame(name);

        if (game == null)
            return new NotFoundResult();

        return new ObjectResult(game);
    }

    // POST: api/Game
    [HttpPost]
    public async Task<IActionResult> Post([FromBody]Game game)
    {
        await _gameRepository.Create(game);
        return new OkObjectResult(game);
    }

    // PUT: api/Game/5
    [HttpPut("{name}")]
    public async Task<IActionResult> Put(string name, [FromBody]Game game)
    {
        var gameFromDb = await _gameRepository.GetGame(name);

        if (gameFromDb == null)
            return new NotFoundResult();

        game.Id = gameFromDb.Id;

        await _gameRepository.Update(game);

        return new OkObjectResult(game);
    }

    // DELETE: api/ApiWithActions/5
    [HttpDelete("{name}")]
    public async Task<IActionResult> Delete(string name)
    {
        var gameFromDb = await _gameRepository.GetGame(name);

        if (gameFromDb == null)
            return new NotFoundResult();

        await _gameRepository.Delete(name);

        return new OkResult();
    }
}
~~~

We're using all methods created on the repository to query and execute actions in the database. Since it's a RestAPI, we have all actions needed, like, _Get_, _Post_, _Put_ and _Delete_. The file has comments, such as, `// GET: api/Game`, this is to show how can we make a request to a certain method on the API.

One thing to point out, there are some actions receiving the id as a string instead of a ObjectId, because is more practical to send just the string that ObjectId generates and create the ObjectId in the back-end code. If we take a look on the _Put_ and _Delete_ methods, we see this behavior happening.

## **Testing API with Postman**

To test our API, I'll be using Postman. [Postman](https://www.getpostman.com/) is a HTTP client used for testing APIs.

In this section I'm gonna show you how to use our API to create, edit, read, delete and from game collection.

## Create a Game

To create a game, we need to make a post request and send a Json object in the request's body.

The first thing is to select the request type as _POST_, and use the following url (don't forget to change to your own. in the localhost section):

~~~
http://localhost:30653/api/Game/
~~~

On the _Headers_ section, let's add the content type as json, as the follow: 

- Key: Content-Type
- Value: application/json

On the _Body_ section, select the format as _raw_ and them _JSON(application/json)_. Now have to create a json object in the body that will be sent on request: 

~~~json
{
    "name": "Rocket League",
    "developer": "Psyonix",
    "publisher": "Psyonix",
    "platforms": [
        "Playstion 4",
        "Xbox One",
        "Nintendo Switch",
        "PC"
    ]
}
~~~

When creating a new object, the _ObjectId_ is not needed in the request, since MongoDB will create it automatically. Now we can hit send button and look at the response:

![]({{ site.baseurl }}/assets/2018/03/mongodbcreatenewgameresponse.png){:.lead}

Create Game Response
{:.figcaption}

If everything works correctly, you will see a response like this. The API returns an OK (200) response and send the game created with the new _ObjectId_.

## Edit a Game

What if we want to update our recently added game?

We have to create a PUT request. In a PUT request we have to send a value on the query string, such as an id, which indicate what object we want to update, and send an object in the body, that will be the updated object.

Create a new request with PUT type selected and use the following url:

~~~
http://localhost:30653/api/Game/Rocket League
~~~

The Header configuration will be the same as in the create.

In the body we have to send the updated json object:

~~~json
{
    "name": "Rocket League",
    "developer": "Psyonix",
    "publisher": "Psyonix",
    "platforms": [
        "PC",
        "Playstion 4",
        "Playstion 4 Pro",
        "Xbox One",
        "Xbox One S",
        "Xbox One X",
        "Nintendo Switch"
    ]
}
~~~

Hit the send button. When everything works fine, you will see a response OK (200) and the game updated:

![]({{ site.baseurl }}/assets/2018/03/mongodbupdategameresponse.png){:.lead}

Update Game Response
{:.figcaption}

## Get All Games

Create a new request with the type GET. Use the following url: 

~~~
http://localhost:30653/api/Game/
~~~

And that's it, very simple request. When send button is clicked, we should see the response with all games in the collection:

![]({{ site.baseurl }}/assets/2018/03/mongodbgetallresponse.png){:.lead}

Get All Games Response
{:.figcaption}

## Delete a Game

To create a Delete request, select _Delete_ as request type and use the following url: 

~~~
http://localhost:30653/api/Game/Rocket League
~~~

In the _Delete_ request we just have to send value on the query string, in our case, is the game name.

If the request is succeeded, the response will be OK (200) and just this.

## **Wrap up**

This is the end of the tutorial, I hope you enjoyed. The tutorial point was present MongoDB and how we can start to use it. My focus was on the basics, like, how to set up the database, how connect to the database and basic operations. 

In the future I'm going to create more tutorials for MongoDB, to show more advanced uses and how to connect to the database more easily and etc.

> [Download the code here](https://github.com/mrodrigues1/MongoDB-WebApi)

## **References and Further Reading**

- [MongoDB Docs](https://docs.mongodb.com/)
- [Using MongoDB .NET Driver with .NET Core WebAPI - Quality App Design](http://www.qappdesign.com/using-mongodb-with-net-core-webapi/)
- [Using MongoDB with Web API and ASP.NET Core - .Net Curry](http://www.dotnetcurry.com/aspnet-mvc/1267/using-mongodb-nosql-database-with-aspnet-webapi-core)
