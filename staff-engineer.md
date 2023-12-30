# Staff engineer: leadership beyond the management track
By Will Larson

My highlights and notes from the book. For full content go to https://staffeng.com/

Another summary is available at: https://github.com/mgp/book-notes/blob/master/staff-engineer.markdown

- [Staff engineer: leadership beyond the management track](#staff-engineer--leadership-beyond-the-management-track)
  * [Types of staff roles:](#types-of-staff-roles-)
    + [What do Staff engineers actually do?](#what-do-staff-engineers-actually-do-)
      - [Setting technical direction](#setting-technical-direction)
      - [Mentorship and sponsorship](#mentorship-and-sponsorship)
      - [Providing engineering perspective](#providing-engineering-perspective)
      - [Exploration](#exploration)
      - [Being Glue](#being-glue)
    + [Does the title even matter?](#does-the-title-even-matter-)
  * [Operating at Staff](#operating-at-staff)
    + [Work on what matters](#work-on-what-matters)
    + [Writing engineering strategy](#writing-engineering-strategy)
      - [Write five design docs](#write-five-design-docs)
        * [Designs docs at google](#designs-docs-at-google)
      - [Synthesize those five design docs into a strategy](#synthesize-those-five-design-docs-into-a-strategy)
      - [Extrapolate five strategies into a vision](#extrapolate-five-strategies-into-a-vision)
    + [Managing technical quality](#managing-technical-quality)
      - [Ascending the staircase](#ascending-the-staircase)
      - [Hot spots](#hot-spots)
        * [How to work the policy](#how-to-work-the-policy)
      - [Best practices](#best-practices)
      - [Quality leverage points](#quality-leverage-points)
      - [Technical vectors](#technical-vectors)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

## Types of staff roles:
- **tech lead**:
Comfortable scoping complex tasks, coordinating their team towards solving them, and unblocking them along the way. Tech Leads often carry the team’s context and maintain many of the essential cross-team and cross-functional relationships necessary for the team’s success. They’re a close partner to the team’s product manager and the first person called when the roadmap needs to be shuffled.

Earlier in their career, they will have implemented their team’s most complex technical projects, but at this point, they default to delegating such projects across the team. They do this both to grow their teammates and in acknowledgment that the team’s impact grows as the Tech Lead’s coding blocks shrink. While they’re coding less, they are still the person defining their team’s technical vision, and stepping in to build alignment within the team on complex issues.

- **architect**:
Architects are responsible for the success of a specific technical domain within their company, for example, the company’s API design, frontend stack, storage strategy, or cloud infrastructure. 

- **solver**: 
The Solver is a trusted agent of the organization who goes deep into knotty problems.
Deployed by organizational leadership as critical and either lacking a clear approach or with a high degree of execution risk.
They generally stop working on problems once they’re contained. The risk: transience, and infuriating the teams left behind to maintain the “solved” problem.

- **right hand**: attend their leader’s staff meetings and work to scale that leader’s impact by removing important problems from their plate. Problems addressed at this level are never purely technical and instead involve the intersection of the business, technology, people, culture, and process.


### What do Staff engineers actually do?

#### Setting technical direction
In earlier roles, you may have tried to influence decisions towards technology choices you were motivated by; in senior positions, you’re accountable to the business and organization first and yourself second.


#### Mentorship and sponsorship
You’re far more likely to change your company’s long-term trajectory by growing the engineers around you than through personal heroics. The best way to grow those around you is by creating an active practice of mentorship and sponsorship

The most effective Staff engineers pair a moderate amount of mentorship with considerably more sponsorship.

#### Providing engineering perspective

In discussions that occur at a level above individual projects and teams, for problems that span teams which are both technical and non-technical in nature. - Dan NaT

To inject an engineering perspective where it would otherwise be missed. Just remember that you’re representing the interests of all of engineering, not just your own.

#### Exploration

In the long-term, companies either learn to explore, or they fade away. Simply assigning a team that’s mastered hill-climbing (ie, going from local maxima/solution to the next above one) to do exploratory work is far from a sure thing, so many companies take a different approach

This isn’t always a business problem either; it can be any ambiguous, important problem that the company’s systems are ill-shaped to address (reduce cloud costs, DB space about to be exhausted w/out ability to increase, etc)

#### Being Glue

Doing the needed, but often invisible, tasks to keep the team moving forward and shipping its work. Expediting the most important work and ensuring it gets finished.

### Does the title even matter?

- Less energy to spend proving credentials => more energy available to make impact
- Be invited in discussions
- Compensation
- Not magic: if you have a problem and believe that your title is the only thing holding you back: recommend focusing on developing your approach and skills: it will be far more impactful than the title. The title will get you over the ledge once you’re close, but it’ll never do as much work as you’d expect.


## Operating at Staff
### Work on what matters
You can deliberately accumulate expertise from doing valuable work. 
Long term bet: focus on work that matters, do projects that develop you, and steer towards companies that value genuine experience.

- avoid low effort/low impact tasks: psychology rewarding, but low impact.
It’s ok to spend some of your time on snacks to keep yourself motivated between bigger accomplishments.
If you’re not deliberately tracking your work, it’s easy to catch yourself doing little to no high-impact work.

- stop preening: you’ll get held accountable for the lack of true impact where others who match the company’s expectation of how a leader appears will somehow slip upward.

- work where there is both room for improvement and attention from the company:

If a topic has no attention from the company: initial work can give decent results, but over time it will decay from support, etc => low impact

As a senior leader, you have an ethical obligation that goes beyond maximizing your company-perceived impact, but it’s important to recognize what you’re up against and time your efforts accordingly.

- Foster growth: lots of resources are invested in hiring. We want to also invest in onboarding, mentoring, and coaching, since it's impactful as well.

- Edits: A surprising number of projects are one small change away from succeeding. Help people succeed

- Finish things: going from unfinished (risk) to finished (leverage)
Eg, help less senior people create buy-in or figure out how to rescope their project into finishable work with a similar impact in one 10th the time.

- What only you can: what wouldn't happen if you don't do it. An intersection of what you’re exceptionally good at and what you genuinely care about. 

eg recruit a key hire, drafting the tech strategy of the company, https://lethain.com/magnitudes-of-exploration/

### Writing engineering strategy

- To write an engineering strategy, write five design documents, and remove the similarities.

- To write an engineering vision, write five engineering strategies, and forecast their implications two years into the future. That’s your engineering vision.


If you realize that you’ve rehashed the same discussion three or four times, it’s time to write a strategy. 
 
When the future’s too hazy to identify investments worth making, it’s time to write another vision.

#### Write five design docs

Design documents describe the decisions and tradeoffs you’ve made in specific projects. Similar to _Architecture Decision Records_

##### Designs docs at google
https://www.industrialempathy.com/posts/design-docs-at-google/:

When not to write a design doc: if the solution is obvious, if there are no alternatives with different trade offs.
 > the vast majority of design docs at Google are created in Google Docs and make heavy use of its collaboration features.

 > Discussion then primarily happens in comment threads in the document.

 > In formal design review meetings, the author presents the doc to a senior engineering audience. 
 Waiting for such meetings to happen can significantly slow down the development process. Mitigate this by seeking the most crucial feedback directly and not blocking progress on wider review.

> When Google was a smaller company, it was customary to send designs to a single central mailing list, where senior engineers would review them at their own leisure. 

Such reviews:
- combine experience of the organization to be incorporated into a design.
- ensure cross-cutting concerns such as observability, security and privacy are accounted for.
- bring feedback early enough in the development lifecycle that it is still relatively cheap to make changes.

#### Synthesize those five design docs into a strategy

Be specific, be opionated.

Good strategies guide tradeoffs and explain the rationale behind that guidance. So it can be adapted to context shifts.

Write alone, review with a group

Ex strategies:
- “When should we write design documents?” 
- “Which databases do we use for which use cases?”
- “How should we stage our migration from monolith to services?”


#### Extrapolate five strategies into a vision

- Write two to three years out
- Ground in your business and your users.
- Be optimistic rather than audacious. 
- Stay concrete and specific. 
- Keep it one to two pages long. 

### Managing technical quality

Technical quality is a long-term game. There’s no such thing as winning, only learning and earning the chance to keep playing.

- *fix the hot spots* that are causing immediate problems
- *adopt best practices* that are known to improve quality
- *prioritize leverage* points that preserve quality as your software changes
- *align technical vectors* in how your organization changes software
- *measure technical quality* to guide deeper investment
- *spin up a technical quality team* to create systems and tools for quality
- *run a quality program to measure*, track and create accountability

#### Ascending the staircase

Start with the easy solutions first (ex: code linting), then progress to more complex solutions.

Goal is to iterate fast, to learn.

Don't add process prematurely: it adds friction.

Be open to try changes, and be prepared to drop if ineffective.

#### Hot spots

At first, identify the hot spot, measure, and focus on where the bulk of the issues happen.

Don't add to process lightly. Process is expensive: it changes how people work.

If problems are created faster in the org that you can fix them, then adopt best practices.

##### How to work the policy
Related: [How to work the policy, and not exceptions](https://lethain.com/work-policy-not-exceptions/)

Policy is supported via constraints. To support it, you have to hold these.

Don't revise the policy at each escalation: at a sufficiently high rate of change, policy is indistinguishable from exception.

Instead: batch escalations, and revise the policy at a future time.

Some challenges:
- Accepting the reduced opportunity space
- Locally suboptimal: eg a team may have to work under challenging circumstances to support a broader goal. The team gets limited benefit from it, and you have to stick with the decision at real personal cost for the folks impacted.

#### Best practices

Adopting best practices requires a level of organizational and leadership maturity that takes some time to develop.

Good process is [evolved](https://lethain.com/good-process-is-evolved/) rather than designed: don't rush it, don't expect it to be finalized on day 1:


- Study how other companies adopt similar practices. 
- Document the approach.
- Start with a few engaged teams.
- Improve the doc based on challenges.
- Roll out further

Limit concurrent best practices rollouts: 
- too much: it dilutes attention and priority
- it is better to adjust few variables independently to understand their effect. Useful if you need to rollback later
- if you still need to add more: move to leverage points

#### Quality leverage points

Leverage points can be done w/out total organizational alignment.

Points where extra investment preserves quality over time by
- preventing gross quality failures
- reducing the cost of future quality investments

The 3 most impactful points:

- interfaces: Durable interfaces expose all the underlying essential complexity and none of the underlying accidental complexity.

- state: has inherent inertia, and gets more complex with more concerns of security, privacy, compliance, etc

- data models: a good data model is rigid (ie, can't express unsupported states), and supports evolution.

Deliberately challenge these:
- interface: integrate 5 clients against a mock implementation  
- state: exercise the failure modes
- data model: represent 5 real scenarii

If you still need more, it's time for broader organizational alignment.


#### Technical vectors

Technical vectors point should be aligned and point towards the goal.

To help align them:
- give direct feedback: have a discussion to share your context and the person's. Give feedback. Start with this before changing process

- refine your engineering strategy

- have your approach reflected by tooling & workflows. Ex:
  - block deploy to prod if the project has no on-call setup
  - provisiong resources only through a website that requires a link to the service specification

- train new hires during onboarding

- Conway's law: org structure will be reflected in the technical structure

- After all this: curate technology change: using architecture reviews, investment strategies, and a structured process for adopting new tools.


If you still need more, it's time to move to heavier approaches. But first, measure.


#### Measure technical quality

Establish a definition of quality based on metrics. It should be adequate for your org. It will help people improve quality based on those metrics.

The metrics should be measured by instrumentation / automated to track over time

Ex: codebase LoC per file, pull request LoC, test coverage, files that changed in 50%+ of the pull requests, functions that own fine-grained locks

Next step: decide between a *quality team* and a *quality program*

#### Technical quality team