# Plan: issue #72 — `verify_password` raises `UnknownHashError` on malformed stored hashes

Issue: https://github.com/codepath/pathreview-ai301-fa26-s1/issues/72
Manifest id: H-05. Base: `main` at `f89c06f` (same commit my repro ran on).
Branch: `fix/72-verify-password-fail-closed`, on my fork.
My reproduction: https://github.com/codepath/pathreview-ai301-fa26-s1/issues/72#issuecomment-5863753449

## Diagnosis

`verify_password` hands the stored hash straight to passlib and lets passlib's
failure escape to the caller. `core/security.py:37` is the single statement
involved:

```python
return bool(pwd_context.verify(plain_password, hashed_password))
```

When `hashed_password` is not a hash passlib can use, `CryptContext.verify`
never reaches a password comparison: it fails earlier, while identifying or
parsing the hash, and raises. My repro pins that down. From my repro report,
step 3:

> ```
>   File "/Users/eldadtolla/pathreview-ai301-fa26-s1/core/security.py", line 37, in verify_password
>     return bool(pwd_context.verify(plain_password, hashed_password))
> ...
>   File ".../passlib/context.py", line 1132, in identify_record
>     raise exc.UnknownHashError("hash could not be identified")
> passlib.exc.UnknownHashError: hash could not be identified
> ```

The traceback is the diagnosis: the raise comes from passlib's
`identify_record` — hash identification — and the frame directly below it is
`core/security.py` line 37. There is no `try`/`except` between them, so the
exception becomes the caller's problem.

The control run in the same step rules out a broader breakage:

> ```
> valid hash, right password: True
> valid hash, wrong password: False
> ```

With a hash passlib *can* identify, the function already returns a bool for
both outcomes — including "wrong password", which is the same fail-closed
behavior the malformed case is missing. So the defect is the absence of
handling at that one call site, not the verification logic.

Expected behavior, from the issue body: verification against a malformed hash
"should fail closed (return `False`), not raise." That is also what the
covering test asserts (`assert result is False`).

### The failure has two shapes, not one

My repro covered one string, `not_a_valid_bcrypt_hash`. Several people on the
thread reported that a bcrypt-*prefixed* but broken hash raises a bare
`ValueError` instead, which would escape a fix that catches only
`UnknownHashError`. That is their finding, not mine, so I ran the shapes
myself on `main` at `f89c06f` in the same venv as my repro before letting it
change this plan:

```
$ .venv/bin/python -c '
from core.security import verify_password
for h in ["not_a_valid_bcrypt_hash", "$2b$notarealhash", "$2b$12$shorttoken", ""]:
    try: print(repr(h), "-> returned", verify_password("password", h))
    except Exception as ex: print(repr(h), "-> raised", type(ex).__module__ + "." + type(ex).__name__, "|", ex)
'
'not_a_valid_bcrypt_hash' -> raised passlib.exc.UnknownHashError | hash could not be identified
'$2b$notarealhash'        -> raised builtins.ValueError | not enough values to unpack (expected 2, got 1)
'$2b$12$shorttoken'       -> raised builtins.ValueError | salt too small (bcrypt requires exactly 22 chars)
''                        -> raised passlib.exc.UnknownHashError | hash could not be identified
```

Confirmed. Identification failure raises `passlib.exc.UnknownHashError`;
parse failure on a bcrypt-prefixed string raises a bare `builtins.ValueError`
from passlib's bcrypt handler. Both are the same defect — a stored value
passlib cannot use makes `verify_password` raise instead of returning `False`
— and both arrive from the same call on line 37.

`passlib.exc.UnknownHashError` is a subclass of `ValueError` (checked:
`UnknownHashError.__mro__` is `UnknownHashError -> ValueError -> Exception`),
so one `except ValueError` covers both shapes, and listing
`UnknownHashError` alongside it would be redundant.

### What I am not claiming

I am not claiming passlib is wrong to raise. Raising on a hash it cannot use
is reasonable library behavior; the bug is that this application's
`verify_password` is documented to return "True if password matches, False
otherwise" and does not hold that contract for a stored value it can be handed
from the database.

## Scope

**In scope — one bounded change, in two parts that have to land together:**

1. `verify_password` in `core/security.py` catches `ValueError` from the
   `pwd_context.verify(...)` call and returns `False`, so a stored hash
   passlib cannot use fails closed instead of raising. This covers both shapes
   above: `UnknownHashError` (a `ValueError` subclass) and the bare
   `ValueError` from the bcrypt parser.
2. The `@pytest.mark.xfail(strict=True, ...)` marker on
   `test_verify_with_wrong_hash_format` in `tests/unit/test_security.py` is
   removed, so the covering test runs and asserts the fixed behavior.

Part 2 is in scope and not optional. The marker is `strict=True`, so once
part 1 lands the test passes and a strict xfail turns that pass into a
failure — leaving the marker makes the suite fail. The issue body asks for it
directly: "The covering test is marked `@pytest.mark.xfail` referencing
manifest id H-05 — remove the marker as part of the fix." Naming it here is
what keeps Unit 4 from drifting from this plan.

**Out of scope — named so I do not drift into it:**

- `hash_password`, `create_access_token`, `decode_access_token`, and the
  `CryptContext` configuration on `core/security.py:12`. The repro's control
  shows them working; nothing in this issue touches them.
- Any catch wider than `ValueError`. No bare `except Exception`, which would
  swallow genuine programming errors (`TypeError` on a non-string, an
  `AttributeError` from a future refactor) and turn them into a silent
  `False` in an auth path.
- Adding a new test for the `$2b$`-prefixed shapes. The issue's covering test
  is the one I have to make pass, and I would rather not expand
  `tests/unit/test_security.py` beyond what H-05 asks for in the same change.
  I will show both shapes in the PR's before/after evidence instead, and if a
  reviewer wants them as test cases I will add them on request.
- The `(trapped) error reading bcrypt version` /
  `module 'bcrypt' has no attribute '__about__'` noise in my repro's step 3.
  I flagged it in my repro report as passlib 1.7.4 reading `bcrypt.__about__`
  on bcrypt 4.x; passlib traps it itself and the control lines still work, so
  it is a separate cosmetic matter and not part of this fix.
- The other 20-plus `@pytest.mark.xfail` markers in `tests/unit/` for other
  manifest ids. Only the H-05 marker comes out.
- Any change prompted by a contribution policy. `docs/CONTRIBUTING.md` is the
  repo's only contributing doc and it says nothing about AI use or disclosure,
  and there is no `AI_POLICY.md`, so no disclosure is required of me here. I
  disclose anyway, because my own voice guide asks me to.
- Any `pyproject.toml` change. I checked for a suppression tied to this issue
  and there is none: `grep -n "72" pyproject.toml` returns nothing, and
  `git grep -n "#72\b"` matches exactly one line in the whole repo — the xfail
  reason string at `tests/unit/test_security.py:220`. There is no
  `pytest.ini`, `setup.cfg`, or `tox.ini` either. So nothing to remove there;
  I am recording the check so that "no pyproject change" is a finding rather
  than an oversight.
- Logging the rejected hash. Tempting for debuggability, but it risks writing
  credential-adjacent material to logs and the issue did not ask for it. If a
  reviewer wants it, it is a follow-up.
- The login route's behavior (`api/routes/auth.py`). One commenter noted a
  malformed hash currently reaches its handler; whatever that route does, it
  is downstream of this contract and not part of H-05.

## Files

- `core/security.py` — `verify_password` (the function at line 27, the return
  at line 37), plus the import needed for the `try`/`except`.
- `tests/unit/test_security.py` — remove the `@pytest.mark.xfail` decorator at
  lines 218–221, above `test_verify_with_wrong_hash_format`.

These are exactly the two files the issue lists under **Relevant files**, and
the two the manifest entry for H-05 lists.

## Approach

1. In `core/security.py`, wrap the `pwd_context.verify(...)` call in
   `verify_password` in a `try`/`except ValueError` and return `False` from the
   handler. Keep the existing `bool(...)` on the success path, and keep the
   signature and return type unchanged — callers see no change.
   No new import is needed: `ValueError` is a builtin, and catching it covers
   `passlib.exc.UnknownHashError` as a subclass.
2. Update the function's docstring so the documented contract matches: note
   that a stored hash passlib cannot identify or parse returns `False` rather
   than raising.
3. In `tests/unit/test_security.py`, delete the `@pytest.mark.xfail(...)`
   decorator (lines 218–221) from `test_verify_with_wrong_hash_format`,
   leaving the test body untouched. The test already asserts
   `result is False`, which is the behavior step 1 adds.
4. Verify in the order under **Test plan** below: the repro first on `main`
   (before), then on the branch (after).

Steps 1–2 are one commit; step 3 is a second commit, so the diff for the
behavior change and the diff for un-parking the test can be read separately.

## Test plan

This is my Unit 2 repro, re-run. Same commands, same venv (`.venv`, Python
3.14.7, passlib 1.7.4, bcrypt 4.3.0), run on `main` at `f89c06f` for the
**before** and on the fix branch for the **after**. Both runs and their full
output go into `~/unit3-evidence.txt`.

**1. The H-05 test as it stands**

```
.venv/bin/pytest tests/unit/test_security.py -k test_verify_with_wrong_hash_format -v
```

- Before (quoted from my repro report):
  `================= 24 deselected, 1 xfailed, 1 warning in 1.73s =================`
- Expected after: `1 passed, 24 deselected` — not `xfailed`, and not
  `xpassed`. With the marker gone the test is an ordinary test, and it passes.

**2. The same test with the marker ignored**

```
.venv/bin/pytest tests/unit/test_security.py -k test_verify_with_wrong_hash_format --runxfail -q
```

- Before (quoted from my repro report):
  `passlib.exc.UnknownHashError: hash could not be identified` and
  `1 failed, 24 deselected, 1 warning in 0.12s`
- Expected after: `1 passed, 24 deselected`, with no `UnknownHashError` in the
  output. `--runxfail` becomes a no-op once the marker is gone, which is the
  point: the test that only failed because of the bug now passes on its own.

**3. Direct call, with the valid-hash control first**

```
.venv/bin/python -u -c '
from core.security import hash_password, verify_password
good = hash_password("password")
print("valid hash, right password:", verify_password("password", good))
print("valid hash, wrong password:", verify_password("nope", good))
print("malformed hash:", verify_password("password", "not_a_valid_bcrypt_hash"))
'
```

- Before (quoted from my repro report): the first two lines print `True` and
  `False`, then the third raises
  `passlib.exc.UnknownHashError: hash could not be identified` from
  `core/security.py", line 37, in verify_password`, and `echo $?` is `1`.
- Expected after: three lines, no traceback —
  ```
  valid hash, right password: True
  valid hash, wrong password: False
  malformed hash: False
  ```
  and `echo $?` is `0`. The first two lines are the control and must not
  change; the third flipping from a traceback to `False` is the fix. The
  `(trapped) error reading bcrypt version` notice may still appear above them
  (the unrelated bcrypt 4.x/passlib matter from my repro's side note) and is
  not part of the pass condition.

**4. Both failure shapes, since the fix now covers both**

```
.venv/bin/python -c '
from core.security import verify_password
for h in ["not_a_valid_bcrypt_hash", "$2b$notarealhash", "$2b$12$shorttoken", ""]:
    try: print(repr(h), "-> returned", verify_password("password", h))
    except Exception as ex: print(repr(h), "-> raised", type(ex).__module__ + "." + type(ex).__name__, "|", ex)
'
```

- Before (run on `main`, output quoted in the Diagnosis above): all four
  raise — two `passlib.exc.UnknownHashError`, two `builtins.ValueError`.
- Expected after: four lines, every one `-> returned False`, and no
  `raised` in the output. This is the step that shows the catch is wide
  enough; step 3 alone would pass even with the too-narrow
  `UnknownHashError`-only fix.

**5. Regression guard, in addition to the above**

```
.venv/bin/pytest tests/unit/test_security.py -q
```

- Expected after: every test in the file passes, with no new failures and no
  `xpassed`. In particular `test_hash_password_long_input`
  (`tests/unit/test_security.py:209`), which hashes a 1000-character password
  and asserts `verify_password(long_password, hashed) is True`, must stay
  `True` — that is the check that the `except ValueError` has not swallowed a
  legitimate verify. 1000 characters is under passlib's 4096 limit, so it does
  not collide with the `PasswordSizeError` case below. This is a guard, not
  the proof; steps 1 and 4 are what show the bug is fixed.

## Risks and unknowns

- **`except ValueError` is wider than `except UnknownHashError`, deliberately,
  and it does swallow one unrelated case.** passlib raises
  `passlib.exc.PasswordSizeError` — also a `ValueError` subclass — when the
  *password* exceeds 4096 characters. After this change that returns `False`
  instead of raising. Output from my venv on `main`:

  ```
  PasswordSizeError.__mro__: PasswordSizeError -> PasswordValueError -> ValueError -> Exception
  5000-char password -> passlib.exc.PasswordSizeError | password exceeds maximum allowed size
  1000-char password -> True
  ```

  I think `False` is the right outcome for an auth path (an over-long password
  should not authenticate, and should not 500 the login route either), but it
  is a real behavior change beyond the issue's text and I am flagging it
  rather than burying it. If a reviewer disagrees, the alternative is
  `except ValueError` with an explicit re-raise for `PasswordSizeError`, and I
  will make that change on request.
  *(Correction on my own earlier reasoning: I first wrote that the bcrypt
  72-byte limit raises here. It does not — the 1000-character line above
  verifies `True` on passlib 1.7.4 with bcrypt 4.3.0. `PasswordSizeError` at
  4096 characters is the real case.)*
- **A caught exception is a silenced signal.** After this change, a malformed
  stored hash looks exactly like a wrong password to every caller. That is
  what the issue asks for ("fail closed") and the safe direction for an auth
  path, but it does mean a corrupted-password-column bug would stop
  announcing itself here. I am not adding logging (out of scope above); if a
  reviewer would rather see a `logger.warning`, I will add it on request.
- **`strict=True` makes the two parts a single landing.** If I commit the
  `core/security.py` fix without removing the marker, `test_security.py`
  fails on `XPASS(strict)`. I know which order to do it in; I am flagging it
  because anyone bisecting the branch will see it at the intermediate commit.
- **Unknown: whether any caller depends on the raise.** I have not yet
  searched for `verify_password` call sites to confirm none of them catch the
  exception themselves and rely on it. I will check that before the PR; the
  issue and manifest both treat `False` as the intended contract, so I expect
  none, but I have not shown it.
- **Unknown: whether the shapes I tested are the whole set.** I tested four
  stored-hash values. There may be other malformed shapes that raise
  something outside `ValueError` entirely (a `TypeError` for a non-string
  stored hash, for instance — I have not tried `None`). Those would still
  escape, and widening to `except Exception` is not the answer. If one turns
  up, it is a follow-up with its own evidence.
- **Environment caveat, carried over from my repro.** My run is macOS 26.5.1
  (arm64) on Python 3.14.7, and I ran only the dependency-install part of
  `make setup` — no `.env`, no Docker, no migrations, since neither
  `core/security.py` nor this test touches the database. I have not run the
  full suite or CI on this change, only `tests/unit/test_security.py`.

## Deviations

Nothing changed. The plan held, and the change on
`fix/72-verify-password-fail-closed` is the plan as posted:

- `verify_password` in `core/security.py` wraps the existing
  `pwd_context.verify(...)` call in `try` / `except ValueError` and returns
  `False` (commit `580e6e7`). No new import was needed, as the Approach said:
  `ValueError` is a builtin and covers `passlib.exc.UnknownHashError` as a
  subclass.
- The docstring now states the fail-closed contract (same commit).
- The H-05 `@pytest.mark.xfail(strict=True, ...)` marker came off
  `test_verify_with_wrong_hash_format` with the test body untouched (commit
  `74f7570`), in a second commit as planned, so the behavior change and the
  un-parking read separately.
- No `pyproject.toml` change, no logging, no new test cases, no other markers
  touched — the out-of-scope list held.

Every expected-after in the Test plan came out as predicted, so no
re-scoping was needed mid-build: `1 xfailed` to `1 passed`, `1 failed` to
`1 passed` under `--runxfail`, the direct call from a traceback with exit 1
to `malformed hash: False` with exit 0, all four malformed shapes from
raising to `returned False`, and the file from `24 passed, 1 xfailed` to
`25 passed` with no `xpassed`. The full commands and output are in
`~/unit3-evidence.txt`.

**One open question from the plan is now answered, in the plan's favour.**
The Risks section said I had not yet checked whether any caller depends on
the raise, and that I would before the PR. I checked:

```
$ git grep -n "verify_password" -- ':!tests'
api/routes/auth.py:12:from core.security import create_access_token, hash_password, verify_password
api/routes/auth.py:80:        if not user or not verify_password(form_data.password, user.hashed_password):
core/security.py:27:def verify_password(plain_password: str, hashed_password: str) -> bool:
```

The only non-test caller is the login route, and its call sits inside a
`try` whose fallback is `except Exception` -> `HTTP_500_INTERNAL_SERVER_ERROR`
with `detail="Login failed"` (`api/routes/auth.py:103-109`). So nothing
depended on the raise; the broad handler was only converting it into a 500.
After this change that same request falls through to the existing
`HTTP_401_UNAUTHORIZED` "Invalid email or password" branch, which is the
correct outcome for a credential that cannot be verified. This confirms the
fix rather than changing it, so it is not a deviation — but it is new
information the posted plan listed as unknown, and it belongs in the PR
description in Unit 4.

The other unknowns the plan listed stay open and unchanged: a non-string
stored hash (a possible `TypeError`) is still untested, and the
`PasswordSizeError` trade-off is still a flagged behavior change awaiting a
reviewer's opinion.
