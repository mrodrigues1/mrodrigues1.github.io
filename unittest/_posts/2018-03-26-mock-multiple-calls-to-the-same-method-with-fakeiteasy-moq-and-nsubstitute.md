---
layout: post
title: Mock Multiple Calls To The Same Method With FakeItEasy, Moq and NSubstitute
date: 2018-03-26 09:00:32.000000000 -03:00
type: post
categories: unittest
tags: .net mock fakeiteasy moq nsubstitute
permalink: "/2018/03/26/mock-multiple-calls-to-the-same-method-with-fakeiteasy-moq-and-nsubstitute/"
image: 
  path: /assets/img/blog/Mock-Multiple-Calls-To-The-Same-Method-With-FakeItEasy-Moq-and-NSubstitute.png
description: >
  Learn how to mock multiple calls to the same method.
---
You can download this article project on [GitHub](https://github.com/mrodrigues1/MockMultipleCalls).

Sometimes when doing unit test, we encounter subsequent calls to the same method. When mocking a method the default behavior is to always return the same result.

In this post, I will show how you can mock multiple calls to the same method with the mocking frameworks&nbsp;[FakeItEasy](https://fakeiteasy.github.io/), [Moq](https://github.com/moq/moq4) and [NSubstitute](http://nsubstitute.github.io/).

## **Code Example**

~~~csharp
public class GameService : IGameService
{
    private readonly IGameRepository _gameRepository;

    public GameService(IGameRepository gameRepository)
    {
        _gameRepository = gameRepository;
    }

    public string GetGameNames(int[] ids)
    {
        var games = ids.Select(id => _gameRepository.FindGameById(id));
        return string.Join(", ", games.Select(g => g.Name));
    }
}
~~~

The&nbsp;_GameService_ class has a method called _GetGameNames_. Inside it, we call our repository method `.FindGameById` for each id on the array.

## **FakeItEasy**

FakeItEasy has two solutions. The first is the `ReturnsNextFromSequence(...)`&nbsp;method:

~~~csharp
[Fact]
public void GetGameNames_ShouldReturnAllGamesNamesSeparatedByComma_FakeItEasy_1()
{
    //Arrange
    var games = new Game[] {
        new Game { Id = 1, Name = "Nier: Automata" },
        new Game { Id = 2, Name = "Rocket League" }
    };

    var gameRepositoryMock = A.Fake<IGameRepository>();
    A.CallTo(() => gameRepositoryMock.FindGameById(A<int>.Ignored))
        .ReturnsNextFromSequence(games);

    var sut = new GameService(gameRepositoryMock);

    //Act
    var result = sut.GetGameNames(games.Select(g => g.Id).ToArray());

    //Assert
    Assert.Equal("Nier: Automata, Rocket League", result);
}
~~~

This method accepts a&nbsp;`T[]`&nbsp;as a parameter, so we can send any type of array needed for the return. We setup the method&nbsp;`FindGameById(...)`&nbsp;to return the array's next object from the sequence each time that the method is called. So will first return&nbsp;`Nier: Automata`&nbsp;and then&nbsp;`Rocket League`.

The other solution is to use the `ReturnsLazily<TReturnType, T1>(Func)`:

~~~csharp
[Fact]
public void GetGameNames_ShouldReturnAllGamesNamesSeparatedByComma_FakeItEasy_2()
{
    //Arrange 
    var games = new Dictionary<int, Game>
    {
        { 1, new Game { Id = 1, Name = "Nier: Automata" } },
        { 2, new Game { Id = 2, Name = "Rocket League" } }
    };

    var gameRepositoryMock = A.Fake<IGameRepository>();
    A.CallTo(() => gameRepositoryMock.FindGameById(A<int>.Ignored))
        .ReturnsLazily<Game, int>(id => games[id]);

    var sut = new GameService(gameRepositoryMock);

    //Act
    var result = sut.GetGameNames(games.Keys.ToArray());

    //Assert
    Assert.Equal("Nier: Automata, Rocket League", result);
}
~~~

`ReturnsLazily<TReturnType, T1>(Func)` specific a function used to generate a return value when the configured call is made. Each time a call happens, it can return a different value.

In our case, we use two type parameters. `TReturnType` stands for the type's return value and&nbsp;`T1` stands for the type of the first argument of the faked method call.

We setup&nbsp;`FindGameById` from our fake&nbsp;`IGameRepository` to return an object. _GameService_ class call this method for each element on the array&nbsp; `games.Keys.ToArray()` sent to the `GetGameNames` method.&nbsp;So&nbsp;`GetGameNames`&nbsp;calls&nbsp;`FindGameById`&nbsp;twice with the id 1 and 2.

A call to&nbsp;`FindGameById` executes&nbsp;`.ReturnsLazily<Game, int>(id => games[id])`. This means that we want to return a _Game_ and the argument in the repository is an _int_. Here `Dictionary<int, Game>` becomes useful because of the expression in&nbsp;`.ReturnsLazily`. In the expression we say to return the object based on the dictionary's key.&nbsp;

Finally,&nbsp;`GetGameNames`&nbsp;will return both game names separated by commas and then we can check if the method is working.

## **Moq**

Now, I'm going to show two ways to make multiple calls to the same method with the Moq framework.

The first is to use `SetupSequence`:

~~~csharp
[Fact]
public void GetGameNames_ShouldReturnAllGamesNamesSeparatedByComma_Moq_1()
{
    //Arrange             
    var games = new Game[] {
        new Game { Id = 1, Name = "Nier: Automata" },
        new Game { Id = 2, Name = "Rocket League" }
    };

    var gameRepositoryMock = new Mock<IGameRepository>();
    gameRepositoryMock.SetupSequence(_ => _.FindGameById(It.IsAny<int>()))
        .Returns(games[0])  //Returned in the first call
        .Returns(games[1]); //Returned in the second call

    var sut = new GameService(gameRepositoryMock.Object);

    //Act
    var result = sut.GetGameNames(games.Select(g => g.Id).ToArray());

    //Assert
    Assert.Equal("Nier: Automata, Rocket League", result);
}
~~~

With the&nbsp;`SetupSequence`&nbsp;we setup our method as we normally do in Moq, but rather than have one call to `.Returns`, we have multiple ones. In the example, we call&nbsp;`.Returns`&nbsp;twice, so&nbsp;each call to `.FindGameById` returns a different object.

The second option is as the following sample:

~~~csharp
[Fact]
public void GetGameNames_ShouldReturnAllGamesNamesSeparatedByComma_Moq_2()
{
    //Arrange             
    var games = new Queue<Game>(new Game[] {
        new Game { Id = 1, Name = "Nier: Automata" },
        new Game { Id = 2, Name = "Rocket League" }
    });

    var gameRepositoryMock = new Mock<IGameRepository>();
    gameRepositoryMock.Setup(_ => _.FindGameById(It.IsAny<int>()))
        .Returns(() => games.Dequeue());

    var sut = new GameService(gameRepositoryMock.Object);

    //Act
    var result = sut.GetGameNames(games.Select(g => g.Id).ToArray());

    //Assert
    Assert.Equal("Nier: Automata, Rocket League", result);
}
~~~

Here, we use the `.Setup` and one `.Returns`&nbsp;setup the mock, the difference is&nbsp;how we configure the return object. Instead of return an object, the mock returns a function. Then, our repository method `.FindGameById` executes this function on each call.

When our function calls&nbsp;`.Dequeue()`,&nbsp;`.FindGameById`&nbsp;return a different object each time.

## **NSubstitute**

In _NSubstitute,_ we configure subsequent calls in the same way used to return one object, using&nbsp;`.Returns` or `.ReturnsForAnyArgs`:

~~~csharp
[Fact]
public void GetGameNames_ShouldReturnAllGamesNamesSeparatedByComma_NSubstitute_1()
{
    //Arrange             
    var games = new Game[] {
        new Game { Id = 1, Name = "Nier: Automata" },
        new Game { Id = 2, Name = "Rocket League" }
    };

    var gameRepositoryMock = Substitute.For<IGameRepository>();
    gameRepositoryMock.FindGameById(Arg.Any<int>())
        .ReturnsForAnyArgs(games[0], games[1]);

    var sut = new GameService(gameRepositoryMock);

    //Act
    var result = sut.GetGameNames(games.Select(g => g.Id).ToArray());

    //Assert
    Assert.Equal("Nier: Automata, Rocket League", result);
}
~~~

Rather than send one object, we can send multiple objects through parameters.

## **Further Reading and References**

- [FakeItEasy - Specifying Return Values](http://fakeiteasy.readthedocs.io/en/stable/specifying-return-values/)
- [Moq - Quick Start](https://github.com/Moq/moq4/wiki/Quickstart)
- [NSubstitute - Multiple return Values](http://nsubstitute.github.io/help/multiple-returns/)
- [Returning multiple fake objects with FakeItEasy](http://blog.jonathanchannon.com/2013/09/20/returning-multiple-fake-objects-with-fakeiteasy/)
