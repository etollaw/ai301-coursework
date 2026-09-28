# Rubric: is this reproduction package ready to post?

<!--
THIS IS THE PART YOU WRITE. The skill in SKILL.md executes whatever
checks you define here. It ships empty on purpose: the judgment is your
work.

A filled rubric must contain:

1. At least one row in the checks table. Each row needs all four
   columns:
   - Check: a short name (used in the output JSON).
   - Evidence: exactly what to look at, and where in the package. Name
     the part (the claim comment, the repro report's environment
     record, the artifacts read against the issue's description, the
     repo-facts block) or a location from your
     references/evidence-guide.md. "The report" is not a source; "the
     output excerpt read against the error the issue describes" is.
   - Pass condition: a decision rule about the OUTCOME that someone
     else could apply and get your answer. Judge the thing itself (does
     the artifact show the issue's behavior?), never the write-up's
     shape (how many steps it has, how long it is, whether it uses a
     template's headings). Structure-shaped checks are what make
     graders disagree with themselves.
   - Weight: `required` (a fail here holds the package) or `preferred`
     (never changes the verdict).

2. A verdict rule below the table: how the check grades combine into
   accept (ready) or reject (hold), including how `unclear` is
   treated. The verdict space is binary. If you write no rule for
   `unclear`, the skill treats it as fail.

Cover what actually gets bad packages posted. The lecture named the
proof families: the environment is recorded, the steps are complete
and followable, the behavior shown matches the issue (not an adjacent
one), the outcome is stated honestly (an evidenced cannot-reproduce is
a pass, a confident wrong-target is not), and the words respect the
repo's conventions. A rubric that ignores a family will fail eval
packages designed around that family.
-->

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| env-recorded | The repro report's environment record (OS, the project's version, and the runtime/toolchain the issue's bug depends on), read against the environment the issue targets (see evidence guide: Environment) | The report names the OS and the exact project version it ran, plus any setting the issue says changes the failure (driver, build profile, shell, dependency version). If the version or setting differs from what the issue targets, the report says so in words. Fail if there is no environment record at all, even when the steps and output look right, or if a deviation from the issue's target is left silent. | required |
| steps-rerunnable | The repro report's commands and inputs, read from starting state to trigger (see evidence guide: Steps) | A stranger with only public materials could re-run the attempt: the exact commands or inputs are given (or a clearly described public setup), and every input the issue names as the trigger is included. Fail if any step depends on private code, an unshared config, or a machine only the author has, or if the steps leave out the trigger setting the issue says matters. | required |
| behavior-matches-issue | The artifacts in the repro report (output excerpts, logs, exit codes, error text) read against the behavior and trigger the issue describes (see evidence guide: Behavior shown) | Either (a) an artifact shows the issue's own behavior (the same error, symptom, or exit code) produced by the issue's own trigger, or (b) the report states it could NOT reproduce and shows the artifact of a real attempt at the issue's trigger. Fail if there is no artifact, if the artifact shows an adjacent behavior (a different error, a syntax/validation error from a changed input, an old version's behavior), or if the artifact contradicts what the report says it shows. Read the artifact itself; formatting and confidence do not count. | required |
| claims-backed | Every factual claim in the claim comment and repro report (reproduced, root cause, frequency, scope, "verified", "guaranteed"), each matched to a shown artifact (see evidence guide: Honesty) | Every claim that something happens, was verified, or is caused by X is backed by an artifact shown in the package, and the stated outcome matches the artifacts (an evidenced cannot-reproduce that names what differed passes). A side observation that supports the outcome but is not the outcome itself (for example, a control run described in words without its own output) does not fail this check when the reported outcome is backed by a shown artifact; the missing control output is what control-run records. Fail if a root cause, certainty, or generalization to environments not tested is asserted with nothing shown, or if expected vs actual is stated in a way the artifacts contradict. | required |
| claim-specific | The candidate claim comment, read against the issue's title and body (see evidence guide: Comms) | The claim comment names something specific to this issue (its function, error, symptom, or trigger) and states a concrete next step the commenter will take, framed as intent. Fail if it is a +1 or me-too, if it is boilerplate that would fit any issue ("please assign me"), or if it promises a fix, a guaranteed outcome, or a delivery date. | required |
| ai-disclosure | The repo-facts block's contribution policy line (AI policy), read against the text of the claim comment and repro report (see evidence guide: Comms) | Treat the package's comments as AI-assisted work. If the stated policy requires disclosing AI use in issue comments or in any contribution without limiting it to pull requests, a comment must disclose the assistance (the tool and its extent). Pass if there is no AI policy, if the policy's disclosure ask covers only pull requests, or if the policy only asks that comments be in the author's own words and the comments read as specific, first-person writing. | required |
| control-run | The repro report's artifacts | The report shows a control run (the same steps without the trigger) whose result differs from the failing run. | preferred |

## Verdict rule

Accept only if every required check passes. `unclear` on a required
check counts as fail, with one exception from the skill: in a live
claim-only draft, the checks that need the repro report (env-recorded,
steps-rerunnable, behavior-matches-issue, and the report half of
claims-backed) are `not yet applicable` and are left out of the verdict.
`control-run` is preferred and never changes the verdict.
