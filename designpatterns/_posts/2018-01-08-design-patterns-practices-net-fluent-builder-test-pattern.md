---
layout: post
title: 'Design Patterns and Practices in .Net: Fluent Builder Test Pattern'
date: 2018-01-08 09:00:24.000000000 -03:00
type: post
categories: designpatterns unittest
tags: tdd
permalink: "/2018/01/08/design-patterns-practices-net-fluent-builder-test-pattern/"
image: 
  path: /assets/img/blog/Design-Patterns-and-Practices-in-.Net_-Fluent-Builder-Test-Pattern.png
description: >
  Making cleaner switch statements.
---
The unit test's arrange phase is where we write more code. Coding data creation can be cumbersome. The worst scenario happen when we have to fill all class' properties, even though, we're only using one property in the test. Besides being tedious, the data creation process can have a lot of code, resulting in less readable and maintainable tests.

To address this problem, we can use the Builder Pattern. The pattern is described in the book design patterns, and solve problems like:

- How can a class create different representation of a complex object?
- How to simplify a class which creates a complex object?

The Builder Pattern brings more flexibility in a complex object's creation, by delegating the construction of the complex object to a builder object. The builder object can create different representations of the same complex object.

In this post I'm focus on the unit test side, and I'm gonna show you a different version of the Builder Pattern, the Fluent Builder Test Pattern.

The main goal of the Fluent Builder Test Pattern is to facilitate the test data creation. Joining the Builder Pattern and the Fluent Interface Pattern makes the complex object's creation straight forward.

## **Example**

To illustrate the pattern's implementation and usage, let's take a look at the `Question` object:

~~~csharp
public class Question
{
    public string DifficultyLevel { get; set; }

    public string QuestionText { get; set; }

    public List<Answer> Answers { get; set; }

    public Question(string difficultyLevel, string questionText, List<Answer> answers)
    {
        DifficultyLevel = difficultyLevel;
        QuestionText = questionText;
        Answers = answers;
    }
}
~~~

A very simple class, which has two properties and a list of the `Answer` object.

### **Builder Class**

Now, we're going to implement the builder version:

~~~csharp
public class QuestionBuilder
{
    private string _questionText = "Easy";
    private string _difficultyLevel = "Question?";
    private List<Answer> _answers = new List<Answer> { new AnswerBuilder() };

    public Question Build()
    {
        return new Question(_difficultyLevel, _questionText, _answers);
    }

    public QuestionBuilder WithDifficultyLevel(string difficultyLevel)
    {
        _difficultyLevel = difficultyLevel;
        return this;
    }

    public QuestionBuilder WithQuestionText(string questionText)
    {
        _questionText = questionText;
        return this;
    }

    public QuestionBuilder WithAnswers(List<Answer> answers)
    {
        _answers = answers;
        return this;
    }

    public static implicit operator Question(QuestionBuilder instance)
    {
        return instance.Build();
    }
}
~~~

The `QuestionBuilder` has a simple implementation as well.

First, we have the same properties as the `Question` class, the difference is that they're private and have default values, more on that later. In the `Build()` method, we return a new `Question` class' instance using those private properties.

We have the methods which start with the prefix `With` + property name. These methods are responsible to provide the values for the private properties. As you can see, these methods return the `QuestionBuilder` class it self with the `return this;`, as a result, we can do a method chaining, making this class a fluent interface.

The last piece of code is the implicit operator. This is an implicit [conversion operator](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/using-conversion-operators), meaning that when we create an  `QuestionBuilder` instance, it will be converted to a `Question` object, by the call to the `Build()` method inside the operator.

### **QuestionBuilder Usage**

Let's see how we can use the `QuestionBuilder`:

~~~csharp
Question question = new QuestionBuilder()
                            .WithQuestionText("What is your name?");
~~~

The builder is pretty easy to use, we just need to create a new `QuestionBuilder` instance and using the "with" methods to build the object as we please.

We can make a chain call to create a question with answers, for example:

~~~csharp
Question question = new QuestionBuilder()
                            .WithQuestionText("What is your name?")
                            .WithAnswers(new List<Answer>
                            {
                                new AnswerBuilder()
                                    .WithAnswerText("Matheus")
                                    .WithCorrect(true),
                                new AnswerBuilder()
                                    .WithAnswerText("Robert")
                            });
~~~

To create the `List<Answer>` we call the `AnswerBuilder` to create each `Answer`. See that in each builder's usage, we don't need to set up every object's property? This is one of the advantages of using a builder. We only need to configure the properties which we're going to use in the test, the other ones have default values.

For instance, check how the `Question` object creation would be without a builder:

~~~csharp
Question question = new Question(
                                "Easy",
                                "What is your name?",
                                new List<Answer>
                                {
                                    new Answer(
                                        "Matheus",
                                        true),
                                    new Answer(
                                        "Robert",
                                        false)
                                });
~~~

Here, we can see how _verbose_ this creation process is. We have to fill every constructor's argument, in result, the code becomes hard to understand.

## **Conclusion**

In conclusion, the Fluent Builder Test Pattern is very useful, because, it take away a lot of verbose code and allow to create new test data more easily and, leave the code more readable and maintainable.

## **References and Further reading**

- [Design Pattern](https://www.amazon.com/Design-Patterns-Object-Oriented-Addison-Wesley-Professional-ebook/)
- [Builder Pattern - Wikipedia](https://en.wikipedia.org/wiki/Builder_pattern)
- [Fluent Interface - Martin Fowler](https://martinfowler.com/bliki/FluentInterface.html)
- [Unit Testing Patterns: Common Patterns to Follow for Error Free Applications - DZONE](https://dzone.com/articles/unit-testing-patterns-common-patterns-to-follow-fo)
