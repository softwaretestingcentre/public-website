---
title: "Lesson 1 - Screenplay Pattern"
date: 2025-05-20
---
# Why UI-centric system tests are bad
In the previous [Lesson Setup](./2025-05-19-lesson-setup.md) article, we had the framework wired up and running a test but we also noted that:
>❗this is not testing user behaviour
>
>❗system tests should not check the website's structure

System tests that just navigate through the application to check that certain elements or strings appear on certain pages are both low value and high cost:
- They are heavily dependent on the structure of the application and on each page not changing, which is very unlikely
- Modern UI frameworks abstract the application design away from the detail of the underlying HTML, so developers are probably unaware that their will break tests
- These checks should be part of front-end unit and component testing, not part of the system tests

***I have seen QA teams barely able to maintain their existing automated regression tests in the face of continual UI refactoring, let alone create tests for new functionality.***

By modelling the tests on user behaviour and outcomes, we insulate ourselves from the overhead of constantly maintaining such UI-centric tests:
- User behaviour is unlikely to change as quickly as UI functionality

# Writing User-centric tests
We want to treat testing as a collaborative exercise between QA and business experts who define user behaviour and expectations. These should be expressed in a common language that ignores the details of how the application is to be implemented (indeed, there should be no implication in the requirement or test definitions that an application exists at all).

## A common language - the Screenplay Pattern
The [Screenplay Pattern](https://serenity-bdd.github.io/docs/tutorials/screenplay#introducing-the-screenplay-pattern) allows us to express requirements and tests from the point of view of User, the Tasks they want to perform and the Questions they ask of a system. It allows QA and business experts to iterate over the definitions of each requirement until it is in an unambiguous, testable state. 
Requirements expressed in the Screenplay Pattern can then be directly used as automated BDD tests.

### Example test - filtering the job list
Let's add a new test class to the features directory `WhenFilterJobs` which will verify that a user can filter the job list so they only see the jobs they are interested in.

We are going to introduce an Actor `John` as a jobseeker user.

So the empty test class would look like this:
```
@ExtendWith(SerenityJUnit5Extension.class)
class WhenFilterJobs {
    
    @CastMember(name = "John")
    Actor john;
    
    @Test
    @DisplayName("Only show Frontend jobs")
    void filterFrontend() {
        
    }
}
```
As a first attempt, we can use the Serenity methods to perform the navigations and UI interactions that achieve the filtering task:
```
    void filterFrontend() {
        john.attemptsTo(
                Open.url(navigate.home),
                Click.on(Button.withText("Browse Jobs")),
                Click.on(PageElement.locatedBy(".filter li").containingText("Frontend"))
        );
    }
```
And then John can ask what jobs are now showing and check they are all in the category "Frontend":
```
    void filterFrontend() {
        john.attemptsTo(
                Open.url(navigate.home),
                Click.on(Button.withText("Browse Jobs")),
                Click.on(PageElement.locatedBy(".filter li").containingText("Frontend"))
        );
        var filteredJobs = john.asksFor(Text.ofEach(".job-detail>.category"));
        assertThat(filteredJobs).allMatch(job -> job.contains("Frontend"));
    }
```
We run the test and see that the test report contains references to the user and the task they are trying to perform
![image](https://github.com/user-attachments/assets/1c4b58b6-3ee9-4516-b819-9aeaa59877ff)

# Summary
> ✔️ The test is now expressed in the Screenplay Pattern, as a User performing a Task

> ❗The test still contains a lot of low-level interaction detail
> 
> ❗The test is limited to a single filter value
>
> ❗The test contains hard-coded CSS locators, which are likely to need ongoing maintenance as the UI evolves

  
