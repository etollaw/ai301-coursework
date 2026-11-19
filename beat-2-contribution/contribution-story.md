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
**Pull Request:** https://github.com/tektronix/python-package-ci-cd/pull/1329

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

**Unit 6: Merged my voice guide into one file, gave it upstream rules, and wrote repo memory for tektronix**

I folded my Unit 2 and Unit 4 voice guides into one guide at `~/.claude/voice-guard/`. I added Upstream rules for claims, questions and pings, using my two real comments on #354 as the "Right" examples. I also wrote a `CLAUDE.local.md` for my clone, recording the signing requirement, the Conventional Commits format and the tox commands. What fell short: I posted both comments on #354 before the guard was installed, so neither was checked. The guard's first real check was a manual run on my PR description on 2026-11-18, which passed. Writing repo memory also surfaced that my global git email belonged to another GitHub account (ELDORADO18), so I switched this clone to etollaw's noreply email so the PR would be credited and verified correctly.

**Unit 7: Wrote a four-check ship manifest for tektronix and got the ship-gate from STOP to PASS**

My manifest runs the repo's own checks from a fresh shell: `tox -ve tests` with and without tox-uv, `tox -ve docs,doctests`, and `pre-commit run --all-files`. Building it caught a real bug before review: the strict docs build failed on the word "uv" in my new docs, so I added it to `doc_config/known_words.txt`. The docs row also failed only because mkdocs read my local `CLAUDE.local.md`, so that row now builds from a clean worktree of HEAD. The first gate run stopped: my pr-precheck rejected the branch on plan-fidelity and reviewable-diff, because my local `main` was stale and the diff pulled in upstream's renovate and SBOM commits, and on repo-standards, because I hadn't said why the README was unchanged. I reset `main` to upstream and added the README reason, and the second run passed.

**Unit 8: Installed the PR monitor and wrote a playbook from tektronix's real rhythm, which had no outside PRs to measure**

The playbook's staleness rule was the hard part: the last 300 PRs in the repo are all from the maintainer or bots, so there were no five outside PRs to time. I based it on the one data point I had, nfelt14 answering my question on #354 in about 7 hours, and set 7 quiet days before a first nudge and 10 after a review round. `monitor.py --state` read the repo correctly (tektronix/python-package-ci-cd, not my fork) and reported all four playbook sections filled. It reported NO PR YET, because opening the PR through Claude Code was blocked by Claude Code's auto-mode safety check, and I opened it myself.

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

- Tool: CLAUDE.local.md repo memory (Unit 6), plus Claude Code reading the two reusable workflows and their docs pages in my clone
- What it produced: the four fields above, and a "subsystem" section in my repo memory naming the 2 workflows, 2 callers, 2 docs pages, and the tox config in pyproject.toml
- Difficulties: the repo has no AGENTS.md or CLAUDE.md, and no tests cover the workflows directly, so understanding came from reading YAML, not from tests
- What changed: I wrote the conclusions into CLAUDE.local.md so later sessions start from them instead of re-reading the workflows

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

- **Commit showing reproduction:** https://github.com/etollaw/python-package-ci-cd/commit/2774f9f38d6b79125ac6538b134bc73cfbce1d4b (branch `repro/354-tox-uv` on my fork: adds `tox-uv` to both install steps in `_reusable-test-code.yml`). Its CI run, https://github.com/etollaw/python-package-ci-cd/actions/runs/42998863934, failed in all 6 test jobs (test-fast and test-general on ubuntu, macos, windows) with `.tox/tests/bin/python: No module named pip` / `tests: FAIL code 1`, the same failure I got locally.
- **Screenshots/logs:** full logs and a summary are in my local evidence file (`~/tektronix-354-repro/evidence.txt`, plus `run*.log` for each run). Timings for the `tests` env: current setup 49.8 s with cold caches, then 17.2 s with warm caches; tox-uv + `uv_seed` 18.0 s with warm caches.
- **My findings:** The missing option reproduces: the workflows give no way to use tox-uv. The bigger finding is that just adding `tox-uv` to the install step breaks a project like this one. The repo's own tox config sets `install_command = python -I -m pip install ...`, and venvs created by tox-uv don't include pip unless `uv_seed = true`. With seeding, the tests pass, but I measured no speedup on this repo (18.0 s vs 17.2 s): installs still go through pip and Poetry, so uv only creates the venv. So the opt-in needs to be documented with this caveat, or the workflow has to handle seeding. Before writing the plan, I want to ask the maintainer which they prefer.

**Toolkit**

- Tool: Claude Code in my clone and in fresh venvs that mirror the workflow's install step; GitHub Actions on my fork for the CI repro. I didn't run repro-check: I posted my repro findings on #354 before installing the Unit 6 tools.
- What it produced: logs for each run in ~/tektronix-354-repro, the fork commit, and a CI run failing in all 6 test jobs with "No module named pip"
- Difficulties: Python 3.14 vs the project's ~=3.13.0 pin; tox-uv picking the wrong interpreter for `basepython = python`; my first CI trigger edit hit the wrong line and needed a second commit
- What changed: pinned basepython locally and recorded why; re-ran the baseline with warm caches so the timing comparison was fair

---

## Solution Approach

**Analysis**

Two things cause it. First, the install steps are hard-coded: `_reusable-test-code.yml` runs `python -m pip install tox tox-gh-actions` in its test-general job and `python -m pip install tox` in test-fast, and `_reusable-test-docs.yml` runs `python -m pip install tox`. No input can add a plugin. Second, adding tox-uv on its own isn't enough for every project. tox-uv creates venvs with `uv venv`, which has no pip, so a tox config that overrides `install_command` with a pip command fails with `No module named pip`. This repo does exactly that in `pyproject.toml` (`[tool.tox] legacy_tox_ini`), which is how I reproduced the failure locally and in CI.

**Proposed Solution**

Add an opt-in `use-tox-uv` input (default `false`) to both reusable workflows. When it's true, each install step appends ` tox-uv`. I'm following nfelt14's answer on #354: document the caveat for overridden install commands, and remove this repo's own override. Deliberately not changing: no automatic pip seeding (I tested that `TOX_OVERRIDE=testenv.uv_seed=true` would work, but the maintainer chose documentation), no change to the default (existing callers behave exactly as before), and no README change (the README only links to the workflow pages, where inputs are documented).

**Implementation Plan**

1. `_reusable-test-code.yml` and `_reusable-test-docs.yml`: add the `use-tox-uv` boolean input and append `${{ inputs.use-tox-uv && ' tox-uv' || '' }}` to all three install steps.
2. `workflows/test-code.md` and `workflows/test-docs.md`: add the input to the Inputs table and Example, plus a note on the `install_command` caveat.
3. `pyproject.toml`: remove the `install_command` override.
4. `test-code.yml`: set `use-tox-uv: true` so CI runs the new path; `test-docs.yml` keeps the default.
5. `CHANGELOG.md`: Unreleased "Added" and "Changed" entries.
6. No test changes: the repo has no tests for workflow YAML, so CI on the PR runs the workflows themselves.

Deviations:
- 2026-11-18: added `uv` to `doc_config/known_words.txt`. The strict mkdocs spellcheck failed on it in my new docs text. It wasn't in the plan I posted.
- 2026-11-18: my first CHANGELOG edit left a blank line inside a list after rebasing, and mdformat split the list, so I moved my bullet to the end of upstream's "Changed" list.

**Toolkit**

- Tool: the maintainer's answer on #354 as the design decision, and my pr-precheck's plan-fidelity check (run by the ship-gate) to hold the diff to plan.md
- What it produced: plan.md with six changes, and a plan-fidelity pass on the second gate run
- Difficulties: the first gate run failed plan-fidelity because my local main was stale, so the diff showed upstream's commits as if they were mine
- What changed: reset local main to upstream before gating; recorded the known_words.txt deviation in plan.md and the PR description

---

## Testing Strategy

- **Repro re-run, before and after:** on main with tox-uv added, `tox -ve tests` fails at once with `.tox-B/tests/bin/python: No module named pip` / `tests: FAIL code 1`. On my branch with tox-uv and no `uv_seed`, it gives `31 passed, 2 deselected` / `tests: OK`.
- **The old path still works:** on my branch with plain tox, `tox -ve tests` gives `31 passed, 2 deselected` / `tests: OK`.
- **Docs:** `tox -ve docs,doctests` gives `docs: OK`, `doctests: OK`. This matters because the docs build runs in strict mode, and it failed before I added `uv` to the known-words list.
- **Lint:** `pre-commit run --all-files` exits 0, covering yamlfix, mdformat, actionlint, ruff and the rest.
- **In CI, not locally:** the windows and macos jobs and the workflow expression itself run on the PR, with `use-tox-uv: true` set in `test-code.yml`.

**Toolkit**

- Tool: the ship-gate (Unit 7) running my ship-manifest.md from a fresh shell
- What it produced: four passing checks on the final commit: tests with plain tox (ok), tests with tox-uv (ok), docs and doctests (ok, 49s), pre-commit on all files (ok, 13s)
- Difficulties: mkdocs reads every Markdown file in the clone, including my local CLAUDE.local.md and ship-manifest.md, so the docs row failed for reasons unrelated to my change
- What changed: the docs row builds from a clean `git worktree` of HEAD and removes it afterwards

---

## Implementation Notes

**2026-11-11**

Posted my claim on #354 and started the reproduction. I mirrored the workflow's install step in fresh venvs with and without `tox-uv` and ran the repo's `tests` env with each. Wrong turn: my first runs used Python 3.14, which the project rejects, so I installed 3.13 with uv. Finding: `tox-uv` alone fails on this repo with "No module named pip", because the repo's tox config forces pip as the install command. `uv_seed = true` fixes that, but it gave no speedup here (18.0 s vs 17.2 s warm). Decision: don't write the plan until I've asked the maintainer whether the opt-in should handle seeding or just document it.

**2026-11-11 (later)**

Tested how a workflow could turn seeding on without editing a project's tox config. `TOX_UV_SEED=true` does nothing (no `--seed`, still "No module named pip"). `TOX_OVERRIDE=testenv.uv_seed=true` works, and the tests pass. I posted this to the maintainer and asked whether the input should enable seeding or just document the caveat: https://github.com/tektronix/python-package-ci-cd/issues/354#issuecomment-6792674016. I also forked the repo and pushed a repro branch. On GitHub runners it fails in all 6 test jobs, on all three operating systems, with the same "No module named pip" error.

**2026-11-17**

nfelt14 had replied on #354 (2026-11-11) and assigned me: "I would just document the caveat for overridden install commands. This repo's install command override should just be removed." That settled the design. I set up SSH commit signing, which CONTRIBUTING requires, and found my git email mapped to another GitHub account (ELDORADO18). I switched this clone to etollaw's noreply email, and GitHub now shows my commit as Verified, by etollaw.

**2026-11-18**

Built the change on `feat/354-optional-tox-uv`, rebased on upstream main, and made one signed `feat:` commit. Two wrong turns: a blank line in my CHANGELOG bullet made mdformat split the list, and the strict docs build failed on "uv" until I added it to `known_words.txt`. Installed the voice guard, the ship-gate and the PR monitor. The ship-gate stopped the first run (stale local main; no README reason) and passed the second.

**Toolkit**

- Tool: Claude Code ran each step; I kept the logs in ~/tektronix-354-repro and dated each decision here as it happened
- What it produced: the dated entries above
- Difficulties: the repo's pre-commit git hook blocked an amend because `poetry` wasn't on PATH, which silently left a file out of my commit until I checked `git show --stat`
- What changed: I now check `git show --stat` after every amend and run commits with the tox env's poetry on PATH

---

## Pull Request

**PR Link:** https://github.com/tektronix/python-package-ci-cd/pull/1329

**PR Description:** Title: "feat: add optional use-tox-uv input to the testing workflows". It follows the repo's PR template. Proposed changes covers the input, the documented `install_command` caveat, and removing the override, as agreed on #354, with `Addresses #354` and one bullet per changed file. It also says why the README is unchanged, and records the `known_words.txt` deviation. Testing pastes the before/after output (`No module named pip` on main with tox-uv; `31 passed` on the branch with and without tox-uv; docs and doctests OK; pre-commit exit 0) and links the fork CI run, then names what runs only in CI. It ends with an AI disclosure saying I used Claude Code to reproduce, edit, run checks and draft, and that I reviewed the diff and output. The CLA and "commented my code" boxes stay unticked, each with a reason.

**Maintainer Feedback:**

- 2026-11-19: none yet. PR opened; CodeFactor and pre-commit.ci passed, and the test workflows are waiting for a maintainer to approve them, as they do for a first-time contributor's PR.

**Status:** Awaiting review

**Toolkit**

- Tool: the ship-gate (Unit 7) before the PR, and the voice guard (Unit 6) on the description
- What it produced: ship-gate PASS on the second run; voice guard "pass. Your rules hold; you can post it." on pr_draft.md
- Difficulties: the first gate run rejected the branch (stale main, missing README reason). Opening the PR through Claude Code was blocked by Claude Code's auto-mode safety check, so I opened it from a fresh Claude Code session where both hooks were active. That session's first attempt was stopped by the ship-gate because it passed the body file as an unexpanded shell variable; it reran with the full path and the gate let it through. It also appended a "Generated with Claude Code" line to the body, which I removed with `gh pr edit` because my disclosure was already in the description
- What changed: fixed both gate findings before opening; restored the PR body to the exact file the gate and voice guard had passed

---

## Learnings and Reflections

**Technical Skills Gained**

- How reusable GitHub Actions workflows take inputs, and how to make a step conditional on a boolean input inside a `run:` line (`${{ inputs.use-tox-uv && ' tox-uv' || '' }}`).
- How tox-uv changes tox: venvs come from `uv venv` without pip, so a pip-based `install_command` breaks. `uv_seed = true` fixes that, and `-x` / `TOX_OVERRIDE` can set config without editing files.
- That `basepython = python` resolves differently under virtualenv (tox's own interpreter) than under tox-uv (it searches PATH), which matters on a machine with several Pythons.
- SSH commit signing, and that GitHub verifies a signature only if the commit email belongs to the account holding the key.

**Challenges Overcome**

- **Making sure the issue was real.** My first "repro" failed for an unrelated reason (Python 3.14). Installing 3.13 showed the real failure, and running it on my fork's CI confirmed it on all three operating systems.
- **Not guessing the design.** I asked one either/or question with my numbers, and the maintainer's answer cut the scope down to documenting the caveat.
- **Hidden failures.** The strict docs build and a pre-commit hook that silently blocked my amend would both have shipped a broken PR. I caught them because the ship manifest runs the repo's real checks, and because I checked `git show --stat` after amending.

**What I'd Do Differently Next Time**

I'd fix my scout's search strategy before scanning, since its `--updated` cutoff hid the very issue I picked. I'd also install the Unit 6–8 tools in the week they were assigned. My claim and repro comments went out before the voice guard existed, and I fell behind the deadline. And I'd check `git config user.email` on day one, not after a week of commits credited to the wrong account.

---

## Resources Used

- nfelt14's answer on #354, which decided the design: https://github.com/tektronix/python-package-ci-cd/issues/354
- tox-uv README, for `uv_seed` and how it creates venvs: https://github.com/tox-dev/tox-uv
- tox docs on config overrides (`-x` / `TOX_OVERRIDE`), which I used to test seeding without editing the config: https://tox.wiki/en/stable/config.html
- tektronix's CONTRIBUTING.md (signing, Conventional Commits, CHANGELOG): https://github.com/tektronix/python-package-ci-cd/blob/main/CONTRIBUTING.md
- GitHub docs on SSH commit signing: https://docs.github.com/en/authentication/managing-commit-signature-verification/about-commit-signature-verification
