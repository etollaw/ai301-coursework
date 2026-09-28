# Evidence guide: where proof lives in a reproduction package

<!--
THIS IS THE PART YOU WRITE (new this week: Unit 1 handed you this file
finished; the scaffolding fades). The skill uses this guide as its map:
for every kind of proof a rubric check names, this file says WHERE to
find it in a package and WHAT GOOD LOOKS LIKE when you do.

Under each family heading below, write:

- Where it lives: the exact places to look. In an eval bundle (which
  section of the package: the issue context, the repo-facts block, the
  claim comment, the repro report and its parts). In live mode (where
  on GitHub or in the draft: the issue thread, the repo's docs, the
  student's draft comment).
- What good looks like: one or two sentences someone else could apply.
  Prefer observable conditions ("the versions named match what the
  issue targets, or the difference is called out") over adjectives
  ("environment is thorough").

A rubric check whose evidence this guide cannot locate is a check
nobody else can execute; the rubric swap showed you what that feels
like. Write the map you wish your grader had.
-->

## Environment

- Where it lives: in an eval bundle, the environment line(s) at the top of the
  candidate repro report, read against the issue section's stated target (the
  version, OS, and settings the reporter names) and the repo-facts block's
  latest release and bug-report template asks. Live: the environment block of
  the student's repro draft, against the issue body and the repo's bug template.
- What good looks like: the OS, the exact project version, and the runtime or
  dependency versions the bug plausibly depends on are named. Any setting the
  issue or thread says changes the failure (driver, build profile, shell,
  release channel) is named too. Where the report's version differs from the
  issue's target, the report says so in a sentence; a silent downgrade or a
  different platform with no comment is a deviation, not a record.

## Steps

- Where it lives: the commands, inputs, and config shown in the candidate repro
  report, in order. Live: the steps section of the repro draft.
- What good looks like: someone with a fresh machine and only public materials
  can run the steps from starting state to trigger: the exact commands or file
  contents are shown, and the trigger the issue names (the specific input,
  flag, setting, or platform detail) appears in the steps unchanged. Steps
  that live in a private repo, depend on an unshared config, or say "set up
  the project" with no commands cannot be re-run.

## Behavior shown

- Where it lives: output excerpts, logs, exit codes, and error text in the
  candidate repro report, read against the behavior described in the issue
  section (its error message, symptom, exit code, and trigger).
- What good looks like: the artifact itself shows the issue's behavior (same
  error text or class, same symptom, same exit code) and was produced by the
  issue's trigger, not a modified input. Compare the artifact, not the
  narration: a graceful validation error where the issue reports a crash, a
  syntax error from a changed input, a compile error where the issue reports a
  runtime error, or an old version's different error are adjacent behaviors.
  A control run (same steps without the trigger, different result) is strong
  support. For a cannot-reproduce, the artifact of the real attempt is the
  evidence.

## Honesty

- Where it lives: every claim sentence in the claim comment and the repro
  report ("I reproduced", "this is caused by", "happens every time", "on all
  platforms", "guaranteed", "verified"), paired with the artifact that backs it.
- What good looks like: each claim points at something shown. Outcomes are
  stated as observations; hypotheses are labeled as hypotheses. An honest
  cannot-reproduce shows the attempt, names what differed from the reporter's
  setup, and says what a triggering setup likely needs. A report fails when
  its certainty outruns its artifacts: a root cause asserted with no trace, a
  "verified" with no transcript, a generalization to a release or platform
  nobody tested, or an expected/actual pair the artifacts contradict.

## Comms

- Where it lives: the candidate claim comment read against the issue title and
  body; both comments read against the repo-facts block's bug-report template
  asks and contribution policy line (including any AI policy). Live: the
  student's drafts against the issue thread, the repo's CONTRIBUTING and any
  AI policy file, and the issue template.
- What good looks like: the claim names this issue's specifics (function,
  error, trigger) and a concrete next step framed as intent, with no promise of
  a fix, an outcome, or a date. It could not be pasted onto another issue.
  When the repo's AI policy requires disclosing AI assistance in comments (or
  in all contributions, not only pull requests), the comment states the tool
  and its extent. A policy that only asks for comments in the author's own
  words is met by specific, first-person writing; a PR-only disclosure ask
  does not apply to issue comments.
