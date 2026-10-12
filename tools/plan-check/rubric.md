# Rubric: is this plan ready to post and build from?

<!--
THIS IS THE PART YOU WRITE. The skill in SKILL.md executes whatever
checks you define here (via your procedure.md). It ships empty on
purpose: the judgment is your work.

A filled rubric must contain:

1. At least one row in the checks table. Each row needs all four
   columns:
   - Check: a short name (used in the output JSON).
   - Evidence: exactly what to look at, and where in the package. Name
     the part (the plan's scope statement, the test plan read against
     the repro evidence's steps, the plan comment read against the
     thread highlights, the repo-facts block) or a location from your
     references/evidence-guide.md. "The plan" is not a source; "the
     plan's stated cause read against what the repro evidence shows"
     is.
   - Pass condition: a decision rule about the OUTCOME that someone
     else could apply and get your answer. Judge the thing itself (is
     this one bounded change? could a stranger start executing it?),
     never the write-up's shape (how many sections it has, how long it
     is, whether it uses headings). Structure-shaped checks are what
     make graders disagree with themselves.
   - Weight: `required` (a fail here holds the package) or `preferred`
     (never changes the verdict).

2. A verdict rule below the table: how the check grades combine into
   accept (ready) or reject (hold), including how `unclear` is
   treated. The verdict space is binary. If you write no rule for
   `unclear`, the skill treats it as fail.

Cover what actually gets bad plans posted. The lecture named the
failure families: the diagnosis ignores or contradicts the reproduced
evidence, the change is unbounded (scope creep), the plan targets the
symptom while the evidence points at the cause, a stranger could not
start executing it, the test plan proves nothing observable, the
unknowns are dressed up as certainty, and the comment ignores what the
thread or the repo's stated conventions ask. A rubric that ignores a
family will fail eval packages designed around that family.
-->

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| grounded-diagnosis | The plan's stated cause (its diagnosis or "Cause:" statement), read against every step, artifact, and control run in the repro-evidence block (see evidence guide: Diagnosis and grounding) | The stated cause explains the behavior the repro evidence actually shows, and nothing in that evidence is inconsistent with it. Fail if a step or control in the repro evidence rules the stated cause out (the blamed component is absent and the failure still happens, or the blamed component is present and the control still passes), if the cause rests on a mechanism the evidence never exercises, or if the plan adopts a cause asserted in the thread that the package's own evidence contradicts. A cause the evidence supports but does not fully prove still passes when no step or control contradicts it; confidence and length do not count either way. | required |
| bounded-scope | The plan's scope statement (its in-scope and not-in-scope lines) together with the changes or approach steps it commits to, read against the one behavior the issue reports and the repro evidence isolates (see evidence guide: Scope) | Everything the plan commits to building serves the one reported behavior; anything beyond it is absent or explicitly deferred as out of scope. Fail if the plan commits to a refactor, rewrite, migration, dependency upgrade, new user-facing option, new abstraction, or CI/test-harness change that the reported behavior does not require, even when a correct narrow fix sits inside it. A plan that deliberately fixes less than the issue asks and names what it is deferring passes. | required |
| executable-approach | The files, functions, or call sites the plan names and the approach or order of work it commits to, read against the code locations the issue and repro evidence point at (see evidence guide: Executability) | The plan names where the change goes (a file, function, or clearly identified call site) and which approach it chose, so a stranger could make the first edit without asking the author to decide anything. Fail if the location is deferred to build time ("somewhere in the editor code", "upstream or vendored, whichever is easier"), if the approach is still a list of candidate layers or an open question ("gocui? tcell? not sure"), or if the first concrete action is to investigate rather than to change something. One named site and one chosen approach pass even in a single terse paragraph. | required |
| decisive-test-plan | The plan's test plan, read against the repro evidence's own steps and artifacts (see evidence guide: Test plan) | The test plan re-runs the repro evidence's trigger (or encodes it as a test) and names the observable result that separates fixed from unfixed: a specific output, exit code, state or color change, assertion, or a named test that currently fails and must pass. Fail if the only verification is running an existing suite, build, or linter with no outcome named for this bug, or if the stated outcome is unobservable or subjective ("should feel fast", "nothing else should feel broken", "undo works after toggling"). | required |
| thread-and-convention | The candidate plan comment, read against (a) the issue's thread highlights for direction already given by a maintainer, owner, or collaborator, and (b) the repo-facts block's contribution policy and its bug-report/AI-disclosure asks (see evidence guide: Comms) | Both clauses hold. (a) Where a maintainer, owner, or collaborator has stated a direction, a diagnosis, a rejected approach, or a request in the thread, the comment addresses it: adopting it, building on it, or saying why the plan departs from it. (b) Treat the plan and comment as AI-assisted work: if the repo's stated policy asks that AI use be disclosed in contributions or comments without limiting that ask to pull requests, the comment names the assistance and its extent. Fail if the comment proceeds as though maintainer direction in the thread were not there, or if a disclosure the stated policy requires is absent. Clause (a) passes when the thread carries no maintainer direction; clause (b) passes when there is no AI policy, when the policy's disclosure ask covers only pull requests, or when the policy only asks that comments be the author's own words and the comment reads as specific first-person writing. | required |
| honest-uncertainty | The plan's risks and unknowns (or their absence) and any certainty words in the plan and comment, read against what the repro evidence has actually established (see evidence guide: Honesty) | A dependency the plan rests on but has not established (an unmeasured cost, an untested platform, a behavior change needing maintainer buy-in, a deferred question) is named as a risk or open question rather than presented as settled. A terse plan that claims nothing beyond its evidence passes. Fail only when the plan asserts as decided something the evidence has not shown and the plan's own success depends on. | preferred |

## Verdict rule

Accept only if every `required` check passes: `grounded-diagnosis`,
`bounded-scope`, `executable-approach`, `decisive-test-plan`, and
`thread-and-convention`. A single required fail is a reject, and no
number of passes elsewhere buys it back.

`unclear` (reported as `?`) is the grade for a required check whose
deciding evidence is absent or too thin to apply the pass condition
to. `?` on a required check counts as a fail and rejects the package:
a plan that cannot be verified from the package is a plan that is not
ready to build from. `?` is not a middle verdict and it is never
rounded up to a pass; the output must still name what was missing.

`honest-uncertainty` is `preferred` and never changes the verdict, in
either direction. It is reported with its grade and evidence so the
author sees it, and a `?` on it is reported as `?` and otherwise
ignored.

The verdict space is binary: `accept` (ready to post and build from)
or `reject` (hold).
