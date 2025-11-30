---
title: "Has AI made us faster?"
date: 2025-11-30
draft: false
summary: "I was recently asked whether AI had helped the team deliver a significant piece of work faster. The 2025 DORA State of AI-Assisted Software Development report has helped me to identify a set of questions thet assess the real impact of AI on delivery performance."
---

I've been using AI assistance in my day to day engineering work since June this year. 

After delivering a significant piece of work, I was asked whether AI had helped us deliver it faster. I struggled to answer the question as I had no reliable way of measuring whether it had actually made a difference.

It wasn’t until I came across the 2025 DORA State of AI-Assisted Software Development report that I gained clarity over what’s required to achieve meaningful productivity gains when adopting AI. The report highlighted that AI’s impact can only be measured by taking into account the capabilities of an organisations underlying system rather than by intuition alone:

> AI’s primary role in software development is that of an amplifier. It magnifies the strengths of high performing organizations and the dysfunctions of struggling ones.
> 

## What does it mean to be a high performing organisation?

The report assesses software delivery performance using two key factors:

- Throughput - how many changes can move through the system over a period of time. Measured by:
    - Lead time for changes
    - Deployment frequency
    - Failed deployment recovery time
- Instability - how consistently and reliably software deployments succeed. Measured by:
    - Change failure rate
    - Rework rate

Broadly speaking, high throughput and low instability are indicators of high performance. However, the authors are keen to point out that these outcomes only describe *what* is happening, not *why*.

> A low deployment frequency might be caused by technical debt, bureaucratic processes, or team burnout—and the metrics alone can’t distinguish between them.
> 

In order to address this, the report also incorporates a set of human experience factors:

- Team performance - perceived effectiveness and collaboration strength of a team
- Product performance - helping users accomplish important tasks plus performance metrics such as latency
- Individual effectiveness - effectiveness and sense of accomplishment of an individual
- Valuable work - time spent doing work an individual feels is valuable and worthwhile
- Friction - the extent to which friction hinders an individuals work
- Burnout - feelings of exhaustion and cynicism related to one’s work

High performers emerge as those experiencing **low friction, low burnout, and low instability**, while achieving **high team performance, high product performance, strong throughput, individual effectiveness, and a high proportion of valuable work**.

## The impact of AI on these factors

The findings show that AI is having a positive impact on many of these factors: product and team performance, individual effectiveness, throughput and time spent doing valuable work are all shown to increase based on adoption of AI. These improvements are attributed to increasing tool maturity and a growing understanding across teams of how to apply AI effectively.

But perhaps most critically, the report highlights an increase in instability. This suggests that wider organisational systems are failing to adapt to the paradigm shift towards AI. For example, lack of a strong software delivery pipeline and high dependency between teams will still lead to instability, regardless of the use of AI. This is where the importance of addressing the wider system is emphasised:

> We believe that the value of AI is not going to be unlocked by the technology itself, but by reimagining the system of work it inhabits.
> 

## The capabilities that enable the benefits of AI adoption

The report introduces the first DORA AI Capabilities Model, a set of seven capabilities that have shown through substantial evidence to amplify the benefits of AI adoption.

### Clear and communicated AI stance

The report finds that unclear AI guidance leads developers to either underuse AI or use it in unsafe or unintended ways. In order to provide a clear stance, organisations should encourage and expect its developers to use AI, support experimentation and be explicit around which tools are permitted.

One important point to note is that this capability measures the clarity and awareness of the AI stance, not the content itself. Organisations should be free to determine what is appropriate for them based on their own unique context.

### Healthy data ecosystems

This capability relates to ensuring AI models are trained on good internal data. A healthy data ecosystem is one where internal data is of a high-quality, accessible and unified.

### AI-accessible internal data

AI tools that are trained on a general set of knowledge can help developers feel more productive but the findings of the report show that the impact can be increased when AI tools are connected to internal data sources that provide company-specific context.

For example, if the engineering team have ADRs defined in an external source, then giving the AI tools context of these will likely yield positive benefits.

### Strong version control practices

Strong version control practices is a long-standing enabler of high-performing teams. The report cites the ability to commit frequently and to rollback changes as the key capabilities here.

AI increases the likelihood of generating greater volumes and frequency of code changes, amplifying the importance of strong version control practices. This can have a direct impact on software instability due to the increased difficulty in reviewing larger batches of code.

Whilst a high volume of code rollbacks is probably not a measure of high quality, it can help to reduce the instability caused by merging large batches of code.

### Working in small batches

This measures the degree to which teams break down changes into manageable units that can be quickly tested and evaluated and is one of DORAs long-time capabilities.

> A team that scores more highly in terms of working in small batches is one that commits fewer lines of code per change, fewer changes per release, and assigns work that can be completed in a shorter amount of time.
> 

Working in this way can help to reduce the likelihood of resorting to code rollbacks, reduce cognitive load on developers and shortens the feedback loop of usage in production.

### User-centric focus

Teams that prioritise user experience, focus on creating value for users and understand how this connects to business success are more likely to find clarity around their goals and orient towards a shared strategy.

I know from my own experience that speaking directly to users and building empathy with their situation leads to a greater chance of solving their actual problems.

The influence of AI is found to be positive for teams that adopt a user-centric focus and negative for teams that do not. Even if AI speeds up development time, solving the wrong problem faster is ultimately harmful to the business.

### Quality internal platforms

> “platforms” refer to a set of capabilities that is shared across multiple applications or services, directed at making these capabilities widely available across the organization.
> 

Having quality internal platforms helps to provide necessary guardrails and shared capabilities for engineering teams, leading to a greater developer experience. Without these guardrails, the use of AI is likely to lead to the use of undesirable capabilities and inconsistency of approaches.

The authors conclude their findings related to these capabilities by stating that simply adopting new tools is not enough:

> successfully leveraging AI in software development is not as simple as just adopting new tools. Rather, organisations must cultivate a specific technical and cultural environment to reap the greatest rewards.
> 

## Value stream management

The report also highlights the importance of a teams ability to understand their workflow and to identify and eliminate constraints as a key enabler in moving from disorganised activity to focused improvement, particularly in the age of AI.

> value stream management (VSM) is the force multiplier that turns AI investment into a competitive advantage, ensuring that this powerful new technology solves the right problems instead of just creating more chaos.
> 

Value stream management focuses on visualising and improving the flow of work from idea to customer. This leads to a shared understanding and rich conversations about where things are currently working well and where they are not. Bottlenecks and inefficiencies can be identified and teams can start to make strides towards eliminating these.

Oftentimes, the things that are identified are not immediately visible or obvious and would have been left to fester over time.

Solely focusing on creating efficiency gains through the use of AI in writing code can lead to localised gains that are absorbed by upstream or downstream bottlenecks. By mapping the entire flow, teams can understand whether they are actually reaping any overall benefit from these improvements.

## So, has AI made us faster?

My takeaway from the DORA report is that **AI only accelerates teams when the environment around it is ready**. Without that foundation, AI can amplify instability through poor or inaccessible training data, large batches of code, misaligned problem-solving, or inconsistent practices. And even when local efficiency improves, upstream or downstream bottlenecks can absorb those gains.

So the next time I’m asked whether AI sped things up, I’ll be assessing my answer against a set of questions that get to the heart of whether the underlying system was primed for AI to have a meaningful effect:

- Did we have clear, shared guidance on how to use AI?
- Did AI have access to the right context and high-quality internal data?
- Were our engineering practices designed for small, safe, frequent changes?
- Were we focused on delivering genuine user value?
- Were robust internal platforms and guardrails in place?
- And did our value stream support efficient flow from idea to production?

With these things in place, I can have greater confidence in saying that AI made us faster.