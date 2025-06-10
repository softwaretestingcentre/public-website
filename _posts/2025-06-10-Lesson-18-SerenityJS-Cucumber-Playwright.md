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

  Background:
    Given Alice starts with the "Form Authentication" example

  Scenario Outline: Using username and password to log in
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

# Under the hood
The Step Definitions are in `features/step-definitions/the-internet.steps.ts`:
```typescript
Given('{actor} starts with the {string} example', async (actor: Actor, exampleName: string) =>
    actor.attemptsTo(
        Navigate.to('/'),
        PickExample.called(exampleName),
    )
);

When('{pronoun} log(s) in using {string} and {string}', async (actor: Actor, username: string, password: string) =>
    actor.attemptsTo(
        Authenticate.using(username, password),
    )
);

Then(/.* should see that authentication has (succeeded|failed)/, async (expectedOutcome: string) =>
    actorInTheSpotlight().attemptsTo(
        VerifyAuthentication[expectedOutcome](),
    )
);
```

So far so standard, except that the `Then` step name uses a RegExp instead of a Cucumber Expression, just to.

I won't go into the `PickExample` definition - there are good comments in the repo for the curious.

`test/authentication/Authenticate.ts` contains the Playwright code used to support the login steps:
```typescript
export const Authenticate = {
    using: (username: string, password: string) =>
        Task.where(`#actor logs in as ${ username }`,
            Enter.theValue(username).into(LoginForm.usernameField()),
            Enter.theValue(password).into(LoginForm.passwordField()),
            Click.on(LoginForm.loginButton()),
        ),
}

/**
 * This is called a "Lean Page Object".
 * Lean Page Objects describe interactive elements of a widget.
 * In this case, the login form widget at https://the-internet.herokuapp.com/login
 */
const LoginForm = {
    usernameField: () =>
        PageElement.located(By.id('username')).describedAs('username field'),

    passwordField: () =>
        PageElement.located(By.id('password')).describedAs('password field'),

    loginButton: () =>
        PageElement.located(By.css('button[type="submit"]')).describedAs('login button'),
}
```

> ℹ️ The original authors have chosen to combine the (minimal) PageObject definition with the helper method in one file.

`test/authentication/VerifyAuthentication.ts` supports the check that authentication has proceeded as expected:
```typescript
export class VerifyAuthentication {
    private static hasFlashAlert = () =>
        Task.where(`#actor verifies that flash alert is present`,
            Ensure.that(FlashMessages.flashAlert(), isVisible()),
        )

    static succeeded = () =>
        Task.where(`#actor verifies that authentication has succeeded`,
            VerifyAuthentication.hasFlashAlert(),
            Ensure.that(Text.of(FlashMessages.flashAlert()), includes('You logged into a secure area!')),
        )

    static failed = () =>
        Task.where(`#actor verifies that authentication has failed`,
            VerifyAuthentication.hasFlashAlert(),
            Ensure.that(Text.of(FlashMessages.flashAlert()), includes('Your username is invalid!')),
        )
}

/**
 * A tiny Lean Page Object, representing the flash messages
 * that show up when the user logs submits the authentication form.
 */
const FlashMessages = {
    flashAlert: () =>
        PageElement.located(By.id('flash')).describedAs('flash message'),
}
```

And we can see in the test report the benefit of Serenity's allowing us to describe actions and elements in plain language:

![image](https://github.com/user-attachments/assets/5f202e94-9b1e-486e-9c7d-aa609fad4c5a)

