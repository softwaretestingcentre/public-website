---
title: "Lesson 5 - More Features, More Questions"
date: 2025-05-27
---
In the previous [Lesson 4 Journey Mapping](/public-website/2025/05/27/Lesson-4-Journey-Mapping.html) article, we started to define and refine requirements for our JobHunt application.

Let's take the first user journey and work through it:

| Actor | Goal | Workflow | Feature | Scenario |
| ----- | ---- | -------- | ------- | -------- |
| John | Get a job | Apply for a job | Job Application | Apply for a React Developer job at Infosys |

# The first Scenario
We create a new `apply_for_job.feature` file:
```gherkin
Feature: Job Application

  Scenario: Apply for a React Developer job at Infosys
    Given John applies for the "React Developer" job at "Infosys"
    When John completes the application form with "John Smith" and "resume.pdf"
    Then he sees that the job application is acknowledged
```

>❗Note that we don't include any detail about how John gets to the job list page or how he identifies the correct job, just his intentions, actions and expectations

And create `ApplyForJobStepDefinitions.java`:
```java
package com.softwaretestingcentre.testjobportal.stepdefinitions;

import com.softwaretestingcentre.testjobportal.helpers.NavigateTo;
import io.cucumber.java.en.Given;
import io.cucumber.java.en.Then;
import io.cucumber.java.en.When;
import net.serenitybdd.screenplay.Actor;

public class ApplyForJobStepDefinitions {
    @Given("{actor} applies for the {string} job at {string}")
    public void actorAppliesForTheJobAt(Actor actor, String role, String company) {
    }

    @When("{actor} completes the form with {string} and {string}")
    public void actorCompletesTheFormWithAnd(Actor actor, String name, String resume) {
    }

    @Then("{actor} sees that the job application is acknowledged")
    public void heSeesThatTheJobApplicationIsAcknowledged(Actor actor) {
    }
}
```

We want a Target in `JobListPage.java` that allows us to click the _Apply Now_ button for a particular role at a named company:

![image](https://github.com/user-attachments/assets/10ed025f-5b3e-4e69-8fc9-8445fde4cb20)

```java
    public static Target JOB_APPLY = Target.the("job card for {0} at {1}")
            .locatedBy("//*[@class='job-card' and contains(., '{1}{0}')]//a[@href='/apply-jobs']");
```
> ✔️ Again we can use a Dynamic Target, passing in the role and company name

Now we want helper methods to apply for a job. In `JobApplications.java`:
```java
public class JobApplications {
    public static Performable applyForJob(String role, String company) {
        return Task.where("{0} applies for " + role + " job at " + company,
                Click.on(JobListPage.JOB_APPLY.of(role, company)));
    }
}
```
So our **Given** step definition is now:
```java
    @Given("{actor} applies for the {string} job at {string}")
    public void actorAppliesForTheJobAt(Actor actor, String role, String company) {
        actor.wasAbleTo(NavigateTo.pageByLink("Jobs"));
        actor.attemptsTo(JobApplications.applyForJob(role, company));
    }
```

And now we want to model the job application page:

![image](https://github.com/user-attachments/assets/ffba0cbc-ae01-47b2-9bbd-95486433f651)

```java
package com.softwaretestingcentre.testjobportal.helpers;

import net.serenitybdd.core.pages.PageObject;
import net.serenitybdd.screenplay.targets.Target;

public class JobApplicationPage extends PageObject {

    public static Target APPLICANT_NAME = Target.the("applicant name")
            .locatedBy("[name='name']");

    public static Target APPLICANT_RESUME = Target.the("resume")
            .locatedBy("#myFile");

    public static Target SUBMIT = Target.the("submit button")
            .locatedBy("[type='submit']");

}
```

and provide a helper to complete the form:
```java
    public static Performable submitApplication(String name, String resume) {
        Path resumePath = Paths.get("src/test/resources/testdata/" + resume);
        return Task.where(name + " submits an application with " + resume,
                Enter.theValue(name).into(JobApplicationPage.APPLICANT_NAME),
                Upload.theFile(resumePath).to(JobApplicationPage.APPLICANT_RESUME),
                Click.on(JobApplicationPage.SUBMIT));
    }
```

So our **When** step definition becomes:
```java
    @When("{actor} completes the application form with {string} and {string}")
    public void actorCompletesTheFormWithAnd(Actor actor, String name, String resume) {
        actor.attemptsTo(JobApplications.submitApplication(name, resume));
    }
```

Finally we want to use Serenity's alert handler to help us check that the job application was acknowledged:

```java
    public static Performable checkJobApplicationIsAcknowledged(Actor actor) {
        String acknowledgement = actor.asksFor(HtmlAlert.text());
        actor.attemptsTo(Switch.toAlert().andAccept());
        return Ensure.that(acknowledgement).isEqualTo("Your Job Application has been Applied Successfully");
    }

    @Then("{actor} sees that the job application is acknowledged")
    public void heSeesThatTheJobApplicationIsAcknowledged(Actor actor) {
        actor.attemptsTo(JobApplications.checkJobApplicationIsAcknowledged(actor));
    }
```

And the test runs successfully:
![image](https://github.com/user-attachments/assets/d1e3854d-6f73-48ac-9f70-57e62b212387)

> ✔️ I'd wager that it's faster to create this test than to implement the actual feature in the application

# Summary

We've taken a feature specified in a Journey Mapping session and created Living Documentation for it in the form of a BDD-style automated test.

We could now think about adding more positive examples (e.g. applying for different jobs) - _if there is any value in that_.

We should add some negative tests:
- user does not supply a name
- user does not supply a resume
- user applies for the same job twice

There are also some questions and suggestions that arise from writing this test:
- The user should see which job they are applying for on the application form
- What is a good way to check that the user's name and resume have been stored correctly in the system?
- How do we stop users entering the wrong name?
- What limits should we impose on the resume file?

> ℹ️ These should form part of further Journey and Example Mapping sessions, rather than being immediately raised as "bugs" 🐛
