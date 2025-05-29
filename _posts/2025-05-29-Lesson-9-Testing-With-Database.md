---
title: "Lesson 9 - Testing with a Database"
date: 2025-05-29
---
# Creating data

One of the first actual tests we want to write is to create a new user account:
```gherkin
  Scenario: Nick creates a user account
    Given Nick creates a user account
    When Nick enters "nick@softwaretestingcentre.com" and "Pa$$w0rd" login details
    Then A new account is created for Nick
```

We'll write step definitions for this scenario:
```java
public class UserManagementStepDefinitions {

    @Given("{actor} creates a user account")
    public void createsAUserAccount(Actor actor) {
        actor.wasAbleTo(Open.browserOn().the(LandingPage.class));
        actor.wasAbleTo(Click.on(LandingPage.NAVBAR_LINK.of("Create User")));
    }

    @When("{actor} enters {string} and {string} login details")
    public void entersLoginDetails(Actor actor, String userEmail, String userPassword) {
        actor.attemptsTo(CreateUser.createUser(userEmail, userPassword));
    }

    @Then("A new account is created for {actor}")
    public void aNewAccountIsCreated(Actor actor) {
        actor.attemptsTo(CreateUser.checkUserWasCreated());
    }

}
```

And a PageObject to represent the Create User form:

![image](https://github.com/user-attachments/assets/e548fdf7-6aa3-4016-a5e1-34df9feaf994)

```java
public class CreateUserPage extends PageObject {

    public static Target EMAIL = Target.the("User email")
            .locatedBy("[name='username']");

    public static Target PASSWORD = Target.the("User password")
            .locatedBy("[name='password']");

    public static Target SIGN_UP = Target.the("Sign up button")
            .locatedBy(".createFormButton>button");

    public static Target SUCCESS_MESSAGE = Target.the("User created successfully message")
            .locatedBy(".successMessage");

}
```

And finally the CreateUser helper methods:
```java
public class CreateUser {

    public static Performable createUser(String email, String password) {
        return Task.where("Create a user with " + email + " and " + password,
                SendKeys.of("q").into(CreateUserPage.EMAIL),
                DoubleClick.on(CreateUserPage.EMAIL),
                Enter.keyValues(email).into(CreateUserPage.EMAIL),

                Click.on(CreateUserPage.PASSWORD),
                SendKeys.of("q").into(CreateUserPage.PASSWORD),
                DoubleClick.on(CreateUserPage.PASSWORD),
                Enter.keyValues(password).into(CreateUserPage.PASSWORD),

                Click.on(CreateUserPage.SIGN_UP));
    }

    public static Performable checkUserWasCreated() {
        return Ensure.that(Text.of(CreateUserPage.SUCCESS_MESSAGE))
                .matches("Congratulations! Your account has been created!");
    }

}
```

> ❗ Note that the create user form has some VERY peculiar behaviour, where it stops responding after typing the first character in each field, so you have to select that character before  typing the full input value

This test works but it is unsatisfactory for 2 reasons:
- Just checking that we get a success message isn't a very good test that a user has been created
- We won't be able to run the test again without either clearing the `customer` row from the DB or using different credentials

# Direct database access
Since we want to check that we actually created a `customer` row in the DB, and we want to be able to delete arbitrary rows, we need to access the DB directly from the tests.

