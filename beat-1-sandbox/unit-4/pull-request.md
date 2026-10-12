# Unit 4 — Test and Submit

Path: `beat-1-sandbox/unit-4/pull-request.md`

Record of the pull request you opened against the Path Review repo, and of the evaluation
runs that produced `eval-run.txt`. This file is graded at the path above; a copy kept
anywhere else in the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Your pull request

**Pull request**

https://github.com/codepath/pathreview-ai301-fa26-s1/pull/146

**Branch**

`fix/72-verify-password-fail-closed` (fixes issue #72)

## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

1. Partial run, `--only pkg-01,pkg-09,pkg-13,pkg-16,pkg-18,pkg-20 --include-calibration` (no bar verdict on partial runs): 6/6 agree.
2. Full run, saved with `--save-run eval-run.txt`: `agreement: 20/20 scored items  (bar: 18/20: PASS)`; `categories: clear-accept 7/7  not-tested 4/4  silent-drift 4/4  standards-wall 2/2  unreviewable 3/3`.

The tool files did not change between the two runs, so run 2 is both the first and the final full run, and its line matches the committed `eval-run.txt`.


**Package analysis**

`pkg-09` (sharkdp/fd#2053, silent-drift). Gold: `reject`. My tool: `reject`, failing `plan-fidelity`, `reviewable-diff`, and `repo-standards`; `decisive-test-evidence` passed.

The plan says "Not in scope: new flags". The diff adds one anyway, in commit `3d0c6b1 cli: add --time-format for custom date parsing`, and rewords help text the issue never mentions. The description still says "Error messages for date parsing improved exactly as planned". My procedure reads the plan before the description and matches every hunk to a scope line or a deviation note. The `--time-format` hunk matched nothing, so it was recorded as unplanned, and `plan-fidelity` fails on a new flag "however small or adjacent, and especially under a 'no other changes' claim". The grader's evidence line was: "Diff adds a --time-format flag (plan's not-in-scope: new flags), rewords other help text ... yet the description says 'exactly as planned'." The testing itself was fine: both the repro and the control were re-run, and `cargo test` passed with 612 tests. That's why it isn't a not-tested reject. The gold note calls this package arguable because the extra work sits next to the fix. My rubric deliberately gives no credit for that: unannounced scope is drift even when it's adjacent.


**Check rationale**

From `tools/pr-precheck/rubric.md`, the `reviewable-diff` row:

> Evidence: The unified diff hunk by hunk and the commit list (see evidence guide: Diff quality)
>
> Pass condition: Every hunk is part of the change: the fix, its tests, comments that explain the fix, and any repo-asked entry (changelog, whatsnew, marker removal). Fail if the diff carries debris: debug prints or logging left in (`eprintln!("DEBUG`, `print(`, `console.log`, `dbg!`), commented-out code or abandoned first attempts, dead functions or `allow(dead_code)`/`_unused` helpers, stray TODO/FIXME added on untouched lines, whitespace, re-indent, import-reorder, or formatting churn on lines the fix does not need, duplicated or re-printed identical lines, or a drive-by edit unrelated to the issue. One such item is enough to fail; a correct fix does not buy it back.

Why it reads this way: I wrote the debris tells as concrete, greppable patterns (`eprintln!("DEBUG`, commented-out code, `allow(dead_code)`, re-indent or import churn, duplicated lines) instead of "the diff is clean". That way two executors find the same item. I also added "One such item is enough to fail; a correct fix does not buy it back", because every unreviewable package in the set has a *correct, evidenced* fix under the debris (bat's clamp, zellij's two fixes, nushell's is_terminal gate). A check that weighed debris against fix quality would let them all through. I also excluded "comments that explain the fix", because neovim's accept package has a multi-line explanatory comment in its hunk, and a naive "comments in diff = debris" rule would have rejected a clear accept.

**Trade-offs**

- **A case I accept it will miss:** `reviewable-diff` has no size or proportion test. A large refactor made entirely of real code (no debug lines, no churn) only fails it through the "drive-by edit unrelated to the issue" clause, which is a judgment call. On starship's `truncate()` rewrite (pkg-03) the grader failed both `plan-fidelity` and `reviewable-diff`, but if it reads a refactor as related to the issue, only `plan-fidelity` catches it. I'm relying on the two checks overlapping.
- **A result the overlap changes:** pkg-09 failed three required checks, not one, so loosening `plan-fidelity` alone would not flip it. The cost is that the harness files it under silent-drift while my tool's reasons are spread across three checks.
- **What `repo-standards` gives up:** it only demands an AI disclosure where the repo's policy asks for one. My own PR passed partly because Path Review states no AI policy. The same draft with its placeholder would have been held in ghostty (pkg-20).
- **What `commit-hygiene` gives up:** it's preferred, so it never moves the verdict. It failed on my own PR (scope `security` is not in CONTRIBUTING's list) and I still got an accept. Clean code with sloppy commits gets through; I accepted that because the "wip/fix" packages (pkg-12) are already held by their debris.
- **Nothing changed elsewhere, and here is how I know:** I made no revisions after the first draft, so no canary re-runs were needed. The partial run already covered the cases a strict rubric is most likely to flip: both standards-wall packages (pkg-01, pkg-20) and both honestly-disclosed-shortfall accepts (pkg-13, pkg-16). The full run agreed on all of them.


---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/pr-precheck/`.
