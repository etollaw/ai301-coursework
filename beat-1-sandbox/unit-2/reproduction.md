# Unit 2 — Claim and Reproduce

Path: `beat-1-sandbox/unit-2/reproduction.md`

Record of your claim and reproduction on the issue you chose in Unit 1, and of the
evaluation runs that produced `eval-run.txt`. This file is graded at the path above; a copy
kept anywhere else in the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Your identity upstream

**GitHub username**

etollaw

---

## Posted upstream

**Claim comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/72#issuecomment-5863705097

Posted text:

````markdown
Hi, I'd like to work on this as my first contribution. I'm going to reproduce it locally on a fork at the current `main`: call `verify_password` with a stored hash that isn't a recognizable bcrypt hash and see whether passlib's `UnknownHashError` escapes instead of the function failing closed and returning `False`. I'll also run `test_verify_with_wrong_hash_format`, the test behind the `xfail` marker for manifest H-05, to check that it currently fails the way the marker says. I'll post a repro report here with my environment, the exact commands, and the output I get, whether or not it reproduces.

(I'm using Claude Code to help run the steps and draft my comments; I check the output myself.)
````

**Reproduction comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/72#issuecomment-5863753449

Posted text:

````markdown
Repro report: I reproduced this on my machine. With a malformed stored hash, `verify_password` lets `passlib.exc.UnknownHashError` escape instead of returning `False`.

**Environment**
- macOS 26.5.1 (arm64), Python 3.14.7
- My fork at commit `f89c06f`, the same as upstream `main`, no local changes
- passlib 1.7.4, bcrypt 4.3.0, pytest 9.1.1 (installed by `pip install -e ".[dev]"`)
- No `.env` and no Docker services. I ran only the dependency-install part of `make setup` (venv + `pip install -e ".[dev]"`) and skipped Docker, migrations, and seeding, since this test and `core/security.py` don't touch the database.

**Steps**
```
git clone https://github.com/etollaw/pathreview-ai301-fa26-s1.git
cd pathreview-ai301-fa26-s1
git checkout f89c06f
python3 -m venv .venv
.venv/bin/python -m pip install --upgrade pip setuptools wheel
.venv/bin/pip install -e ".[dev]"
```

**1. The H-05 test as it stands (marker in place):**
```
$ .venv/bin/pytest tests/unit/test_security.py -k test_verify_with_wrong_hash_format -v
...
================= 24 deselected, 1 xfailed, 1 warning in 1.73s =================
```

**2. The same test with the xfail marker ignored (`--runxfail`):**
```
$ .venv/bin/pytest tests/unit/test_security.py -k test_verify_with_wrong_hash_format --runxfail -q
...
>           raise exc.UnknownHashError("hash could not be identified")
E           passlib.exc.UnknownHashError: hash could not be identified

.venv/lib/python3.14/site-packages/passlib/context.py:1132: UnknownHashError
...
FAILED tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format
1 failed, 24 deselected, 1 warning in 0.12s
```
(The one warning is a Pydantic deprecation notice from `core/config.py:7`, not related.)

**3. Direct call, with a valid-hash control first:**
```
$ .venv/bin/python -u -c '
from core.security import hash_password, verify_password
good = hash_password("password")
print("valid hash, right password:", verify_password("password", good))
print("valid hash, wrong password:", verify_password("nope", good))
print("malformed hash:", verify_password("password", "not_a_valid_bcrypt_hash"))
'
(trapped) error reading bcrypt version
Traceback (most recent call last):
  File "/Users/eldadtolla/pathreview-ai301-fa26-s1/.venv/lib/python3.14/site-packages/passlib/handlers/bcrypt.py", line 620, in _load_backend_mixin
    version = _bcrypt.__about__.__version__
              ^^^^^^^^^^^^^^^^^
AttributeError: module 'bcrypt' has no attribute '__about__'
valid hash, right password: True
valid hash, wrong password: False
Traceback (most recent call last):
  File "<string>", line 6, in <module>
    print("malformed hash:", verify_password("password", "not_a_valid_bcrypt_hash"))
                             ~~~~~~~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/Users/eldadtolla/pathreview-ai301-fa26-s1/core/security.py", line 37, in verify_password
    return bool(pwd_context.verify(plain_password, hashed_password))
                ~~~~~~~~~~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/Users/eldadtolla/pathreview-ai301-fa26-s1/.venv/lib/python3.14/site-packages/passlib/context.py", line 2343, in verify
    record = self._get_or_identify_record(hash, scheme, category)
  File "/Users/eldadtolla/pathreview-ai301-fa26-s1/.venv/lib/python3.14/site-packages/passlib/context.py", line 2031, in _get_or_identify_record
    return self._identify_record(hash, category)
           ~~~~~~~~~~~~~~~~~~~~~^^^^^^^^^^^^^^^^
  File "/Users/eldadtolla/pathreview-ai301-fa26-s1/.venv/lib/python3.14/site-packages/passlib/context.py", line 1132, in identify_record
    raise exc.UnknownHashError("hash could not be identified")
passlib.exc.UnknownHashError: hash could not be identified
$ echo $?
1
```

**Expected vs. actual**
- Expected: `verify_password("password", "not_a_valid_bcrypt_hash")` fails closed and returns `False`, which is what `test_verify_with_wrong_hash_format` asserts.
- Actual: it raises `passlib.exc.UnknownHashError: hash could not be identified`, raised from `pwd_context.verify(...)` at `core/security.py` line 37. The control shows that `verify_password` works normally with a valid bcrypt hash (`True` for the right password, `False` for a wrong one), so the failure is specific to an unrecognizable hash.

**Side note:** the "(trapped) error reading bcrypt version" traceback at the top of step 3 is passlib 1.7.4 failing to read `bcrypt.__about__` on bcrypt 4.x. passlib catches it itself, and hashing and verifying with a valid hash still work (see the control lines), so I don't think it's part of this bug. I'm noting it so nobody mistakes it for the error.

Next I'll look at how `verify_password` should handle hashes passlib can't identify, and check what removing the H-05 marker means for the test.

(I'm using Claude Code to help run the steps and draft this comment; the output above is from my machine, and I checked it myself.)
````

## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

1. First full eval run (rubric with six required checks plus one preferred): 19/20 agreement,
   PASS. Categories: `clear-accept 7/8  disclosure 1/1  no-evidence 4/4  unfollowable-comms 3/3  wrong-target 4/4`.
   The one disagreement was pkg-03 (gold accept, graded reject, `failed: claims-backed, control-run`).
2. Revised `claims-backed` (loosened it; see Check rationale), then re-ran
   `--only pkg-03,pkg-20,pkg-15,pkg-17`: 4/4 agreement. pkg-03 flipped to accept; the three
   canaries held.
3. Confirming full run with `--save-run eval-run.txt`: 20/20 agreement, PASS. Categories:
   `clear-accept 8/8  disclosure 1/1  no-evidence 4/4  unfollowable-comms 3/3  wrong-target 4/4`.

Agreement line from the committed `eval-run.txt`:

```
agreement: 20/20 scored items  (bar: 18/20: PASS)
```

**Package analysis**

pkg-03 (BurntSushi/ripgrep#2779). Gold label: accept. My rubric's verdict: reject on the
first full run, accept after the revision.

The report reproduces the bug with the issue's exact command and shows its output, and my
`behavior-matches-issue` check passed it: "Report's output `1:fnord / 2:boccob /
3:d321fdddffff / 4:clowns` matches the issue's stated actual behavior from the same trigger
command". But the report also says, without showing any output: "Dropping `-r '$1'` from the same
command reports 1, 4, 7, 10 correctly, which matches the owner's note that `--replace` is
required to trigger it." My first `claims-backed` check required every claim to have a shown
artifact, so the grader failed it, with the evidence line "is asserted with no output artifact shown for
that run" (quoting that sentence). That sentence is a control run
described in words, not the outcome of the report. The outcome itself is backed. The gold
note calls this an "exact-steps repro on current version with a minus-replace control
matching the owner's trigger note", so the missing control output is a gap for my preferred
`control-run` check, not a reason to hold the package.

**Check rationale**

Quoted from `tools/repro-check/rubric.md` as it reads now:

| claims-backed | Every factual claim in the claim comment and repro report (reproduced, root cause, frequency, scope, "verified", "guaranteed"), each matched to a shown artifact (see evidence guide: Honesty) | Every claim that something happens, was verified, or is caused by X is backed by an artifact shown in the package, and the stated outcome matches the artifacts (an evidenced cannot-reproduce that names what differed passes). A side observation that supports the outcome but is not the outcome itself (for example, a control run described in words without its own output) does not fail this check when the reported outcome is backed by a shown artifact; the missing control output is what control-run records. Fail if a root cause, certainty, or generalization to environments not tested is asserted with nothing shown, or if expected vs actual is stated in a way the artifacts contradict. | required |

Why it reads this way: the first version stopped at "(an evidenced cannot-reproduce that
names what differed passes)" and then listed the fail cases. That rejected pkg-03, because the
grader applied "every claim ... is backed by an artifact" to a side sentence about a control
run. I added the sentence starting "A side observation that supports the outcome but is not
the outcome itself" so that claims-backed judges the claims that carry the report (the
outcome, root cause, certainty, generalization), and handed the missing control output to
`control-run`, which is preferred and never changes the verdict. I kept the fail list as it
was, because it is what catches pkg-13 ("guaranteed reproducible" backed by nothing), pkg-15
("I verified this race condition" with no transcript) and pkg-17 (generalizing to a release
nobody tested).

**Trade-offs**

This was a loosening, so I re-ran canaries with `--only` before spending a full run:
`--only pkg-03,pkg-20,pkg-15,pkg-17`. pkg-20 is the only package in the single-package
`disclosure` category, as the canary rule requires. pkg-15 and pkg-17 are rejects whose
failures depend on claims-backed (an unbacked root cause, an artifact that contradicts the
narration). All three stayed reject and pkg-03 flipped to accept (4/4), and the confirming
full run then agreed on all 20.

What the check gives up: a report can now describe a side observation in words that is
actually wrong, and claims-backed will not catch it as long as the main outcome has an
artifact. For example, a control run that was never really run, described as "works fine
without the flag", passes this check. Only the preferred `control-run` check notices the
missing output, and it cannot hold the package. I accept missing that case because the
decision about whether to post rests on the reported outcome, and holding pkg-03-style
reports for a narrated side note rejected a package the gold set accepts.

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/repro-check/`.
