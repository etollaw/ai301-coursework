# Unit 1 — Issue Selection

Path: `beat-1-sandbox/unit-1/selection.md`

Record of the issue carried into Unit 2, and of the evaluation runs that produced
`eval-run.txt`.

---

## Selected issue

**Issue link**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/72

**Verdict output**

Live-mode run over three candidates (#71, #72, #73); #72 is the first entry in the JSON
block and records `accept`.

```
⏺ Skill(issue-select)
Successfully loaded skill

  Read 2 files, ran 5 shell commands

No explicit AI-generated-contribution ban in CONTRIBUTING.md — ai-policy-ok passes (silence passes per rubric). Now let's fetch the three issues' full details, comments, labels, assignees, and linked PRs.

  Ran 3 shell commands

Summary

All three candidates are in-scope (codepath/pathreview-ai301-fa26-s1) and all pass every required check — all three: accept.

Repo-wide facts (apply to all three): last default-branch commit f89c06f (2026-09-16, 6 days before capture) → maintainer-active pass; not archived, pushed 2026-09-16 → repo-in-use pass; docs/CONTRIBUTING.md states no ban on AI-generated contributions (only disclosure-adjacent norms like CI-must-be-green and xfail-marker hygiene) → ai-policy-ok pass.

Ranked (best fit first):

1. #72 — verify_password raises UnknownHashError (fit: backend Python bug in core/security.py, requires removing an xfail marker tied to CI — directly exercises the "reading unfamiliar code + CI/testing" goal from your profile; carries good-first-issue + tier-1). Unclaimed: no comments on the issue itself; the two cross-references found are other students' own coursework submission PRs in their personal repos merely citing the issue URL — not a PR opened against this repo, so per the Path-Review house rule this doesn't block claiming.
2. #71 — heading-hierarchy test fixture is indented. Also touches an xfail-marked test (CI/testing exposure) and requires understanding the markdown heading-extraction logic in ingestion/parsers/readme_parser.py, but the fix itself is a one-line indentation removal — thinner learning surface. tier-2, no good-first-issue label.
3. #73 — README vs .env.example disagree on LLM key. Bounded and safe, but pure docs/config reconciliation — least alignment with your stated interest in codebase-reading/CI work; closest to the "avoid" end of your profile even though it's not frontend/CSS. Has good-first-issue + tier-1.

[
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/72",
    "checks": [
      {"name": "maintainer-active", "grade": "pass", "evidence": "Last default-branch commit f89c06f dated 2026-09-16, 6 days before capture (2026-09-22)"},
      {"name": "repo-in-use", "grade": "pass", "evidence": "archived: false; pushed_at 2026-09-16T21:48:27Z, within 90 days"},
      {"name": "scope-bounded", "grade": "pass", "evidence": "Body describes one coherent fix ('should fail closed... not raise') with a single named xfail marker (H-05) to remove; no TBDs or unresolved thread"},
      {"name": "unclaimed", "grade": "pass", "evidence": "No assignees, no comments on the issue; the two 'cross-referenced'/'referenced' timeline entries are other students' own coursework-submission PRs in unrelated personal repos citing the issue URL, not a PR against this repo — Path Review house rule: classmates' claim signals don't block"},
      {"name": "ai-policy-ok", "grade": "pass", "evidence": "docs/CONTRIBUTING.md has no AI-generated-contribution ban"},
      {"name": "good-first-label", "grade": "pass", "evidence": "labels: ['bug', 'good first issue', 'api', 'tier-1']"}
    ],
    "verdict": "accept"
  },
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/71",
    "checks": [
      {"name": "maintainer-active", "grade": "pass", "evidence": "Last default-branch commit f89c06f dated 2026-09-16, 6 days before capture"},
      {"name": "repo-in-use", "grade": "pass", "evidence": "archived: false; pushed_at 2026-09-16T21:48:27Z, within 90 days"},
      {"name": "scope-bounded", "grade": "pass", "evidence": "Single coherent fix ('Remove the indentation') plus removing xfail marker H-04; no undecided core requirement"},
      {"name": "unclaimed", "grade": "pass", "evidence": "No assignees, no comments, no cross-referenced PRs in timeline"},
      {"name": "ai-policy-ok", "grade": "pass", "evidence": "docs/CONTRIBUTING.md has no AI-generated-contribution ban"},
      {"name": "good-first-label", "grade": "fail", "evidence": "labels: ['bug', 'ingestion', 'tier-2'] — no 'good first issue' label"}
    ],
    "verdict": "accept"
  },
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/73",
    "checks": [
      {"name": "maintainer-active", "grade": "pass", "evidence": "Last default-branch commit f89c06f dated 2026-09-16, 6 days before capture"},
      {"name": "repo-in-use", "grade": "pass", "evidence": "archived: false; pushed_at 2026-09-16T21:48:27Z, within 90 days"},
      {"name": "scope-bounded", "grade": "pass", "evidence": "Single coherent goal ('Make the two files agree'); no sub-items, no TBD, no unresolved disagreement"},
      {"name": "unclaimed", "grade": "pass", "evidence": "No assignees, no comments, no cross-referenced PRs in timeline"},
      {"name": "ai-policy-ok", "grade": "pass", "evidence": "docs/CONTRIBUTING.md has no AI-generated-contribution ban"},
      {"name": "good-first-label", "grade": "pass", "evidence": "labels: ['bug', 'good first issue', 'docs', 'tier-1']"}
    ],
    "verdict": "accept"
  }
]
```

---

## Eval iterations

**Run history**

1. First full eval run with initial 6-check rubric: 15/20 agreement, below the 18 bar.
   Categories: claimed 4/4, clear-accept 5/8, dead-repo 3/3, policy 1/1, scope 2/4.
2. Diagnosed all 5 disagreements via `--only` reruns with `--out` for per-check JSON; all
   traced to the scope-bounded check being either too strict (rejecting bounded multi-file
   issues) or too loose (accepting issues with real unresolved scope).
3. Revised scope-bounded's pass condition three times across `--only` reruns on the same 5
   disputed issues: first to stop failing issues for touching multiple files, then to catch
   author-stated unresolved core scope vs. thread-level design debate, then to distinguish
   genuinely independent sub-items (umbrella issue) from one coherent feature shown via
   several examples.
4. Final full run: 18/20, PASS. Categories: claimed 4/4, clear-accept 6/8, dead-repo 3/3,
   policy 1/1, scope 4/4. Saved via `--save-run eval-run.txt`.

Final agreement line from `eval-run.txt`:

```
agreement: 18/20 scored items  (bar: 18/20: PASS)
```

**Issue analysis**

issue-04 — gold: accept, my rubric: reject (failed: scope-bounded). The issue lists several
missing rule previews for one existing rendering feature: "Including remove identity, fuse
spiders, remove self loops, etc." My scope-bounded check read this as multiple independent,
separately-trackable sub-items (an umbrella issue) because of the open-ended "etc." and list
format, which is clause (a) of the check: "the issue's sub-items are independent, separately
assignable pieces of work meant to be split off individually". In reality it's one bounded
task — add previews for a known feature — illustrated with concrete examples, not several
issues that would ever be tracked separately. This is the main remaining gap in my rubric:
distinguishing "one task with several examples" from "several independent tasks" is a
genuinely hard line, and my current wording still leans too far toward flagging any
multi-item list as an umbrella issue.

**Check rationale**

Check: `scope-bounded` (required). Pass condition, as currently written in
`tools/issue-select/rubric.md`:

> Pass unless: (a) the issue's sub-items are independent, separately assignable pieces of work meant to be split off individually — not a single coherent goal that happens to touch multiple files or pages; (b) the issue's own body leaves the core required work undecided — phrasing like "TBD" or "not sure if X is needed" applied to a central part of the task, not a minor optional aside — showing even the author hasn't decided what's required, OR the comment thread shows a live, unresolved disagreement between multiple people with no maintainer decision reached (do not fail merely because the author confidently diagnoses multiple causes or proposes several possible fixes on their own); (c) a maintainer explicitly states the fix touches core internals; (d) issue is a pure usage/support question; or (e) issue history shows multiple closed, unmerged linked PRs or abandoned attempts. Do not fail solely for touching multiple files or listing more than one possible fix — grade the bound of the work, not its size.

Reasoning behind its current form: my first draft failed issues for touching multiple files,
which rejected bounded multi-file issues the gold set accepts. Each clause was added on an
`--only` rerun of the disputed issues. The closing sentence ("grade the bound of the work,
not its size") fixed the multi-file false rejects. Clause (b) separates an author who leaves
the core work undecided ("TBD", "not sure if X is needed") or a live, unresolved thread
disagreement from an author who confidently proposes several fixes. Clause (a) was narrowed
to "independent, separately assignable pieces of work", not "a single coherent goal that
happens to touch multiple files or pages", to tell an umbrella issue from one feature shown
through several examples. Clauses (c)–(e) name concrete signals (a maintainer calling it
core internals, a support question, abandoned PRs) so the check is not a pure judgment call.

**Trade-offs**

Loosening scope-bounded to stop punishing multi-file issues fixed real false rejects, but it
still rejects issue-01 and issue-04 in the final run. Both are gold accepts that my check
read as unbounded, so the line between "one task, several examples" and "several independent
tasks" isn't fully solved by my wording. I also noticed real run-to-run variance from the
grading model: issue-04 and issue-19 flipped verdicts across reruns with no rubric change,
meaning some score movement reflects grading noise rather than pure rubric improvement. I
stopped tuning once I passed the 18/20 bar, since further attempts kept trading one edge
case for another.

---

## Selection rationale

**Selection rationale**

1. Fit to interests and time: chosen from among 3 accepted candidates (#71, #72, #73)
   because it's a backend Python bug (verify_password raising UnknownHashError) requiring
   removal of a CI xfail marker — best matches my goal of getting better at reading
   unfamiliar code and working with CI/testing setups, and carries good-first-issue + tier-1
   labels. The fix is one function in core/security.py plus one xfail marker (H-05), which
   is a size I can finish within the unit.
2. What the verdict got right, and what I weighed that it couldn't: the verdict correctly
   passed every required check and correctly read the two cross-references as classmates'
   coursework PRs in their own repos, not claims against this repo. But my rubric is binary,
   and all three candidates were accepted, so it couldn't choose between them. I made that
   call on fit to what I want to learn: #71 is a one-line indentation fix with a thinner
   learning surface, and #73 is docs/config reconciliation, away from the code-reading and
   CI work I'm after.
3. Anticipated difficulty in claiming it: nobody has commented on or been assigned to #72,
   but two classmates' coursework PRs already cite it, so others are likely considering it
   too. I expect to need to post my claim comment early in Unit 2 and check the thread
   first for anyone who got there before me.

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/issue-select/`.
