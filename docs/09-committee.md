# The committee

## The pool

Twelve named committee members. **Three reviews are needed per swim.** Nobody is assigned.
It is an open call, and whichever three respond first make the determination.

There is no panel, no rotation, and no recusal mechanism, because there is no assignment to
recuse from.

Committee members' personal email addresses are not recorded in this repository.

## Why the form is cloned per swim

Jotform submission limits are per form. To cap a review at three votes for one swim, that swim
needs its own form. That is the entire reason for the manual clone.

| Setting | Value |
|---|---|
| Unique Submission, cookies or IP | No check |
| Unique Field | `Your Email` |
| Closes at | 3 submissions |

**These controls have already failed once.** The SWIM-1035 form holds four submissions.

## Per clone, what has to be edited

| Field | Example |
|---|---|
| Swim ID default | `SWIM-1035` |
| Swimmer Name default | The swimmer's name |

## How reviewers reach the form

The Jotform link is never sent. The cloned form is **embedded in the WordPress ratification page**,
in its Committee Review section. The post is published private with a password, and that page
link goes to the committee.

A reviewer opens one page, reads the complete ratification, the narrative, the swim details, the
map, the observer log, the gallery and the documents, and submits their determination from the
bottom of that same page.

This is why the page has to exist before the committee can review. The page is not the output of
the decision. It is the thing the decision is made against.

## The dispatch email

Composed manually, sent by Rose or sometimes Quinn, usually batched across several pending swims.

Structure, repeated per swim:

> Ratification N: `SWIM-XXXX` `Swimmer Name`
> View here: `https://openwaterswimming.com/ratifications/swim-xxxx-name/`
> Password: `[COMMITTEE_PASSWORD]`
> Status: N reviews received, needs N more

The status lines are counted manually by opening each form.

The password is one static string, the same for every swim, sent to all twelve members in a
message that also lists which swims are pending and where to find them.

## What a reviewer submits

A four-point assessment on each of photo and video evidence, observer logs and report, and GPS
tracks, plus a four-point overall recommendation, each with optional notes. Then three conflict
of interest questions and a signature.

**Three reviewers produce twelve graded assessments and up to twelve notes.**

## How those become a decision

**There is a rule, and it is a count.**

> Three ratified reviews from three committee members means the swim is ratified.

No judgment is exercised at the recording step. The admin reads the count and records the outcome.
The committee decides.

### The conflict this creates

The ratification page link and the review request go to **twelve** people. The cloned form is
capped at **three** submissions.

So the first three responses close the form. If one of those three returns not ratified, the swim
is recorded as not ratified immediately, even though a fourth reviewer might have ratified it and
produced three ratified reviews.

**The rule and the mechanism do not agree.** Two readings are possible and nobody has stated which
is intended:

| Reading | What it means |
|---|---|
| Three ratified reviews, regardless of any not ratified reviews from others | The pool stays open until three ratified reviews exist, or until it is clear they cannot be reached |
| The first three to respond decide | What the cap currently enforces |

A related ambiguity sits underneath it. Reviewers submit a four-point scale, not a binary. Whether
"Approve with qualifications" counts toward the three has never been written down either.

## What gets published

Two tables, built manually in the League Table plugin, replacing the embedded form on the page.

**`SWIM-XXXX Name - Evaluation`**, the conflict disclosures.

| | RELATIONSHIP | COMPENSATION | IMPARTIALITY IMPACTED |
|---|---|---|---|

**`SWIM-XXXX Name - Recommendations`**, the assessments.

| | Logs | Photos/Videos | GPS | Overall |
|---|---|---|---|---|

Reviewer names are published. The four-point scale is not: the published table shows a tick, so
"Approve with qualifications" and "Approve (no issues)" are indistinguishable to a reader, and
the notes exist only inside the Jotform submissions.

## Three shapes for one thing

Worth stating plainly, because it affects any redesign.

| Where | Shape |
|---|---|
| What reviewers submit | Four-point scale, four questions |
| How the outcome is reached | A count of ratified reviews, threshold three |
| What is published | A tick |
| What the workflow records | Ratified or Not Ratified |
