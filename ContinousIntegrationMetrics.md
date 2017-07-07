Terms
-----

Green - "All appropriate validations are performed to trust the state of the code."
Commit - "A complete discreet and Green code change."
Trunk - "A location where the current code is stored that is considered the source of truth."
Merge - "Integrating Commits into a Trunk."
Real Time - "Median measured business clock time."



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