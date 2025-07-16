---
title: "Lesson 21 - AI-Acceptance-Test-Behaviour-Driven-Development"
date: 2025-07-02
---
Although I didn't think much of the experience of using AI to generate tests, there is an obvious link between BDD and the current fad for prompt engineering:

> What if the specification _was_ the code?

Now we know BDD is meant to be a dynamic specification (not test) language, so can we use BDD scenarios to prompt AI to build an app?

Let's try it.

### Creating an app from BDD

Say Felicity is a Data Centre Site Manager and she wants an application to help her keep track of her Site's performance.

Outlining that as a BDD feature, we can start to create scenarios, e.g.

```gherkin
Feature: Manage Site
  As a Facility Manager, I want to:
    View my site performance
    View my site projections
    Configure the system
    Manage the Cooling AI
    Manage Cooling Approval
    View Cooling SLAs

  Scenario: Facility Manager sees current KPIs
    Given Felicity has opened their portal
    When they check their KPIs
    Then they see that the KPI data is current
      | KPI | Expected Value |
      | PUE |           1.33 |
      | WUE |           1.56 |
```

Using GPT-4.1 in CoPilot, we will try a series of prompts:
```
Please create a simple website to implement a UI for the "Manage Site" feature.
Put the code in /src folder.
Please create a landing page that links to the "Manage Site" page
On site-manager page, create a link from "View site performance" to a new page showing the "Current KPIs" table as a barchart
Create a link on the performance page back to the site-manager page
Create a link on the site-manager page back to the landing page
```
We get a simple flat folder structure with the pages we asked for and some rudimentary styling:

<img width="209" height="117" alt="image" src="https://github.com/user-attachments/assets/a43a287c-eaff-4102-9b1b-aa98cddded54" />

And the pages have the features we asked for, along with some sensible text (that we didn't have to specify):

<img width="626" height="265" alt="image" src="https://github.com/user-attachments/assets/5806eefb-690b-4096-af4a-a8d2f5458b6b" />

<img width="484" height="616" alt="image" src="https://github.com/user-attachments/assets/4a1ceb94-1427-40e3-ae71-6a300f0fd6fa" />

<img width="715" height="756" alt="image" src="https://github.com/user-attachments/assets/a5c1e2f0-93fa-4a21-819b-18a09ff3d68e" />

For fun, let's change the barchart to a timechart with some fake data:
```
Change the chart on the performance page to a timechart.
Use the current values as the values for today and add monthly values for the last 6 months.
Randomize these values to within 20% of the current values.
Change the WUE line to blue and the PUE line to green
```

<img width="764" height="655" alt="image" src="https://github.com/user-attachments/assets/41d7efad-5eea-41ae-8104-9df9a76f53be" />

That was quite fun and a good way to prototype new features in a testable way. 

❗It forgot to update the page sub-title `Current KPIs as Barchart`

✔️ Obviously not a production-grade application but it is created almost as fast as we can specify it.

### Creating step definitions with BDD
Throwing caution to the wind, let's see GPT test its own work:
```
Using the serenity/js framework, write the step definitions for the "Facility Manager sees current KPIs" scenario
```

The results are ... puzzling:

```Typescript
Given('Felicity has opened their portal', async function () {
    await actorCalled('Felicity').attemptsTo(
        Navigate.to('/src/site-manager.html')
    );
});

When('they check their KPIs', async function () {
    // No action needed, KPIs are visible on page load
});

Then('they see that the KPI data is current', async function () {
    await actorCalled('Felicity').attemptsTo(
        Ensure.that(
            Text.of(PageElement.located(By.css('table tbody tr:first-child td:last-child'))),
            equals('1.33')
        ),
        Ensure.that(
            Text.of(PageElement.located(By.css('table tbody tr:last-child td:last-child'))),
            equals('1.56')
        )
    );
});
```

It's not really using Serenity/JS to its full potential:
- ✖️ putting UI interactions directly into the step definitions
- ✖️ it's assumed that "opened their portal" means go directly to the site-management page, rather than the landing page
- ✖️ "no action needed" would be fair enough - if the last point was true
- ✖️ We really shouldn't be using long, specific xpaths

I guess it's not bad at a first attempt, maybe some more prompting about using the screenplay pattern or hiding the interactions in helper classes would have yielded better results.

### So how would you do it?

```gherkin
// facility-management.steps.ts
Given('{actor} has opened their portal', async (actor: Actor) => 
    actor.attemptsTo(
        User.login()
    )
)

When('{pronoun} check their KPIs', async (actor: Actor) => 
    actor.attemptsTo(
        Navigate.to('/site-manager.html')
    )
)

Then('{pronoun} see(s) that the KPI data is current', async (actor: Actor, data: DataTable) => 
    actor.attemptsTo(
        Data.compareToTable(data, 'KPI')
    )
)
```

```Typescript
// User.ts
export const User = {
    login: () =>
        Task.where(`#actor logs in`,
            Navigate.to('/landing.html'),
        ),
}
```

```Typescript
// Data.ts
export const Data = {

    fromTable: (metric: string) =>
        Question.about(`data in table for ${metric}`, async actor => {
            let rowIndex = (await actor.answer(Table.rowNames())).indexOf(metric);
            return (await actor.answer(Table.values()))[rowIndex * 2 + 1];
        }),
    
    compareToTable: (expectedData: DataTable, metric: string) =>
        Task.where(`#actor compares data to expectations`,
            List.of(expectedData.hashes()).forEach(({actor, item}) => 
                actor.attemptsTo(
                    Ensure.that(
                        item["Expected Value"], 
                        equals(
                            Data.fromTable(item[metric])
                        )
                    )
                )
            )
        ),

}

const Table = {

    rows: () =>
        PageElements.located(By.css('table tr td:nth-child(1)')).describedAs('table rows'),

    rowNames: () =>
        Table.rows().eachMappedTo(Text),

    cells: () =>
        PageElements.located(By.css('table td')).describedAs('table cells'),

    values: () =>
        Table.cells().eachMappedTo(Text),

}
```

And the test passes with all the specified steps:

<img width="732" height="811" alt="image" src="https://github.com/user-attachments/assets/435482ae-6885-48cb-8301-06371188e933" />







