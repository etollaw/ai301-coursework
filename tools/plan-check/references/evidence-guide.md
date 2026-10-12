# Evidence guide: where evidence lives in a plan package

<!--
THIS IS THE PART YOU WRITE (second week running: the judgment files
stay in your hands). The skill uses this guide as its map: for every
kind of evidence a rubric check names, this file says WHERE to find it
in a plan package and WHAT GOOD LOOKS LIKE when you do.

Under each family heading below, write:

- Where it lives: the exact places to look. In an eval bundle (which
  section of the package: the issue context, the repro-evidence block,
  the candidate plan's scope statement or test plan, the plan comment,
  the repo-facts block). In live mode (where on GitHub or in the
  draft: the issue thread, the student's posted repro comment, the
  repo's docs, the draft plan and comment).
- What good looks like: one or two sentences someone else could apply.
  Prefer observable conditions ("the stated cause cites behavior the
  repro evidence actually shows") over adjectives ("diagnosis is
  solid").

A rubric check whose evidence this guide cannot locate is a check
nobody else can execute, and this week that cuts twice: your
procedure.md tells the skill WHEN to gather each family, and this
guide tells it WHERE. Write the map you wish your executor had.
-->

## Diagnosis and grounding

- Where it lives: in an eval bundle, the candidate plan's `Cause:`
  sentence or its `### Diagnosis` section; the behavior it must explain
  is in the repro-evidence block, specifically its numbered steps, its
  `Actual:` line, and its artifacts (output excerpts, timings, exit
  codes, panics, tracebacks). The control runs are the deciding part:
  they appear in the repro-evidence block as a `Control:` line or as a
  numbered step that changes or removes one variable. A cause the
  thread proposed is in the thread highlights, but the thread is not
  evidence for it; the repro-evidence block is.
- What good looks like: the mechanism the plan blames, if switched off,
  would stop the behavior the artifacts show, and every control run in
  the package is consistent with that. Read the controls as a test of
  the plan's claim: if the blamed component is already out of the loop
  in a step that still fails (bat's step 3 at 25.8s with
  `--paging=never`, so pager bindings cannot be the cost), or present
  in a control that passes (yq's top-level `[.a] | length == 1`
  synthesizing null fine), the diagnosis is ruled out no matter how
  confidently it is written or how many people in the thread agreed.
  A plan whose cause the evidence supports but does not fully prove is
  still grounded, as long as no step or control contradicts it.
- Live: the plan's diagnosis is in `plan.md`; the behavior it must
  explain is in the student's own posted repro comment on the issue
  (their week-2 report), not in a fresh local run and not in the issue
  body. On the house issue with no posted repro of the student's own,
  the repro evidence is the house repro pack as quoted in the drafts;
  if the drafts quote none, there is no grounding evidence to read.

## Scope

- Where it lives: in an eval bundle, the candidate plan's `Scope`
  section (its in-scope sentence and its `Not in scope:` line) plus
  every numbered item under `Changes`, `Approach`, or `Plan`, since
  what the plan commits to building is the real scope and the heading
  is only a label. The yardstick is the issue section's reported
  behavior and the single behavior the repro evidence isolates.
- What good looks like: every committed item traces back to the one
  reported behavior, and work beyond it is either absent or named on
  the not-in-scope line. One bounded change looks like lazygit's
  "add the commits context to the post-push refresh scope; out: how
  push status is computed" — one site, one behavior, the neighbours
  explicitly left alone. A drive-by rewrite looks like the correct
  one-constant fix arriving with an HTTP-client migration, a settings
  panel, and a retry framework attached: the fix is in there, but the
  plan commits to four things the reported behavior never asked for.
  Deliberately fixing less than the issue asks is not a scope fault
  when the plan says what it is deferring and why; a not-in-scope line
  that defers the deep fix is scope working, not scope shrinking.
- Live: `plan.md`'s Scope section including its out-of-scope lines,
  read against the issue body on GitHub. Housekeeping the plan must
  own counts as in scope when the issue's own test setup carries it:
  an `xfail` marker or a config suppression that the issue's test is
  parked behind belongs in the plan's scope, because removing the
  marker is part of making the fix observable.

## Executability

- Where it lives: in an eval bundle, the plan's named files, functions,
  or call sites (anywhere in the plan: the scope line, the approach
  steps, or the diagnosis), and the ordered steps under `Approach` or
  `Changes`. Read against the code locations the issue and the repro
  evidence point at (a traceback's frames, a panic's file:line, the
  culprit a maintainer named in the thread).
- What good looks like: a stranger could make the first edit without
  asking the author a question. That needs two things present: a
  location specific enough to open (`src/output.rs`,
  `crates/core/flags/hiargs.rs`, "the push completion callback in
  `pkg/gui/controllers/sync_controller.go`") and one approach chosen
  rather than surveyed. "Add `recover()` somewhere", "upstream or
  vendored, whichever is easier", and "gocui? tcell? not sure" all
  name the work that remains to be decided, not the work to be done;
  so does a plan whose first action is to profile or investigate. A
  single terse paragraph that names one site and one approach is
  executable; three polished pages that defer every real decision to
  build time are not.
- Live: `plan.md`'s Files and Approach sections, read against the
  actual paths in the fork's working tree. A file path in the plan
  that does not exist in the repo is a location nobody can open.

## Test plan

- Where it lives: in an eval bundle, the plan's `Test plan` or `Test:`
  section, read against the repro-evidence block's numbered steps and
  its artifacts. The test plan's anchor is whatever the repro evidence
  used as its trigger.
- What good looks like: the test plan re-runs the repro's own trigger,
  or encodes it as a test case, and names the observable that will
  differ once the fix lands: an exit code, a specific output line, a
  state or colour change at a named step ("at step 3 the color must
  flip without leaving the view"), a named test that currently fails
  and must pass, a timing with a number. A vague test plan names an
  activity instead of an outcome: "run the full test suite and make
  sure nothing regresses" is a regression guard, not a proof the bug is
  fixed, and "should feel fast", "nothing else should feel broken", and
  "undo works after toggling" name nothing anyone else could check.
  Regression-suite runs are fine as an addition; they do not stand in
  for the per-bug observable.
- Live: `plan.md`'s Test plan section, which must re-run the exact
  commands from the student's posted repro comment and state the
  expected after for each. Where the repro ran a test behind an `xfail`
  marker, the expected after is that test passing without
  `--runxfail`, which only holds if the plan also removes the marker.

## Honesty

- Where it lives: in an eval bundle, the plan's `Risk`, `Risks`,
  `Unknowns`, or `Open questions` lines, plus the certainty words
  anywhere in the plan and comment ("confirmed", "this is caused by",
  "will", "guaranteed"), read against what the repro-evidence block
  actually established. In live mode also the plan's `## Deviations`
  section.
- What good looks like: the things the plan's success depends on but
  has not established are on the page as unknowns, with what the author
  would do about each: an unmeasured hot-path cost flagged with the
  fallback if the benchmark moves, a platform not tested named as
  untested, a behavior change presented as a proposal the maintainers
  may reject. False confidence is the opposite shape: a dependency the
  plan rests on stated as decided, a cause asserted as established
  when only the thread (not the evidence) said so, a cost called
  negligible with nothing measured. Silence is not false confidence: a
  terse plan that claims nothing beyond its evidence is honest.
- Live: `plan.md`'s Risks/unknowns section before posting, and its
  `## Deviations` section after the build. A deviation recorded in the
  plan is honest work; a deviation that exists only in the diff is
  not, so an empty Deviations heading after a build that changed
  course is a honesty fault even when the diff is correct.

## Comms

- Where it lives: two sources, read together against the candidate
  plan comment. (1) The thread highlights, where each entry carries its
  author's role: direction from an `OWNER`, `MEMBER`, `COLLABORATOR`, or
  `CONTRIBUTOR` is maintainer-side direction; a claim from a `NONE`
  participant is not, however confident. (2) The repo-facts block's
  `contribution policy` line and `bug reports` line, which carry the
  stated templates, the contributing asks, and any AI-use policy.
- What good looks like: thread-aware means the comment engages the
  specific direction that is already on the thread, by name. When an
  owner has isolated the culprit to a file and line, posted a patched
  test binary, and asked for testing, a thread-aware comment either
  builds on that patch or says why it is going a different way; a
  comment that proposes a docs-only workaround without mentioning any
  of it is boilerplate that happens to be about the right bug. When a
  maintainer has rejected an approach as too expensive, a thread-aware
  comment does not re-propose it. Where the thread carries no
  maintainer direction at all (a package with zero comments), there is
  nothing to engage and this half is satisfied.
  On policy: treat every package as AI-assisted work. A policy that
  requires disclosing all AI usage, stating the tool and the extent,
  without limiting that ask to pull requests, needs a disclosure line
  in the comment itself; its absence is a convention fault even when
  the plan is excellent. A policy that only makes the contributor
  responsible for what they submit, that asks for disclosure in pull
  requests only, or that asks comments be in the contributor's own
  words, is satisfied by a specific first-person comment with no
  disclosure line. "No stated AI policy" asks for nothing.
- Live: the live issue thread (`gh issue view <n> --comments`, or the
  issue page) for direction and commenter roles, `CONTRIBUTING.md`,
  `AI_POLICY.md`, and `.github/ISSUE_TEMPLATE/` in the repo for the
  stated asks, and `scope.md` for the house rules that override the
  general ones in the Path Review classroom repo. In the classroom
  repo, a classmate's plan comment on the same issue is not maintainer
  direction and does not need engaging; the student's own posted repro
  comment is the evidence their plan must follow from.
