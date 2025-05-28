---
title: "Lesson 6 - System Test Coverage"
date: 2025-05-28
---
# What should System Tests cover?

In the previous [Lesson 5 Feature Mapping](/public-website/2025/05/27/Lesson-5-Feature-Implementation.html) article we went through the process of creating a Feature file and BDD-style automated test for a user journey.

At the end of the article, I started to list possible additional negative test cases:
> We should add some negative tests:
> - user does not supply a name
> - user does not supply a resume
> - user applies for the same job twice

On reflection, the first two test cases feel off. 

> ❗ Input validation does not belong in system testing.

Our initial training as QAs may include thinking of ways to validate user input, but I think this hearkens back to the early days of the web, when we had to build websites using dumb HTML elements and write all our own Javascript and/or backend logic to sanitise user inputs.

This shouldn't be the case now. HTML elements are more configurable and front-end frameworks do most of the heavy lifting for us. 

Of course, input validation **does** need to be checked - but by automated tests at the **component test** level.

> ✔️ System tests should be checking that the system can help the user towards their goals.

With this context, only the last of the test cases above feels like a genuine system test:
- it represents a possible **full user journey** that we want to prevent
- detection of this error state requires **multiple components of the application** working together

We can easily reuse the existing step definitions and refactor the helper methods to implement this test:
```gherkin
  Scenario: John should be warned if he applies for the same job twice
    Given John applies for the "Fullstack Developer" job at "Amazon"
    When John completes the application form with "John Smith" and "resume.pdf"
    And he sees that the job application is acknowledged
    And John reapplies for the "Fullstack Developer" job at "Amazon"
    And John completes the application form with "John Smith" and "resume.pdf"
    Then he sees that the job application is rejected
```
> ❗ I normally try to keep my Scenarios under 5 lines - and avoid using **And** whenever I can but the duplication nature of this test allows it

Note that we can write step definitions with alternative wordings for clarity - as in reapplies/applies for the 1st and 4th step in the above scenario:
```java
    @Given("{actor} applies/reapplies for the {string} job at {string}")
    public void actorAppliesForTheJobAt(Actor actor, String role, String company) {
        actor.wasAbleTo(NavigateTo.pageByLink("Jobs"));
        actor.attemptsTo(JobApplications.applyForJob(role, company));
    }
```

Add a Step Definition to check the rejection step:
```java
    @Then("{actor} sees that the job application is rejected")
    public void heSeesThatTheJobApplicationIsRejected(Actor actor) {
        actor.attemptsTo(JobApplications.checkDuplicateJobApplicationIsRejected(actor));
    }
```

And refactor the alert text handlers:
```java
public class JobApplications {

    public static String SUCCESSFUL_APPLICATION = "Your Job Application has been Applied Successfully";
    public static String DUPLICATE_APPLICATION = "You Already Applied For This Job!";

    public static Performable applyForJob(String role, String company) {
        return Task.where("{0} applies for " + role + " job at " + company,
                Click.on(JobListPage.JOB_APPLY.of(role, company)));
    }

    public static Performable submitApplication(String name, String resume) {
        Path resumePath = Paths.get("src/test/resources/testdata/" + resume);
        return Task.where(name + " submits an application with " + resume,
                Enter.theValue(name).into(JobApplicationPage.APPLICANT_NAME),
                Upload.theFile(resumePath).to(JobApplicationPage.APPLICANT_RESUME),
                Click.on(JobApplicationPage.SUBMIT));
    }

    public static Performable applicationAlertHandler(Actor actor, String expectedMessage) {
        String outcome = actor.asksFor(HtmlAlert.text());
        actor.attemptsTo(Switch.toAlert().andAccept());
        return Ensure.that(outcome).isEqualTo(expectedMessage);
    }

    public static Performable checkJobApplicationIsAcknowledged(Actor actor) {
        return applicationAlertHandler(actor, SUCCESSFUL_APPLICATION);
    }

    public static Performable checkDuplicateJobApplicationIsRejected(Actor actor) {
        return applicationAlertHandler(actor, DUPLICATE_APPLICATION);
    }
}
```

And when we run this test, we see that duplicate applications haven't been handled:
```
  Then he sees that the job application is rejected
      java.lang.AssertionError: Expected: a value that is equal to: <"You Already Applied For This Job!">
      Actual:   <"Your Job Application has been Applied Successfully">
```
We need to go back to a Journey Mapping session to describe the expected behaviour.

> ❗ If the test just discovered that the wording of the message was wrong, then we would simply raise a bug.
> 
> ❗ Because the feature is entirely absent in this case, we need to go back to the Business Expert for a new requirement definition.

