# Adding Opt-in tox-uv to Tektronix's CI Workflows

## Overview

Issue #354 in tektronix/python-package-ci-cd adds optional tox-uv to their testing workflows, which fits my CI interest. I built an opt-in switch plus docs. The PR is open, awaiting review: https://github.com/tektronix/python-package-ci-cd/pull/1329

## What happened

### Finding the issue

My first scan wasted grades on taken issues. Adding `-linked:pr` drops them before grading. From my scan notes (2026-10-27):

> Scan 2, to check the `-linked:pr` fix. It worked: 0 of the top 5 had a PR attached (scan 1: 4 of 5).

### Proving the problem

Plain tox passed 31 tests, but with tox-uv added it failed. The repo's pip `install_command` override breaks, since uv venvs lack pip. My fork's CI failed the same way in all 6 test jobs:

```
test-code / test-fast (ubuntu)   .tox/tests/bin/python: No module named pip
                                 tests: FAIL code 1
test-code / test-fast (windows)  D:\a\...\.tox\tests\Scripts\python.EXE: No module named pip
test-code / test-general (macos) .tox/py313/bin/python: No module named pip
```
https://github.com/etollaw/python-package-ci-cd/actions/runs/42998863934

### Asking instead of guessing

I didn't want to guess the design, so I posted my numbers on #354 and asked. The answer shrank the scope to docs, plus removing the repo's own override:

> "I would just document the caveat for overridden install commands. This repo's install command override should just be removed, it doesn't really need to use that install command anyway." (nfelt14, maintainer, on #354, 2026-11-11, about 7 hours after my question; I was assigned the issue)

### The fix

The fix adds a `use-tox-uv` input, off by default, and the docs explain the caveat. From my diff to `_reusable-test-code.yml`:

```diff
+      use-tox-uv:
+        description: A boolean indicating if the tox-uv plugin should be installed
+          alongside tox, so that tox uses uv to create virtual environments and install
+          packages.
+        required: false
+        default: false
+        type: boolean
...
-        run: python -m pip install tox
+        run: python -m pip install tox${{ inputs.use-tox-uv && ' tox-uv' || '' }}
```

### What the checks caught

The spellcheck caught "uv", so I added it to the known words:

```
WARNING -  mkdocs_spellcheck: (symspellpy) workflows/test-code.md: Misspelled 'uv', did you mean 'us', 'up', 'guv'?
Aborted with 2 warnings in strict mode!
```

The gate caught a stale main and a missing README note:

```
ship-gate: STOP. your own pr-precheck rejected feat/354-optional-tox-uv: verdict reject.
  - plan-fidelity [fail]: git diff main...HEAD includes .github/renovate.json, _reusable-sbom-scan.yml, ... (likely stale local main)
  - repo-standards [fail]: CONTRIBUTING.md asks that a PR adding functionality update the README; README is not in the diff and the description gives no reason
```
I reset main and added a README note, and the second run passed:
```
ship-gate: PASS. feat/354-optional-tox-uv passed your manifest's checks and your pr-precheck.
```

### After the fix

The tests pass with and without tox-uv:

```
# branch, plain tox
31 passed, 2 deselected
  tests: OK
# branch, + tox-uv, no uv_seed
31 passed, 2 deselected
  tests: OK
```

The PR is open, awaiting review. I'm proudest of the ship-gate catching bugs before a maintainer had to.

## Limitations and what's next

There's no speedup on this repo: 17.2 s without tox-uv vs 18.0 s with it, because Poetry still does the installing. Next I'd benchmark a uv-only project.
