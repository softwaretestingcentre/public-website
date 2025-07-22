---
title: "Lesson 22 - Using AI to repair web test locators"
date: 2025-07-22
---

# Web UI changes cause flaky tests

Famously, one of the hardest problems in web testing is keeping up with changes in the UI.

Our web testing tools, such as Selenium, Playwright and Cypress all depend on being able to reliably identify elements on the page to be interacted with.

Our front-end development frameworks, however, abstract away the physical structure of the webpage. 


**Making development easier has made testing harder.**


When the front-end changes, testers usually have to go through a tedious manual process of determining which tests were broken by that change (or more often, series of changes) and manually update the affected locators.

# Can AI help us to make the updates?

This tedious process begs to be automated. Can AI tools determine from the context of the tests and the application code how the locators in the tests relate to the appication?

ℹ️ For this to work, it helps to have a well-structured test framework - one where just a few locator strings are referenced in one distinct area of the test code.


Here's a series of prompts (to `Copilot/GPT-4.1`) that I used to scope this out:

---

## softwaretestingcentre: Find all the web locators used by the objects defined in the /test folder

## GitHub Copilot: Here are all the web locators used by objects defined in the test folder:

### In Explainer.ts
- `By.css('[data-tab="${sectionName}"]')` — used for tab selection.
- `By.id(sectionName)` — used for advice section identification.

### In Data.ts
- `By.css('table th')` — column headers.
- `By.css('table tr td:nth-child(1)')` — table rows.
- `By.css('table td')` — table cells.
- `By.css('button[aria-label="Show filters"]')` — filter button.
- `By.css('button[aria-label="Export"]')` — export button.

### In User.ts
- `By.id('username')` — username field.
- `By.id('password')` — password field.
- `By.css('button[type="submit"]')` — login button.

These locators are used with Serenity/JS's `PageElement` and `PageElements` objects for UI testing. No direct locators are used in LLM_Explainer.ts or Actors.ts.

> ✔️ This is a pretty good analysis. It has recognised the framework that I'm using and identified all the locators (_and their usage_)


---

## softwaretestingcentre: Compare these locators to any defined _or implied_ by the application code under the /src folder. Highlight any discrepancies.

## GitHub Copilot: Here is a comparison of the locators used in test with those defined or implied in the src application code:

### Locators in test:
- **Tabs:** `By.css('[data-tab="${sectionName}"]')`
- **Advice Section:** `By.id(sectionName)`
- **Table:** `By.css('table th')`, `By.css('table tr td:nth-child(1)')`, `By.css('table td')`
- **Buttons:** `By.css('button[aria-label="Show filters"]')`, `By.css('button[aria-label="Export"]')`, `By.css('button[type="submit"]')`
- **Login Fields:** `By.id('username')`, `By.id('password')`

### Locators in src:

#### Tabs
- In explainer.html, tabs are defined as:
  - `<button class="tab-btn active" data-tab="TimeMachine">`
  - `<button class="tab-btn" data-tab="AgentRecommendation">`
  - `<button class="tab-btn" data-tab="SLABoundaries">`
- **Match:** The test locator for tabs (`[data-tab="${sectionName}"]`) matches the actual HTML.

> ✔️ Good - it sees that the test locators are correctly using the `data-tab` property to identify the tabs on the page.


#### Advice Section
- No direct evidence of elements with `id=sectionName` in the HTML files. If advice sections are expected to have IDs matching section names, this should be confirmed in the HTML.

> ✖️ This is just wrong, it has missed `<section id="TimeMachine" class="tab-content active">`, etc. 🤔


#### Table
- Tables and their headers/cells are present in site-manager.html, deployment.html, etc.:
  - `<table>`, `<th>`, `<td>`
- **Match:** Table locators in tests match the HTML structure.

> OK but the tables are generic, so there is nothing specific to find here yet 😐
  

#### Buttons
- In explainer.html, tab buttons use `data-tab`, but no buttons with `aria-label="Show filters"` or `aria-label="Export"` are found in the HTML files.
- **Discrepancy:** The test expects filter and export buttons with specific `aria-label` attributes, but these are not present in the HTML.

#### Login Fields
- No evidence of elements with `id="username"` or `id="password"` in the HTML files.
- **Discrepancy:** The test expects login fields with these IDs, but they do not exist in the provided HTML.

> ✔️ OK this is good - these filter/export/login locators were specified for testing an earlier version of the application and can now be deleted. 👍

### Summary of Discrepancies

- **Advice Section:** Test expects elements with `id=sectionName`, but these are not found in the HTML.
- **Filter/Export Buttons:** Test expects buttons with `aria-label="Show filters"` and `aria-label="Export"`, but these are missing in the HTML.
- **Login Fields:** Test expects fields with `id="username"` and `id="password"`, but these are not present in the HTML.

**Recommendation:**  
To resolve these discrepancies, update the HTML files to include the missing IDs and `aria-label` attributes, or adjust the test locators to match the actual application markup.

---


# Following the advice and repeating the analysis gets weird
So I deleted the unused locators from the test objects and repeated the analysis:

### Here are all the web locators used by objects defined in the test folder:

### Explainer.ts
- `By.css('[data-tab="${sectionName}"]')` — tab buttons
- `By.id(sectionName)` — advice section

### Data.ts
- `By.css('table th')` — table column headers
- `By.css('table tr td:nth-child(1)')` — first column of table rows
- `By.css('table td')` — all table cells

### User.ts
- `By.id('username')` — username field
- `By.id('password')` — password field
- `By.css('button[type="submit"]')` — login button

These locators are used with Serenity/JS's `PageElement` and `PageElements` for UI testing. No direct web locators are used in LLM_Explainer.ts or Actors.ts.

---

> :warning The locators it found in `User.ts` no longer exist

This is the entire content of `User.ts`:
```typescript
import { Task } from "@serenity-js/core";
import { Navigate } from "@serenity-js/web";

export const User = {
    login: () =>
        Task.where(`#actor logs in`,
            Navigate.to('/landing.html'),
        ),
}
```

> ⚠️ Even if I specifically instruct it to refresh its view of the files, it still picks up these zombie locators from somewhere. 🧟
>
> ✖️ Clicking the `User.ts` file link in the chat takes me to the **only** version of this file that exists in the workspace, so I'm baffled. 🤷‍♂️

> ℹ️ The only way to get past this was to run the analysis on just the `User.ts` file, then repeat it for the `/test` folder.

### Here are all the web locators used by objects defined in the test folder:

**Explainer.ts**
- `By.css('[data-tab="${sectionName}"]')` (tab buttons)
- `By.id(sectionName)` (advice section)

**Data.ts**
- `By.css('table th')` (table column headers)
- `By.css('table tr td:nth-child(1)')` (first column of table rows)
- `By.css('table td')` (all table cells)

**User.ts**
- No web locators are used (only navigation).

No direct web locators are used in LLM_Explainer.ts, Actors.ts, or `index.ts`.

---

# Running the comparison with the application again still has an odd oversight

## Comparison of test locators vs src application code:

### Locators in test (from all relevant files):

- `By.css('[data-tab="${sectionName}"]')` — tab buttons
- `By.id(sectionName)` — advice section
- `By.css('table th')` — table column headers
- `By.css('table tr td:nth-child(1)')` — first column of table rows
- `By.css('table td')` — all table cells

**Note:** No locators in User.ts (only navigation).

### Locators in src (HTML structure):

- **Tab buttons:**  
  Present in explainer.html as `<button class="tab-btn" data-tab="...">`  
  ✅ Matches test locator.

- **Advice section (`id=sectionName`):**  
  No evidence of elements with dynamic IDs matching section names in the HTML.  
  ❌ Discrepancy: Expected IDs for advice sections are missing.

- **Table structure:**  
  Present in multiple HTML files as `<table>`, `<th>`, `<td>`, and rows with multiple columns.  
  ✅ Matches test locators.

### Summary of Discrepancies

- **Advice section IDs (`id=sectionName`)** are not present in the HTML.
- All other locators (tabs, table headers/cells/rows) match the HTML structure.
  
**Recommendation:**  
Update the HTML to include advice section IDs if needed, or adjust the test locator to match the actual markup. No changes needed for table and tab locators.

> ✔️ For some reason the presentation of the analysis is more attractive and succinct in the **Locators in src** section. 🙂
>
> ✖️ It's still wrong about the Advice section lacking ids.

Let's try to help it out by changing the locator:
``` typescript
  adviceSection: (sectionName: string) =>
    PageElement.located(By.css(`section[id="${sectionName}"]`)).describedAs(`${sectionName} advice section`),
```

And refind the locators
> ⚠️ Again, I had to focus on `Explainer.ts` before it would update its view of the locators properly 🤦

And it seems happier now:

---
Here is a comparison of all locators in the test folder versus those defined or implied in the src application code, with discrepancies highlighted:

### Locators in test

**Explainer.ts**
- `By.css('[data-tab="${sectionName}"]')` — tab buttons
- `By.css('section[id="${sectionName}"]')` — advice section

**Data.ts**
- `By.css('table th')` — table column headers
- `By.css('table tr td:nth-child(1)')` — first column of table rows
- `By.css('table td')` — all table cells

**User.ts**
- No web locators (only navigation)

### Locators in src (HTML structure)

**explainer.html**
- Tab buttons: `<button class="tab-btn" data-tab="...">`  
  ✅ Matches test locator.
- Advice sections: `<section id="TimeMachine" ...>`, `<section id="AgentRecommendation" ...>`, `<section id="SLABoundaries" ...>`  
  ✅ Matches test locator.

**site-manager.html, deployment.html, etc.**
- Tables: `<table>`, `<th>`, `<td>`  
  ✅ Matches test locators.

**No login form or fields with `id="username"` or `id="password"` found in src.**
- (No discrepancy, as current User.ts does not use these locators.)

### Discrepancies

- **None found.**  
  All locators currently used in test match the structure and attributes present in the src HTML files.

**Summary:**  

All locators in the test folder are consistent with the elements and attributes defined in the src application code. No discrepancies need to be addressed.

---

> ℹ️ We need to be careful how we define selectors for GPT to recognise them properly
>
> ℹ️ Its view of the locators after any change to the test code needs to be carefully refreshed

> ⚠️ It still looks for the zombie login locators - but chooses to ignore the fact that they don't exist

# What happens when we update the application?

Let's add a specific id to the site manager table (but get it wrong in the test code):
```html
      <h2>Current KPIs</h2>
      <table id="site-kpi">
        <thead>
          <tr><th>KPI</th><th>Expected Value</th></tr>
        </thead>
        <tbody>
```

```typescript
    columnHeaders: () =>
        PageElements.located(By.css('table#site-kpis th')).describedAs('column headers'),
```

And re-run the analysis:

---

Here is a comparison of all locators in the test folder versus those defined or implied in the src application code, with discrepancies highlighted:

### Locators in Data.ts (current focus)

- `By.css('table#site-kpis th')` — table column headers
- `By.css('table tr td:nth-child(1)')` — first column of table rows
- `By.css('table td')` — all table cells

---

### Locators in src (HTML structure)

**site-manager.html**
- Table is defined as:
  ```html
  <table>
    <thead>
      <tr><th>KPI</th><th>Expected Value</th></tr>
    </thead>
    <tbody>
      <tr><td>PUE</td><td>1.33</td></tr>
      <tr><td>WUE</td><td>1.56</td></tr>
    </tbody>
  </table>
  ```
- **Discrepancy:**  
  The test locator expects a table with `id="site-kpis"` (`table#site-kpis th`), but the actual table in the HTML does **not** have an `id` attribute.  
  The other locators (`table tr td:nth-child(1)`, `table td`) match the HTML structure.

### Other Locators in test

**Explainer.ts**
- `By.css('[data-tab="${sectionName}"]')` — matches `<button class="tab-btn" data-tab="...">` in explainer.html
- `By.css('section[id="${sectionName}"]')` — matches `<section id="...">` in explainer.html

**User.ts**
- No web locators (only navigation)

### Summary of Discrepancies

- **Table column header locator:**  
  `By.css('table#site-kpis th')` expects a table with `id="site-kpis"`, but the table in site-manager.html does not have this ID.  
  **Resolution:** Add `id="site-kpis"` to the table in the HTML, or update the locator to match the actual markup.

- **All other locators** match the structure and attributes present in the src HTML files.

**Recommendation:**  
- Update the table in site-manager.html to include `id="site-kpis"`, or change the locator in the test to `By.css('table th')` if you want to match any table.
- No other changes needed; all other locators are consistent with the application code.

---

> ⚠️ Again, it is just not updating its view (of the application code this time)
>
> ⚠️ It fails to pick up the new id in the table 🧐

# Summary - No
Honestly this turned out much worse than I expected, for the entirely unexpected reason that GPT seems to ignore file changes at random.
As a result, it was a lot of work, considering I am only dealing with a tiny application with a handful of locators.

> ✖️ NOT RECOMMENDED
