# Evidence guide: where evidence lives in a PR package

Each family below maps to one required check and one harness
category. For each: where it lives in an eval bundle, where it lives
in my live PR (issue #72 on `codepath/pathreview-ai301-fa26-s1`, fork
branch `fix/72-verify-password-fail-closed`), and what good looks
like.

## Plan fidelity (harness category: silent-drift)

- Where it lives, eval bundle: the plan's scope sits in the
  `## Plan context` block: its `Scope` sentence, `Not in scope:` line,
  `Files:` list, and the steps after `Plan:`; a deviation or deferral
  note is a sentence there beginning "Deviation", "Known limit",
  "deferred", or "Not handled". What actually changed is the
  `### Diff` under `## Candidate PR` (each `--- a/` / `+++ b/` file
  header and its hunks). The fidelity claims are in `### Description`
  ("per the plan", "exactly as planned", "no functional changes
  outside", "now documents").
- Where it lives, live: `plan.md` in the fork clone (its Scope, Files,
  Approach, and `## Deviations` sections); `git diff main...HEAD` and
  `git diff main...HEAD --stat` on the branch; the description body in
  `pr_draft.md` (its Summary and Changes bullets).
- What good looks like: every file in the diff's header list is in
  the plan's Files list (or is the planned test file, a repo-asked
  changelog, or a marker/suppression removal the issue's test needs),
  every planned step has a hunk, and the description's Changes list
  has exactly one bullet per hunk group. Drift in the *more* direction
  looks like starship's one-line fallback fix arriving with a
  `truncate()` rewrite and a new config option under "implements the
  posted plan exactly"; in the *less* direction, like vue's plan
  promising a warning and a docs update while the diff holds only the
  warning and the description says the docs "now document the
  limitation". An honest shortfall looks like neovim's PR: the `%VAR%`
  case deferred in the plan's note and restated in the description, so
  the diff matches what the PR claims.

## Test evidence (harness category: not-tested)

- Where it lives, eval bundle: the `### Test evidence` section (and
  any before/after block in `### Description`), read against the
  plan context's `Test plan:` sentence and its `Repro evidence:` line
  (the trigger and the control). The repo's stated suite is in the
  repo-facts block's `pull requests:` and `contribution policy:` lines
  ("pass the test suite (yarn test)", "cargo test", pre-commit).
- Where it lives, live: `test_evidence.md` in the fork clone, holding
  the real output of my repro and of `make test-unit`,
  `make test-integration`, `make lint`, `make typecheck`, each on
  `main` and on the branch; the plan's Test plan section in `plan.md`;
  the checks named in `docs/CONTRIBUTING.md` and the template's
  Testing checklist.
- What good looks like: a command, its before output, its after
  output, and the plan's expected-after named next to it, for every
  repro the test plan lists, plus the suite's pass line or count. For
  #72: `verify_password("x", "$2b$notarealhash")` raising `ValueError`
  on `main` and returning `False` on the branch, and
  `tests/unit/test_security.py::test_verify_with_wrong_hash_format`
  passing without its xfail marker. Weak evidence looks like
  "verified working on my machine for a full day", "cargo test
  passes" with the repro never re-run, or a re-run of the control
  (golangci-lint's single-file case, httpie's GET header-casing path)
  instead of the case that broke.

## Diff quality (harness category: unreviewable)

- Where it lives, eval bundle: the `### Diff` (every `+`/`-` line)
  and `### Commits` under `## Candidate PR`.
- Where it lives, live: `git diff main...HEAD` (every hunk) and
  `git log --oneline main..HEAD`; also `git status` to confirm no
  scratch files (`plan.md`, `comment.md`, `pr_draft.md`,
  `test_evidence.md`) were committed.
- What good looks like: every added line is the fix, its test, or a
  comment explaining the fix, and every removed line is the bug or a
  marker the fix retires. The debris tells: `eprintln!("DEBUG ...")`,
  `print(`/`console.log` left in, `// let x = ...` commented-out
  first attempts, `fn _unused_probe` or `#[allow(dead_code)]`
  helpers, a TODO added on an untouched import, re-indented or
  re-ordered blocks, identical lines removed and re-added, and edits
  to files unrelated to the issue. Commit subjects like "wip", "fix",
  or "misc cleanups while debugging" are the commit-list version of
  the same tell (graded by `commit-hygiene`).

## Standards and comms (harness category: standards-wall)

- Where it lives, eval bundle: the repo-facts block's
  `pull requests:` line (template sections, required checklist,
  `closes #xxxx`, changelog/whatsnew asks) and
  `contribution policy:` line (AI policy, with its scope), read
  against `### Title` and `### Description` and, for repo-asked
  files, the `### Diff` file list.
- Where it lives, live: `.github/PULL_REQUEST_TEMPLATE.md` (sections
  Summary, Issue with `Closes #`, Changes, Testing checklist,
  Screenshots / Demo, Notes for Reviewers), `docs/CONTRIBUTING.md`
  (branch format `<type>/<issue-number>-<slug>`, Conventional Commits
  with listed scopes, the five CI jobs, the xfail-marker and
  `pyproject.toml` suppression rule; no stated AI policy at capture),
  the `scope.md` house rules (template always used, disclosure section
  where it applies), and `pr_draft.md`.
- What good looks like: every template section present with content
  about this PR, `Closes #<n>` with the real number, only true boxes
  ticked (an unticked "CI is green" before the PR exists is honest),
  repo-asked entries in the diff, and an AI-use disclosure naming the
  tool and the extent wherever the policy asks for one (ghostty: "all
  AI usage in any form must be disclosed"). A visibly ignored ask
  looks like pandas' PR with no `closes #`, no checklist, and no
  whatsnew entry, however good the fix. Whether the description's
  claims match the diff is plan fidelity, not this family.
