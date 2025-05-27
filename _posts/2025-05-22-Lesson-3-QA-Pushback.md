---
title: "Lesson 3 - QA Pushback"
date: 2025-05-22
---
In the [previous BDD lesson](/public-website/2025/05/22/Lesson-2-Cucumber.html) we started to create a BDD framework that allows us to express Requirements in a testable way.

By writing the framework in distinct layers - Business, Logic and Interaction, we saw how easy it was to go from one filtering test example to a Scenario Outline that covers all the job categories:
```gherkin
  Scenario Outline: Filter jobs by category
    Given John is browsing jobs
    When he filters on "<Category>"
    Then he only sees "<Category>" jobs
    Examples:
      | Category          |
      | Frontend          |
      | Backend           |
      | Devops            |
      | Full Stack        |
      | Digital Marketing |
```
# What should we NOT test next?
It would be tempting at this point to write a test that checks that the Filter values themselves are correct and complete. This would involve either accessing the original job data directly or scraping the page to obtain it, then checking that all the categories in the job cards are represented in the filter. 

I would argue that we should not do this as a system test:
- It does not reflect any expected user behaviour
- It verifies the integrity of the application and should be an automated check in the development project
- It introduces a dependency on test data that is not under QA or Business control
- It is dependent on a particular UX design that may change

# What should we test?
There is another filtering test that we can write - showing jobs by the Experience level.

When we do this, we should aim to reuse as much of the framework as we can, rather than writing new test code.

Again we only have a limited set of examples to cover and the Scenario Outline will be very similar to the Category filtering one:
```gherkin
  Scenario Outline: Filter jobs by experience level
    Given John is browsing jobs
    When he filters on "<Experience>" level
    Then he only sees jobs that match the "<Experience>" level
    Examples:
      | Experience |
      | 0-1 year   |
      | 2-3 year   |
      | 4-5 year   |
      | 5+ year    |  
```
Because the filtering for Experience is not the same as for Category, however, we do need new Step Definitions for the **When** and **Then** clauses.

The **When** clause requires some variations on the existing Step Definitions and helper class methods:
```java
// FilterStepDefinitions
    @When("{actor} filters on {string} level")
    public void actor_filters_on_experience(Actor actor, String experience) {
        actor.attemptsTo(FilterJobs.byExperience(experience));
    }

// FilterJobs
    public static Performable byExperience(String experience) {
        return Task.where("{0} filters jobs by " + experience + " experience",
                Click.on(JobListPage.JOB_EXPERIENCE_FILTER.of(experience))
        );
    }

// JobListPage
    public static Target JOB_EXPERIENCE_FILTER = Target.the("experience filter {0}")
            .locatedBy("//*[@class='job-category']//li[.='{0}']//input");
```

The **Then** clause is more complicated because:
- The experience level is not shown in the UI
- The filter is a range not a single value

![image](https://github.com/user-attachments/assets/b1e330ae-b670-402f-b92f-ca6249623f29)


# Don't try to be too clever 
Rather than try to do anything complicated to get the experience data, I discussed this with the developer (me) and we agreed that it would be reasonable for jobseekers to see the required experience level in the job cards, so I updated the UI and pushed a new version of the application:

![image](https://github.com/user-attachments/assets/0143c4ff-10e9-4a67-8fda-f7a9c1523313)

Because we are dealing with comparing this value to a range expressed as a string, however, I needed to write a more involved method for the comparison in `FilterJobs`
```java
// FilterStepDefinitions
    @Then("{actor} only sees jobs that match the {string} level")
    public void actor_only_sees_experience_level_jobs(Actor actor, String experience) {
        actor.attemptsTo(checkAllJobsMatchExperience(experience));
    }

// FilterJobs
    private static Boolean experienceIsInRange(String experience, String range) {
        int experienceYears = Integer.parseInt(experience);
        int experienceMin = Integer.parseInt(String.valueOf(range.charAt(0)));
        int experienceMax;
        try {
            experienceMax = Integer.parseInt(String.valueOf(range.charAt(2)));
        } catch (Exception ignored) {
            experienceMax = 99;
        }
        return experienceYears >= experienceMin && experienceYears <= experienceMax;
    }

    public static Performable checkAllJobsMatchExperience(String experienceRange) {
        return Ensure.that(Text.ofEach(JobListPage.JOB_EXPERIENCE_LIST))
                .allMatch("experience",
                        jobExperience -> experienceIsInRange(jobExperience, experienceRange));
    }

// JobListPage
    public static Target JOB_EXPERIENCE_LIST = Target.the("job experience list")
            .locatedBy(".job-detail>.experience>p>div");
```

# Don't just accept that the UI is correct
While doing exploratory testing of the UI, it becomes apparent that a jobseeker can select multiple experience levels.

This does not make sense - why would a jobseeker look specifically for jobs with with both 2 years _and_ 5 years required experience?

The expected behaviour _should be_ when a jobseeker filters by experience level, all jobs with required experience _up to and including that value_ are shown.

The application UI needs to be updated to reflect this definition of the feature.

> ❗ Don't just accept that the UI is correct and needs to be tested as-is
>
> ❗ Yes - I just wasted my time writing an automated test before questioning the design



