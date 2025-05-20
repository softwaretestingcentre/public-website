---
title: "Lesson 1 - Screenplay Pattern"
date: 2025-05-20
---
# Why UI-centric system tests are bad
In the previous [Lesson Setup](_posts/2025-05-19-lesson-setup.md) article, we had the framework wired up and running a test but we also noted that:
>❗this is not testing user behaviour
>
>❗system tests should not check the website's structure

System tests that just navigate through the application to check that certain elements or strings appear on certain pages are both low value and high cost:
- They are heavily dependent on the structure of the application and on each page not changing, which is very unlikely
- Modern UI frameworks abstract the application design away from the detail of the underlying HTML, so developers are probably unaware that their will break tests
- These checks should be part of front-end unit and component testing, not part of the system tests

***I have seen QA teams barely able to maintain their existing automated regression tests in the face of continual UI refactoring, let alone create tests for new functionality.***

By modelling the tests on user behaviour and outcomes, we insulate ourselves from the overhead of constantly maintaining such UI-centric tests:
- User behaviour is unlikely to change as quickly as UI functionality

  
