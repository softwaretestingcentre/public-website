---
title: "Side Quest 1 - Running Tests in CI"
date: 2025-05-21
---
# Using Github for CI/CD 

We want to be able to run our tests in a consistent environment and to share the results with the rest of the team.

To this end, we can use Github Actions to:
- Detect commits
- Run our tests against our web application
- Capture any artifacts (e.g. the test report)

# Hosting our web application

We want our web application hosted so that it can be navigated to by the Github CI runner at a consistent url.

[Netlify](https://www.netlify.com/) allows us to host a web application build from code in a Githib repo.

Any commits to the application code will trigger a rebuild and redeploy on Netlify.

We can create a project in Netlify and link it to our web application repo.

Then open **Build & deploy settings** and set these values in **Build settings**

| Setting | Value |
| ------- | ----- |
| Build Command | `CI=false npm run build` |
| Publish Directory | `build` |

Give the Netlify project a name, e.g. `stc-job-portal` from which it will create a url.

Run Deploy and the application will be hosted at `stc-job-portal.netlify.app`:
![image](https://github.com/user-attachments/assets/22a339ba-403c-4559-ac90-c58f7daffda0)

# Creating a Github Action
We need to run tests in headless mode, so we update `serenity.conf`:
```
serenity.test.root = com.softwaretestingcentre.testjobportal.features
webdriver {
    capabilities {
        "goog:chromeOptions" {
            args = ["headless=chrome"]
        }
    }
}
```
And create a Github Action to run the tests and save the report whenever we commit to the test project:
```yaml
name: Test Job Portal

on:
  push:
    branches: [ "master" ]
  pull_request:
    branches: [ "master" ]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4
    - name: Set up JDK 23
      uses: actions/setup-java@v4
      with:
        java-version: '23'
        distribution: 'temurin'
        cache: maven

    - uses: browser-actions/setup-chrome@latest
    - run: chrome --version
    
    - name: Run Serenity Tests
      run: mvn clean verify

    - name: Test Report Generation
      uses: actions/upload-artifact@v4
      if: success() || failure()
      with:
          name: Serenity Report                 # Name of the folder
          path: target/site/serenity/           # Path to test results
```
Now our tests will run against the hosted web application whenever we push changes to the Github test repo:
![image](https://github.com/user-attachments/assets/db266291-d82f-4989-ba71-37b74fd84f9e)

![image](https://github.com/user-attachments/assets/ad959ed3-0460-4248-93a6-8b528be39664)

And we have captured the test report:
![image](https://github.com/user-attachments/assets/907d8a6e-59e1-4504-add4-d79e12f190ac)

