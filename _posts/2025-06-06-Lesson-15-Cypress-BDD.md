---
title: "Lesson 15 - BDD with Cypress"
date: 2025-06-06 16:00:00 -0000
---
I'd like to use BDD with Cypress to test the [OWASP Juice Shop](https://github.com/juice-shop/juice-shop).

# Cypress BDD configuration
I found some of the [online advice](https://www.npmjs.com/package/cypress-cucumber-preprocessor) didn't quite work for this project, so here are the steps I went through:

Install the `cypress-cucumber-preprocessor` dependency:
```
npm install --save-dev cypress-cucumber-preprocessor
npm install --save-dev @types/cypress-cucumber-preprocessor
```

Add some step definition path config to `package.json`:
```json
  "cypress-cucumber-preprocessor": {
    "nonGlobalStepDefinitions": false,
    "stepDefinitions": "test/cypress/integration"
  }
```

Update `cypress.config.ts` to include Cucumber config:
```typescript
...
import * as otplib from 'otplib'

const browserify = require('@cypress/browserify-preprocessor');
const cucumber = require('cypress-cucumber-preprocessor').default;
const resolve = require('resolve');

const options = {
  ...browserify.defaultOptions,
  typescript: resolve.sync('typescript', { baseDir: "/test/cypress/integration/" }),
};

export default defineConfig({
  projectId: '3hrkhu',
  defaultCommandTimeout: 10000,
  retries: {
    runMode: 2
  },
  e2e: {
    baseUrl: 'http://localhost:3000',
    specPattern: '**/*.feature',//'test/cypress/e2e/**.spec.ts',
    downloadsFolder: 'test/cypress/downloads',
    fixturesFolder: false,
    supportFile: 'test/cypress/support/e2e.ts',
    setupNodeEvents (on: any) {
      on("file:preprocessor", cucumber(options)),
      on('task', {
...
```

# BDD implementation in layers
Under the `cypress` folder, add `integration` and create a feature file `integration/basic.feature`:
```gherkin
Feature: Basic navigation

  Scenario: Open Juice Shop
    Given Haxxor goes to the Juice Shop
    When she selects "Apple Juice"
    Then she can see the details include "The all-time classic"
```

We want clean separation between the step definitions and the UI implementation, so create a PageObject file `integration/pages/HomePage.cy.js':
```javascript
class HomePage {
    navigate() {
        cy.visit('/#/');
    }
    openProduct(productName) {
        cy.get("mat-grid-tile").contains(productName).click();
    }
    productStrapLine() {
        return cy.get("app-product-details");
    }
}

const homePage = new HomePage();
export default homePage;
```

The step definitions need to go in a folder that matches the feature name, e.g. `integration/basic/basic.js`:
```javascript
import { Given, When, Then } from "cypress-cucumber-preprocessor/steps";
import homePage from "../pages/HomePage.cy";

Given("Haxxor goes to the Juice Shop", () => {
    homePage.navigate();
});

When('she selects {string}', juiceName => {
    homePage.openProduct(juiceName);
});

Then('she can see the details include {string}', juiceDetails => {
    expect(homePage.productStrapLine().contains(juiceDetails));
});
```

So our file structure looks like this:

![image](https://github.com/user-attachments/assets/f212b260-6d8e-4136-bb93-6cb465c8c803)


# Run the BDD scenario
Now when you run `npx cypress open`, you only see the feature file(s):

![image](https://github.com/user-attachments/assets/80d30129-422f-4551-a397-beee0e41e560)

Note that the test run still uses the Before/After steps defined for the main Cypress e2e tests:

![image](https://github.com/user-attachments/assets/659f662e-7d4e-4767-bd0f-d0a3e5c152ef)

And we can still step through the test:

![image](https://github.com/user-attachments/assets/37f9bfa3-39d5-4431-8709-55efd644f65f)

# BDD implementation of an OWASP challenge solution
Let's solve the **Score Board** challenge as a BDD Scenario `integration/score_board.js`:
```gherkin
Feature: Score Board

Scenario: Hidden Score Board can be opened
Given Haxxor goes to the Juice Shop
When she opens the Score Board
Then she sees she has solved 1 challenge
```

The first step was used before, so we move its definition to `integration/common/steps.js`. 

Define a PageObject `integration/pages/ScoreBoardPage.cy.js`:
```javascript
class ScoreBoardPage {
    navigate() {
        cy.visit('/#/score-board');
    }
    getSolvedCount() {
        return cy.get(".score");
    }
}

const scoreBoardPage = new ScoreBoardPage();
export default scoreBoardPage;
```

And create the other step definitions in `integration/score_board/steps.js`:
```javascript
import { When, Then } from "cypress-cucumber-preprocessor/steps";
import scoreBoardPage from "../pages/ScoreBoardPage.cy";

When("she opens the Score Board", () => {
    scoreBoardPage.navigate();
});

Then("she sees she has solved {int} challenge/challenges", solvedCount => {
    expect(scoreBoardPage.getSolvedCount().contains(" " + solvedCount + "/"));
});
```

Our file structure is now:

![image](https://github.com/user-attachments/assets/a5421d22-91de-4609-b8ff-5e7dfa4d2b95)

And we have tests for 2 features:

![image](https://github.com/user-attachments/assets/0dc71fc0-1574-4d95-88f4-06931b11845c)

And the new test passes:

![image](https://github.com/user-attachments/assets/c4ffbed0-b0d2-4713-b8c4-434a4e152940)

# Run in CI
We can trigger a test run as a Github Action whenever we push changes to the tests (or application):
```yaml
name: "CI/CD Pipeline"
on:
  push:
    branches: [ "master" ]
    paths-ignore:
      - '*.md'
      - 'LICENSE'
      - 'monitoring/grafana-dashboard.json'
      - 'screenshots/**'
    tags-ignore:
      - '*'
  pull_request:
    paths-ignore:
      - '*.md'
      - 'LICENSE'
      - 'data/static/i18n/*.json'
      - 'frontend/src/assets/i18n/*.json'
env:
  NODE_DEFAULT_VERSION: 20

jobs:
  e2e:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest]
        browser: [chrome] 
      fail-fast: false
    steps:
      - uses: actions/checkout@v4

      - name: "Use Node.js"
        uses: actions/setup-node@39370e3970a6d050c480ffad4ff0ed4d3fdee5af #v4.1.0
        with:
          node-version: ${{ env.NODE_DEFAULT_VERSION }}
      
      - name: "Install application"
        run: npm install
      
      - name: "Execute end-to-end tests on Ubuntu"
        if: ${{ matrix.os == 'ubuntu-latest' }}
        uses: cypress-io/github-action@v6
        with:
          install: false
          browser: ${{ matrix.browser }}
          start: npm start
          wait-on: http://localhost:3000
```

I implemented this as a matrix in case I want to use other browsers or OSes at any point.

The results can be seen in the run console:
```
...

────────────────────────────────────────────────────────────────────────────────────────────────────
                                                                                                    
  Running:  score_board.feature                                                             (2 of 2)


  Score Board
info: Solved 1-star scoreBoardChallenge (Score Board)
info: Cheat score for tutorial scoreBoardChallenge solved in 1min (expected ~1min) with hints allowed: 0.42679999999999996
    ✓ Hidden Score Board can be opened (1526ms)


  1 passing (2s)


  (Results)

  ┌────────────────────────────────────────────────────────────────────────────────────────────────┐
  │ Tests:        1                                                                                │
  │ Passing:      1                                                                                │
  │ Failing:      0                                                                                │
  │ Pending:      0                                                                                │
  │ Skipped:      0                                                                                │
  │ Screenshots:  0                                                                                │
  │ Video:        false                                                                            │
  │ Duration:     1 second                                                                         │
  │ Spec Ran:     score_board.feature                                                              │
  └────────────────────────────────────────────────────────────────────────────────────────────────┘


====================================================================================================

  (Run Finished)


       Spec                                              Tests  Passing  Failing  Pending  Skipped  
  ┌────────────────────────────────────────────────────────────────────────────────────────────────┐
  │ ✔  basic.feature                            00:01        1        1        -        -        - │
  ├────────────────────────────────────────────────────────────────────────────────────────────────┤
  │ ✔  score_board.feature                      00:01        1        1        -        -        - │
  └────────────────────────────────────────────────────────────────────────────────────────────────┘
    ✔  All specs passed!                        00:03        2        2        -        -        -
```
And in the run summary:

![image](https://github.com/user-attachments/assets/fb6bf0f0-4291-4686-86df-9d0cdd091fea)
