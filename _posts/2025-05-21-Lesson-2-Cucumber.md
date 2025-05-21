---
title: "Lesson 2 - Cucumber"
date: 2025-05-21
---
# Using a shared language for requirements and tests
In the previous [Lesson 1 Screenplay Pattern](2025-05-20-lesson-1-screenplay-pattern.md) article, we used the Screenplay Pattern to write a test with the concepts of:
- Actors
- Tasks
- Questions

This has resulted in a more readable test, but we also noted that:
> ❗The test still contains a lot of low-level interaction detail

This is not the kind of information that business experts need or want to see when we collaborate with them on requirements and test cases.

We can use BDD as a common language shared between QA and Business experts. 

This involves splitting the test framework into 3 main layers:
| Layer | Purpose | BDD representation |
| ----- | ------- | ------------------ |
| Business | expresses user journeys and expectations | Feature files and Scenarios |
| Logic | translates user journeys into Actors, Tasks and Questions | Step definitions |
| Interaction | performs tasks and returns answers by interaction with the application | Page and Component helper classes |

# BDD Test Framework structure
We are going to delete the existing framework structure and start again based on the above layers
![image](https://github.com/user-attachments/assets/44a818ab-e8c3-425a-b153-29919139bbaf)

# Writing a feature for the Business Layer
We start by defining a feature - a user journey that describes a task they want to perform and their expectations about the result.

> ℹ️ Note that we do not specify any implemention details about the application at this stage.

> ❗ A common mistake at this stage is to write steps as if they are user manual for the application - e.g. with fine detail about where to click and what to enter into the login fields, for instance.
>
> ❗ This is NOT what we want to see in a Feature file
>
> ❗ It alienates business users, makes tests heavily dependent on implementation details and will quickly become a maintenance nightmare

With this in mind, we create a new feature file, `filter_jobs.feature` with one example scenario:
```gherkin
Feature: Filter jobs based on category
  
Scenario: Only show Frontend jobs
  Given John is browsing jobs
  When he filters on "Frontend"
  Then he only sees "Frontend" jobs
```
# Step Definitions for the Logic layer
Now we need to write Step Definitions to translate the above Given, When, Then steps into executable actions.

Create a step definition file `FilterStepDefinitions.java` and add boilerplate implementations for these 3 steps:
```java
package com.softwaretestingcentre.testjobportal.stepdefinitions;

import io.cucumber.java.PendingException;
import io.cucumber.java.en.Given;
import io.cucumber.java.en.Then;
import io.cucumber.java.en.When;
import net.serenitybdd.screenplay.Actor;

public class FilterStepDefinitions {

    @Given("{actor} is browsing jobs")
    public void actor_is_browsing_jobs(Actor actor) {
        throw new PendingException("Implement this step");
    }

    @When("{actor} filters on {string}")
    public void actor_filters_on_category(Actor actor, String category) {
        throw new PendingException("Implement this step");
    }

    @Then("{actor} only sees {string} jobs")
    public void actor_only_sees_category_jobs(Actor actor) {
        throw new PendingException("Implement this step");
    }

}
```
Now we should see the feature file has picked up the Step Definitions and added syntax highlighting:
![image](https://github.com/user-attachments/assets/1adb1ad9-1503-4c10-bee6-d23dde773699)

From the previous lesson, we know that `John` is a jobseeker Actor. Serenity tracks the current Actor through the test, allowing us to refer to them with prounouns in later steps.

We could run this test now to check that everything is wired up correctly - and we should see instructions to implement the steps:
```
[ERROR] Tests run: 1, Failures: 0, Errors: 1, Skipped: 0, Time elapsed: 1.678 s <<< FAILURE! -- in com.softwaretestingcentre.testjobportal.CucumberTestSuite
[ERROR] Filter jobs based on category.Filter on Frontend jobs -- Time elapsed: 0.457 s <<< ERROR!
io.cucumber.java.PendingException: Implement this step
	at com.softwaretestingcentre.testjobportal.stepdefinitions.FilterStepDefinitions.actor_is_browsing_jobs(FilterStepDefinitions.java:11)
	at ✽.John is browsing jobs(classpath:features/filter_jobs.feature:4)
[INFO] 
[INFO] Results:
[INFO] 
[ERROR] Errors: 
[ERROR]   Implement this step
[INFO] 
[ERROR] Tests run: 1, Failures: 0, Errors: 1, Skipped: 0
```

# Helper classes for the Interaction layer
So that we can implement the step definitions fully, we need helper classes to map steps onto (UI or API) interactions with the application.

We are going to implement a number of helper classes to implement these steps. It might seem like overkill but it pays off in terms of reusability and maintenance.

## PageObjects
One pattern that we can use to express these interactions is to create **PageObjects** that implement interactions on a per page basis, often a good model for web applications.

It is important that a PageObject only provides access to a web page and its elements - i.e. we do not implement any logic or interactions here.

e.g. `JobListPage.java`
```java
package com.softwaretestingcentre.testjobportal.helpers;

import net.serenitybdd.annotations.DefaultUrl;
import net.serenitybdd.core.pages.PageObject;
import net.serenitybdd.screenplay.targets.Target;

@DefaultUrl("https://stc-job-portal.netlify.app/")
public class JobListPage extends PageObject {
    public static Target JOB_FILTER = Target.the("filter category {0}")
            .locatedBy("//li[text()='{0}']");

    public static Target JOB_CATEGORY_LIST = Target.the("job list")
            .locatedBy(".job-detail>.category");

}
```
> ℹ️ Note that because this is a SinglePageApplication, the `@DefaultUrl` for this page is the base url for the entire application.
>
> ℹ️ Note also that we have only created Target objects for the elements we intend to use **now**
> 
> we don't try to map the entire set of elements for the page, because they will probably change as the application is being developed
>
> ✔️ using Serenity's Target objects rather than Selenium selectors allows us to augment them with descriptive text for more readable test reports

## Helper classes
We need a method that allows us to navigate to the job list page, so we create a class `NavigateTo.java`:
```java
package com.softwaretestingcentre.testjobportal.helpers;

import net.serenitybdd.screenplay.Performable;
import net.serenitybdd.screenplay.Task;
import net.serenitybdd.screenplay.actions.Click;
import net.serenitybdd.screenplay.actions.Open;
import net.serenitybdd.screenplay.targets.Target;

public class NavigateTo {

    public static Performable theJobListPage() {
        return Task.where("{0} opens the Job list page",
                Open.browserOn().the(JobListPage.class),
                Click.on(Target.the("Browse Jobs button")
                        .locatedBy("[data-testid='btn']"))
        );
    }

}
```
For now this just has a single function that opens the home page and clicks the button that takes us to the job list.

Now we want another helper class to do filtering tasks, `FilterJobs.java`:
```java
package com.softwaretestingcentre.testjobportal.helpers;

import net.serenitybdd.screenplay.Performable;
import net.serenitybdd.screenplay.Task;
import net.serenitybdd.screenplay.actions.Click;
import net.serenitybdd.screenplay.ensure.Ensure;
import net.serenitybdd.screenplay.questions.Text;

public class FilterJobs {

    public static Performable byCategory(String jobCategory){
        return Task.where("{0} filters jobs by "+jobCategory,
                Click.on(JobListPage.JOB_FILTER.of(jobCategory))
        );
    }

    public static Performable checkAllJobsMatchCategory(String category) {
        return Ensure.that(Text.ofEach(JobListPage.JOB_CATEGORY_LIST))
                .allMatch("category",
                        it -> it.contains(category));
    }

}
```
This provides a Task to do the filtering and an Ensure clause to check that the filtering has worked.

## Full Step Definitions
Now we can use these helper classes to complete Step Definitions for this Scenario:
```java
public class FilterStepDefinitions {

    @Given("{actor} is browsing jobs")
    public void actor_is_browsing_jobs(Actor actor) {
        actor.wasAbleTo(NavigateTo.theJobListPage());
    }

    @When("{actor} filters on {string}")
    public void actor_filters_on_category(Actor actor, String category) {
        actor.wasAbleTo(FilterJobs.byCategory(category));
    }

    @Then("{actor} only sees {string} jobs")
    public void actor_only_sees_category_jobs(Actor actor, String category) {
        actor.attemptsTo(checkAllJobsMatchCategory(category));
    }

}
```

# Full BDD Test Report
Now when we run the test, we can expand the report to see the full detail of the tasks and any underlying interactions:
![image](https://github.com/user-attachments/assets/3405ce3a-e274-4681-ba55-9ffb1ad8f04c)

# Reusability
Given that our helper methods are parameterised it would be trivial to repeat the test for different inputs, e.g.
```gherkin
  Scenario Outline: Filter jobs by category
    Given John is browsing jobs
    When he filters on "<Category>"
    Then he only sees "<Category>" jobs
    Examples:
      | Category |
      | Frontend |
      | Backend  |
```


# Summary
> ✔️ The Features and Scenarios are expressed in a common language understandable to both QA and business teams
> 
> ✔️ The test report shows the Actors and Tasks being performed and allows access to the details of the interactions with the application
> 
> ✔️ The framework has a clear deliniation between Business, Logic and Interaction layers
> 
> ✔️ We have parameterised helper methods that can be reused to cover many inputs and scenarios








