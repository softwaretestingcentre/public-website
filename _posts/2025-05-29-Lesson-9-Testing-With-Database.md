---
title: "Lesson 9 - Testing with a Database"
date: 2025-05-29
---
# Creating data during a test

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
- We won't be able to run the test again without either clearing the `customer` row from the DB or using different customer input data

# Database access via API
Since we want to check that we actually created a `customer` row in the DB, and we want to be able to delete arbitrary rows, we need to access the DB from the tests.

This app supplies an API for managing data - [Documentation](https://github.com/softwaretestingcentre/atsea-sample-shop-app/blob/master/REST.md).

So if we want to create a new customer, we have to go through some steps:
- Find out if there is a customer record with that email
- If there is, delete it

This will be a combination of `GET /api/customer/username={username}` and `DELETE /api/customer/{customerId}`
> ❗ I'm just starting to realise what a mess the schema is in. I will stick with it for now, for demonstration purposes.

We can also use the `username` endpoint to check that a user has been created.

So, replacing the `CreateUser` helper class with a more general `UserManagement` class, we can use RestAssured methods to work with the REST API:
```java
public class UserManagement {

    public static Performable createUser(String username, String password) {
        deleteUserByName(username);
        theActorInTheSpotlight().remember("username", username);
        return Task.where("Create a user with " + username + " and " + password,
                SendKeys.of("q").into(CreateUserPage.EMAIL),
                DoubleClick.on(CreateUserPage.EMAIL),
                Enter.keyValues(username).into(CreateUserPage.EMAIL),

                Click.on(CreateUserPage.PASSWORD),
                SendKeys.of("q").into(CreateUserPage.PASSWORD),
                DoubleClick.on(CreateUserPage.PASSWORD),
                Enter.keyValues(password).into(CreateUserPage.PASSWORD),

                Click.on(CreateUserPage.SIGN_UP));
    }

    public static Performable checkUserWasCreated() {
        String username = theActorInTheSpotlight().recall("username");
        return Ensure.that(Text.of(CreateUserPage.SUCCESS_MESSAGE))
                .matches("User created",
                        message ->
                                message.equals("Congratulations! Your account has been created!")
                                && getUserIdFromUsername(username) > 0);
    }

    public static int getUserIdFromUsername(String username) {
        int userId = 0;
        try {
            userId = when().get("http://localhost:8080/api/customer/username=" + username)
                    .then().contentType(ContentType.JSON)
                    .extract().path("customerIf"); //sic
        } catch (Exception ignored) {
        }
        return userId;
    }

    public static void deleteUserWithId(int userId) {
        when().delete("http://localhost:8080/api/customer/" + userId)
                .then().statusCode(204);
    }

    public static void deleteUserByName(String username) {
        int userId = getUserIdFromUsername(username);
        if (userId == 0) return;
        deleteUserWithId(userId);
    }

}
```
> ❗ Note that `customerId` is mis-spelled `customerIf` in the JSON response `getUserIdFromUsername()`

> ℹ️ Because we are doing a check on the username now, we have added commands to have it remembered and recalled by the current Actor.

So now we:
- delete the user record with a matching username (if it exists)
- check both that the success message is displayed AND that we have a record for the new customer in the database

> ✔️ Note that all the changes were made in the helper class.
> 
> ✔️ The Scenario and Step Definitions remained the same as before.
>
> ✔️ This is the great advantage of layering your test framework correctly.




