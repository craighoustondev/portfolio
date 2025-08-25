---
title: "How testing in production helps truly solve your customers problems"
date: 2025-08-23T22:30:00+01:00
draft: true
---

Ever shipped a feature that didn’t solve your customers problem or struggled to measure whether it made any difference at all?

In this post, I’ll share how testing in production and working closely with users has helped me close that gap and deliver changes that truly work for customers.

## What does it mean to test in production?

<div style="display:flex; justify-content:center;">
    {{< figure src="i-dont-always-test.jpg" width="400">}}
</div>

I first came across the concept of testing in production from a Principal Engineer I worked with when I was a junior developer. It initially filled me with a lot of fears. What if I mess up some real-user data or settings? What if I bring the entire system down? I’ve already tested in a staging environment so why do I need to test in production? It felt at odds with everything I’d been told about keeping production locked down to end users and admins, with testing done in a “safe” environment. I refrained from doing this at the time as it seemed too risky.

Since then, I've discovered my concerns were misplaced as I had completely misunderstood both the risks involved and the value that testing in production delivers.

Drawing on my experience, I’ve identified three core aspects of what “testing in production”  means to me, which I’ll outline in more detail below.

## Explore the production system

Charity Majors, cofounder and CTO of [honeycomb.io](http://honeycomb.io/), is a leading voice on this topic. She has often emphasised that it is impossible to spin up production clones of large and complex systems and to accurately simulate real traffic patterns. Testing in production is the only way you will ever understand the real impact of the changes you deploy.

In recent years I have done a lot of ensemble testing of features in production. To do this, we gather together people from different parts of the business such as customer support, customer success, engineering, product and design and explore production using a tenant reserved for internal use. 

In advance of the session, we get everyone involved to add the goals we want to achieve to a Miro board. Having a diverse array of perspectives, knowledge and experience involved in this reduces the likelihood of missing something important.

We assign roles for the session, including a driver to share screen and interact with the system, a navigator to guide the actions of the driver and a note taker to map out the journey and results of the session. The others might be simply observing or monitoring logs and unhandled errors.

We routinely use feature flags during build and only enable it for our internal tenant for the purposes of the testing session.

The outcome of the session is that we have either built enough confidence that things are working as expected and without undue friction or that there is some additional work that we need to do to get to that point. This helps us to determine the rollout plan and at what point we enable the feature for users.

Once we’ve built confidence internally, the real learning comes from watching what happens in production.

## Observe and monitor real-user experience

PostHog notes in their blog [*How to safely test in production](https://posthog.com/product-engineers/testing-in-production)* that observability and monitoring are crucial for testing in production. Metrics, error tracking, and real-user analytics give teams the visibility to catch issues fast and see how features perform in the real world.

In my experience, tools such as Datadog, Sentry and Metabase have proven to be invaluable for this purpose.

In a recent piece of work, we had to take the output from a third party and map that to a format that was meaningful for our domain and consumable by our UI. We knew that the data we received was sometimes incomplete or did not contain the keys we needed. We could have agonised over all the potential permutations and spent a long time covering these in our tests and code. Instead, we shipped what we considered would cover the majority of cases and created a page in Datadog that monitored the output every time we run our mapping code. We regularly checked this page to help us uncover the error conditions and iteratively build out our logic. This allowed us to deliver value to our customers at an early stage knowing that we could proactively fix the few instances that resulted in errors.

At the start of each piece of work we define a measurable aspirational goal. Dependent on the goal, we can use data from our production database to measure this over time. In one instance, we wanted to increase usage of a particular part of our product, which we could measure by the the number of user actions performed within 7 days of that action being assigned to the user. 

Before we started the work we could see that around 40% of actions were performed within 7 days. We set a goal of getting this to 70% and were able to create a query in Metabase to track this over time.

{{< figure src="goal-line.jpg">}}

In this particular case, we released a first iteration and were able to see that we were trending in the right direction (close to 50%) but it quickly became obvious to us that this change alone probably would not get us close to the goal. This helped us to understand that we needed to iterate and deliver more positive changes.

Monitoring tells you what’s happening, but it doesn’t always explain why. That’s where direct conversations with users come in.

## Listen to your users

“A modern software engineer’s job is not done until they have watched users use their code in production.” ([Charity Majors](https://increment.com/testing/i-test-in-production/))

In the last year, I have had lots of conversations with users at various points of delivery. We speak to customers before we have even written a single line of code to help us to understand their problem and what their current workflow looks like. 

We follow up during the build in order to get feedback on each iteration. Sometimes, we realise at an early stage that we have done enough to ease their pain and they are happy to get early access while we iterate. 

Once they have started using the new feature, we speak to them again to understand exactly how they are using it and what we need to do to make it even more valuable.

If they feel comfortable, I like to get users to share screen and describe exactly how they use the system in their day-to-day work. It’s important when doing this to let them do the talking and to actively listen to what they are saying. This can uncover surprising insights as often their workflow doesn’t match what was expected.

In one example, we received feedback from another part of the business that users were struggling to complete certain actions due to usability issues. It was suggested that performing this action was actually unnecessary and we should remove the button that led them towards the troublesome UX. When we observed our users behaviour it became clear that this step was necessary and that removing the button would actually cause more issues than it would solve. This invalidated our early assumptions and forced us to rethink how we solve the problem.

## From Fear to Confidence

Testing in production isn’t about being reckless — it’s about embracing reality. By combining exploration, observability, monitoring, and conversations with users, you move past assumptions and start solving the real problems your customers face. That shift turns delivery from a gamble into a learning loop, where every release makes your product stronger.