# Voice guide: how I talk upstream

## Who I am in threads

I'm a student making my first contributions, working in the course's Path Review
repo. I read code I didn't write and I care about tests and CI. When I comment,
readers get what I actually ran and saw, and a plain next step, not a verdict on
their code.

## Rules I write by

### Rule: Promise the investigation, not the fix

I say what I'll look into next. I never promise a fix, an outcome, or a date.

- Wrong: "I'll have a fix for this up by Friday."
- Right: "Next I'll reproduce this locally and post what I find here."

### Rule: Name this issue, not any issue

Every comment names something only this issue has: the function, the error,
the marker, the trigger.

- Wrong: "Hi, I'd like to work on this, please assign me!"
- Right: "I'd like to look into `verify_password` raising `UnknownHashError` on a malformed hash."

### Rule: Output, not adjectives

If I say something happened, the output is right there. If I only think
something, I label it as a guess.

- Wrong: "Confirmed, it's definitely a passlib bug."
- Right: "On my machine the test raises `UnknownHashError` (output below). My guess is the hash-identification step, but I haven't checked that yet."

### Rule: Say what I didn't do

If I skipped part of the setup, used a different version, or couldn't
reproduce, I say so in a sentence instead of leaving it out.

- Wrong: "Reproduced on the latest version."
- Right: "Reproduced on Python 3.12 on macOS; I didn't try the Docker setup."

### Rule: Disclose AI help plainly

When an AI assistant helped me write a comment or run the steps, I say so in
one line.

- Wrong: (no mention, comment drafted with an assistant)
- Right: "I used Claude Code to help run the steps and draft this comment; I checked the output myself." 

### Rule: The title says what changed, in the repo's format

A PR title is a commit subject: Conventional Commits type and scope,
then the behavior that changes, in under 72 characters. No issue
number alone, no "Fix bug", no exclamation marks.

- Wrong: "Fix #72!!"
- Right: "fix(security): verify_password returns False on malformed stored hashes"

### Rule: The description promises exactly the diff

One Changes bullet per change in the diff, nothing the diff doesn't
contain, nothing in the diff left out. If the build changed course
from the posted plan, the deviation is stated in the description too,
not only in `plan.md`.

- Wrong: "Implements the plan exactly; also cleaned up some tests."
- Right: "- Catch `ValueError` around `pwd_context.verify` and return `False`
  (covers `UnknownHashError` and malformed bcrypt strings)."

### Rule: Evidence is pasted, not summarized

Every check result in the description points at real output: the
command and the line it printed. A box is ticked only when I ran the
thing and it passed; anything I couldn't run (CI before the PR
exists) stays unticked with a sentence saying why.

- Wrong: "- [x] All tests pass"
- Right: "- [x] Unit tests pass (`make test-unit`, pass line pasted under Testing)"

### Rule: Shortfalls are stated plainly, not apologized for

If the PR deliberately does less than the issue could ask, I name
what is left and why in one sentence, without "sorry" or "hopefully".

- Wrong: "Sorry, I didn't get to the other cases, hopefully that's ok."
- Right: "Not in this PR: `hash_password` input validation; it is a separate behavior."

## Things I never post

- A fix date or a "this will be quick"
- "Same as above" or "+1": my proof goes up in my own words
- A root cause stated as fact before I've shown it
- Blame or frustration aimed at the maintainers or the code
- Anything I didn't run myself presented as if I had
- A ticked checkbox for something I didn't run, or "tests pass" with no output
- A PR description that claims coverage the evidence doesn't show
