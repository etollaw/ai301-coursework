# Rubric: is this a good first issue?

<!--
THIS IS THE PART YOU WRITE. The skill in SKILL.md executes whatever checks
you define here. It ships empty on purpose: the judgment is your work.

A filled rubric must contain:

1. At least one row in the checks table. Each row needs all four columns:
   - Check: a short name (used in the output JSON).
   - Evidence: exactly what to look at, and where. Name the source
     (repo-facts block, issue body, comment thread, or the locations in
     references/evidence-guide.md). "The repo" is not a source; "the last
     5 default-branch commit dates" is.
   - Pass condition: a condition someone else could apply and get your
     answer. Prefer thresholds with numbers ("a maintainer commented
     within 30 days") over adjectives ("maintainer is responsive").
   - Weight: `required` (a fail here rejects the issue) or `preferred`
     (never changes the verdict; a nice-to-have that helps rank the
     issues you accept).

2. A verdict rule below the table: how the check grades combine into
   accept or reject, including how `unclear` is treated. The verdict
   space is binary. If you write no rule for `unclear`, the skill treats
   it as fail.

Cover what actually kills first contributions. The lecture named four
families: the maintainer is alive, the repo is in use, the scope fits a
newcomer, and nobody else is already on it. A rubric that ignores a family
will fail eval issues designed around that family.
-->

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| maintainer-active | "last 5 default-branch commits" under Repo facts | At least one of the last 5 default-branch commits is dated within 90 days of the bundle's capture date | required |
| repo-in-use | "archived:" flag and "latest release" under Repo facts | Repo is not archived, AND (latest release is within 12 months of capture date OR last push to any branch is within 90 days of capture date) | required |
| scope-bounded | Issue body and comment thread | Pass unless: (a) the issue's sub-items are independent, separately assignable pieces of work meant to be split off individually — not a single coherent goal that happens to touch multiple files or pages; (b) the issue's own body leaves the core required work undecided — phrasing like "TBD" or "not sure if X is needed" applied to a central part of the task, not a minor optional aside — showing even the author hasn't decided what's required, OR the comment thread shows a live, unresolved disagreement between multiple people with no maintainer decision reached (do not fail merely because the author confidently diagnoses multiple causes or proposes several possible fixes on their own); (c) a maintainer explicitly states the fix touches core internals; (d) issue is a pure usage/support question; or (e) issue history shows multiple closed, unmerged linked PRs or abandoned attempts. Do not fail solely for touching multiple files or listing more than one possible fix — grade the bound of the work, not its size. | required |
| unclaimed | "this issue: assignees:" and "linked PRs:" under Repo facts, plus the Comments section | No assignee is listed, no linked PR is open, and no "I'll take this"-style claim comment in the last 30 days (relative to capture date) is unanswered and unabandoned | required |
| ai-policy-ok | "contribution policy" line under Repo facts | Fails only if the policy states an outright ban on AI-generated contributions; disclosure/testing/review conditions and silence both pass | required |
| good-first-label | Issue labels | Issue carries a "good first issue" (or equivalent) label | preferred |

## Verdict rule

Accept only if every required check passes. A `?` (not enough evidence) on
a required check counts as a fail. The `good-first-label` check never
changes the verdict — in live mode it's used only to help rank issues
your rubric already accepted.
