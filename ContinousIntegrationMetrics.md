Metrics for Continuous Integration
==================================

Thesis
------

All software development workflows can be summarized into a universal model of software integration: code is written, code is verified, code is integrated with other code. Given this universal nature, simple metrics exist that will quantify how much real time the integration cycle takes. Using this metric, a software team can characterize their current cycle, and use that information to suggest and adopt changes that will improve the total cycle time, with the goal of making integration more continuous.

Terms
-----

Green - "All appropriate validations are performed to trust the state of the code."

Commit - "A complete discrete and Green code change."

Trunk - "A location where the current code is stored that is considered the source of truth."

Merge - "Integrating Commits into a Trunk."

Real Time - "Median measured duration of a task, in business hours."

Universal Software Development Workflow
---------------------------------------

Work on every software project follows this form:

A developer or developer pair cycles through three states:

- Think "Consider the task at hand and plan the execution path"
- Pull "Update the developer's working code to include changes from the Trunk"
- Code "Make modifications to the source code locally."

After one or more rotations through these states, the developer will create a Commit.

Later, the Commit will be Merged.

Once the Commit is Merged, it is available to all developers during the Pull step of their workflow.

Metrics
-------

### Commit Time

This is the Real Time that passes between the initial code change and the creation of a Commit.

### Integration Time

This is the Real Time that passes between the Commit and the Merge that integrates the Commit.

### Commit-Integration Time

This is the Real Time for getting a change into the Trunk; the sum of Commit Time and Integration Time.

### Daily Max Throughput

This is the maximum number of new Commits integrated per day, per committer. This can be calculated as the floored whole value of a business day divided by Commit-Integration Time. For typical business days, DMT = floor(8/CIT)

### Days-to-Integration

This is the minimum number of days it will take for a Commit to show up in Trunk. This can be calculated as the ceiling whole value of Commit-Integration Time divided by a business day. For typical business days, DtI = ceil(CIT/8)

### Green Time

This is the Real Time it takes in order to verify that the code is Green.

### Absolute Minimum Metrics

Absolute Minimums can be calculated by considering how long it takes a "null" code change to propagate through each step.

The Absolute Minimum Commit Time is the Green Time.

The Absolute Minimum Integration Time is theoretically however long it takes to transport the Merge. This is possible when the Green assessment of the Absolute Minimum Commit Time is trustworthy, and there are never conflicts to be resolved in the Merge.

### Practical Minimums

Practical Minimums can be calculated for each project how long it takes a "null" code change to propagate through the team's infrastructure.

Targets
-------

The Commit-Integration Time is the metric that best illustrates a team's continuous integration interval. This number can be uses to summarize the health of the team's integration process. Daily Max Throughput and Days-to-Integration  are useful for illustrating CIT in a more business-friendly way. Commit Time and Integration Time are useful to know as a diagnostic tool when a team intends to improve CIT.

For reference, projects of widely-ranging sizes with extremely healthy continuous integration processes have achieved Commit-Integration Times of about 2 hours (or less). Commit-Integration Times of 8 hours might be considered acceptable (given project constraints). Projects with Commit-Integration Times over 8 hours should not be considered "continuous" and will likely exhibit the problems CI ameliorates.

The following table comically illustrates how different values of Commit Time and Integration Time should be assessed.

For values of Commit Time:

| < 2     | < 4            | < 8              | < 16         | < 32      | < 64  |
|---------|----------------|------------------|--------------|-----------|-------|
| :smile: | :blush:        | :expressionless: | :confounded: | :worried: | :cry: |

For values of Integration Time:

| < 2     | < 4              | < 8          | < 16  | < 32        | < 64     |
|---------|------------------|--------------|-------|-------------|----------|
| :smile: | :expressionless: | :confounded: | :cry: | :grimacing: | :scream: |

Note that higher Integration Times are considered more painful than higher Commit Times. This is because Integration Time decays value: the Commit is getting more stale the longer the wait.

### Extremely Healthy Project Example Numbers

Commit Time = 1h

Integration Time = 1h

Commit-Integration Time = 2h

Daily Max Throughput = 4 Commits

Days-to-Integration = 1 day

Every Committer can make four changes per day, every day sees new Commits.

Measurement
-----------

Collecting these numbers with precision can be challenging. Because these numbers are not likely to change much over the course of a week, it should be considered acceptable to ask a development team to estimate what their current personal CT and IT values are, and then produce team medians of those values. These estimates are usually sufficient to suggest process improvements.

For teams that want to produce more accurate measurements, then using a timer to collect the CT and IT of each Commit will be required. Be sure that the timer does not collect non-business time - for example, if a commit is started at 4pm one day, and the developers stop working at 5pm, make sure the timer is stopped and restarted when the developers start again the next day.

Understanding Continuous Integration Metrics, and Improving
===========================================================

Having collected the metrics suggested in "Metrics for Continuous Integration", a team may not be satisfied with their results and seek to improve. The following sections are intended to help understand what different values of the metrics may mean, and then suggest ways to improve a team's process in order to improve the metrics.


### To Improve Commit Time

Commit Time (CT) is composed of two major components:

 1. The amount of work developers perform in the slice of work
 2. The Green Real Time... the amount of time it takes to confirm that the code is safe.

#### Improving CT.1: work quantity

Usually when CT is large, CT.1 is the biggest bite. Improving this is principally cultural - work with the developers to practice breaking down large tasks into smaller, discrete parts. This may involve teaching skills like:

- Object Modeling away from the keyboard
- Breaking down stories into tasks, and tasks into commits.
- Branch by Abstraction.
- Feature Toggling
- setting short-term team architectural goals
- tolerating temporary inconsistency
- Adding new acceptance tests to the Green validations only after the feature is complete

#### Improving CT.2: Green Real Time

When teams are excellent at CT.1, then, over time, CT.2 becomes more prominent. Test suites tend to grow along with the codebase and feature set, and with great complexity comes great validation requirements.

A common way to deal with these issues is with smarter modularization - organizing your code modules according to your team's development needs. Smart modularization allows the team to use the clear dependency rules to determine the correct suite of tests to run given a particular code change.

This may be a little difficult to understand, so allow me to suggest some examples:

##### Example 1

A team has built a web application that will accept data, perform some substantial custom statistics on that data, then present it in a single page app. In order to maximize speed of development, it has been developed in one version control system: a single git repository. The team hits a moment where their total test time is approaching ten minutes for the whole system, and they recognize that this is a substantial productivity cost to them over the course of a day. After some investigation, they see that about three minutes of the build time is attributed to the core engine of the statistics system, and they note that over the last three months, only minor changes have been made to this core system.

Given this knowledge, they decide to extract the core statistics system to its own, independent module still within their git repository. The independence is *important*: that module has *zero* dependencies on other modules within this repository. Because of this, they can make the following optimization to the Green process: the tests for the core module will *only* run when a modification has been made to code in that core module; otherwise they can be safely skipped. In this way, the team safely cuts out three minutes from a typical Green process, bringing their median Commit Time down substantially. They also decided to daily run the *full* test suite, just to ensure that there are not any time-related problems in that module code. And of course, when changes *are* made to that module, the full build must be run. To maximize safety, instead of relying on the programmer's discretion as to whether they should run these tests or not, the team uses a build tool that will detect when changes are made to that module, and include those tests automatically.

##### Example 2

A team has built a comprehensive web application over the course of a few years. There are a *lot* of web browser based end to end tests on account of this, and the e2e test suite is part of the Green requirements. This is appropriate - these are tests managed by the developers (the QA team has their own suite of tests that are not required as part of Green). However, the burden of these tests is getting fairly onerous - they take four minutes to run, of the total test time of six minutes.

The team considers upcoming work - there's a lot more stuff that will require e2e testing! On account of this, they decide to take the problem seriously now. They assess that the frontend of their application can be reasonably divided into four parts - each satisfying a different user. They decide to take advantage of this - they modularize their frontend into five modules. One module will act as their shared library, which will be deliberately thin, rarely-modified, and consumed by the other four modules. The other four will act as almost independent applications.

Why did the team find this valuable, other than the general satisfaction of modularization? They did this because after the modularization they can use logical guarantees to skip unrelated tests. "When I change something in module C, it is *impossible* for it to affect A, B and D".

By doing this, and upgrading their build system to optimize test runs based on the dependency tree, they were able to cut their median build time to a *third* of what it was. And only the rare builds that update the shared library or updated other shared dependencies will trigger the full build.

### To improve Integration Time 

Depending on a particular team's process, Integration Time (IT) can be composed of many different things. Here they are in a numbered list:
  1. The technical merge process itself, which may include resolving conflicts with other Commits.
  2. Many teams also require that *all* commits related to a particular feature be complete before *any* commits related to the feature may be merged.
  3. Some teams add an additional Green validation step, to demonstrate that the source can build twice before being shared.
  4. Many teams add an explicit code review step before accepting a Commit, that includes one or more developers. Sometimes the review step will also require a team architect or tech lead to approve.

Teams suffering from large Integration Times may have substantial bottlenecks for any or all of these processes. Lets discuss some strategies for reducing the burden.

#### Improving IT.1: Merge Process

Actually merging multiple Commits can be a pain point for some teams. The pain usually comes from one of these sources:

- Bad tooling that does not automate trivial merges, or is not trustworthy for trivial merges
- Developers are concurrently working on shared code and changing it in non-compatible ways, making merging difficult and error-prone
- Code structure has bottlenecks - developers must modify similar files to check in
- Large Commits, or letting Commits pile up before Merge will make Merge resolution more difficult and more error-prone
- The test-suite is not trustworthy enough: features may break during a merge and the bug escapes detection.

Of course, the best answer for bad tooling is... improve the tooling. There are many third party tools that can be trusted to mostly correctly merge code.

For developer concurrency problems, the *best* way to solve this is by communication. Find ways to ensure that your team is aware of what the rest of the team is working on. Daily stand-up meetings are an important tool to help with this. Making tasks-in-progress as visible as possible is another key tool - ideally the team should incidentally over the course of the day notice tasks-in-progress. Physical story boards are a fine tool for communicating this information. Co-location or strict chat channel guidelines will also be a great boon. 

Once the possible collision is detected, having the developers break related work down into tasks that can be explicitly divided amongst them will encourage the developers to work closely and keep in touch. In extreme cases, blocking work at the story-planning level can be considered; however, this should be considered a *last resort*. Blocking work is when a story is not allowed to be started until other related stories in the same area are completed. Blocking work at the planning level indicates that the development team is not trusted to detect and resolve collisions as they arise. A planning process that requires substantial technical knowledge will tend to freeze out product owners, and this defeats the purpose of planning.

When code-structure bottlenecks occur, adding structural features to the source code can reduce the pressure. Take this example: the code requires a "test suite" dictionary file that lists all of the tests being added to the system, and every time a developer adds work they have to modify the file. This file is constantly being merged, and this merge has caused some pain for the team. Given that, inverting the relationship from an explicit "list" into a functional "search" will eliminate the collision point. That is to say, replace the dictionary file with a program that finds and collects the tests. This sort of technique can be applied at many layers of a system, and is the foundation of dependency injection. There are trade-offs when using this technique, and the team should consider these when solving these problems.
  
Large Commits are an issue that is also discussed in relation to CT.1. However, improving CT.1 will not help when many Commits have been completed but not merged. The simple suggestion - attempt to merge after as few Commits as possible. Depending on the team process, this might not be as simple as it sounds. Team process related issues will be discussed in more detail in suggestions for improving IT.2, IT.3, and IT.4.

When the test-suite is not trustworthy, there are a number of problems that could be in play here - much more than this document will be able to cover in brief. In short, the team's goal should be to work to create "safe-zones" within the source code, where the test-suite IS trustworthy, and then work to separate the untrustworthy code from the trustworthy code. Over time, pull as much code as is appropriate from the untrusted category into the trusted category.

#### Improving IT.2: Feature-complete before Merge

Many teams currently require that a feature be complete before Merge. Teams will vary on definition of what a "feature" is - some will define it using story-card boundaries, others will be less strict. This document touches on this subject in the "Large Commits" paragraph of "Improving IT.1". However, there are a few more things worth noting here.

A Commit, as defined in the Terms section, must be discrete and Green. Being Green means that it has not broken any existing functionality, and is not broken itself.  Given that, *every Commit should be considered feature-complete for the features it provides*. Now of course, being feature-complete for those features does not mean that those features are ready for public consumption - there is a difference between being *feature-complete* and *presentable*. Because of this distinction, there is a problem - for some Commits, merely being Green may not be sufficient to safely Merge. These Commits are those that expose un-presentable features to the user.

One way to avoid this problem is by improving the Team culture such that it understands that *every Commit will be shown to users*. Given this knowledge, the team should use a variety of techniques to *hide* un-presentable features within those Commits. This may be as simple as not including an element on the main page, or as complex as an environment-specific feature-toggle.

The important thing is that the team is aware of any and all *additional Commit requirements* and adheres to them consistently. Given that, work to Merge after every Commit, and work to end the relationship between story-completion and Merge.

#### Improving IT.3: Repeated Green

When the Real Time value of checking Green is reasonable, this is unlikely to be a major bottleneck. However, for projects of substantial size, rerunning a build may become more painful. When total Green time is over twenty minutes, this can be excruciating. Assuming that the recommendations from improving CT.2 have already been applied (in-repository modularization and caching), this is a good time to consider stronger measures. These measures may include:
  
  - Use concurrency to maximize the number of modules that can be built and tested simultaneously.
  - Add additional hardware to the build process such that concurrency-enabled builds can be performed faster
  - Explore smart caching at the build-server level; only build modules that the changed code can affect
  - Reducing the depth of the project's module hierarchy. That is to say, minimize the number of modules that 'everything depends on', and when possible, divide them into more highly specified modules.
  - Extracting isolated modules from the project into true independent libraries, with their own semver. Be warned, this solution will bring a new category of continuous integration problems into play (CI with dependencies).
  - Consider converting some dependencies from build dependencies to service dependencies. This should *only* be considered if this approach is appropriate for the business. This also brings a new category of continuous integration problems into play (CI with in-development services)
  
#### Improving IT.4: Blocking Formal Code Review (BFCR)

It has become fairly common that a team will institute a formal code review step that will block the Commit from being Merged until a minimum number of reviewers have approved the Commit. This process can vary radically in terms of time-cost: some teams are able to do formal review within an hour of the Commit. Most teams wait substantially longer; in extreme cases this may be over a week.

Formal code review of this sort should be considered "blocking" - that is to say, no work can be completed until it has been reviewed by an approved authority. Any blocking formal code review (BFCR) process needs to be able to answer these questions:

  - What kind of issues is the BFCR intended to protect against - which issues require the code review process to be *blocking*? This could reasonably include things like "the team cannot be trusted to test", "the team cannot be trusted to respect the threading model", or "the team cannot be trusted to follow the project standards/conventions"
  - What are the non-critical benefits of the code review process?
  - What is the process by which a team member becomes a "reviewer"?
  - What is the expectation of responsiveness of the review team?
  - How do reviews get assigned?
  - Does work done by the review team also need review?
  - How long will the team work using BFCR? Under what conditions will it be dropped? 

Sometimes just answering these questions will suggest how things can be improved; some examples: 
  - Maybe there is no process by which a team member becomes a reviewer, and thus the reviewer pool is too small.
  - Maybe the review team had no formal expectation on responsiveness and was thus acting inconsistently
  - Maybe the reviews are not assigned and thus the review team only performs reviews when they think to check the queue 
Fixing the obvious holes in the process should help streamline the BFCR process.

##### Non-blocking Formal Code Review

To take the formal code review process even further, consider doing two things: 
  1. Start using team processes that will move the team from "untrusted" to "trusted"
  2. Work to automatically detect critical problems that only BFCR would catch
  3. Begin transitioning from a BFCR process to a non-blocking formal code review process (NBFCR). 

What does this mean? Essentially, if BFCR is the root of the IT.3 problem, then working to switch to a NBFCR will alleviate much of that pain by divorcing the code review process from the Merging process. All of the benefits of BFCR can be maintained with a NBFCR - appropriate strategies for doing so will vary by team. As a starting point, here are some team processes that may be used to switch from BFCR to NBFCR:

  - Regular team education time, so all are aware of mandatory project standards / problems that require a tech lead consult 
  - Paired programming with regular rotation for continuous informal code review (CICR)
  - Regular (possibly daily) formal code review of all recent work by the review team. Optionally, only reviewed builds get promoted to "release". Stories are not considered complete until passing through formal code review.
  
  
Blocking formal code review can be a massive barrier to getting a healthy Daily Max Throughput. Being cognizant of how this process is affecting the team can yield big improvements.