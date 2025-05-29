---
title: "Lesson 7 - Questions, Questions, Questions"
date: 2025-05-29
---
> What's the most powerful language feature for Automated Testing?
>
> The _Question_

In the previous 6 lessons we have forked and deployed a simple [React App](https://github.com/softwaretestingcentre/job-portal) and built a [test framework](https://github.com/softwaretestingcentre/test-job-portal) for it to demonstrate how BDD techniques lead to testable requirements and relevant system tests.

The power of this process does not come from the automation, from Gherkin or from Java, but from our ability and willingness to ask _questions_.

These questions may at any one time be regarded as:
- Incisive
- Intrusive
- Irrelevant
- Idiotic

But we should not shy away from asking them. 

> ❗Bugs hide behind unasked questions.

> ℹ️ QA == "Question Asking"

We saw in [Lesson 4 Journey Mapping](/public-website/2025/05/27/Lesson-4-Journey-Mapping.html) how Journey Mapping helps us to break down requirements so that they are easier to define and dig into for further detail. You may have your own way of doing this process but it should always be a structured conversation, where the results are recorded and categorised.

Journey Mapping has the advantage of translating directly and easily into readable BDD scenarios, from which we can build automated tests. The table structure shown in the examples is a good start, but it is not the only way to achieve this. If you are more of a visual or relational thinker, you may prefer to use a mind map for requirements discovery. 

> ✔️ Choose and use whatever tools work for you and your team.

We also saw that we can't always capture **all** the requirements - or **all the detail of** the requirements - in a single session. We may only discover nuances and edge-cases after starting to write tests. This is OK, it is an iterative process - we just need to make sure that non-QA resources have the capacity to answer further questions when they come up.

# Going further
So far we have used the JobHunt React app to learn about functional testing and BDD. My fork of this is missing major areas of functionality. Feel free to take a fork of this app and the test framework if you want to practise writing BDD tests and learn the Serenity BDD framework.

For the following lessons, I am going to use an application that has distinct front-end, back-end and database components to look into API, Performance and Security testing.
