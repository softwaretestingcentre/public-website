---
title: "Lesson 16 - XSS injection"
date: 2025-06-09 15:00:00 -0000
---

Moving onto the next challenge, injecting a script into the page.

We are going to try to get an alert message to appear:

![image](https://github.com/user-attachments/assets/3dc21756-498c-40b1-9807-260d86c4fd15)

# Setup

Switching back to Serenity BDD (because Cypress has an alert-handling bug), we create a feature file for the second challenge:
```gherkin
Feature: Juice Shop is susceptible to XSS attacks

  Scenario: Haxxor injects HTML into the search input
    Given Haxxor goes to the Juice Shop
    When she injects HTML into the search input
    Then she sees an alert message containing "xss"
    And she sees she has solved the "DOM XSS" challenge
```

We need new step definitions:
```java
    @When("{actor} injects HTML into the search input")
    public void sheInjectsHTMLIntoTheSearchInput(Actor actor) {
        actor.wasAbleTo(JuiceShop.searchFor("<iframe id='injection' src='javascript:alert(\"xss\")'>"));
    }

    @Then("{actor} sees an alert message containing {string}")
    public void sheSeesAnAlertMessage(Actor actor, String alertMessage) {
        actor.attemptsTo(JuiceShop.alertIsEqualTo(alertMessage));
    }


    @Then("{actor} sees she has solved the {string} challenge")
    public void sheSeesSheHasSolvedTheChallenge(Actor actor, String solvedChallenge) {
        actor.wasAbleTo(JuiceShop.openScoreBoard());
        actor.wasAbleTo(JuiceShop.challengeIsSolved(solvedChallenge));
    }
```

PageObjects:
```java
// HomePage
    public static Target SEARCH_BUTTON = Target.the("Search button")
            .locatedBy("#searchQuery");

    public static Target SEARCH_INPUT = Target.the("Search input")
            .locatedBy("app-mat-search-bar input");

// ScoreBoardPage
    public static Target SPINNER = Target.the("Spinner")
            .locatedBy(".loading-spinner-wrapper");

    public static Target SOLVED_CHALLENGE = Target.the("Solved Challenge")
            .locatedBy("//challenge-card[contains(concat(' ', normalize-space(@class), ' '),
                          ' solved ')]//*[text()='{0}']");
```

And helper methods:
```java
// JuiceShop

    public static Performable searchFor(String searchTerm) {
        return Task.where("{0} searches for " + searchTerm,
                Open.browserOn().the(HomePage.class),
                Click.on(HomePage.SEARCH_BUTTON),
                Enter.theValue(searchTerm).into(HomePage.SEARCH_INPUT),
                SendKeys.of(Keys.ENTER).into(HomePage.SEARCH_INPUT)
        );
    }

    public static Question<String> getAlertText() {
        return Question.about("Alert text").answeredBy(
                actor -> {
                    actor.attemptsTo(Switch.toAlert());
                    String acknowledgement = actor.asksFor(HtmlAlert.text());
                    actor.attemptsTo(Switch.toAlert().andDismiss());
                    return acknowledgement;
                }
        );
    }

    public static Performable alertIsEqualTo(String alertText) {
        return Ensure.that("Alert message is " + alertText,
                getAlertText()).isEqualTo(alertText);
    }

    public static Performable openScoreBoard() {
        return Task.where("{0} opens their Score Board",
                Open.browserOn().the(ScoreBoardPage.class),
                WaitUntil.the(ScoreBoardPage.SPINNER, WebElementStateMatchers.isNotVisible())
        );
    }

    public static Performable challengeIsSolved(String solvedChallenge) {
        return Ensure.that(ScoreBoardPage.SOLVED_CHALLENGE.of(solvedChallenge)).isDisplayed();
    }
```

# Refactoring
There were a couple of things I wasn't happy about:
- Handling cookies through the UI
- Hiding test data in the Step Definition

> ❗ Using the UI for cookie handling could introduce flakiness

So I added an event handler to the `HomePage` PageObject to set the cookies and refresh the page:
```java
    @WhenPageOpens
    public void setCookies() {
        if (getDriver().manage().getCookieNamed("cookieconsent_status") == null) {
            getDriver().manage().addCookie(new Cookie("cookieconsent_status", "dismiss"));
            getDriver().manage().addCookie(new Cookie("welcomebanner_status", "dismiss"));
            getDriver().manage().addCookie(new Cookie("language", "en"));
            getDriver().navigate().refresh();
        }
    }
```

> ❗ Test input data should be in the Scenario, where it can be reviewed more easily

I also moved the XSS value from the Step Definition to the Scenario:
```gherkin
  Scenario: Haxxor injects HTML into the search input
    Given Haxxor goes to the Juice Shop
    When she searches for "<iframe src=\"javascript:alert(`xss`)\">"
    Then she sees an alert message containing "xss"
    And she sees she has solved the "DOM XSS" challenge
```

> ℹ️ Note that I had to use the _exact_ quotes that the Challenge is expecting before it would be marked as solved

```java
    @When("{actor} searches for {string}")
    public void sheSearchesFor(Actor actor, String searchTerm) {
        actor.wasAbleTo(JuiceShop.searchFor(searchTerm));
    }
```
