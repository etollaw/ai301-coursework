# Rubric: is this pull request ready to submit?

Four required checks, one per way a PR gets held (silent drift, not
tested, unreviewable, standards wall), plus one preferred check that
never moves the verdict. Every check reads the artifact against
something outside it: the diff against the plan, the evidence against
the test plan, the description against the diff, the PR against the
repo's stated asks. None of them reads the write-up's length or shape.

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| plan-fidelity | The diff's changed files and hunks, read against the plan's scope (in-scope line, Files list, Changes/Approach steps), its not-in-scope line, and its deviation notes; then the description's claims (what it says changed, and any fidelity claim such as "exactly as planned" or "no changes beyond the plan"), read against the diff (see evidence guide: Plan fidelity) | Both directions hold, and the description tells the truth about both. (1) Nothing more: every changed hunk serves a change the plan commits to (the fix itself, its regression test in a planned or obviously matching test file, a changelog or doc entry the repo asks for, removal of a marker or suppression the issue's test is parked behind) or is covered by a deviation note. (2) Nothing less: every change the plan commits to is in the diff, or is explicitly deferred in the plan's deviation note or in the description. (3) The description claims nothing the diff does not contain and hides nothing it does. Fail if the diff adds a refactor, rename pass, rewrite, new option, new flag, new setting, or edit to a file the plan never names, with no deviation note (however small or adjacent, and especially under a "no other changes" claim); if a planned change (a doc update, a second fix site, a test the plan promises) is missing with no deferral; or if the description claims work the diff does not deliver. A PR that deliberately delivers less than the plan and says so in a deviation note or the description passes. | required |
| decisive-test-evidence | The test-evidence section (and any before/after in the description), read against the plan's test plan and the repro evidence's trigger; plus the repo's stated checks or suite (from the repo-facts block's PR and contribution asks) and whether their outcome is shown (see evidence guide: Test evidence) | (1) The evidence re-runs the plan's repro trigger, or a test that encodes it, on the changed code path, and shows the observable after-result the plan's expected-after names (an output line, an exit code, a value, a named test passing); the before-result is shown or is the one the repro evidence already established. (2) Every distinct failure mode or repro the plan's test plan names is re-run, not just the first. (3) The repo's own test suite or stated checks were run and their outcome is visible (a pass line, a count, or a named command reported passing). Fail if the evidence is assertion only ("tested locally", "works now", "verified on my machine", "tests pass" with no command or outcome); if it exercises only a control or the unchanged path (the case that never broke, a different input shape than the repro); if a repro the plan's test plan names is never re-run; or if the repo's stated suite or checks are never shown run. A shown failure counts as evidence when it is disclosed with its reason and is unrelated to the change. | required |
| reviewable-diff | The unified diff hunk by hunk and the commit list (see evidence guide: Diff quality) | Every hunk is part of the change: the fix, its tests, comments that explain the fix, and any repo-asked entry (changelog, whatsnew, marker removal). Fail if the diff carries debris: debug prints or logging left in (`eprintln!("DEBUG`, `print(`, `console.log`, `dbg!`), commented-out code or abandoned first attempts, dead functions or `allow(dead_code)`/`_unused` helpers, stray TODO/FIXME added on untouched lines, whitespace, re-indent, import-reorder, or formatting churn on lines the fix does not need, duplicated or re-printed identical lines, or a drive-by edit unrelated to the issue. One such item is enough to fail; a correct fix does not buy it back. | required |
| repo-standards | The repo-facts block's (live: `.github/PULL_REQUEST_TEMPLATE.md`, `docs/CONTRIBUTING.md`, any AI policy) stated PR asks and AI-use policy, read against the PR title, description, and diff (see evidence guide: Standards and comms) | Every ask the repo states for PRs is met. (1) Template: where the repo has a PR template or required checklist, each of its sections or checklist items appears in the description with real content for this PR (not the template's placeholder text, not left blank), and a checked box is backed by evidence in the package. (2) Issue link: where the template or guide asks for `Closes #`/`fixes #` or that the PR reference the issue, the issue number is there. (3) Repo-asked artifacts: where the repo states a bug fix needs a changelog/whatsnew entry or a test, it is in the diff or the description says why not. (4) AI disclosure: treat every package as AI-assisted work; if the repo's stated policy requires disclosing AI use (in any contribution, or in PRs), the description names the tool and the extent of the help. Fail if a stated template section or checklist is visibly absent or left as boilerplate, a required issue link is missing, a repo-asked entry is missing without explanation, or a required AI disclosure is absent. Pass each clause where the repo asks nothing: no template means clause 1 passes; "no stated AI policy" means clause 4 passes; a policy that only asks that text be the author's own words passes on a specific first-person description. A placeholder left for the author to fill (for example `[AI-use disclosure: to be written]`) is not a disclosure. | required |
| commit-hygiene | The commit list (subjects) and, live, the branch name, read against the repo's stated commit and branch conventions (see evidence guide: Diff quality) | Each commit subject says what that commit changes, and follows the repo's stated convention where one exists (for example Conventional Commits with a listed scope, `fix/<issue>-<slug>` branch). Fail on subjects like "wip", "fix", "misc cleanups", "fmt", or a convention the repo states and the commits ignore. | preferred |

## Verdict rule

Accept only if all four `required` checks pass: `plan-fidelity`,
`decisive-test-evidence`, `reviewable-diff`, and `repo-standards`. A
single required fail is a reject; no number of passes elsewhere buys
it back, and an excellent fix with one required fail is still held.

`unclear` is the grade for a required check whose deciding evidence
is absent from the package (no test-evidence section at all, no plan
context, a named live file that does not exist), so the pass
condition cannot be applied either way. **`unclear` on a required
check counts as a fail and rejects the package**: a PR that cannot be
verified from the package is not ready to submit. `unclear` is never
rounded up to a pass and is not a middle verdict; the evidence line
must say what was missing. Absence of a fault is not unclear: a repo
with no template passes clause 1 of `repo-standards`, and a diff with
no debris passes `reviewable-diff`.

`commit-hygiene` is `preferred` and never changes the verdict, in
either direction, whatever its grade (`pass`, `fail`, or `unclear`).
It is reported so the author sees it.

The honest-outcome rule: a PR that delivers less than everything and
says so (a deferral recorded in the plan's deviation note or stated in
the description) is graded on what it claims, and can be accepted.
Only an unannounced shortfall, or a claim the diff contradicts, holds
the package.

The verdict space is binary: `accept` (ready to submit) or `reject`
(hold).
