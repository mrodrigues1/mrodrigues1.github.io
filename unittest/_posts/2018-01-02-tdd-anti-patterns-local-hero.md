---
layout: post
title: 'TDD Anti-patterns: Local Hero'
date: 2018-01-02 09:00:31.000000000 -03:00
type: post
categories: unittest
tags: tdd
permalink: "/2018/01/02/tdd-anti-patterns-local-hero/"
image: 
  path: /assets/img/blog/TDD-Anti-patterns_-Local-Hero.png
description: >
  Continuing the series of posts about unit tests anti patterns.
---
_This series is inspired by a [Stack Overflow thread](https://stackoverflow.com/questions/333682/unit-testing-anti-patterns-catalogue)._

You're creating unit tests in your local environment, everything looks nice and all the tests are green. But, after you check-in the code, in effect, the tests aren't working elsewhere, only in your machine. They aren't working in your team members machines and the continuous integration is failing.

What could have gone wrong?

This is a typical symptom of the local hero anti-pattern. It's usually cause by Hardcoded paths, locally installed libraries and difference in the environment configuration.

## **Local Hero Code Sample**

To illustrate the local hero anti-pattern, take a look at the following unit test:

~~~csharp
[Fact]
public void AnalyzeFile_ReturnFileDataAnalyzed()
{
    //Arrange
    var fileLocation = "C:\\User\\Matheus\\My Documents\\file.txt";
    var sut = new FileAnalyzer();

    //Act
    var result = sut.AnalyzerTxtFile(fileLocation);

    //Assert
    //...
}
~~~

As you can see, in this unit test we're trying to analyze a text file but, the file is stored on my machine. This unit test will break in other people machine and on the server, because they don't have the file at that location.

A solution to this problem is to store the file inside the solution and add it to the source control. As a result, they won't fail in other environments anymore.

## **Wrap Up**

To conclude, look for possible implementations that are linked to your local environment when writing unit tests.

