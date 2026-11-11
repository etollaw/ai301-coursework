# Contribution: [FEAT]: Enable optional usage of `tox-uv` in the code and documentation testing workflows

Path: `beat-2-contribution/contribution-story.md`

This file is Assignment 5, and it is graded at the path above; a copy kept anywhere else in
the repository is not read. **Keep every section header below under its exact name** — the
format criteria grade exactly that. Your course repo link is the whole submission: no tool
files and no evidence uploads are part of Assignment 5.

**Contribution Number:** [1]
**Student:** [Student name]
**GitHub Username:** etollaw
**Issue:** https://github.com/tektronix/python-package-ci-cd/issues/354
**Pull Request:** [GitHub PR link, once opened]

One repository, built across units 5 to 8, telling the whole story of your real
contribution. Fill it as you work, not at the end — the sections below ask for evidence that
is far easier to write down as it happens than to reconstruct in the last weekend.

Contribution Number stays `[1]` unless you make more than one contribution this term; it
just numbers them.

---

## Toolkit

Every story section below, from Why I Chose This Issue through Pull
Request, ends with a Toolkit block: which tool did that phase's work
and how you used it, what it produced, what difficulties or
shortcomings it presented, and what you changed or would change
because of them. Together the blocks are the thread the rubric's
Toolkit points read, and they are the raw material for your unit-9
portfolio, so keep them current as you work rather than
reconstructing them later.

A Toolkit block looks like this (copy it into each story section and
fill it there):

```text
**Toolkit**

- Tool: [which tool did this phase's work, and how you used it]
- What it produced: [described in a sentence or a short excerpt]
- Difficulties: [what fought you or fell short, or "none met"]
- What changed: [the patch, revision, or workaround, what you would
  update, or "nothing"]
```

This section also keeps one short dated entry per week of tool work,
starting below: what you built or changed that week, what broke, and
what you changed because of it. The blocks tell the story where it
happened; these entries are the toolkit's own timeline.

All four entries are graded. Keep every heading below, and fill each
one in the week it belongs to.

**Unit 5: Turned issue-select into issue-scout, wrote its search strategy, and fixed a ranking that spent my grading budget on taken issues**

I installed the scout over my Unit 1 skill and wrote three `gh search issues` queries from my fit profile, two of them naming established owners, because label-plus-language searches came back almost entirely from repos only a few weeks old. My first scan graded five issues, and four failed `unclaimed`: each already had an open PR, one filed 25 minutes after its issue, because ranking by "fewest comments, then newest" pushed hours-old issues to the top. I added `-linked:pr` to every query, and on the second scan none of the top five had a PR. Grading by hand and with the scout also exposed rubric gaps I logged but haven't patched yet: `scope-bounded` passed apache/airflow#16793 despite a maintainer's "I hope you know what you are signing up for"; `unclaimed` would pass Canner/WrenAI#1421 (I checked that one by hand) even though a maintainer said a team member was building it and the UI it targets is now a frozen legacy branch; and no check looks at an issue's age, so scikit-hep/pyhf#1653 passed after four silent years.

**Unit 6: [one line on the tool work]**

[2-4 sentences: the voice-guard, the repo-memory file. What broke or
fell short, and what changed because of it. "Nothing broke" counts when
it is true - say how you know.]

**Unit 7: [one line on the tool work]**

[2-4 sentences: the ship-gate and your gate manifest. What it produced
on your real branch, what fought you, what changed because of it.]

**Unit 8: [one line on the tool work]**

[2-4 sentences: the PR-monitor and your playbook. What it read, what it
drafted, what you did with the draft.]

---

## Why I Chose This Issue

My scout's two scans never surfaced this issue. They ranked apache issues on top (magpie#1824, airflow#16793), and I found tektronix/python-package-ci-cd#354 on the AI301 Issue Selection site instead, through a "test" title search after filtering for Python and good first issue / help wanted. I checked about 30 candidates there by hand with free `gh` calls, and most already had an open PR, a fresh claim, or a maintainer saying the fix had landed. Three survived, and I graded all three with the scout. It accepted all three and ranked this one the best fit: one clear goal, no assignee, no linked PR, no comments, and no AI policy against contributors like me. I re-checked on 2026-11-10 before claiming, and it was still unclaimed. I chose it because my fit profile asks for CI and testing setup work, and this is exactly that. The repo's reusable GitHub Actions workflows (`_reusable-test-code.yml`, `_reusable-test-docs.yml`) install tox with plain `pip install tox`, and the issue asks for an optional way to use `tox-uv` instead. I want to learn how reusable workflows take inputs, and how to change a workflow other projects depend on without breaking them.

The scout held up on grading but fell short on search. My queries keep only issues updated since 2026-07-27, and this one was last touched in February 2025, so my own strategy filtered it out before ranking ever saw it. Its owner isn't on my owner lists either, and its title doesn't contain "CI". The rubric also passed `maintainer-active` on five renovate[bot] commits. I confirmed by hand that the maintainer who opened the issue (nfelt14) still commits, most recently on 2026-11-04. Nothing checks issue age either: this one is 21 months old and has had no comments. I logged these gaps in my scan notes but haven't patched them. My next changes would be to count only human commits in `maintainer-active`, and to drop the `--updated` cutoff in favour of ranking on creation date.

Claim comment: https://github.com/tektronix/python-package-ci-cd/issues/354#issuecomment-6792472005 (posted 2026-11-11)

**Toolkit**

- Tool: issue-scout. I ran two scans with my own search strategy and rubric, then used `issue-scout: grade` on my three picks from the Issue Selection site.
- What it produced: an accept on all six checks for #354, ranked first of my three picks as the closest fit for CI/testing work. The only caveat it flagged was that the recent commits were all from the Renovate bot.
- Difficulties: my search strategy's `--updated` cutoff excluded this issue, so the scout found it only through grading, not scanning. `maintainer-active` counts bot commits, and no check looks at issue age.
- What changed: I verified human maintainer activity and the unclaimed state by hand (2026-10-27 and again 2026-11-10). The rubric and strategy patches are logged in scan-notes but not made yet.

---

## Understanding the Issue

**Problem Description**

Nothing is broken; an option is missing. python-package-ci-cd ships reusable GitHub Actions workflows that other Python projects call to run their tox test suites. Those workflows always install plain `tox` with pip, so a project that wants the faster `tox-uv` plugin (which builds tox's virtual environments with uv instead of virtualenv/pip) has no way to turn it on.

**Expected Behavior**

A project calling the code-testing or docs-testing reusable workflow can opt in to `tox-uv` (for example, through a new workflow input), and the workflow then installs `tox-uv` alongside tox so its tox runs use uv. Projects that don't opt in keep today's behavior, and the workflow docs explain the new option.

**Current Behavior**

Every job installs tox with a hard-coded pip command and offers no input to change it. In `_reusable-test-code.yml`, the general job runs `python -m pip install tox tox-gh-actions` and the fast job runs `python -m pip install tox`. In `_reusable-test-docs.yml`, the job runs `python -m pip install tox`. Neither workflow's `inputs:` block has anything about uv or tox-uv.

**Affected Components**

- `.github/workflows/_reusable-test-code.yml`: the reusable code-testing workflow (its inputs and its install steps)
- `.github/workflows/_reusable-test-docs.yml`: the reusable docs-testing workflow
- `workflows/test-code.md` and `workflows/test-docs.md`: the user docs for those workflows, including their Inputs sections
- `.github/workflows/test-code.yml` and `test-docs.yml`: this repo's own callers, where the new option can be exercised
- `CHANGELOG.md`: its Unreleased section (CONTRIBUTING.md requires an entry)

**Toolkit**

- Tool:
- What it produced:
- Difficulties:
- What changed:

---

## Reproduction Process

**Environment Setup**

[What a stranger would need to stand this up: your operating system,
the relevant versions (language, runtime, package manager, the project
itself), and how you set the project up. Then the challenges you faced
and how you solved them, or that none came up.]

**Steps to Reproduce**

1. [Step 1]
2. [Step 2]
3. [Observed result]

**Reproduction Evidence**

- **Commit showing reproduction:** [link to commit in your fork]
- **Screenshots/logs:** [if applicable]
- **My findings:** [what you discovered during reproduction; an
  honest, evidenced cannot-reproduce that led you to re-pick or ask
  the maintainer belongs here and costs you nothing]

**Toolkit**

- Tool:
- What it produced:
- Difficulties:
- What changed:

---

## Solution Approach

**Analysis**

[Your analysis of the root cause: what is causing the issue, traced
to the files and functions involved.]

**Proposed Solution**

[High-level description of your fix approach, and what you are
deliberately NOT changing.]

**Implementation Plan**

1. [Modify file X to do Y]
2. [Add or change Z]
3. [Update tests]

[Record deviations here as they happen: what changed mid-build and
why. A recorded deviation is honest work; silent drift is not.]

**Toolkit**

- Tool:
- What it produced:
- Difficulties:
- What changed:

---

## Testing Strategy

[What observable checks prove the fix: your reproduction steps re-run
against the built change (before and after), and the repo's own test
suite or checks with the outcome shown. "Tests pass" with nothing
observable named is not a strategy.]

**Toolkit**

- Tool:
- What it produced:
- Difficulties:
- What changed:

---

## Implementation Notes

[A running log, added to as you work. Date each entry. What you
built, decisions you made when you made them, wrong turns included.]

**[Date]**

[Entry]

**[Date]**

[Entry]

**Toolkit**

- Tool:
- What it produced:
- Difficulties:
- What changed:

---

## Pull Request

**PR Link:** [GitHub PR URL when opened]

**PR Description:** [what the PR says; much of the content above
adapts into it]

**Maintainer Feedback:**

- [Date]: [feedback received, or "none yet"; silence is a normal
  state and costs you nothing]
- [Date]: [how you addressed it]

**Status:** [Awaiting review / Iterating / Merged / Closed. The
status is a fact you record, never a grade: the PR is graded as
opened.]

**Toolkit**

- Tool:
- What it produced:
- Difficulties:
- What changed:

---

## Learnings and Reflections

**Technical Skills Gained**

[What you learned technically, named concretely.]

**Challenges Overcome**

[What was hard and how you solved it; point at the real events, the
Toolkit blocks above are full of them.]

**What I'd Do Differently Next Time**

[Reflection on your process.]

---

## Resources Used

[What actually helped, not a reading list. List things this story
already shows you using: the doc you followed to set the environment
up, the thread that explained the bug, the person who answered you.
One real entry beats ten plausible ones.]

- [Link to documentation that helped]
- [Discussion, issue, or post that helped]
