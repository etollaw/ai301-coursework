# Procedure: how this tool grades a PR package

## Read order

The plan and the repo's asks are read before the candidate PR, and
the description is read last. Every required check is a side-by-side:
the diff against the plan, the evidence against the test plan, the
description against the diff, the PR against the stated asks. If you
read the description first, its own summary ("implements the plan
exactly", "tests pass") tells you what the diff and evidence say;
reading the yardsticks first means you hold the PR against them
instead.

1. **Repo facts** (live: `.github/PULL_REQUEST_TEMPLATE.md`,
   `docs/CONTRIBUTING.md`, any root `CONTRIBUTING.md` or
   `AI_POLICY.md`). Write down, as a numbered list called the *asks
   list*: every PR template section and checklist item by name;
   whether an issue link (`Closes #`, `fixes #`, "reference the
   issue") is asked; any repo-asked artifact for bug fixes (changelog,
   whatsnew entry, tests); the stated test suite or checks
   (`cargo test`, `yarn test`, `make test-unit`, pre-commit, CI); the
   commit and branch conventions; and the AI policy, recorded as one
   of: none stated / own-words only / disclosure required (in PRs or
   in all contributions).
2. **Issue and thread highlights** (live: `gh issue view <n> --repo
   <owner/repo> --comments`). Write one sentence: the single behavior
   the issue reports. Note any maintainer-side direction (OWNER,
   MEMBER, COLLABORATOR, CONTRIBUTOR) the PR should engage.
3. **Plan context** (live: `plan.md`, including `## Deviations`).
   Write down three lists: the *scope list* (every change the plan
   commits to, one line each, plus every file it names), the
   *out list* (its not-in-scope line and any deviation note that
   defers or adds work), and the *test-plan list* (every repro,
   trigger, or failure mode the test plan says to re-run, each with
   its expected-after observable, plus any suite it says to run).
4. **Diff** (live: `git diff main...HEAD`, with `--stat` for the file
   list). List every changed file, and every hunk with a one-line
   label of what it does. Do not read the description yet.
5. **Commits** (live: `git log --oneline main..HEAD`; also
   `git branch --show-current`). Copy the subjects.
6. **Test evidence** (live: the named evidence file, e.g.
   `test_evidence.md`). List every command shown with its visible
   output line, marked before or after.
7. **Title and description** last (live: line 1 of the named draft
   file is the title, the rest is the body). List every claim it
   makes about what changed, any fidelity claim, any deferral it
   states, every template section and checked box, and any AI-use
   disclosure.

Live mode, before step 1: read `scope.md` and stop per SKILL.md's
scope seam (placeholder `Repo:` line, or an issue or target repo
outside scope). Read `voice-guide.md` before step 7 and collect any
broken rule as a summary note; it feeds no check.

## Evidence gathering

One pairing per required check. Record each fact as a short quote or
a one-line paraphrase with its location (file and hunk, command, or
description sentence), because every grade must name its fact.

1. **Diff against plan** (for `plan-fidelity`, direction 1): take
   each hunk label from read-order step 4 and match it to a line of
   the scope list or the out list's deviation notes. Treat as matched:
   the fix, its regression test, comments explaining the fix, a
   repo-asked changelog or whatsnew entry, and removal of a marker or
   suppression the issue's test is parked behind. Record every hunk
   that matches nothing as *unplanned* (file and what it does). A hunk
   that edits a file on the plan's not-in-scope line is unplanned even
   if the file is named elsewhere.
2. **Plan against diff** (for `plan-fidelity`, direction 2): take
   each line of the scope list and find the hunk that delivers it.
   Record every planned change with no hunk as *missing*, then strike
   any missing item that a deviation note or the description
   explicitly defers.
3. **Description against diff** (for `plan-fidelity`, clause 3):
   take each claim from read-order step 7 and find the hunk that backs
   it. Record any claim with no backing hunk (claims more), and record
   any fidelity claim ("exactly as planned", "no other changes")
   sitting over an unplanned hunk from move 1 (hides more).
4. **Evidence against test plan** (for `decisive-test-evidence`):
   for each test-plan list item, find the evidence command that
   re-runs that trigger on the changed path and record the shown
   after-output next to the plan's expected-after. Record *not re-run*
   for any item with no matching command, *control only* where the
   only command run is the plan's control or a different input shape
   than the repro, and *assertion only* where the evidence is prose
   with no command or output.
5. **Suite against asks** (for `decisive-test-evidence`, clause 3):
   take the stated suite or checks from the asks list (and any suite
   the test plan names) and record the evidence line showing each was
   run and its outcome, or *not shown*. Live mode: `make test-unit`,
   `make test-integration`, `make lint`, `make typecheck` are the
   stated checks; a failure shown with its reason, unrelated to the
   diff, counts as run.
6. **Debris scan** (for `reviewable-diff`): read each hunk's added and
   removed lines once more for the debris tells in the rubric (debug
   output, commented-out code, dead functions, stray TODOs,
   whitespace/indent/import churn, duplicated lines, unrelated edits).
   Record the first one found with its file. A comment that explains
   the fix is not debris.
7. **Asks against PR** (for `repo-standards`): walk the asks list item
   by item and record where the description or diff meets it, or
   *absent*. For template sections, record whether each has content
   about this PR or only the template's placeholder. For checked boxes,
   record whether the evidence backs them. For the AI policy, record
   whether the description names the tool and extent when the policy
   requires disclosure.
8. **Commits against convention** (for `commit-hygiene`): record each
   subject that does not say what it changes or breaks a stated
   convention, and (live) whether the branch matches
   `<type>/<issue-number>-<slug>`.

## Check execution

Checks execute in rubric order: `plan-fidelity`,
`decisive-test-evidence`, `reviewable-diff`, `repo-standards`,
`commit-hygiene`. Every check is graded even after an earlier one
fails, because the author needs the whole picture.

For each check:

1. Apply the pass condition to the facts gathered for that check and
   nothing else. Return to the package only to confirm an exact quote;
   the gathering moves already pulled what the condition names.
2. Grade `fail` when a recorded fact meets a fail clause: any
   *unplanned*, *missing* (not deferred), or claims-more item for
   `plan-fidelity`; any *not re-run*, *control only*, *assertion only*,
   or *not shown* item for `decisive-test-evidence`; any debris item
   for `reviewable-diff`; any *absent* or placeholder-only item for
   `repo-standards`. One such fact is enough.
3. Grade `pass` when every gathered fact for the check satisfies the
   pass condition.
4. Grade `unclear` only when the package lacks the evidence the
   check's Evidence column names (no plan context, no test-evidence
   section, no diff, a named live file missing), so the condition
   cannot be applied. Absence of a fault is a pass, not unclear: no
   template, no AI policy, no debris, no deferral needed.
5. Write one evidence line: the fact that decided it, not a
   restatement of the rule. "Evidence shows only `rg` single-file
   control; the plan's two-file repro is never re-run" is evidence;
   "testing is insufficient" is not.
6. Never let length, polish, confidence, or the description's own
   self-assessment move a grade. A terse PR and a long one are graded
   identically.
7. If the procedure is silent on a situation the check needs, grade
   that check `unclear` and name the missing step in the summary.

## Verdict assembly

1. Collect the grades of the four required checks.
2. If all four are `pass`, the verdict is `accept`; otherwise
   `reject`. `unclear` on a required check counts as a fail.
3. Ignore `commit-hygiene` when computing the verdict, whatever its
   grade; report it anyway.
4. Name the deciding check. On a reject, it is the first required
   check graded `fail` or `unclear` in rubric order; quote its evidence
   line as the reason. On an accept, say every required check passed
   and quote the evidence line of the check that came closest to
   failing, so the author knows where the PR is thinnest.
5. Emit the readable summary (one line per check: name, grade,
   evidence; the deciding check; live-mode voice notes), then the
   fenced JSON block from SKILL.md last, every check in rubric order,
   with the verdict from step 2 and nothing after it.
