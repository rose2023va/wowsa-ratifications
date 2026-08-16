# Platform constraints

Things the current tooling genuinely cannot do, as distinct from things that were never set up.

**Read this before proposing any change.** Each of these has already been hit, and each one is
the direct cause of manual work elsewhere in the process. Proposals that assume any of them away
will not survive contact with the system.

---

## 1. Jotform cannot schedule anything relative to a date field

Every email fires on an event. None can fire a set number of days before or after a date held in
a form field.

### What this causes

| Symptom | Where |
|---|---|
| The good luck email arrives when the observer agreement clears, which can be weeks or months before the swim | [03-flow.md](03-flow.md) |
| No reminder as a swim date approaches | |
| No prompt on the rule that Pre-Swim Planning is due 30 days before the swim | |
| No chase when a case sits at the same stage for weeks | |
| No detection of a case that has silently broken, because detecting one means noticing that time has passed | |
| **A person has to be the clock.** Someone checks Google Calendar for swim dates that have passed and marks the swim complete manually, and nothing downstream can happen until they do | Step 6 |

That last one is the single largest consequence in the process. Until a person notices, the
swimmer and observer cannot submit their post-swim documentation, and from their side that is
indistinguishable from being forgotten.

### What would resolve it

Anything outside Jotform that can read a date and act on it. This is the clearest justification
for a second system existing at all, and it does not require replacing Jotform.

---

## 2. Jotform cannot run steps in parallel

**Tested, not assumed.** The post-swim observer report and the swimmer narrative were built as
parallel branches. The workflow terminated as soon as one branch completed.

### What this causes

The entire workflow has to be a single sequential chain. In particular, the observer's report
blocks the swimmer's narrative, even though neither depends on the other. A swimmer who is home
and ready to write up their swim waits on an observer who may be travelling, slow or unreachable.

### Important for anyone reading the builder

The long single chain looks like a deliberate sequence. It is not. It is the only shape available.
Do not preserve that ordering in a redesign on the assumption that it encodes intent.

### The apparent exception

The Google Calendar branch runs alongside the main path and ends on its own. It works only
because nothing downstream waits for it.

---

## 3. Jotform email nodes have no CC or BCC

Copying anyone means adding them to the recipient list as a full To recipient.

### What this causes

On the Observer Confirmed email, Quinn, the observer and the swimmer appear in one To line,
visible to each other. There is no way to copy someone internally without them appearing on a
customer facing message, short of sending a second separate email.

---

## Field conditions

Not a tooling limit, but it constrains solutions just as hard.

**Nothing captured during a swim can depend on being connected.** A crew is on a boat in open
water. There is often no internet and frequently no laptop.

Two things have already been built and tested against this and neither held up:

1. An **online observer log** intended to capture entries live during the swim
2. A **real time photo submission form** intended to capture timestamps at the moment of capture

Both failed for the same reason. Any proposal that involves the crew submitting something while
the swim is happening will fail the same way.

What this leaves: the observer records on paper or in a fillable PDF, and the evidence is
submitted afterwards. Improvements have to work on what arrives after the fact, not on capturing
it live.

---

## Related, not a platform limit

**Airtable will not let the API edit an automation that contains a script step.** An attempt to
update the Ratification Order automation programmatically was rejected as a read-only node. Any
change to that script has to be pasted in through the Airtable interface by a person.

This is why the extended script covering product, add-ons and order link, drafted on
6 August 2026, was never applied. See [issue: order automation script](12-defects.md).
