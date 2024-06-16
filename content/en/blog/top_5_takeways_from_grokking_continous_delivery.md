---
author: Eve Kolb
title: My top 5 takeaways from reading Grokking Continuous Delivery
date: 2024-04-21
description: The most impactful changes you can make to your CI/CD workflows
tags: ["ci", "cd", "book"]
type: blog
---

## Intro

I recently finished reading
[Grokking Continuous Delivery](https://www.manning.com/books/grokking-continuous-delivery)
by Christie Wilson.
It's a super easy introduction to a lot of CI/CD Concepts and should give any
reader a good start in building more resilient software.
Today I wanna share my biggest takeaways from reading this book.

## What this article is not

This article is not an exhaustive list of "lessons-learned" from the book,
by any stretch of the imagination.
i recommend checking the book out yourself
if you want to learn all the book has to offer.

## My takeaways

### 1. The test pyramid

Christie Wilsons shares a widely accepted split between **unit tests**,
**integration tests** and **end-2-end tests** (called e2e tests going forward).

here's an explainatory diagram

![test pyramid](../images/grokking-continuous-delivery/test-pyramid.svg)

As you can see, the following split between test types is recommend:

- 70% unit tests
- 20% integration tests
- 10% e2e tests

You might think "Why this split?". Well, one big reason is the time
these tests each take.
Unit Tests usually only take a few milliseconds, while integration tests
can easily take 5-10 seconds, and e2e tests even longer.

This prompts the question "Why aren't we just writing unit tests then?".
At first this seems like a good idea, they are the fastest,
so we can run more of them.

But if we look closer at the kinds of bugs these tests can surface,
we see that each test has its own strengths and areas it excells at.

Unit Tests only test a small unit of the code as the name suggests.
Therefore they are good at testing isolated logic, but fail at testing
how that logic interacts with other parts of the system.
Thats where integration tests come in. A good example of a usecase
of integration tests are database interactions. They test how your code
integrates with the database.

E2E tests on the other hand are really good at finding issues in deployment
strategies and processes, since they test the system as a whole.

Together, these 3 test types surface the most complete set of bugs.
But even then, they will never surface everything, a perfect system doesnt exist.
Thats why it is important to continously write tests for bugs as they
are found.

### 2. Deploying more often reduces deployment pains

If our deployments fail often the intuitive thing to do is to deploy less,
allowing for more time to testing before release.
This introduces another issue though: With each deployment you
integrate more changes, which have more chance at failing.

Therefore deploying more oten, ideally at every change, reduces the risk
of any particular deployment failing, as there is less changes that can
affect the health of the system, and they are easier to identify and roll back.

If issues and failures aren't caught often enough before deployment,
you should look at adding more E2E tests, as they surface deployment issues.
