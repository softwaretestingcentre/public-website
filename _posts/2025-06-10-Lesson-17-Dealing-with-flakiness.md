---
title: "Lesson 17 - Dealing with flakiness"
date: 2025-06-10 10:00:00 -0000
---
In the previous lesson on [XSS Injection](/public-website/2025/06/09/Lesson-16-XSS-Injection.html), I switched from Cypress to SerenityBDD because Cypress does not deal with the alert dialog properly.

Sadly, SerenityBDD only fares a little better, so I had to refactor the tests to deal with it.

Generally flakiness comes down to:
- Unexpected UI events
- Timing

According to this [SerenityBDD issue](https://github.com/serenity-bdd/serenity-core/issues/2470), there may be a problem with closing "unexpected" Alert dialogs too soon.

This was manifesting in failed test runs with:
```
org.openqa.selenium.NoAlertPresentException: 
no such alert
```

I was also having problems with the test checking for the Solved Challenge elements before they had been rendered:
```
Assertion error: Expected: Solved Challenge that is displayedActual:   web element is not displayed
```

So I needed to be more explicit about waiting in both cases:
```java
// JuiceShop helper
    public static Question<String> getAlertText() {
        // wait for the alert to appear - also flags to WebDriver that we are expecting it
        Alert alert = new WebDriverWait(getDriver(), Duration.ofSeconds(5))
                .until(ExpectedConditions.alertIsPresent());
        return Question.about("Alert text").answeredBy(
                actor -> {
                    actor.attemptsTo(Switch.toAlert());
                    String acknowledgement = actor.asksFor(HtmlAlert.text());
                    actor.attemptsTo(Switch.toAlert().andDismiss());
                    return acknowledgement;
                }
        );
    }

    public static Performable challengeIsSolved(String solvedChallenge) {
        // wait for Challenge cards to be rendered before checking for solved ones
        WaitUntil.the(ScoreBoardPage.CHALLENGE_CARD, WebElementStateMatchers.isVisible());
        return Ensure.that(ScoreBoardPage.SOLVED_CHALLENGE.of(solvedChallenge)).isDisplayed();
    }


// ScoreBoardPage

    @WhenPageOpens
    public void waitForSpinner() {
        WaitUntil.the(ScoreBoardPage.SPINNER, WebElementStateMatchers.isNotVisible());
    }

    public static Target CHALLENGE_CARD = Target.the("Challenge Card")
            .locatedBy("challenge-card");
```

And the tests are now running on CI:
```
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running com.softwaretestingcentre.securitytesting.CucumberTestSuite
Scenario: Haxxor opens the Score Board                     
  Given Haxxor goes to the Juice Shop                      
  When she opens the score board                           
  Then she sees she has solved the "Score Board" challenge 
Scenario: Haxxor injects HTML into the search input                
  Given Haxxor goes to the Juice Shop                              
  When she searches for "<iframe src=\"javascript:alert(`xss`)\">" 
  Then she sees an alert message containing "xss"                  
  And she sees she has solved the "DOM XSS" challenge              
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 20.40 s
```

And we can quickly add a scenario for Challenge 3 just by writing a new Scenario in `xss-injection.feature`:
```gherkin
 Scenario: Haxxor can inject a payload into the page
    Given Haxxor goes to the Juice Shop
    When she searches for "<iframe width=\"100%\" height=\"166\" scrolling=\"no\" frameborder=\"no\" allow=\"autoplay\" src=\"https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/771984076&color=%23ff5500&auto_play=true&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true\"></iframe>"
    Then she sees she has solved the "Bonus Payload" challenge
```
