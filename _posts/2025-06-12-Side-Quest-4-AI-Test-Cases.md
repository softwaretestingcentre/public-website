---
title: "Side Quest 4 - Generating BDD scenarios with AI"
date: 2025-06-12
---
> ❗ This is entirely experimental, I wouldn't trust AI with writing test scripts, but it might be useful as an automated generator of BDD scenarios

In this lesson I'm going to play with [Workik](https://workik.com/) and see what it can offer the weary/forgetful tester.

From the previous lessons, we have a couple of XSS test cases to run against Juice Shop, so let's see if AI can suggest any more.

Our prompt:
```
Generate Cross Site Scripting scenarios in Cucumber.
Run as a user called "Haxxor".
Try to inject XSS into the OWASP Juice Shop website search form.
User does not need to login
```

In a few seconds it responds with:
```
This feature defines XSS injection test cases
targeting only the search form on OWASP Juice Shop
as user "Haxxor" without logging in,
checking the site resists typical reflected XSS attacks via search input.
```
And it produces 1 scenario, with multiple examples of payloads:
```gherkin
Feature: Cross Site Scripting (XSS) Injection in OWASP Juice Shop Search Form as Unauthenticated User

  As a security tester named "Haxxor"
  I want to inject XSS payloads into the OWASP Juice Shop search form without logging in
  So that I can verify if the search functionality is vulnerable to Cross Site Scripting attacks

  Background:
    Given I am identified as user "Haxxor"
    And I am on the OWASP Juice Shop homepage

  @xss_search_injection
  Scenario Outline: Inject XSS payloads into the search form and verify no script execution
    Given I am on the Search page
    When I enter the XSS payload "<payload>" into the search field
    And I submit the search form
    Then I should not see any alert popup triggered by the injected script
    And the search results page should render safely without executing any script

    Examples:
      | payload                            |
      | <script>alert('XSS')</script>      |
      | '"><img src=x onerror=alert(1)>    |
      | <svg/onload=alert('XSS')>          |
      | <body onload=alert('XSS')>         |
      | "><iframe src=javascript:alert(1)> |
```
These are good suggestions but I'm not sure it's adding much over a good process and set of standards for XSS testing.

Let's try running them anyway (rewritten to match the existing step definitions):
```gherkin
  Scenario Outline: Inject XSS payloads into the search form and verify no script execution
    When she searches for "<payload>"
    Then she sees an alert message containing ""

    Examples:
      | payload                              |
      | <script>alert('XSS')</script>        |
      | '"><img src=x onerror=alert(`1`)>    |
      | <svg/onload=alert('XSS')>            |
      | <body onload=alert('XSS')>           |
      | "><iframe src=javascript:alert(`1`)> |
```
> ❗ OK we **strictly shouldn't see an alert at all** but I know the site is vulnerable and I just wanted to see what happens.

# And what did happen?
It broke the [BDD Report](https://github.com/serenity-js/serenity-js/issues/2879)

There we go - I knew AI was dangerous 🤖💣
