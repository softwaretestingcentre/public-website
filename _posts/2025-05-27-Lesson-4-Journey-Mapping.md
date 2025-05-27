---
title: "Lesson 4 - Journey Mapping"
date: 2025-05-27
---
# Asking the right questions at the right time

In the previous [Lesson 3 QA Pushback](/public-website/2025/05/22/Lesson-3-QA-Pushback.html) article we wrote a test to filter jobs by experience level but found that the functionality was not what a typical user would expect to see.

Sometimes this just happens - it is not apparent that something is wrong until we try to test it. 

> ❗ You are not obliged to implement **every** test you write. It's ok to use them for discovery _and throw them away_.

One advantage of the Screenplay Pattern is that breaking down the framework into layers makes it easier to distinguish between design and implementation issues even before you run a test.

If it's hard to express the requirement as a scenario or if the logic layer is hard to implement, then this is a strong signal that the application **design** needs rework. 

Automated tests are about more than just driving the application successfully. 

> ❗If you get tests “working” when the user journey is ambiguous - or just not being met fully - then the test has no value.

> ❗I have often seen QA Engineers (myself included) hunker down and thrash their way through automating a test for a feature that isn't well defined. Trust your gut instinct - if something feels unusually difficult to automate, stop what you are doing and start asking questions.

> ℹ️ Bugs often hide in unasked questions.

How could we have caught this earlier?

# The practice of Journey Mapping

A BDD technique that we can use during Requirements Discovery is Journey Mapping. 

This is a discussion (or a series of discussions) that seeks to define and refine requirements in terms of:
- Actors
- Goals
- Workflows
- Features
- Scenarios
- Steps

We met Actors before when we started using the Screenplay pattern. Features, Scenarios and Steps we saw in the last 2 lessons when we started writing BDD tests using Cucumber.

> ✔️ By structuring the discussion around this hierarchy, it is easier to think in concrete terms about what our users' aims are and how they will achieve them through the application.

One user journey might be _"John wants to get a new job"_. 

We can express this in terms of the Journey Mapping technique (ignoring Steps for brevity) as follows:
- Actor: John
- Goal: Get a job
- Workflow: Apply for a job
- Feature: Job Application
- Scenario: Apply for a React Developer job at Infosys

Once we have one example, it is easier to think about variations or negative examples that lead to both a more complete design and a wider range of tests - all before a line of code is written.

Thinking of a few more user journeys:

| Actor | Goal | Workflow | Feature | Scenario |
| ----- | ---- | -------- | ------- | -------- |
| John | Get a job | Apply for a job | Job Application | Apply for a React Developer job at Infosys |
| John | Learn about a job | View job | Job Detail | See details for the Software Engineer role at Cloud Mentor |
| John | Highlight interesting jobs | Job saving | Saved Jobs | Save the Jr Backend Developer job at IBM and the Full Stack Engineer job at Facebook |
| John | Browse saved jobs | Job saving | Saved Jobs | See saved jobs |
| John | Manage saved jobs | Job saving | Saved Jobs | Unsave the Jr Backend Developer job |
| John | Only see relevant jobs | Filter Jobs | Filter | By role |
| John | Only see relevant jobs | Filter Jobs | Filter | By experience |
| John | Only see relevant jobs | Filter Jobs | Filter | By salary |
| John | Only see relevant jobs | Filter Jobs | Filter | By applied |
| Emily | Grow the team | Create Jobs | Upload | Create a job listing for a Devops at LockedIn |

# Write Tests Early

Now that we have started the Journey Mapping, we can create BDD-style tests to exercise the features and scenarios we have uncovered.

> ✔️ It doesn't matter at this stage that we have no application features to test - or that we haven't defined the Steps yet - we have some useful documentation about the system that we can write in a BDD style for review by the Business Expert(s).

> ℹ️ I have tried to be consistent throughout these lessons by talking about QA collaborating with "Business Experts" - in your organisation these might be any combination of Product Owners, Business Analysts, Subject Matter Experts and Expert Users.

For example, we can write a test to filter jobs by salary **whether the feature is ready or not**:
```gherkin
  Scenario: Filter jobs by salary
    Given John is browsing jobs
    When he filters on salary above "50000" pa
    Then he only sees jobs that pay more than "50000" pa
```

And we simply mark the Step Definitions as "Pending":
```java
    @When("he filters on salary above {string} pa")
    public void heFiltersOnSalaryAbovePa(String annualSalary) {
        throw new PendingException("Implement me");
    }

    @Then("he only sees jobs that pay more than {string} pa")
    public void heOnlySeesJobsThatPayMoreThanPa(String annualSalary) {
        throw new PendingException("Implement me");
    }
```

When we run the test framework, we can see that this requirement has been considered but the test is in the Pending state:
![image](https://github.com/user-attachments/assets/236c41d7-d390-4c9b-b4c2-15bcfaf53172)




  
