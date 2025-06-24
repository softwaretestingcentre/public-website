---
title: "Lesson 20 - Python BDD with Behave"
date: 2025-06-24
---
So far we have seen how to write BDD style tests in Java, Javascript and Typescript.

For this lesson, we are going to use Python.

> ⚠️ Trigger Warning! - Python is not my strongest suit, so there may be _non-pythonic_ content 😱

At first I tried to reimplement some of the existing tests in [pytest-bdd](https://pytest-bdd.readthedocs.io/en/stable/#) but I couldn't get on with it, so I switched to [Behave](https://behave.readthedocs.io/en/stable/index.html).

I implemented the "hidden page" challenges from Juice Shop, with a refactor to reuse the framework code for both pages.

Going from 2 Scenarios with mostly duplicated steps:
```gherkin
  Scenario: Haxxor reads the privacy policy
    Given Haxxor goes to the Juice Shop
    When she opens the Privacy Policy
    Then she sees she has solved the "Privacy Policy" challenge

  Scenario: Haxxor opens the Score Board
    Given Haxxor goes to the Juice Shop
    When she opens the score board
    Then she sees she has solved the "Score Board" challenge
```

to a single Scenario Outline:
```gherkin
# /features/hidden-pages.feature
  Scenario Outline: Haxxor finds hidden pages
    Given Haxxor goes to the Juice Shop
    When she opens the "<Hidden Page>"
    Then she sees she has solved the "<Hidden Page>" challenge

    Examples:
      | Hidden Page    |
      | Privacy Policy |
      | Score Board    |
```

# All Context
Behave has the concept of `context` which allows us to store arbitrary data to be shared around between steps.

We can use this for our test setup  - e.g to initialise a Playwright browser, set a base url, etc:
```python
# /features/environment.py

from behave import fixture, use_fixture
from playwright.sync_api import sync_playwright

@fixture
def browser_page(context):
    with sync_playwright() as p:
        browser = p.chromium.launch(channel="chrome")
        context.page = browser.new_page()
        yield context.page
        browser.close()

def before_all(context):
    context.base_url = "https://stc-owasp-juice-dnebatcgf2ddf4cr.uksouth-01.azurewebsites.net"
    use_fixture(browser_page, context)
```

Our step definitions can (should) be simple one-liners, using ui and api helper methods:
```python
# /features/steps/juice-shop.py

from behave import *
from features.steps.ui.juice_shop_ui import *
from features.steps.api.juice_shop_api import *

@given("Haxxor goes to the Juice Shop")
def open_juice_shop(context):
    open_shop(context)

@when('she opens the "{}"')
def open_hidden_page(context, page):
    open_page(context, page)

@then('she sees she has solved the "{}" challenge')
def check_challenge(context, challenge_name):
    assert check_challenge_solved(context, challenge_name)
```

> ℹ️ Note that the {} construct isn't strictly needed for Behave, but VS Code gets confused if you include a parameter name

For the ui helpers, we are going to use Playwright methods via the `context.page` object initialised in `environment.py` before the tests run :
```python
# /features/steps/ui/juice_shop_ui.py

pages = {
    "Privacy Policy": "/privacy-security/privacy-policy",
    "Score Board": "/score-board"
}

def open_shop(context):dict
    context.page.goto(f"{context.base_url}/#/")

def open_page(context, page):
    context.page.goto(f"{context.base_url}{pages[page]}")
```
By implementing `pages` as a **dict**, we are able to use the page name for both the `When` and `Then` steps.

For API calls, we just use the `requests` library:
```python
# /features/steps/api/juice_shop_api.py

import requests

def check_challenge_solved(context, challenge):
    challenge_statuses = requests.request("GET", f"{context.base_url}/api/Challenges/?name={challenge}").json()
    return challenge_statuses["data"][0]["solved"]
```

# Debugging Behave tests in VS Code
Create a `launch.json` file to enable run/debug of individual files and all features:
```json
{  "configurations": [
    {
    "name": "Python: Behave current file",
    "type": "debugpy",
    "request": "launch",
    "module": "behave",
    "console": "integratedTerminal",
    "args": ["${file}"],
    },
    {
    "name": "Python: Behave all features",
    "type": "debugpy",
    "request": "launch",
    "module": "behave",
    "console": "integratedTerminal",
    "args": ["${workspaceFolder}/features"],
    },
  ]
}
```

Now if we press **F5** while `hidden-pages.feature` is open, the tests run:
```
Feature: Users can find hidden pages # features/hidden-pages.feature:1

  Scenario Outline: Haxxor finds hidden pages -- @1.1           # features/hidden-pages.feature:10
    Given Haxxor goes to the Juice Shop                         # features/steps/juice-shop.py:5 3.945s
    When she opens the "Privacy Policy"                         # features/steps/juice-shop.py:9 2.059s
    Then she sees she has solved the "Privacy Policy" challenge # features/steps/juice-shop.py:13 2.822s

  Scenario Outline: Haxxor finds hidden pages -- @1.2        # features/hidden-pages.feature:11
    Given Haxxor goes to the Juice Shop                      # features/steps/juice-shop.py:5 0.744s
    When she opens the "Score Board"                         # features/steps/juice-shop.py:9 2.330s
    Then she sees she has solved the "Score Board" challenge # features/steps/juice-shop.py:13 3.812s

1 feature passed, 0 failed, 0 skipped
2 scenarios passed, 0 failed, 0 skipped
6 steps passed, 0 failed, 0 skipped, 0 undefined
Took 0m15.711s
```

and we can add breakpoints as needed:

![image](https://github.com/user-attachments/assets/1756cbde-6c81-4244-ad45-156bab07a857)

