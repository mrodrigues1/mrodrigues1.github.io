---
layout: post
title: 'TDD Anti-patterns: The Slowpoke'
date: 2018-03-12 10:00:05.000000000 -03:00
type: post
categories: unittest
tags: tdd integrationtest
permalink: "/2018/03/12/tdd-anti-patterns-the-slowpoke/"
image: 
  path: /assets/img/blog/TDD-Anti-patterns_-The-Slowpoke.png
description: >
  Continuing the series of posts about unit tests anti patterns.
---
_A [Stack Overflow thread](https://stackoverflow.com/questions/333682/unit-testing-anti-patterns-catalogue)&nbsp;inspired this series._

Every once in a while we stumble upon a particular unit test which takes ages to run, taking more than 10 seconds sometimes.&nbsp;Whenever a slow poke test is created,&nbsp;it takes more time to run the tests. In the beginning it seems to cause no harm, it's one slow test, what could go wrong? you may ask.

Time goes by and tests are slower and slower, so the developers stop running the tests. When checking the unit tests, more than one-third are running slow and you don't know how it happened.

Unit tests should run fast as possible, if the tests are slow, developers will start to avoid it or not bother to run them at all.

## **Causes**

There are many cause for this problem, I'm going to list the most common.

The first one is the excessive setup for tests. When the tests have so much setup that it makes the tests slow.

Another possible cause is that the test have some hidden dependency and it cause the test to slow down because it's trying to access the database.

One other cause, that is not that common, but I've seen sometimes, is when you want to simulate a timeout from a web service. This definitely will slow down the unit test, because a timeout usually can take a considerable amount of time to happen.

## **How to avoid**

There are some approaches to take when end up in this situation, but the best thing to do is to prevent that it happens.

About excessive setup, instead of having a massive set up method, which contains configuration needed to all tests. Try to break up the setup method into smaller methods and use them in the arrange phase where it makes sense.

When encountering a hidden dependency, like a unit test trying to use the database, or trying to reach a file on the server. What you have to do isolate those dependencies in their own classes and expose it as an interface, to be able to mock them. If testing with the dependency is needed, move the test to its own project, because it configures an integration test.

In case the test has something related to wait an amount of time for something to happen. This kind of test should also moved to the integration test project. Even though they are unit tests, they are too slow to be considered unit tests.

One thing to have in mind is that **unit test have to run fast** , otherwise you may have a problem.

I've already talked about this and more techniques to write better unit tests in more depth, all the links are on the section bellow.&nbsp;

## **Further Reading and References**

- [What Makes Good Unit Test? Readability - Matheus Rodrigues](https://www.matheus.ro/2018/01/15/makes-good-unit-test-readability/)
- [What Makes Good Unit Test? Maintainability - Matheus Rodrigues](https://www.matheus.ro/2017/12/04/what-makes-good-unit-test-maintainability/)
- [What Makes Good Unit Test? Reliability - Matheus Rodrigues](https://www.matheus.ro/2017/10/23/unit-test-reliability/)
- [TDD Antipatterns - Agile in a Flash](http://agileinaflash.blogspot.com.br/2009/06/tdd-antipatterns.html)
- [Unit testing Anti-patterns catalogue - Stack Overflow](https://stackoverflow.com/questions/333682/unit-testing-anti-patterns-catalogue)
