---
title: "Lesson Setup"
date: 2025-05-19
---
# The Application
To provide some real-world lessons for learning how to do QA, we are going to use a website as our Application Under Test.

I have forked the job-portal app [https://github.com/softwaretestingcentre/job-portal](https://github.com/softwaretestingcentre/job-portal).

Some updates were made using `npm audit fix --force` (see change history).

Run the app locally with `npm start`


# The Test Project
After checking that the app could run, I created a new Selenium Java project in IntelliJ:

![image](https://github.com/user-attachments/assets/7db8bcae-7d3d-450f-91a5-9ddf6cd761ab)

Add `assertj-core` and [SerenityBDD](https://serenity-bdd.github.io/) as dependencies and upgrade from `serenity-junit` to `serenity-junit5` in the POM.

## Writing the First Test
We want to check that available jobs are listed

Create a new (test) class `WhenBrowsingJobs`
```java
package com.softwaretestingcentre.testjobportal;
import net.serenitybdd.annotations.Managed;
import net.serenitybdd.junit5.SerenityJUnit5Extension;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.openqa.selenium.WebDriver;

@ExtendWith(SerenityJUnit5Extension.class)
public class WhenBrowsingJobs {
    @Managed(driver = "chrome")
    WebDriver driver;
    
    @Test
    void jobsShouldBeListed() {

    }
}
```

## Adding support for our test
### Navigation
We want to navigate to the Home Page

Create a class `Navigate Actions`
```java
package com.softwaretestingcentre.testjobportal;
import net.serenitybdd.core.steps.UIInteractions;

public class NavigateActions extends UIInteractions {
    public void toJobPortalHomePage() {
        openUrl("http://localhost:3000/Home");
    }
}
```
### Interactions
And another class `Browse Actions` which will allow us to access the job list
```java
package com.softwaretestingcentre.testjobportal;
import net.serenitybdd.core.steps.UIInteractions;

public class BrowseActions extends UIInteractions {
    public void browseJobs() {
        $("[data-testid='btn']").click();
    }
}
```
## PageComponents
Finally, we add a PageComponent that allows us to interrogate the job listing page
```java
package com.softwaretestingcentre.testjobportal;
import net.serenitybdd.core.pages.PageComponent;

public class JobList extends PageComponent {
    public String firstResult() {
        return $(".job-detail").getText();
    }
}
```
## Completing the Test
Add the above helper classes to the test class and use them to write the test
```java
package com.softwaretestingcentre.testjobportal;
import net.serenitybdd.annotations.Managed;
import net.serenitybdd.junit5.SerenityJUnit5Extension;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.openqa.selenium.WebDriver;
import static org.assertj.core.api.Assertions.assertThat;

@ExtendWith(SerenityJUnit5Extension.class)
public class WhenBrowsingJobs {
    @Managed(driver = "chrome", options = "headless")
    WebDriver driver;

    NavigateActions navigate;
    BrowseActions browse;
    JobList jobList;

    @Test
    void jobsShouldBeListed() {
        navigate.toJobPortalHomePage();
        browse.browseJobs();
        assertThat(jobList.firstResult()).contains("Fullstack Developer");
    }

}
```
Running the jobsShouldBeListed test now will open Chrome, browse to the job list and check that the expected text is displayed.
We should get `Process finished with exit code 0` in the Console.

## Adding BDD-style Steps and Assertions
We can add to the readability of the resulting test report by annotating the helper functions as BDD Steps
```java
public class NavigateActions extends UIInteractions {
    @Step("Navigate to the home page")
    public void toJobPortalHomePage() {
        openUrl("http://localhost:3000/Home");
    }
}

public class BrowseActions extends UIInteractions {
    @Step("Open the Job List")
    public void browseJobs() {
        $("[data-testid='btn']").click();
    }
}
```
Additionally, we wrap the test's assertion in `Serenity.reportThat()`
```java
    @Test
    void jobsShouldBeListed() {
        navigate.toJobPortalHomePage();
        browse.browseJobs();

        Serenity.reportThat("The first job listed should be for a Fullstack Developer",
                () -> assertThat(jobList.firstResult()).contains("Fullstack Developer")
        );
    }
```
Now if we run `mvn verify` we get a more informative report
```
[INFO] Test results for 1 tests generated in 2.1 secs in directory: file:/test-job-portal/target/site/serenity/
[INFO] ------------------------------------------------
[INFO] | SERENITY TESTS:               | SUCCESS
[INFO] ------------------------------------------------
[INFO] | Test scenarios executed       | 1
[INFO] | Total Test cases executed     | 1
[INFO] | Tests passed                  | 1
[INFO] | Tests failed                  | 0
[INFO] | Tests with errors             | 0
[INFO] | Tests compromised             | 0
[INFO] | Tests aborted                 | 0
[INFO] | Tests pending                 | 0
[INFO] | Tests ignored/skipped         | 0
[INFO] ------------------------------- | --------------
[INFO] | Total Duration| 4s 177ms
[INFO] | Fastest test took| 4s 177ms
[INFO] | Slowest test took| 4s 177ms
[INFO] ------------------------------------------------
[INFO] 
[INFO] SERENITY REPORTS
[INFO]   - Full Report: file:///test-job-portal/target/site/serenity/index.html
[INFO] 
[INFO] --- failsafe:3.1.2:verify (default) @ test-job-portal ---
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  14.266 s
[INFO] Finished at: 2025-05-19T17:33:37+01:00
[INFO] ------------------------------------------------------------------------

Process finished with exit code 0
```
And a full HTML report in the `target/site/serenity` folder with screenshots for each interaction
![image](https://github.com/user-attachments/assets/48af42cf-7351-4797-b020-7a02b684f0f0)

# Summary
In this lesson we: 
- created a web application to use for testing
- set up a test framework with helper classes
- wrote and ran our first test
- viewed the test report

> ✔️ our Serenity BDD test framework is wired up correctly
> 
> ✔️ the test script uses our custom classes and Serenity methods, it does not call Selenium methods directly

***It is bad practice to have test scripts that consist of endless lines of Selenium (or Playwright) commands.***

***These commands should only appear in the low level interaction helper classes, around which we build our framework.***

***The test scripts should abstract away any detail about the application, expressing the test in terms that a user can understand.***


> ❗this is not testing user behaviour
> 
> ❗system tests should not check the website's structure
>
> ❗there are magic strings in test script and helper classes

***This first example is only provided to check that the framework is building correctly, it is NOT an example of what we should be testing.***



