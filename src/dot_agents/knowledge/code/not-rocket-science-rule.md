---
slug: not-rocket-science-rule
categories: [testing, prs]
priority: 3
description: Before merging, run CI on the actual merge commit — not the branch tip — and merge only if it goes green. Keeps main green and forces tests to stay fast.
applies_when:
  - main branch breaks despite all PRs passing CI individually
  - setting up CI gating or a merge queue
  - debating whether to require rebase-on-main before merge
  - test suite is creeping past the threshold where developers can wait for it
related: [creating-prs, writing-tests]
source: https://graydon2.dreamwidth.org/1597.html
---

# Not Rocket Science Rule

> Test the merge, not the branch. A green branch tip says nothing about the post-merge state.

## Key Takeaways

- Named by Graydon Hoare. The rule: *"the answer to keeping a tree green is to never let it go red in the first place — by automatically testing each change before merging it."* What gets tested must be the merge commit, not the PR branch as it sat in isolation.
- A branch that passed CI hours ago may break when merged because main moved. Testing the branch tip is necessary but not sufficient — semantic conflicts (a renamed function, a deleted helper) slip past `git merge` cleanly.
- Implemented by tools like Bors, Homu, GitHub merge queues, GitLab's "merge train". Each PR is rebased onto current main, CI runs on that synthetic commit, and merge happens only on green.
- Side effect: creates real economic pressure to keep the test suite fast. A 45-minute suite means the merge queue stalls; a 4-minute suite scales. The rule and `sans-io` reinforce each other.
- Without it, you get the "main is broken again" pattern — every contributor wastes time bisecting other people's regressions.

## Source

Graydon Hoare — [The Not Rocket Science Rule of Software Engineering](https://graydon2.dreamwidth.org/1597.html); cited in Alex Kladov, [How to Test](https://matklad.github.io/2021/05/31/how-to-test.html).
