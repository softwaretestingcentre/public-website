---
title: "Lesson 12 - Golden Rules of Functional Testing"
date: 2025-06-03
---
# A Recap
Over the previous lessons we have used a couple of simple applications to learn how to use a BDD framework to test them.

Along the way, there have been some key lessons, that I will summarise here:
1. Quality is not a reactive task, nor is it something injected at the end of a process, but an ongoing discipline.
2. It's really difficult to build a test framework that makes tests easy to write and maintain, so don't do it - use an existing one.
3. A test framework does more than drive UI interactions, it has its own expressive high-level testing language and produces shareable reports.
4. If you are writing raw Selenium or Playwright commands, you are already creating more work for yourself.
5. System tests should prove that user behaviour is supported by the application.
6. System tests should **not** check the UI structure, input validation or individual system components.
7. Requirements discovery and refinement should be a joint activity of the QA and business teams.
8. Requirements can be automated using BDD before the application is implemented.
9. Splitting the test framework into layers makes test design, implementation, refactoring and maintenance much easier.
10. Thinking about the test framework in layers makes it easier to discover gaps in the requirements.
11. Good BDD Scenarios are short, focussed and devoid of any implementation detail.
12. BDD Scenarios can be reviewed by non-QA teams before any tests are automated.
13. PageObjects should consist of nothing but the (Target) locators we find that we need as we write the tests.
14. Using Dynamic Target locators drastically cuts down the amount of the UI that we need to specify as individual locators.
15. Test code that performs Interaction and Question tasks lives in helper classes - separate from both Step Definitions and PageObjects.
16. Helper classes should do one thing but be flexible enough to handle different inputs - allowing us to expand test coverage easily.
17. If the requirements or implementation make it hard to write tests, QA needs to ask for either or both to change.
18. Never trust the UI - and do not scrape it for test inputs.
19. If the requirements are unclear, ask for concrete examples.
20. **Bugs hide behind unasked questions**.
21. Requirements discovery is an iterative process, not a one-off task.
22. It's never too late to ask questions.
23. We don't need to test everything through the UI. Use APIs and database access whenever needed.
24. As we iterate over the tests, it's OK to rewrite them at any level, including the Scenarios if we can make the tests clearer.
25. Always check the simplest case first.
26. Always vary your inputs.
