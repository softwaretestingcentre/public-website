---
title: "Side Quest 5 - Adding CI features with Github Actions"
date: 2025-06-13
---
I appreciated the github action in the juice-shop repo that published test results to the repo's github pages and wanted to have the same feature in the security-testing repo (the SerenityBDD tests for Juice Shop).

# Publishing test results
All I had to do was add the [github-deploy-action](https://github.com/JamesIves/github-pages-deploy-action) to `.github/workflows/maven.yml`:
```yaml
    - name: Test Report on GitHub Pages
      uses: JamesIves/github-pages-deploy-action@v4.7.3
      if: github.ref == 'refs/heads/main'
      with:
        branch: gh-pages
        folder: target/site/serenity
        clean: true
```
And change the **Settings/Actions/General/Workflow Permissions** for the repo to read/write:

![image](https://github.com/user-attachments/assets/ae2803cf-1915-4aaa-8780-a8a596ca13f0)

And now every time I push to the repo, the tests run and publish the results to the repo's [github pages](https://softwaretestingcentre.github.io/security-testing/).

Which is much easier than downloading them as an artifact and opening them locally.

# Creating a test summary
The [publish-unit-test-result-action](https://github.com/EnricoMi/publish-unit-test-result-action) can provide a nice summary on the workflow run page when the tests have finished.

I added the config to the same workflow file
```yaml
    - name: Publish Test Results
      uses: EnricoMi/publish-unit-test-result-action@v2
      if: (!cancelled())
      with:
        files: |
          target/failsafe-reports/*CucumberTestSuite.xml
```

The tests ran and produced a summary. I pushed a new test:
```gherkin
  Scenario: Haxxor reads the privacy policy
    Given Haxxor goes to the Juice Shop
    When she opens the Privacy Policy
    Then she sees she has solved the "Privacy Policy" challenge
```

and this updated the results:

![image](https://github.com/user-attachments/assets/d12f3221-ed18-4a98-ad2d-e9a9f5238029)

> ✔️ You can see that it compares the results to the previous run, picking up the fact that we have a new test

Then I deliberately broke the test and pushed it again:
```gherkin
    Then she sees she has solved the "Reflected XSS" challenge
```

And the failing test is named in the summary:

![image](https://github.com/user-attachments/assets/d8e0bd71-ffd5-4c61-8c15-5b01de9fbb98)

> ✔️ If you click on the test name, it takes you to the commit, so you can see what changed.

> ✔️ The Test Results link in the sidebar lets you drill in to see the error:

![image](https://github.com/user-attachments/assets/f4553ca7-13af-4bbf-9585-3572004865d1)

After we fix the test, we can see the effect on the results:

![image](https://github.com/user-attachments/assets/fc212998-7677-4676-a4a3-b37e68560758)

# Alternative Action when that doesn't work
This Action does not recognise the test results from Serenity/JS, so I created an alternative job for the `serenity-js-cucumber-playwright-template` repo:

```yaml
      - name: Publish Results
        run:
          echo "[Test Results](https://softwaretestingcentre.github.io/serenity-js-cucumber-playwright-template/)" >> $GITHUB_STEP_SUMMARY
```

And when the job runs, we get a link to the results page in the Job Summary:

![image](https://github.com/user-attachments/assets/f1d41135-b5e1-40a3-93c1-b0030a774af4)
