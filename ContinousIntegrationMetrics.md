Metrics for Continous Integration
=================================

Thesis
------

All software development workflows can be summarized into a universal model of software integration: code is written, code is verified, code is integrated with other code. Given this universal nature, a simple metric can be created that will quantify the real time integration cycle. Using this metric, a software team can characterize their current cycle, and use that information to suggest and adopt changes that will improve the total cycle time, with the goal of making integration more continous.


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

Let B be the Real Time that passes between the Commit and the Merge including that Commit.

Let C be the sum of A and B; this is the Real Time for getting a change into the Trunk.

Let D be the floored whole value of a business day divided by C. This is the maximum number of Commits integrated per day, per committer. D = floor(8/C), typically.

Let E be the ceiling whole value of C divided by a business day. This is the minimum number of days it will take for a particular commit to show up in Trunk. E = ceil(C/8), typically.


Absolute Minimums can be calculated by considering how long it takes a "null" code change to propegate through each step.

The Absolute Minimum A is the Real Time it takes to verify that the code is Green.

The Absolute Minimum B is theoretically however long it takes to transport the Merge. This is possible when the Green assessment of the Absolute Minimum A is trustworthy.

Practical Minimums can be calculated for each project how long it takes a "null" code change to propegate through the team's infrastructure.


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

 1. The amount of work developers intend to do in the slice of work
 2. The Green Real Time... the amount of time it takes to confirm that the code is safe.

#### Improving A.1

Usually when A is large, A.1 is the biggest bite. Improving this is principally cultural - work with the developers to practice breaking down large tasks into smaller, discrete parts. This may involve teaching skills like:

- Object Modeling away from the keyboard
- Breaking down stories into tasks, and tasks into commits.
- Branch by Abstraction.
- Feature Toggling
- setting short-term team architectural goals
- tolerating temporary inconsistancy
- Adding new acceptance tests to the Green validations only after the feature is complete

#### Improving A.2

When teams are excellent at A.1, then, over time, A.2 becomes more prominent. Test suites tend to grow along with the codebase and feature set, and with great complexity comes great validation requirements.

A common way to deal with these issues is with smarter modularization - organizing your code modules according to your team's development needs. Smart modularization allows the team to use the clear dependency rules to determine the correct suite of tests to run given a particular code change.

This may be a little unclear to understand, so lets start with an example:

A team has built a web application that will accept data, perform some substantial custom statistics on that data, then present it in a single page app. In order to maximize speed of development, it has been developed in one version control system: a single git repository. The team hits a moment where their total test time is approaching ten minutes for the whole system, and they recognize that this is a substantial productivity hit to them over the course of a day. After some investigation, they see that about three minutes of the build time is attributed to the core engine of the statistics system, and they note that over the last three months, only minor changes have been made to this core system.

Given this knowledge, they decide to extract the core statistics system to its own, independant module still within their git repository. The independance is *important*: that module has *zero* dependencies on other modules within this repository. Because of this, they can make the following optimization to the Green process: the tests for the core module will *only* run when a modification has been made to code in that core module; otherwise they can be safely skipped. In this way, the team safely cuts out three minutes from a typical Green process, bringing their median A down substantially. They also decided to daily run the *full* test suite, just to ensure that there aren't any time-related problems in that module code.
