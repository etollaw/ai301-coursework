# Contribution: [FEAT]: Enable optional usage of `tox-uv` in the code and documentation testing workflows

Path: `beat-2-contribution/contribution-story.md`

This file is Assignment 5, and it is graded at the path above; a copy kept anywhere else in
the repository is not read. **Keep every section header below under its exact name** — the
format criteria grade exactly that. Your course repo link is the whole submission: no tool
files and no evidence uploads are part of Assignment 5.

**Contribution Number:** [1]
**Student:** Eldad Tolla
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

- macOS 26.5.1 (Apple silicon). tektronix/python-package-ci-cd cloned at `7d896b9` (main, 2026-11-11).
- Python 3.13.18, installed with `uv python install 3.13`, because the project requires `~=3.13.0`. My system Python is 3.14.7.
- tox 4.67.13, tox-gh-actions 3.5.0, and tox-uv 1.37.7 (uv 0.13.7), each in a fresh venv outside the repo. These mirror the workflow's install step: `python -m pip install tox tox-gh-actions` for the current setup, plus `tox-uv` for the opt-in setup.
- I ran the repo's `tests` env with `tox -ve tests`, the same command the workflow's fast job uses, giving each run its own `--workdir` so runs didn't share environments.
- Challenges:
  - My first runs used Python 3.14 and Poetry refused it ("Current Python version (3.14.7) is not allowed by the project (~=3.13.0)"). I fixed that by installing 3.13.
  - Under tox-uv, the venv still came out as 3.14 even with 3.13 first on PATH. The `tests` env sets `basepython = python`: plain tox uses the interpreter tox runs on, but tox-uv searches for one and found 3.14. Pinning `basepython=python3.13` fixed it locally. On GitHub runners, setup-python installs only the matrix version, so this is likely local-only.

**Steps to Reproduce**

1. Read the install steps on main: `_reusable-test-code.yml` runs `python -m pip install tox tox-gh-actions` (line 72) and `python -m pip install tox` (line 116); `_reusable-test-docs.yml` runs `python -m pip install tox` (line 52). Neither workflow's `inputs:` mentions uv.
2. In a fresh Python 3.13 venv, run that exact install step, then `tox --version`. Registered plugins: only `tox-gh-actions-3.5.0`.
3. Run `tox -ve tests` from the repo. The log shows `create virtual environment via CPython3Posix` (virtualenv, not uv). Result: `31 passed, 2 deselected`, `tests: OK`.
4. Repeat steps 2–3 with `tox-uv` added to the install, which is what the issue asks for. `tox --version` now lists `tox-uv-bare-1.37.7 ... with uv==0.13.7`, and the venv is created by `uv venv`. The run then fails at once: `.tox-B/tests/bin/python: No module named pip`, `tests: FAIL code 1`.
5. Re-run step 4 with `-x testenv.uv_seed=true` (plus `basepython=python3.13`, see setup). Result: `31 passed, 2 deselected`, `tests: OK`.

**Reproduction Evidence**

- **Commit showing reproduction:** fork not created yet (pending)
- **Screenshots/logs:** full logs and a summary are in my local evidence file (`~/tektronix-354-repro/evidence.txt`, plus `run*.log` for each run). Timings for the `tests` env: current setup 49.8 s with cold caches, then 17.2 s with warm caches; tox-uv + `uv_seed` 18.0 s with warm caches.
- **My findings:** The missing option reproduces: the workflows give no way to use tox-uv. The bigger finding is that just adding `tox-uv` to the install step breaks a project like this one. The repo's own tox config sets `install_command = python -I -m pip install ...`, and venvs created by tox-uv don't include pip unless `uv_seed = true`. With seeding, the tests pass, but I measured no speedup on this repo (18.0 s vs 17.2 s): installs still go through pip and Poetry, so uv only creates the venv. So the opt-in needs to be documented with this caveat, or the workflow has to handle seeding. Before writing the plan, I want to ask the maintainer which they prefer.

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

**2026-11-11**

Posted my claim on #354 and started the reproduction. I mirrored the workflow's install step in fresh venvs with and without `tox-uv` and ran the repo's `tests` env with each. Wrong turn: my first runs used Python 3.14, which the project rejects, so I installed 3.13 with uv. Finding: `tox-uv` alone fails on this repo with "No module named pip", because the repo's tox config forces pip as the install command. `uv_seed = true` fixes that, but it gave no speedup here (18.0 s vs 17.2 s warm). Decision: don't write the plan until I've asked the maintainer whether the opt-in should handle seeding or just document it.

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
