# Unit 3 — Plan and Build

Path: `beat-1-sandbox/unit-3/plan-and-implement.md`

Record of your plan, the branch you built it on, and the evaluation runs that produced
`eval-run.txt`. This file is graded at the path above; a copy kept anywhere else in the
repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Posted upstream

**GitHub username**

etollaw

**Plan comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/72#issuecomment-6117905091

Posted text:

````markdown
Plan for this issue, building on my repro above.

**Diagnosis.** `verify_password` passes the stored hash straight to passlib and has nothing around the call, so passlib's failure escapes to the caller. The traceback in my repro report shows both ends of that: the raise comes from passlib's `identify_record` ("hash could not be identified"), and the frame directly below it is `core/security.py", line 37, in verify_password` — `return bool(pwd_context.verify(plain_password, hashed_password))`. The valid-hash control in the same step printed `True` for the right password and `False` for a wrong one, so verification itself is fine; what's missing is handling on the unusable-hash path.

**The failure has two shapes, and that changed my plan.** My repro only covered `not_a_valid_bcrypt_hash`. Several people above reported that a bcrypt-*prefixed* but broken hash raises a bare `ValueError` instead — thanks, that's a real gap in what I'd tested. It's their finding, not mine, so I ran the shapes myself on `main` at `f89c06f` before letting it change anything:

```
'not_a_valid_bcrypt_hash' -> raised passlib.exc.UnknownHashError | hash could not be identified
'$2b$notarealhash'        -> raised builtins.ValueError | not enough values to unpack (expected 2, got 1)
'$2b$12$shorttoken'       -> raised builtins.ValueError | salt too small (bcrypt requires exactly 22 chars)
''                        -> raised passlib.exc.UnknownHashError | hash could not be identified
```

Confirmed: identification failure raises `UnknownHashError`, and parse failure on a bcrypt-prefixed string raises a bare `ValueError` from passlib's bcrypt handler. Same defect, same call site, two exception types. So a fix catching only `UnknownHashError` would leave half of it in place, and I've scoped to `ValueError` instead — `UnknownHashError` is a `ValueError` subclass, so one catch covers both and naming both would be redundant.

**Scope.** One bounded change, in two parts that have to land together:

1. `verify_password` in `core/security.py` catches `ValueError` from the `pwd_context.verify(...)` call and returns `False`, so a stored hash passlib can't use fails closed instead of raising.
2. The `@pytest.mark.xfail(strict=True, ...)` marker on `test_verify_with_wrong_hash_format` in `tests/unit/test_security.py` comes off, as the issue asks. It has to: the marker is `strict=True`, so once the fix lands, leaving it in turns the now-passing test into a failure.

Deliberately out of scope: `hash_password` and the JWT helpers (the control shows them working); any catch wider than `ValueError` (no bare `except Exception` — that would turn a `TypeError` into a silent `False` in an auth path); new test cases for the `$2b$` shapes, which I'll show in the PR's evidence rather than expanding `test_security.py` beyond H-05; the `(trapped) error reading bcrypt version` noise I flagged in my repro side note, which is a passlib/bcrypt 4.x matter and not this bug; and the login route's handling, which is downstream of this contract. I checked for a `pyproject.toml` suppression tied to this issue and there isn't one — `git grep "#72"` matches only the xfail reason string — so no config change is needed.

**Test plan.** My repro's commands, re-run unchanged, before on `main` at `f89c06f` and after on the branch:

1. `pytest ... -k test_verify_with_wrong_hash_format -v` — before: `1 xfailed`. Expected after: `1 passed`, not `xfailed` and not `xpassed`.
2. The same with `--runxfail` — before: `passlib.exc.UnknownHashError: hash could not be identified`, `1 failed`. Expected after: `1 passed`, no `UnknownHashError` in the output.
3. The direct call with the valid-hash control first — before: `True`, `False`, then a traceback, exit code 1. Expected after: `True`, `False`, `malformed hash: False`, exit code 0. The first two lines are the control and must not change.
4. The four-shape loop above — before: all four raise. Expected after: all four `-> returned False`. This is the step that shows the catch is wide enough; step 3 would pass even with the too-narrow fix.

Plus `pytest tests/unit/test_security.py -q` as a regression guard, where `test_hash_password_long_input` (it hashes a 1000-character password and asserts `verify_password(...) is True`) staying `True` is what shows the catch hasn't swallowed a legitimate verify.

**The trade-off I'd most like a second opinion on.** `except ValueError` also catches `passlib.exc.PasswordSizeError`, which passlib raises when the *password* is over 4096 characters — so that case returns `False` instead of raising. I checked, on `main`:

```
PasswordSizeError.__mro__: PasswordSizeError -> PasswordValueError -> ValueError -> Exception
5000-char password -> passlib.exc.PasswordSizeError | password exceeds maximum allowed size
1000-char password -> True
```

I think `False` is right for an auth path (an over-long password shouldn't authenticate, and shouldn't 500 the login route), but it's a behavior change beyond the issue's text, so I'd rather flag it than bury it. If a maintainer prefers, the alternative is catching `ValueError` with an explicit re-raise for `PasswordSizeError`.

**Other things I don't know yet.** Whether any existing caller catches the raise and depends on it — I'll check the call sites before the PR. And whether the four shapes I tested are the whole set: a non-string stored hash might raise a `TypeError`, which would still escape, and I haven't tried it. Widening to `except Exception` isn't the answer there; it'd be a follow-up with its own evidence.

On what I haven't done: as in my repro, I ran only the dependency-install part of `make setup` (no `.env`, no Docker, no migrations), and I've run only `tests/unit/test_security.py` — not the full suite and not CI.

I'll report back here with the before/after output once the change is on a branch. I see several classmates working this issue too — this is my own plan from my own repro, and where I've used their findings above I've said so and re-ran them myself.

(I'm using Claude Code to help run the steps and draft this comment; the plan and the reasoning are mine and I checked the output myself.)
````

---

## Your branch

**Branch**

`fix/72-verify-password-fail-closed`

Pushed to my fork: https://github.com/etollaw/pathreview-ai301-fa26-s1/tree/fix/72-verify-password-fail-closed

Two commits off `main` at `f89c06f`:

```
580e6e7 fix(security): verify_password fails closed on malformed stored hashes
74f7570 test(security): un-park test_verify_with_wrong_hash_format
```

```
 core/security.py            | 16 ++++++++++++++--
 tests/unit/test_security.py |  4 ----
 2 files changed, 14 insertions(+), 6 deletions(-)
```

**Evidence**

My Unit 2 reproduction steps, re-run unchanged. BEFORE is on `main` at `f89c06f`; AFTER is on
`fix/72-verify-password-fail-closed`. Same venv both times: macOS 26.5.1 (arm64), Python
3.14.7, passlib 1.7.4, bcrypt 4.3.0, pytest 9.1.1. The complete transcript of both runs is in
`~/unit3-evidence.txt`.

### BEFORE (on `main`, unfixed)

**1. The H-05 test as it stands (marker in place)**

```
$ .venv/bin/pytest tests/unit/test_security.py -k test_verify_with_wrong_hash_format -v
================= 24 deselected, 1 xfailed, 1 warning in 0.34s =================
$ echo $? -> 0
```

**2. The same test with the xfail marker ignored (`--runxfail`)**

```
$ .venv/bin/pytest tests/unit/test_security.py -k test_verify_with_wrong_hash_format --runxfail -q
>       result = verify_password("password", wrong_hash)
>           raise exc.UnknownHashError("hash could not be identified")
E           passlib.exc.UnknownHashError: hash could not be identified
.venv/lib/python3.14/site-packages/passlib/context.py:1132: UnknownHashError
1 failed, 24 deselected, 1 warning in 0.24s
$ echo $? -> 1
```

**3. Direct call, with a valid-hash control first**

```
$ .venv/bin/python -u -c '
from core.security import hash_password, verify_password
good = hash_password("password")
print("valid hash, right password:", verify_password("password", good))
print("valid hash, wrong password:", verify_password("nope", good))
print("malformed hash:", verify_password("password", "not_a_valid_bcrypt_hash"))
'
valid hash, right password: True
valid hash, wrong password: False
Traceback (most recent call last):
  File "<string>", line 6, in <module>
    print("malformed hash:", verify_password("password", "not_a_valid_bcrypt_hash"))
  File "/Users/eldadtolla/pathreview-ai301-fa26-s1/core/security.py", line 37, in verify_password
    return bool(pwd_context.verify(plain_password, hashed_password))
  File ".../passlib/context.py", line 1132, in identify_record
    raise exc.UnknownHashError("hash could not be identified")
passlib.exc.UnknownHashError: hash could not be identified
$ echo $? -> 1
```

**4. Both failure shapes**

```
$ .venv/bin/python -c '
from core.security import verify_password
for h in ["not_a_valid_bcrypt_hash", "$2b$notarealhash", "$2b$12$shorttoken", ""]:
    try: print(repr(h), "-> returned", verify_password("password", h))
    except Exception as ex: print(repr(h), "-> raised", type(ex).__module__ + "." + type(ex).__name__, "|", ex)
'
'not_a_valid_bcrypt_hash' -> raised passlib.exc.UnknownHashError | hash could not be identified
'$2b$notarealhash' -> raised builtins.ValueError | not enough values to unpack (expected 2, got 1)
'$2b$12$shorttoken' -> raised builtins.ValueError | salt too small (bcrypt requires exactly 22 chars)
'' -> raised passlib.exc.UnknownHashError | hash could not be identified
```

**5. Regression guard: the whole security test file**

```
$ .venv/bin/pytest tests/unit/test_security.py -q
24 passed, 1 xfailed, 1 warning in 8.25s
```

### AFTER (on `fix/72-verify-password-fail-closed`)

**1. The H-05 test as it stands (marker now removed)**

```
$ .venv/bin/pytest tests/unit/test_security.py -k test_verify_with_wrong_hash_format -v
================= 1 passed, 24 deselected, 1 warning in 0.30s ==================
$ echo $? -> 0
```

**2. The same test with `--runxfail`**

```
$ .venv/bin/pytest tests/unit/test_security.py -k test_verify_with_wrong_hash_format --runxfail -q
1 passed, 24 deselected, 1 warning in 0.19s
$ echo $? -> 0
```

No `UnknownHashError` anywhere in the after output.

**3. Direct call, with a valid-hash control first**

```
$ .venv/bin/python -u -c '
from core.security import hash_password, verify_password
good = hash_password("password")
print("valid hash, right password:", verify_password("password", good))
print("valid hash, wrong password:", verify_password("nope", good))
print("malformed hash:", verify_password("password", "not_a_valid_bcrypt_hash"))
'
valid hash, right password: True
valid hash, wrong password: False
malformed hash: False
$ echo $? -> 0
```

The two control lines are unchanged; the third flipped from a traceback to `False`.

**4. Both failure shapes**

```
$ .venv/bin/python -c '
from core.security import verify_password
for h in ["not_a_valid_bcrypt_hash", "$2b$notarealhash", "$2b$12$shorttoken", ""]:
    try: print(repr(h), "-> returned", verify_password("password", h))
    except Exception as ex: print(repr(h), "-> raised", type(ex).__module__ + "." + type(ex).__name__, "|", ex)
'
'not_a_valid_bcrypt_hash' -> returned False
'$2b$notarealhash' -> returned False
'$2b$12$shorttoken' -> returned False
'' -> returned False
```

**5. Regression guard: the whole security test file**

```
$ .venv/bin/pytest tests/unit/test_security.py -q
25 passed, 1 warning in 8.17s
```

`24 passed, 1 xfailed` became `25 passed`: no new failures and no `xpassed`.
`test_hash_password_long_input` (a 1000-character password) still passes, which is what shows
the `except ValueError` has not swallowed a legitimate verify.

One note on the after output: the `(trapped) error reading bcrypt version` /
`module 'bcrypt' has no attribute '__about__'` notice still appears above step 3's output. That
is passlib 1.7.4 reading `bcrypt.__about__` on bcrypt 4.x, it was there before the change too,
and my Unit 2 repro flagged it as unrelated. It is not the issue's error.

## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

> **DRAFT, Eldad to rewrite.**

1. `--only calib-01,calib-02,calib-03,calib-04 --include-calibration` (partial, calibration
   carries no score): 4/4 per-item agreement. First run of the first draft of all three
   components. Nothing changed before it.
2. `--only pkg-04,pkg-20,pkg-03,pkg-09` (partial, scored): 4/4. Categories
   `clear-accept 2/2  thread-convention 2/2`. Nothing changed before it either — this was a
   risk spot-check on the 2-package `thread-convention` category before spending on a full run.
3. Full run with `--save-run eval-run.txt`: **19/20, PASS**. Categories
   `clear-accept 6/7  scope-creep 4/4  thread-convention 2/2  unbuildable 3/3  wrong-cause 4/4`.
   Nothing changed before this run: runs 1 and 2 surfaced no disagreement to iterate on, so the
   first full run was also the confirming run.

Agreement line from the committed `eval-run.txt`:

```
agreement: 19/20 scored items  (bar: 18/20: PASS)
```

The one disagreement (pkg-14) was left unrevised on purpose; see **Trade-offs**.

**Package analysis**

> **DRAFT, Eldad to rewrite.**

pkg-14 (`zellij-org/zellij#5174`), category `clear-accept`. Gold label: **accept**. My rubric's
verdict: **reject**, on `executable-approach`.

The plan names the areas and one chosen approach: drain pending OSC colour-query responses in
the client attach path before pane input is wired, in `zellij-server`'s client connection
handling. But its Files section then says the "exact functions to be pinned in the PR after
tracing the query issuance with debug logs". My `executable-approach` fail clause fires when
"the location is deferred to build time", and that sentence defers the location by its own
words, so the check failed and the required-check rule rejected the package.

Reading the gold note, the deferral it calls "arguable" is a different one — the untestable
Windows session-switch variant, which the plan defers explicitly and with a reason. The gold set
treats naming the area plus the approach as enough to start, and treats pinning exact function
names during the build as normal work rather than an unmade decision. On reflection I think the
gold label is the better read for this package: a stranger handed that plan could open
`zellij-server`'s connection handling and start, because the approach is settled even though
the line numbers are not.

**Check rationale**

> **DRAFT, Eldad to rewrite.**

Quoted from `tools/plan-check/rubric.md` as it reads now:

| executable-approach | The files, functions, or call sites the plan names and the approach or order of work it commits to, read against the code locations the issue and repro evidence point at (see evidence guide: Executability) | The plan names where the change goes (a file, function, or clearly identified call site) and which approach it chose, so a stranger could make the first edit without asking the author to decide anything. Fail if the location is deferred to build time ("somewhere in the editor code", "upstream or vendored, whichever is easier"), if the approach is still a list of candidate layers or an open question ("gocui? tcell? not sure"), or if the first concrete action is to investigate rather than to change something. One named site and one chosen approach pass even in a single terse paragraph. | required |

Why it reads that way. The three examples in the fail clause are quoted from the packages the
check exists to catch: "somewhere in the editor code" is calib-02, "upstream or vendored,
whichever is easier" is pkg-18, and "gocui? tcell? not sure" is pkg-17. I wrote the condition
around *what decision is still open* rather than *how much detail is present*, because the
unbuildable packages are not short — pkg-10 and pkg-18 are wordy. What they share is that the
real choice (which layer, which file, which of two fixes) is postponed to build time.

The last sentence, "One named site and one chosen approach pass even in a single terse
paragraph", is there to stop the check turning into a length test. calib-01 is three short
paragraphs and is a gold accept, so any condition that rewards elaboration would have failed it.

What I rejected in favour of this: an earlier phrasing I considered required the plan to name a
file *and* a function. I dropped the function half before running anything, because calib-01
names "the push completion callback in `pkg/gui/controllers/sync_controller.go`" — a call site
described in prose, not a function signature — and a file-and-function rule would have rejected
a gold accept. The cost of that choice is pkg-14, which is exactly the case where a plan names
the area and says the function comes later; see **Trade-offs**.

**Trade-offs**

> **DRAFT, Eldad to rewrite.**

The honest answer is that nothing changed after my first full run, and here is how I know what
it cost.

`executable-approach` is the check that gives something up, and pkg-14 is the package whose
result it changes: gold accept, my verdict reject, `failed: executable-approach`. The loosening
that would recover it is one sentence — something like "naming the area plus the chosen approach
is executable even when the exact function is pinned while tracing". I did not add it, for two
reasons:

1. That sentence is close to what pkg-18 says about itself. pkg-18 defers "upstream or vendored,
   whichever is easier" and "maybe also check other linters", and it is a gold reject. The
   distinction between "approach settled, line numbers pending" (pkg-14) and "approach pending"
   (pkg-18) is real, but writing it as a pass condition someone else could apply the same way is
   harder than it looks, and `unbuildable` is a 3-package category standing at 3/3. Trading a
   certain 3 for a possible 4 was not worth it.
2. The bar was already met: 19/20 against a bar of 18/20, with every category matched, so
   `eval-run.txt` was valid from the confirming run. Chasing the twentieth point meant another
   full run to re-fingerprint the saved file.

What the check will keep missing: a plan that names a plausible area and a confident approach
but has not actually located the code will pass, as long as it does not say so. My condition
reads what the plan *admits* about its own open decisions, so a plan that is vague without
announcing its vagueness slips through. pkg-14 is the mirror image — it was penalised precisely
for being honest about what it had not pinned down yet, which is the uncomfortable part of this
trade-off.

Canaries: none were needed, because no revision was made. Had I loosened
`executable-approach`, the `--only` list would have been
`pkg-14,pkg-10,pkg-17,pkg-18,calib-02` — the package to recover plus every `unbuildable`
package, since that is the small category the change would touch.

---

Related paths: `plan.md` and `eval-run.txt` in this directory; your skill's files in
`tools/plan-check/`.
