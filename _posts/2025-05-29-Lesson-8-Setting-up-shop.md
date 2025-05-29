---
title: "Lesson 8 - Setting Up Shop"
date: 2025-05-29
---

# The online shop application
For the next few lessons, we are going to use a more complete sample website "At Sea Shop", including separate front-end and back-end code and a database.

![image](https://github.com/user-attachments/assets/38be042e-4150-4211-8937-26e9979d7f2a)

The [repo](https://github.com/softwaretestingcentre/atsea-sample-shop-app) contains the full code but we will run it in a Docker container:

`docker-compose up` starts the website on http://localhost:8080/

# A new test framework
We are going to create a new test framework - basically a clone of the one we used for JobHunt:

![image](https://github.com/user-attachments/assets/4509e9b3-7b3c-4a21-9716-a652a361d6da)

Then we'll start to build a BDD-style test to check the wiring:
```gherkin
Feature: Landing Page

  Scenario: Betty sees the landing page
    When Betty opens the shop website
    Then she sees the welcome banner
```

```java
@DefaultUrl("http://localhost:8080/index.html")
public class LandingPage extends PageObject {
    public static Target WELCOME_BANNER = Target.the("Welcome Banner")
            .locatedBy(".headerTitle");
```

```java
public class LandingPageStepDefinitions {
    @When("{actor} opens the shop website")
    public void OpensTheShopWebsite(Actor actor) {
        actor.wasAbleTo(Open.browserOn().the(LandingPage.class));
    }

    @Then("{actor} sees the welcome banner")
    public void sheSeesTheWelcomeBanner(Actor actor) {
        Ensure.that(LandingPage.WELCOME_BANNER.resolveFor(actor).getText())
            .equals("Welcome to the atsea shop");
    }
}
```

> ❗Note that I haven't implemented a helper for Landing Page actions because this isn't a proper test, it's just to check the wiring.

We run the test and everything works:
```
Scenario: Betty sees the landing page # 
  When Betty opens the shop website   # 
  Then she sees the welcome banner    # 
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 4.386 s
```

And we push to the repo at [https://github.com/softwaretestingcentre/test_sample_shop-app]
