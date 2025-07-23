---
title: "Lesson 22a - Using AI PROPERLY to repair web test locators"
date: 2025-07-23
---

In my previous post I was bemoaning the fact that GPT wasn't picking up some of my code changes when looking for discrepancies in locators defined in test objects compared to those in the application source code.

After a bit of [RTFM](https://code.visualstudio.com/docs/copilot/chat/copilot-chat-context#_add-files-as-context) and using the `#codebase` hashtag, I am happy to say that it is working much better:

e.g. I've done a bit of refactoring of the `Table` test object to provide a specific id:

```typescript
const Table = {

    kpi_table: 'table#site-kpis',

    columnHeaders: () =>
        PageElements.located(By.css(`${Table.kpi_table} th`)).describedAs('column headers'),

    columnNames: () =>
        Table.columnHeaders().eachMappedTo(Text),

    rows: () =>
        PageElements.located(By.css(`${Table.kpi_table} tr td:nth-child(1)`)).describedAs('table rows'),

    rowNames: () =>
        Table.rows().eachMappedTo(Text),

    cells: () =>
        PageElements.located(By.css(`${Table.kpi_table} td`)).describedAs('table cells'),

    values: () =>
        Table.cells().eachMappedTo(Text),

}
```

and I made a (deliberate) typo to ensure that they won't match the `site-manager.html` page:
```html
      <table id="site-kpi">
        <thead>
          <tr><th>KPI</th><th>Expected Value</th></tr>
        </thead>
        <tbody>
          <tr><td>PUE</td><td>1.33</td></tr>
          <tr><td>WUE</td><td>1.56</td></tr>
        </tbody>
      </table>
```

Now when I ask GPT to compare the tests to the app, will it find this discrepancy?

### Compare all the locators in the #codebase  test folder to any defined or implied by the application code under the #codebase src folder. Highlight any discrepancies.

### Summary of Relevant Code

- **Test locators for table:** `table#site-kpis`, `th`, `tr td:nth-child(1)`, `td`
- **Test locators for tabs/sections:** `[data-tab="..."]`, `section[id="..."]`
- **Application code for table:** `table#site-kpi`, `th`, `td`
- **Application code for tabs/sections:** `button[data-tab="..."]`, `section[id="..."]`

---

This is the complete set of code relevant to the locator comparison request. If you need a detailed discrepancy analysis, let me know!

Here is a direct comparison of all locators in the test folder (Data.ts, Explainer.ts) to those defined or implied by the application code in the src folder (site-manager.html, explainer.html, etc.), with discrepancies highlighted:

---

## Table Locators

**Test code:**
- `PageElements.located(By.css('table#site-kpis th'))`
- `PageElements.located(By.css('table#site-kpis tr td:nth-child(1)'))`
- `PageElements.located(By.css('table#site-kpis td'))`

**Application code (site-manager.html):**
- Table is defined as `<table id="site-kpi">` (not `site-kpis`)
- Table rows and cells structure matches the test selectors

**Discrepancy:**  
- **Test code uses `table#site-kpis`, but application code uses `table#site-kpi`.**  
  The test locator will not match the table in the application.  
  **Correction needed:** Change all test selectors from `site-kpis` to `site-kpi`.

---

## Tab and Section Locators

**Test code (Explainer.ts):**
- `PageElement.located(By.css('[data-tab="${sectionName}"]'))` (tab button)
- `PageElement.located(By.css('section[id="${sectionName}"]'))` (tab content section)

**Application code (explainer.html):**
- Tab buttons: `<button class="tab-btn" data-tab="...">`
- Tab content sections: `<section id="..." class="tab-content">`

**Discrepancy:**  
- **None.** Test selectors match the application code.

---

## Other Elements

- **Advice text:**  
  Test code does not use a locator for `.advice-text`, but this is not required for current step definitions.
- **Charts:**  
  No test locators for `#slaChart` or `#slaPoints`, which is consistent with the current feature requirements.

---

## Summary Table

| Locator in Test Code                | Locator in Application Code      | Discrepancy?         |
|-------------------------------------|----------------------------------|----------------------|
| `table#site-kpis`                   | `table#site-kpi`                 | **Yes** (ID mismatch)|
| `table#site-kpis tr td:nth-child(1)`| `<tr><td>...</td></tr>`          | Yes (ID mismatch)    |
| `table#site-kpis td`                | `<td>...</td>`                   | Yes (ID mismatch)    |
| `[data-tab="..."]`                  | `<button data-tab="...">`        | No                   |
| `section[id="..."]`                 | `<section id="...">`             | No                   |

---

**Discrepancies found:**
1. Table ID mismatch (`site-kpis` vs `site-kpi`) in all table-related locators.

**Action required:**  
Update all test selectors using `table#site-kpis` to `table#site-kpi` for consistency with the application code. All other locators are correct.


# Summary - Yes
So when you use GPT properly for codebase analysis, it **is** useful.

It's also faster and more thorough than manual analysis, but I do wonder if it would scale - I already blew up my free account just getting this far.

# AI - Redemption

To determine this, I ran some analysis on a production test framework that I was asked to review a few months back.

This framework consists of 20K LOC and includes over 500 Selenium selector statements.

As part of my analysis, I spent _several days_ painstakingly going through the framework looking for duplicate and/or poorly formed selectors.

GPT did the same analysis _in under a minute_:

### softwaretestingcentre: Find any selenium locators in the #codebase src folder. Highlight any duplicates or badly formed selectors.

### GitHub Copilot: I searched for Selenium locators in the src folder, specifically usages of By.id, By.xpath, By.cssSelector, By.className, By.name, By.linkText, and By.partialLinkText. Here are the findings:

### Selenium Locator Usages

#### By.xpath (most common)
- Many usages in:
  - `MDDLogsTasks.java`
  - `ThingCreationPage.java`
  - `ThingDirectoryPage.java`
  - `OtherThings_ProducerPage.java`
  - `AdminPage.java`
  - `ThingManagementPage.java`
  - `TasksPage.java`
- Example patterns:
  - `"//*[contains(text(),'" + fieldText + "')]/../../following-sibling::*/*/*[2]/*[1]"`
  - `"//span[contains(text(),'" + rsa_values.get(0) + "')]/../../..//div[@class=\"c-ThingCreationTable__table-card-top-right\"]//button//span[contains(text(),\"Approve Thing ID\")]"`
  - `"//td[normalize-space()='" + row.get("Thing ID") + "']"`
  - `"//a[normalize-space()='" + Thing.getAssignedThingID() + "']"`
  - `"//h1[normalize-space()='RAG Status:']/following-sibling::button"`
  - `"//*[normalize-space()='" + ThingID + "'][@data-field='Things']"`

#### By.cssSelector
- Found in:
  - `ThingManagementPage.java`
- Example:
  - `By.cssSelector("[data-testid='" + elementLocator.replaceAll("\\$", "") + "']")`
  - `allocationRoundSelector.findElements(By.cssSelector("option"))`

#### By.name
- Found in:
  - `LoginPage.java`
- Example:
  - `By.name("action")`

### Duplicates

- There are repeated patterns for finding elements by Thing ID, name, or other dynamic values, especially in `OtherThings_ProducerPage.java` and `TasksPage.java`:
  - `"//td[normalize-space()='" + row.get("Thing ID") + "']"` and similar for other fields.
  - `"//*[normalize-space()='" + ThingID + "'][@data-field='Things']"` appears multiple times for different Thing types.

### Badly Formed Selectors

- Many selectors use deeply nested and chained XPaths, e.g.:
  - `"//*[contains(text(),'" + fieldText + "')]/../../following-sibling::*/*/*[2]/*[1]"`
  - These are fragile and likely to break if the DOM structure changes.
- Some selectors concatenate dynamic values directly into the XPath, which can lead to injection issues or errors if the value contains special characters.
- Selectors like `"//*[contains(text(),'" + fieldText + "')]/../../../following-sibling::*/*/*[1]/*"` are overly complex and hard to maintain.

### Summary

- **Duplicates:** Many selectors for Thing IDs, names, and similar fields are repeated across files.
- **Badly Formed:** Deeply nested XPaths, direct string concatenation, and overly complex selectors are common.
- **Files with most issues:** `MDDLogsTasks.java`, `OtherThings_ProducerPage.java`, `ThingManagementPage.java` and `TasksPage.java`.

If you need the exact code snippets or want to review specific files for refactoring, let me know which files or patterns you'd like to focus on.Here is a summary of Selenium locators found in your pages classes, with notes on duplicates and potential issues:

---

### 1. Duplicates

#### By.xpath("//a[normalize-space()='" + ... + "']")
- Found in:
  - `AdminPage.java`: `selectThing(Thing Thing)`
  - `UserManagementPage.java`: Used for Thing ID verification (hardcoded value: "HAR1-AAA-020")
- **Note:** These are similar but not identical. Both select an anchor by its normalized text, but one is parameterized and one is hardcoded. If the Thing ID is dynamic, hardcoding may cause fragility.

#### By.xpath("//span[normalize-space()='" + ... + "']")
- Found in:
  - `ThingDirectoryPage.java`: Uses Thing name from `ViewmoreCCPPage.Thing_Name`
- **Note:** This pattern is reused in other files for different purposes. If the Thing name is not unique on the page, this could select the wrong element.

#### By.xpath("//td[normalize-space()='" + ... + "']")
- Found in:
  - `OtherThings_ProducerPage.java`: Used for verifying producer details
- **Note:** Used for multiple fields (Thing ID, Name, etc.). If table cells are not unique, this could cause false positives.

---

### 2. Badly Formed Selectors

#### By.xpath("//input[@type='submit'")
- Found in:
  - `AdminPage.java`: In `find("input[type='submit'").click();`
- **Issue:** Missing closing bracket for the XPath predicate. Should be .

#### By.xpath("//span[contains(text(),'" + ... + "')]/../../..//div[@class=\"c-ThingCreationTable__table-card-top-right\"]//button//span[contains(text(),\"Approve Thing ID\")]")
- Found in:
  - `ThingCreationPage.java`: Used for RSA, DPA, LCHA approval buttons
- **Issue:** This is a very long and brittle XPath. If the DOM structure changes, this will break. Consider using a more robust selector or adding unique attributes.

#### By.xpath("//*[contains(text(),'" + fieldText + "')]/../../following-sibling::*/*/*[2]/*[1]")
- Found in:
  - `MDDLogsTasks.java`: Used for editing fields
- **Issue:** Relies on a deep and specific DOM structure. If the hierarchy changes, this will break.

#### By.xpath("//*[normalize-space()='" + ThingID + "'][@data-field='Things']")
- Found in:
  - `TasksPage.java`: Used for selecting a Thing
- **Issue:** If multiple elements have the same Thing ID and data-field, this could select the wrong one.

---

### 3. Other Observations

- Many selectors use normalized text, which is good for whitespace robustness, but if the text is not unique, it can cause issues.
- Some selectors are hardcoded with specific values (e.g., Thing IDs), which reduces flexibility and maintainability.
- Deeply nested XPaths (with many .. or `/following-sibling::*`) are fragile and should be avoided if possible.

---

### Recommendations

- Review selectors for uniqueness and robustness.
- Avoid hardcoding values unless necessary.
- Prefer CSS selectors or shorter XPaths when possible.
- Add unique attributes (e.g., `data-testid`) to elements in the application for easier selection.

If you want a full list of all selectors or want to refactor any, let me know!
