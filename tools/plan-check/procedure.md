# Procedure: how this skill grades a plan package

<!--
THIS IS THE PART YOU WRITE, and it is a new kind of part. Weeks 1 and
2, SKILL.md carried a numbered workflow and you only wrote judgment
files. This week the workflow is gone from the frame: SKILL.md says
"execute procedure.md", and these are the operating steps you author.
The machinery is in your hands now.

Your operator swap is the design brief. When your executor stalled
because your rubric said WHAT to decide but not HOW to find the
evidence, that was a procedure gap. This file is where those gaps get
closed: a complete procedure lets someone who has never seen a plan
package before (a groupmate, or the skill itself) grade one exactly the
way you would.

Under each stage heading below, write the concrete steps for that
stage. The one-line note under each heading says what a complete
procedure must decide there. Write steps, not intentions: "read the
repro evidence before the plan, and note what behavior it pins down"
is a step; "understand the context" is a wish.
-->

## Read order

The repro evidence is read before the plan, always. Every check in the
rubric that can reject a plan compares the plan against evidence that
already exists; reading the plan first lets its confidence tell you
what the evidence said. So:

1. Read the issue section (title, body, reported behavior). Write down
   one sentence: the single behavior this issue reports. Every later
   scope judgment measures against this sentence.
2. Read the repo-facts block. Write down two things: whether the
   stated contribution policy asks for AI-use disclosure and whether
   that ask is limited to pull requests, and any bug-report or
   contribution ask the comment would have to satisfy.
3. Read the thread highlights. For each comment, write down the
   author's role as the package states it (OWNER, MEMBER, COLLABORATOR,
   CONTRIBUTOR, NONE) and, for maintainer-side authors only, any
   direction given: a diagnosis, a named culprit location, an approach
   proposed, an approach rejected, a request made of the reporter, or
   a linked PR. This list is the only thing clause (a) of
   `thread-and-convention` is graded against.
4. Read the repro-evidence block last among the evidence sections, and
   note three things separately: the behavior it pins down, the
   artifacts that pin it down (output, timings, exit codes, panics),
   and every control run with its result. Write the controls out as
   "with X removed/changed, the result was Y". The controls are what
   decide `grounded-diagnosis`, so they get written down before the
   plan is read.
5. Only now read the candidate plan, then the candidate plan comment.
   Read them once end to end without grading, then return to them per
   check.

In live mode, do steps 1-3 from GitHub and the repo's docs per
`references/evidence-guide.md`, take step 4 from the student's posted
repro comment on that issue, and take step 5 from `plan.md` and the
draft comment file. Before step 1, read `scope.md` and stop if the
issue is outside the scoped repo or the scope's repo line is still a
placeholder. Read `voice-guide.md` before grading the comment, and
collect any rule the draft breaks as a note for the summary; it does
not feed any check's grade.

## Evidence gathering

One gathering move per evidence family. Record the fact as a short
quote or a one-line paraphrase with its location, because every check
must name the fact that decided it.

1. **Stated cause** (for `grounded-diagnosis`): take the plan's
   diagnosis or "Cause:" sentence. Record it as a claim of the form
   "X causes Y". If the plan states more than one candidate cause,
   record the one its approach steps actually act on.
2. **Contradiction set** (for `grounded-diagnosis`): take the control
   runs and step results written down in read-order step 4 and test
   each against the recorded "X causes Y". Record any one of them
   whose result cannot be true if X causes Y. One such item is enough;
   name it and stop looking.
3. **Scope commitments** (for `bounded-scope`): list every distinct
   thing the plan commits to building, from both the in-scope line and
   the numbered approach steps, one line each. Mark each one "serves
   the reported behavior" or "beyond it". Then read the not-in-scope
   line and strike any "beyond it" item the plan explicitly defers.
   Record the surviving "beyond it" items.
4. **Location and approach** (for `executable-approach`): record the
   file, function, or call site the plan names, and the one approach it
   chose. If the plan names several candidate locations or layers
   without choosing, record that as "no choice made" rather than
   recording the list.
5. **Test plan and its anchor** (for `decisive-test-plan`): record
   what the test plan runs, and separately record the observable
   result it says to look for. Then check that the thing it runs is
   the repro evidence's own trigger (or a test encoding it). "Run the
   suite" with no per-bug outcome is recorded as "no observable named".
6. **Thread direction and policy ask** (for `thread-and-convention`):
   from the read-order step 3 list, take the maintainer-side direction
   items. For each, find the sentence in the plan comment that
   addresses it, and record that sentence or record "not addressed".
   Then take the policy note from read-order step 2 and record whether
   the comment names AI assistance and its extent.
7. **Unestablished dependencies** (for `honest-uncertainty`): list
   what the plan's success depends on that the repro evidence does not
   show (a cost not measured, a platform not tested, a behavior change
   needing maintainer agreement). For each, record whether the plan
   names it as a risk or open question, or states it as decided.

## Check execution

Checks execute in rubric order: `grounded-diagnosis`,
`bounded-scope`, `executable-approach`, `decisive-test-plan`,
`thread-and-convention`, `honest-uncertainty`. Every check is graded;
none is skipped once another has failed, because the author needs the
whole picture.

For each check:

1. Apply the pass condition to the facts gathered for that check and
   nothing else. Do not re-read the whole package; the gathering step
   for that check already pulled what the condition names. Return to
   the package text only to confirm an exact quote.
2. Grade `pass` when the pass condition holds on the gathered facts,
   `fail` when the condition's fail clause is met by a named fact.
3. Grade `?` (unclear) only when the package does not contain the
   evidence the check's Evidence column names, so the pass condition
   cannot be applied either way. Absence of a *fault* is never
   unclear: a plan with no risks section and no overclaim passes
   `honest-uncertainty`; a thread with no maintainer direction passes
   clause (a) of `thread-and-convention`; a short plan that names one
   site and one approach passes `executable-approach`. Unclear is for
   missing evidence, not for thin writing.
4. Write the one line of evidence that decided the grade: the quote or
   the recorded fact, not a restatement of the rule. "Looks fine" is
   not evidence and neither is "the diagnosis is grounded"; "step 3
   times 25.8s with `--paging=never`, so the pager is not in the loop"
   is.
5. Judge only what the condition names. Length, polish, headings,
   tone, and confidence never move a grade. A terse plan and a long
   one are graded identically.
6. If the pass condition genuinely cannot be applied because this
   procedure is silent on the situation, grade the check `?`, and say
   in the summary which step was missing rather than inventing one.

## Verdict assembly

1. Collect the grades for the five `required` checks.
2. If all five are `pass`, the verdict is `accept`. Otherwise it is
   `reject`.
3. Treat `?` on a required check as a fail for this purpose, per the
   rubric's verdict rule. Do not round `?` up to a pass and do not
   treat it as a separate verdict; the verdict space is binary.
4. Ignore `honest-uncertainty` when computing the verdict, whatever
   its grade. Report it anyway.
5. Name the deciding check. On a reject, that is the first required
   check graded `fail` or `?` in rubric order; quote its evidence line
   in the summary as the reason. On an accept, the summary says every
   required check passed and quotes the evidence line of the check
   that was closest to failing, so the author knows where the plan is
   thinnest.
6. Emit the readable summary (one line per check with its grade and
   evidence, plus any voice-guide notes in live mode), then the fenced
   JSON block from SKILL.md as the last thing in the output, with every
   check listed in rubric order and the verdict from step 2.
