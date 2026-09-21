# Notes

Fill this in and send it back, together with your workflows.

Both files go in this `submission` folder:

- `NOTES.md`, this file, filled in.
- your two workflows, exported from n8n with **Download**: the repaired daily
  report, and the backfill. Any file names are fine. Send one file if you did
  not get to part 3.

Keep it short. Short and honest beats long and polished. Delete the italic
prompts as you go.

A root cause is not a symptom. "It reports 100 orders" is a symptom. "It asks
for one page and ignores the rest" is a cause. We are marking the causes.

---

## Fault 1: "the report says exactly 100 orders on busy days"

**Root cause, in one sentence:**



**What I changed:**



**How to prove it is gone:**

*Give the exact command or check we can run, and the number it should
produce. "It looks right now" is not proof.*



---

## Fault 2: "the revenue is too low"

**Root cause, in one sentence:**



**What I changed:**



**How to prove it is gone:**



---

## Fault 3: "late orders land on the wrong day"

**Root cause, in one sentence:**



**What I changed:**



**How to prove it is gone:**



---

## Part 2: how did you make a second run safe?

**What the workflow does now when the day already has a report:**



**How to prove it:**

*Trigger the same date twice and show us what to look at.*



---

## Part 3: the backfill

**How it decides which days to do:**



**What it does with a day that has no orders, and why:**



**What it does when one day in the range fails:**



**How to prove it:**



---

## Which fault would have been caught earlier, and by what?

*A test, an alert, a check on the data. Something that would have caught it
before the sales lead noticed.*



## What is still wrong that you did not fix?

*Anything unfinished, half-done, or that you know is wrong. This section
counts in your favour. We would rather read "I did not fix the third one" than
find it ourselves.*



---

## Anything else

*Optional. Assumptions you made, questions the brief did not answer, a
decision you are unsure about.*
