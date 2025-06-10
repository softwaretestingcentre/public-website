---
title: "Lesson 18 - SerenityJS with Cucumber and Playwright"
date: 2025-06-10 14:00:00 -0000
---
I want to play with the JavaScript version of Serenity which uses Playwright as the browser driver.

Fork [the repo](https://github.com/serenity-js/serenity-js-cucumber-playwright-template)

Make sure I am running node v22 and JRE v17 then build the project with `npm ci`.

There is a default Feature already set up:
```gherkin
Feature: Form-based authentication

  In order to learn how to use Serenity/JS with Cucumber and Playwright
  As a Curious Developer
  I'd like to see an example

  Background:
    Given Alice starts with the "Form Authentication" example

  Scenario Outline: Using username and password to log in

    ["The Internet"](https://the-internet.herokuapp.com/) is an example application
    that captures prominent and ugly functionality found on the web.
    Perfect for writing automated acceptance tests against 😎
    With **Serenity/JS** you can use [Markdown](https://en.wikipedia.org/wiki/Markdown)
    to better describe each `Feature` and `Scenario`.

    When she logs in using "<username>" and "<password>"
    Then she should see that authentication has <outcome>

    Examples:
      | username | password             | outcome   |
      | tomsmith | SuperSecretPassword! | succeeded |
      | foobar   | barfoo               | failed    |
```

Which we can run with `npm test`:
```
[test:execute] ✓ Execution successful (2s 581ms)
[test:execute] ================================================================================
[test:execute] Execution Summary
[test:execute] 
[test:execute] Form-based authentication: 2 successful, 2 total (5s 537ms)
[test:execute] 
[test:execute] Total time: 5s 537ms
[test:execute] Real time: 5s 640ms
[test:execute] Scenarios:  2
[test:execute] ================================================================================
```

And `npm start` to see the SerenityBDD report:

![image](https://github.com/user-attachments/assets/2471b4f8-4baa-41bc-a401-58b1d0b1b3a0)

