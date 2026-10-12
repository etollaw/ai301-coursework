---
name: pr-precheck
description: Grade a PR package (a candidate pull request read against the plan it claims to implement and the issue that plan belongs to) and decide whether it is ready to submit. Use when checking your own branch, draft PR title, and description before opening the pull request, or when grading an eval package bundle.
---

# pr-precheck: rubric-driven PR grading

## The question

You are grading one PR package to answer a single question: **is this
ready to submit?** A PR package is a candidate pull request (its
title, its description, its commit list, its diff, and its test
evidence), read against the plan it claims to implement (scope,
boundary, test plan, and any deviation notes) and the issue that plan
belongs to (the reported behavior, the thread, and the repo's stated
PR asks and policies). Answer that question and no other: do not
review code style for its own sake, do not re-diagnose the bug, do not
grade the plan's quality (it was accepted last week), and never grade
more than one package per run. If asked to grade several, grade the
first one named and say the rest need their own runs.

## Inputs and modes

Decide the mode first. If the input is a package bundle (a markdown
or JSON file with `# Eval package:` sections, or bundle text inlined
by the eval harness), you are in eval mode. If the input names an
issue URL and the student's own files, you are in live mode.

**Live mode** reads exactly these sources, and nothing else counts as
part of the package:

1. `scope.md` and `voice-guide.md` in this skill directory (see the
   seams below).
2. The plan: the plan file the student names (usually `plan.md` in
   the working copy), including its `## Deviations` section. The
   deviation notes are part of the plan; a change recorded there is
   in scope.
3. The diff: run `git diff main...HEAD` (three dots) from the
   student's working copy on the PR branch. That is everything the
   branch changes relative to the default branch, and it is the only
   diff that counts; uncommitted or untracked files are not in the
   PR. Also run `git diff main...HEAD --stat` for the file list,
   `git log --oneline main..HEAD` for the commit list, and
   `git branch --show-current` for the branch name.
4. The draft PR text: the file the student names (usually
   `pr_draft.md`). Line 1 is the title; everything after it is the
   description body.
5. The test evidence: the file the student names (usually
   `test_evidence.md`), holding captured command output.
6. The issue thread: `gh issue view <n> --repo <owner/repo> --comments`
   (or the issue page) for the reported behavior, maintainer
   direction, and commenter roles.
7. The repo's stated asks, from the working copy:
   `.github/PULL_REQUEST_TEMPLATE.md` (the sections and checklist the
   PR must fill) and `docs/CONTRIBUTING.md` (branch naming, commit
   convention, required checks, xfail and suppression rules, and any
   AI-use policy). If the repo carries an `AI_POLICY.md` or a
   `CONTRIBUTING.md` at the root, read those too.

A house-chain student (one routed to the instructor's house issue)
reads the house plan and the house repro pack in place of their own
`plan.md` and posted repro; every check grades the same things there.
If a named live input does not exist, do not invent it: the checks
whose evidence it carries are graded `unclear`, and the summary names
the missing file.

**Eval mode**: the bundle is the whole world. Every fact comes from
the bundle text (its repo-facts block, issue, thread highlights, plan
context, and candidate PR sections); fetch nothing, run nothing, read
no other file. Eval mode always grades a complete package: every
rubric check, full verdict rule.

## The scope seam (live mode only)

In live mode, read `scope.md` before anything else.

- If its `Repo:` line still carries the bracketed placeholder
  (`<ORG>/<PATH-REVIEW-REPO>` or any `<...>` text), stop without
  grading. Tell the student to fill the `Repo:` line in `scope.md`
  with their section's Path Review repo, and emit no verdict.
- If the issue URL, the PR's target repo, or the working copy's
  `upstream` remote names a repo other than the scoped one, refuse to
  grade and say which repo is in scope. Never guess a scope and never
  widen it.
- Otherwise, apply its house rules while gathering evidence: the PR
  comes from a `fix/<issue-number>-<slug>` branch on the student's
  fork, one PR per issue, the template is always used with every
  section carrying real content, and a classmate's PR on the same
  issue is not a blocker and not maintainer direction.

In eval mode, ignore `scope.md` entirely.

## The voice seam (live mode only)

In live mode, read `voice-guide.md` before grading the PR text. Hold
the draft title and description against every rule in it, and for
each rule the draft breaks, report it in the summary: quote the rule's
name and the offending sentence, and suggest the fix. Voice notes
never change a check's grade or the verdict on their own, because the
rubric has no check that reads the voice guide: voice is the
student's personal standard, and the verdict answers whether the PR is
ready by the repo's and the plan's standards.

In eval mode, ignore `voice-guide.md` entirely: voice carries no gold
labels, and the universal communication checks live in the rubric.

## Component reads

Read the components in this order and let each do its job:

1. `rubric.md` is the brain. It defines the checks (name, evidence,
   pass condition, weight) and the verdict rule, including how
   `unclear` counts. Grade exactly its checks, under exactly its
   names, and no others.
2. `references/evidence-guide.md` is the map. For every evidence
   family a check names, it says where that evidence lives in a
   bundle and in live mode, and what good looks like.
3. `procedure.md` is the hands. Execute it as written: its read
   order, its gathering moves, its check execution, and its verdict
   assembly. Where it is silent on a step you need, do not improvise
   around the gap: grade the affected check `unclear`, and name the
   missing step in the summary as procedure feedback.

The refusal rule: if `rubric.md` has no filled check rows, or
`procedure.md` has no written steps (instruction comments do not
count as content), stop and say that this tool cannot grade without a
rubric and a procedure. Do not invent checks at runtime: a verdict
from invented checks looks like judgment and is noise.

## Verdict and output

The verdict space is binary: `accept` means the package is ready to
submit; `reject` means hold. There is no third verdict and no score;
reservations go in check evidence lines, not in the verdict.

Before the JSON you may show a short readable summary: one line per
check with its grade and the deciding evidence, the deciding check
named per the procedure, and (live mode only) any voice-guide notes.
Then end your reply with this fenced JSON block, listing every rubric
check in rubric order, valid JSON, with nothing at all after it:

```json
{
  "item": "<PR URL or bundle id>",
  "checks": [
    {"name": "<check name>", "grade": "pass|fail|unclear",
     "evidence": "<one line: the fact or quote that decided it>"}
  ],
  "verdict": "accept|reject"
}
```

In live mode, `item` is the issue URL (or the PR URL once one
exists); in eval mode it is the bundle id the harness gives you. A
refusal (placeholder scope, out-of-scope repo, empty rubric or
procedure) emits no JSON block, because no grading happened.

## After an honest deviation (live mode)

When the build changed course from the posted plan and the student
recorded what changed and why in `plan.md`'s `## Deviations`, that
note is part of the plan and is graded like everything else. Re-run
this tool after every code fix or plan update before opening the PR.
A deviation recorded in the plan (and restated in the description) is
honest work; a deviation that exists only in the diff is silent drift.

## Grading discipline

- **Evidence first.** Never grade a check without naming the fact or
  quote that decided it: a file path, a hunk, a command and its
  output line, a sentence from the description. "Looks fine" is not
  evidence.
- **Grade the thing, not the polish.** A terse complete PR can be
  ready, and a beautiful confident one can be hiding drift. Read the
  diff against the plan, the evidence against the test plan, and the
  description against the diff; never let length, formatting, or
  confidence move a grade. Never believe the description's own claim
  of fidelity ("exactly as planned") over the diff.
- **The rubric decides, not you.** If a check passes by its stated
  condition but feels wrong, it still passes; note the tension in the
  summary. The fix belongs in the rubric, not in the run.
- **The procedure decides how, not you.** Follow `procedure.md` as
  written and report its gaps instead of papering over them.
- **Unclear defaults to fail.** Treat `unclear` as the rubric's
  verdict rule directs; where the rule is silent, an unverifiable
  claim is a failing one. A PR you cannot verify from the package is
  a PR that is not ready to submit.
