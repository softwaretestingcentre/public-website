---
title: "Lesson 19 - Security Testing with SerenityJS+Playwright"
date: 2025-06-11
---

# Adapting our Security Tests to SerenityJS

In this lesson we are going to cover:
- Navigation
- Cookie handling
- UI Interactions
- Lean Page Objects
- API response parsing
- Data interfaces
- Alert handling

As before we are going to try to run the "Bonus Payload" scenario:
```gherkin
# features/xss-dom/xss-dom.feature
Feature: Juice Shop is susceptible to XSS attacks

  Scenario: Haxxor can inject a payload into the page
    Given Haxxor goes to the Juice Shop
    When she searches for "<iframe width=\"100%\" height=\"166\" scrolling=\"no\" frameborder=\"no\" allow=\"autoplay\" src=\"https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/771984076&color=%23ff5500&auto_play=true&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true\"></iframe>"
    Then she sees she has solved the "Bonus Payload" challenge
```

All of the step definitions will live in `features/step-definitions/juice-shop.steps.ts`:
```typescript
import { Given, Then, When } from '@cucumber/cucumber';
import { Actor } from '@serenity-js/core';

import { JuiceShop, ScoreBoard } from '../../test/juiceshop';

Given('{actor} goes to the Juice Shop', async (actor: Actor) =>
    actor.attemptsTo(
        JuiceShop.open()
    )
);

When('{pronoun} searches for {string}', async (actor: Actor, searchTerm: string) =>
    actor.attemptsTo(
        JuiceShop.searchFor(searchTerm)
    )
)

Then('{pronoun} sees he/she/they has/have solved the {string} challenge', async (actor: Actor, challengeName: string) => {
    actor.attemptsTo(
        ScoreBoard.confirmChallengeSolved(challengeName)
    );
})
```

The `test/juiceshop/JuiceShop.ts` helper object:
```typescript
import { Task } from '@serenity-js/core';
import { By, Click, Cookie,Enter,Key,Navigate, PageElement, Press } from '@serenity-js/web';

export const JuiceShop = {
    open: () => 
        Task.where(
            '#actor opens the Juice Shop',
            Navigate.to('/'),
            Cookie.set({
                name: 'cookieconsent_status', 
                value: 'dismiss'
            }),
            Cookie.set({
                name: 'welcomebanner_status', 
                value: 'dismiss'
            }),
            Cookie.set({
                name: 'language', 
                value: 'en'
            }),
            Navigate.reloadPage()
        ),
        
    searchFor: (searchTerm: string) =>
        Task.where(
            `#actor searches for ${searchTerm}`,
            Click.on(SearchBar.searchButton()),
            Enter.theValue(searchTerm).into(SearchBar.searchInput()),
            Press.the(Key.Enter).in(SearchBar.searchInput())
        ),

}

const SearchBar = {
    searchButton: () =>
        PageElement.located(By.css('#searchQuery')).describedAs('Search button'),

    searchInput: () =>
        PageElement.located(By.css('app-mat-search-bar input')).describedAs('Search input'),

}
```

> ✔️ we set the cookies and reload the page so we don't have to deal with the popups.

> ✔️ the search page elements are defined in a Lean Page Object.

The `test/juiceshop/ScoreBoard.ts` helper object:
```typescript
import { Ensure, isTrue } from '@serenity-js/assertions';
import { Task } from '@serenity-js/core';
import { GetRequest, LastResponse, Send } from '@serenity-js/rest';

export const ScoreBoard = {

    confirmChallengeSolved: (challengeName: string) => 
        Task.where(`#actor confirms that ${challengeName} has been solved`,
            Send.a(GetRequest.to(`/api/Challenges/?name=${challengeName}`)),
            Ensure.that(
                LastResponse.body<ChallengeData>()
                .data[0]
                .solved,
                isTrue()
            )
        ),
    
};

interface ChallengeData {
    status: string,
    data: Challenge[]
}

interface Challenge {
    name: string,
    solved: boolean
}
```
This just makes a call to the API to get the Challenges data and checks that the named Challenge has the property `solved === true`.

> ✔️ we define (minimal) interfaces for returned JSON data, so that our method knows what structure to expect

And when we run with `npm test` we see the test passing:

![image](https://github.com/user-attachments/assets/1f6ba24c-141d-476a-81e3-cdd95434c636)


# More tests

Adding the test for finding the Score Board page is simple. For `features/hidden-page/score-board.feature`:
```gherkin
Feature: Juice Shop has a hidden Score Board

  Scenario: Haxxor opens the Score Board
    Given Haxxor goes to the Juice Shop
    When she opens the score board
    Then she sees she has solved the "Score Board" challenge
```

We just need a step definition and helper method to open the score board:
```java
// features/step-definitions/juice-shop.steps.ts
When('{pronoun} opens the score board', (actor: Actor) => {
    actor.attemptsTo(
        ScoreBoard.open()
    )
})

// test/juiceshop/ScoreBoard.ts
export const ScoreBoard = {

...

    open: () => 
        Task.where('#actor opens the Score Board',
            Navigate.to('/#/score-board'),
        ),

};
```

![image](https://github.com/user-attachments/assets/a39b7814-ee2a-491d-bc26-30da3963f697)


# CI Pipeline and results
Because the original repo comes with Github Actions defined, the tests run whenever there is a push to `main`:
```
[test:execute] Juice Shop has a hidden Score Board: Haxxor opens the Score Board
[test:execute] 
[test:execute]   Given Haxxor goes to the Juice Shop
[test:execute]     Haxxor opens the Juice Shop
[test:execute]       ✓ Haxxor navigates to "/" (2s 332ms)
[test:execute]       ✓ Haxxor sets cookie: { name: "cookieconsent_status", value: "dismiss" } (3ms)
[test:execute]       ✓ Haxxor sets cookie: { name: "welcomebanner_status", value: "dismiss" } (2ms)
[test:execute]       ✓ Haxxor sets cookie: { name: "language", value: "en" } (2ms)
[test:execute]       ✓ Haxxor reloads the page (1s 1ms)
[test:execute]   When she opens the score board
[test:execute]     Haxxor opens the Score Board
[test:execute]     Then she sees she has solved the "Score Board" challenge
[test:execute]       Haxxor confirms that Score Board has been solved
[test:execute]       ✓ Haxxor navigates to "/#/score-board" (67ms)
[test:execute] 
[test:execute]     ✓ Execution successful (4s 50ms)
[test:execute]     --------------------------------------------------------------------------------
[test:execute]     /home/runner/work/serenity-js-cucumber-playwright-template/serenity-js-cucumber-playwright-template/features/xss-dom/xss-dom.feature:11
[test:execute] 
[test:execute]     Juice Shop is susceptible to XSS attacks: Haxxor can inject a payload into the page
[test:execute] 
[test:execute]       Given Haxxor goes to the Juice Shop
[test:execute]         Haxxor opens the Juice Shop
[test:execute]           ✓ Haxxor navigates to "/" (2s 588ms)
[test:execute]           ✓ Haxxor ensures that true does equal true (1ms)
[test:execute]           ✓ Haxxor sets cookie: { name: "cookieconsent_status", value: "dismiss" } (6ms)
[test:execute]           ✓ Haxxor sets cookie: { name: "welcomebanner_status", value: "dismiss" } (2ms)
[test:execute]           ✓ Haxxor sets cookie: { name: "language", value: "en" } (3ms)
[test:execute]           ✓ Haxxor reloads the page (588ms)
[test:execute]       When she searches for "<iframe width=\"100%\" height=\"166\" scrolling=\"no\" frameborder=\"no\" allow=\"autoplay\" src=\"https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/771984076&color=%23ff5500&auto_play=true&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true\"></iframe>"
[test:execute]         Haxxor searches for <iframe width="100%" height="166" scrolling="no" frameborder="no" allow="autoplay" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/771984076&color=%23ff5500&auto_play=true&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true"></iframe>
[test:execute]           ✓ Haxxor clicks on Search button (112ms)
[test:execute]           ✓ Haxxor enters "<iframe width="100%" height="166" scrolling="no" frameborder="no" allow="autoplay" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/771984076&color=%23ff5500&auto_play=true&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true"></iframe>" into Search input (12ms)
[test:execute]           ✓ Haxxor presses key Enter in Search input (25ms)
[test:execute]       Then she sees she has solved the "Bonus Payload" challenge
[test:execute]         Haxxor confirms that Bonus Payload has been solved
[test:execute] 
[test:execute]       ✓ Execution successful (4s 650ms)
[test:execute]       ================================================================================
```

And as a bonus, the test report is published to the [repo's Github Pages](https://softwaretestingcentre.github.io/serenity-js-cucumber-playwright-template/index.html)

# Handling the dreaded Alert dialog
Back to our nemesis 😱, the XSS scenario that creates an Alert:
```gherkin
  Scenario: Haxxor injects HTML into the search input
    Given Haxxor goes to the Juice Shop
    When she searches for "<iframe src=\"javascript:alert(`xss`)\">"
    Then she sees an alert message containing "xss"
    And she sees she has solved the "DOM XSS" challenge
```

By default, SerenityJS [will dismiss any alert dialog](https://serenity-js.org/api/web/class/ModalDialog/), so we need special handlers to capture its message text.

Because the alert dialog is triggered in the "searches for" step, we need to handle the alert in its helper method:
```typescript
// test/juiceshop/JuiceShop.ts
    searchFor: (searchTerm: string) =>
        Task.where(
            `#actor searches for ${searchTerm}`,
            Click.on(SearchBar.searchButton()),
            Enter.theValue(searchTerm).into(SearchBar.searchInput()),
            Press.the(Key.Enter).in(SearchBar.searchInput()),
            Check.whether(searchTerm, includes('javascript:alert'))
            .andIfSo(
                Wait.until(ModalDialog, isPresent()),
                notes().set('alert_message', ModalDialog.lastDialogMessage())
            )
            
        ),
```
We check whether the search term is going to create an alert, then we capture its message into a `note`, which _can_ be shared between steps.

So we can now write a step definition to check the alert message:
```typescript
// features/step-definitions/juice-shop.steps.ts
Then('{pronoun} sees an alert message containing {string}', async (actor: Actor, alertMessage: string) => {
    actor.attemptsTo(
        JuiceShop.confirmAlertMessageIs(alertMessage)
    )
})
```

And the helper method picks up the `note` from before:
```typescript
// test/juiceshop/JuiceShop.ts
    confirmAlertMessageIs: (alertMessage: string) =>
        Task.where(
            `#actor confirms alert message is ${alertMessage}`,
            Ensure.that(notes().get('alert_message'), equals(alertMessage))
        )
```

When we look at the test report, we can see that the alert is only handled in one scenario, as we wanted:

![image](https://github.com/user-attachments/assets/eb6bb980-64e2-4c3f-9157-965b830d5270)

![image](https://github.com/user-attachments/assets/26857990-4228-4327-b0e7-fa0ce3702be0)

