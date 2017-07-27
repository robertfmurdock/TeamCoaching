Metrics for Continuous Integration
=================================

Thesis
------

All software development workflows can be summarized into a universal model of software integration: code is written, code is verified, code is integrated with other code. Given this universal nature, a simple metric can be created that will quantify the real time integration cycle. Using this metric, a software team can characterize their current cycle, and use that information to suggest and adopt changes that will improve the total cycle time, with the goal of making integration more continuous.


Terms
-----

Green - "All appropriate validations are performed to trust the state of the code."

Commit - "A complete discreet and Green code change."

Trunk - "A location where the current code is stored that is considered the source of truth."

Merge - "Integrating Commits into a Trunk."

Real Time - "Median measured duration of a task, in business hours."


Big Questions
-------------

How much Real Time passes between Commit and Merge?

How much Real Time passes between initial code change and Commit?



Universal Software Development Workflow
---------------------------------------

Work on every software project follows this form:

A developer or developer pair cycles through three states:

- Think "Consider the task at hand and plan the execution path"
- Pull "Update the developer's working code to include changes from the Trunk"
- Code "Make modifications to the source code locally."

After one or more rotations through these states, the developer will Commit.

At some point after the Commit occurs, the Commit will be Merged.

Once the Commit is Merged, it is available to all developers during the Pull step of their workflow.

Metrics
-------

Let A be the Real Time that passes between the initial code change and a Commit.

Let B be the Real Time that passes between the Commit and the Merge that contains the Commit.

Let C be the sum of A and B; this is the Real Time for getting a change into the Trunk.

Let D be the floored whole value of a business day divided by C. This is the maximum number of Commits integrated per day, per committer. D = floor(8/C), typically.

Let E be the ceiling whole value of C divided by a business day. This is the minimum number of days it will take for a particular commit to show up in Trunk. E = ceil(C/8), typically.


Absolute Minimums can be calculated by considering how long it takes a "null" code change to propagate through each step.

The Absolute Minimum A is the Real Time it takes to verify that the code is Green.

The Absolute Minimum B is theoretically however long it takes to transport the Merge. This is possible when the Green assessment of the Absolute Minimum A is trustworthy.

Practical Minimums can be calculated for each project how long it takes a "null" code change to propagate through the team's infrastructure.


Targets
-------

For values of A

| < 2     | < 4            | < 8              | < 16         | < 32      | < 64  |
|---------|----------------|------------------|--------------|-----------|-------|
| :smile: | :blush:        | :expressionless: | :confounded: | :worried: | :cry: |

For values of B

| < 2     | < 4              | < 8          | < 16  | < 32        | < 64     |
|---------|------------------|--------------|-------|-------------|----------|
| :smile: | :expressionless: | :confounded: | :cry: | :grimacing: | :scream: |


Scenarios
---------

### Pretty Great Scenario:

A = 1h

B = 1h

C = 2h

D = 4 Commits

E = 1 day

Every pair can make four changes per day.

Suggestions
-----------

### To improve values of A

A is composed of two major components:

 1. The amount of work developers perform in the slice of work
 2. The Green Real Time... the amount of time it takes to confirm that the code is safe.

#### Improving A.1

Usually when A is large, A.1 is the biggest bite. Improving this is principally cultural - work with the developers to practice breaking down large tasks into smaller, discrete parts. This may involve teaching skills like:

- Object Modeling away from the keyboard
- Breaking down stories into tasks, and tasks into commits.
- Branch by Abstraction.
- Feature Toggling
- setting short-term team architectural goals
- tolerating temporary inconsistency
- Adding new acceptance tests to the Green validations only after the feature is complete

#### Improving A.2

When teams are excellent at A.1, then, over time, A.2 becomes more prominent. Test suites tend to grow along with the codebase and feature set, and with great complexity comes great validation requirements.

A common way to deal with these issues is with smarter modularization - organizing your code modules according to your team's development needs. Smart modularization allows the team to use the clear dependency rules to determine the correct suite of tests to run given a particular code change.

This may be a little difficult to understand, so allow me to suggest some examples:

##### Example 1

A team has built a web application that will accept data, perform some substantial custom statistics on that data, then present it in a single page app. In order to maximize speed of development, it has been developed in one version control system: a single git repository. The team hits a moment where their total test time is approaching ten minutes for the whole system, and they recognize that this is a substantial productivity cost to them over the course of a day. After some investigation, they see that about three minutes of the build time is attributed to the core engine of the statistics system, and they note that over the last three months, only minor changes have been made to this core system.

Given this knowledge, they decide to extract the core statistics system to its own, independent module still within their git repository. The independence is *important*: that module has *zero* dependencies on other modules within this repository. Because of this, they can make the following optimization to the Green process: the tests for the core module will *only* run when a modification has been made to code in that core module; otherwise they can be safely skipped. In this way, the team safely cuts out three minutes from a typical Green process, bringing their median A down substantially. They also decided to daily run the *full* test suite, just to ensure that there are not any time-related problems in that module code. And of course, when changes *are* made to that module, the full build must be run. To maximize safety, instead of relying on the programmer's discretion as to whether they should run these tests or not, the team uses a build tool that will detect when changes are made to that module, and include those tests automatically.

##### Example 2

A team has built a comprehensive web application over the course of a few years. There are a *lot* of web browser based end to end tests on account of this, and the e2e test suite is part of the Green requirements. This is appropriate - these are tests managed by the developers (the QA team has their own suite of tests that are not required as part of Green). However, the burden of these tests is getting fairly onerous - they take four minutes to run, of the total test time of six minutes.

The team considers upcoming work - there's a lot more stuff that will require e2e testing! On account of this, they decide to take the problem seriously now. They assess that the frontend of their application can be reasonably divided into four parts - each satisfying a different user. They decide to take advantage of this - they modularize their frontend into five modules. One module will act as their shared library, which will be deliberately thin, rarely-modified, and consumed by the other four modules. The other four will act as almost independent applications.

Why did the team find this valuable, other than the general satisfaction of modularization? They did this because after the modularization they can use logical guarantees to skip unrelated tests. "When I change something in module C, it is *impossible* for it to affect A, B and D".

By doing this, and upgrading their build system to optimize test runs based on the dependency tree, they were able to cut their median build time to a *third* of what it was. And only the rare builds that update the shared library or updated other shared dependencies will trigger the full build.

### To improve values of B

Depending on a particular team's process, B time can be composed of many different things. Here they are in a numbered list:
  1. The technical merge process itself, which may include resolving conflicts with other Commits.
  2. Some teams add an additional Green validation step, to demonstrate that the source can build twice before being shared.
  3. Many teams add an explicit code review step before accepting a Commit, that includes one or more developers. Sometimes the review step will also require a team architect or tech lead to approve.
  4. Many teams also require that *all* commits related to a particular feature be complete before *any* commits related to the feature may be merged.

Teams suffering from large B values may have substantial bottlenecks for any or all of these processes. Lets discuss some strategies for reducing the burden.

#### Improving B.1

Actually merging multiple changes sets can be a pain point for some teams. The pain usually comes from one of these sources:

- Bad tooling that does not automate trivial merges, or is not trustworthy for trivial merges
- Developers are concurrently working on shared code and changing it in non-compatible ways, making merging difficult and error-prone
- Code structure has bottlenecks - developers must modify similar files to check in
- Large Commits, or letting Commits pile up before Merge will make Merge resolution more difficult and more error-prone
- The test-suite is not trustworthy enough: features may break during a merge and the bug escapes detection.

Of course, the best answer for bad tooling is... improve the tooling. There are many third party tools that can be trusted to mostly correctly merge code.

For developer concurrency problems, the *best* way to solve this is by communication. Find ways to ensure that your team is aware of what the rest of the team is working on. Daily stand-up meetings are an important tool to help with this. Making tasks-in-progress as visible as possible is another key tool - ideally the team should accidentally over the course of the day notice tasks-in-progress. Physical story boards are a fine tool for communicating this information. Co-location or strict chat channel guidelines will also be a great boon. 

Once the possible collision is detected, having the developers break related work down into tasks that can be explicitly divided amongst them will encourage the developers to work closely and keep in touch. In extreme cases, blocking work at the story-planning level can be considered; however, this should be considered a *last resort*. Blocking work at the planning level indicates that the development team is not trusted to detect and resolve collisions as they arise. A planning process that requires substantial technical knowledge will tend to freeze out product owners, and this defeats the purpose of planning.

When code-structure bottlenecks occur, adding structural features to the source code can reduce the pressure. Take this example: the code requires a "test suite" dictionary file that lists all of the tests being added to the system, and every time a developer adds work they have to modify the file. This file is constantly being merged, and this merge has caused some pain for the team. Given that, inverting the relationship from an explicit "list" into a functional "search" will eliminate the collision point. That is to say, replace the dictionary file with a program that finds and collects the tests. This sort of technique can be applied at many layers of a system, and is the foundation of dependency injection. There are trade-offs when using this technique, and the team should consider these when solving these problems.
  
Large Commits are an issue that is also discussed in relation to A.1. However, improving A.1 will not help when many Commits have been completed but not merged. The simple suggestion - attempt to merge after as few Commits as possible. Depending on the team process, this might not be as simple as it sounds. Team process related issues will be discussed in more detail in suggestions for improving B.2, B.3, and B.4.

When the test-suite is not trustworthy, there are a number of problems that could be in play here - much more than this document will be able to cover in brief. In short, the team's goal should be to work to create "safe-zones" within the source code, where the test-suite IS trustworthy, and then work to separate the untrustworthy code from the trustworthy code. Over time, pull as much code as is appropriate from the untrusted category into the trusted category.

#### Improving B.2

When the Real Time value of checking Green is reasonable, this is unlikely to be a major bottleneck. However, for projects of substantial size, rerunning a build may become more painful. When total Green time is over twenty minutes, this can be excruciating. Assuming that the recommendations from improving A.2 have already been applied (in-repository modularization and caching), this is a good time to consider stronger measures. These measures may include:
  
  - Use concurrency to maximize the number of modules that can be built and tested simultaneously.
  - Add additional hardware to the build process such that concurrency-enabled builds can be performed faster
  - Explore smart caching at the build-server level; only build modules that the changed code can affect
  - Reducing the depth of the project's module hierarchy. That is to say, minimize the number of modules that 'everything depends on', and when possible, divide them into more highly specified modules.
  - Extracting isolated modules from the project into true independent libraries, with their own semver. Be warned, this solution will bring a new category of continuous integration problems into play (CI with dependencies).
  - Consider converting some dependencies from build dependencies to service dependencies. This should *only* be considered if this approach is appropriate for the business. This also brings a new category of continuous integration problems into play (CI with in-development services)
  
