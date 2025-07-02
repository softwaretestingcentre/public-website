---
title: "Observations on AI in Testing"
date: 2025-07-02
---
# Uses of AI in software testing
I have just watched a webinar on using AI for testing applications and I have some concerns.

It was suggested that AI can help testers in a number of areas:
- Creating Test cases
- Setting up Test data
- Configuring Test environments
- Test Results analysis

This seemed like a decent list but unfortunately there was only time to talk about test case creation.

❗ Even with the truncated content, I saw a number of things in the presentation that gave me pause.

# AI writing functional test cases
In the example given, they used an AI plugin in Jira to analyse the (suspiciously detailed) requirements in a Jira ticket and turn them into test cases.
I had some thoughts while this was going on:

| What I saw | What I thought |
|-------|--------|
| The original requirement was a simple behaviour and a list of test inputs/outputs | <ul><li>This is already a well-specified test, what are we gaining from AI?</li><li>QA should have already worked with the requirement author to make sure the specification is testable</li></ul> |
| The AI tool seemed to be churning for a minute | <ul><li>Why does this take any time at all?</li><li>Where is the data going?</li><li>What is the cost/impact of AI doing this while the QA sits idly watching?</li></ul> |
| A list of test cases is produced | <ul><li>How are QAs meant to learn the **craft** of writing tests?</li><li>Are we really saving any time at all when QA will need to review every test?</li><li>No test framework will scale if every test is created individually</li></ul> |
| The tests include a lot of UI element visibility checks | It's painful to see system-level tests that just check the structure of the UI | 

# AI writing non-functional test cases
Another example demonstrated how AI could detect Accessibility issues in the application.
- This is being done far too late in the development process
- Set your non-functional standards up front and enforce them in the codebase using static checkers and peer review.
- Performance, Accessibility, etc should be designed-in, not bolted on after failing tests.

# Why AI isn't helping
My impression was that AI was being used to support a reactive process where QA had no input to the requirements specification or development standards.
This is a **process** issue, not one to be solved by complex tooling. 

❗If AI is just covering up problems - and actively making them harder to fix - the damage may be irreversible.

Do you need to use AI at all or do you need to:
- Design tests _in conjunction with_ the requirements?
- Use templates/checklists to design tests for **your** specific coverage needs?
- Use checks **during development** to cover non-functional requirements?

# Where I would like to see AI helping
✔️ There's some useful work being done with AI helping to "heal" broken tests at runtime by detecting UI elements that have updated in some way, for instance.

⚠️ I would still tread cautiously around this, however, as some changes might be accidental and/or have side-effects that need to be discussed **before** the application code is changed.

⚠️ There is also a danger that when the test framework is subject to changes not coming from human input, the amount of noise in your VCS could overwhelm your capacity to review changes in enough detail.

✔️ What I think AI could be good at is spotting _trends_ in historic test results:
- Is test execution performance improving or degrading?
- Are tests becoming more or less flaky?
- What are the problem areas in the application - or in the test framework?
