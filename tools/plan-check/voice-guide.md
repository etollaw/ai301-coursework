# Voice guide: how I talk upstream

<!--
THIS IS THE PART YOU WRITE (new this week). Live mode reads this file
before any comment of yours goes out the door; eval mode ignores it
entirely, because your voice is yours and carries no gold labels.

This is not etiquette. "Be polite and concise" is advice for everyone
and therefore rules for no one. Write rules YOU need, in your own
words, each one concrete enough that the skill can hold a draft
against it and say which rule it breaks.

Three sections. Fill all three.
-->

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

## Things I never post

- A fix date or a "this will be quick"
- "Same as above" or "+1": my proof goes up in my own words
- A root cause stated as fact before I've shown it
- Blame or frustration aimed at the maintainers or the code
- Anything I didn't run myself presented as if I had
