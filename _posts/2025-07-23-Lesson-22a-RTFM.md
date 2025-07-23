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
