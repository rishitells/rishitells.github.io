---
title: The zen of testing
date: "2021-07-11T19:21:20.376Z"
description: "Why testing your code is important"
---

Writing and maintaining automated tests and test suites take significant time and energy of software development teams, so do the benefits of writings tests outweigh the cost associated with them? Can’t we just check the correctness of our code manually and ship it to production instead of spending time writing tests for every new implementation, or fixing failing tests when we make a change? These are some questions which bug many software developers working in a fast paced environment.
I will try to address it in this short write-up.

### Tests give quantitative metrics about code correctness
Running automated tests is probably the quickest way we can reason about the correctness of our code in a systematic manner, instead of a guessing, speculating or fearing about it. Also, for large codebases with many interconnected components, it is imperative to know whether a newly introduced change has impacted the existing components, and if so — how much? Tests give you quick feedback about any deviance from requirement or specification.

> “Rather than apply minutes of suspect reasoning, we can just ask the computer by making the change and running the tests.”
>
> ― Kent Beck, Test-Driven Development: By Example

It gives a tremendous peace of mind when you make a change in code and all of your existing tests still pass. It gives you a confidence to ship the code without worrying about breaking stuff. And because you do not have to think about and manually re-verify what you might have broken, it gives you more time to focus on important things.


### Tests make refactoring a breeze
With automated tests in place, refactoring your code is a breeze. You could just rewrite your whole implementation differently and run tests to see if the new implementation works exactly the way it was working before. No more worrying about large scale refactoring in code, tests have your back!

### Writing tests helps you write good code
To be able to write tests, you have to design your modules in a way that they can be isolated and put under test, independent of other modules. So if you are not able to test a piece of code without touching unrelated code, it’s not designed well in the first place, and is a sign that you should refactor.

### Tests also serve as a documentation for your code.
Writing tests helps other developers, and even non-developers who are trying to understand your implementation. They could just read meaningfully written tests and tell what a piece of code is about and what all it does. 