# Ratifications: defects found

Small, individually fixable problems found while walking through the live systems on
16 August 2026. None of these depend on any rebuild.

---

## Live and customer facing

| Where | What |
|---|---|
| Airtable order automation | Script step shows "Fix configuration", `payload` reads Invalid value, trigger test failed twice |
| Airtable order automation | Add-ons are never captured, so anyone buying Record Attempt, Live Map Tracking or Blog Recap receives no instructions for what they paid for |
| Jotform workflow | Relay was never wired in. The entry email hardcodes the solo form, so every relay has run down the solo path |
| Route Approval email | Never states that route approval is not ratification, contrary to WOWSA's own support guidance |
| Confirmation email | Quotes GPS track distance rather than the shortest straight-line tangent, in a message headed RATIFICATION SUMMARY |
| Confirmation email | Says "I've copied the ratification committee" when the recipient list is the swimmer alone |
| Good luck email | Fires when the observer agreement clears, so it can arrive months before the swim |

---

## Forms

| Where | What |
|---|---|
| Independent Observer Agreement | The link labelled "WOWSA Marathon Swimming Rules & Regulations" points at the observer form itself |
| Independent Observer Agreement | Category and swim type are checkboxes, so both Assisted and Unassisted can be ticked at once |
| Independent Observer Agreement | Typo in the assisted equipment sublabel: "pourous" |
| Independent Observer Agreement | Relay appears as a swim type on a form no relay can currently reach |
| Pre-Swim Planning | "Will this be an unassisted or assisted swim?" is hidden, so the swimmer never declares the category that determines record eligibility |
| Pre-Swim Planning | "Who is your pilot?" is hidden and marked required |
| Post-Swim Swimmer Narrative | Swim ID is optional, has no input mask, and prefill is off on that workflow node |
| Post-Swim Swimmer Narrative | Publication consent is required with only one option, so it cannot be declined |
| Committee form | Conflict of interest questions are checkboxes, so Yes and No can both be ticked |
| Committee form, SWIM-1035 | Four submissions received where three is the cap. Needs correcting |

---

## Email copy

| Where | What |
|---|---|
| Observer notification | "Name has submitted **her** pre-swim planning form" hardcodes a pronoun on an email sent for every swim |
| Observer Confirmed | One `Name` field used twice in one sentence for two different people, so one is always wrong |
| Post-Swim Observer Report notification | Same `Name` field problem |
| Not Ratified email | Points the swimmer to the page for "the committee's full notes", where the page shows ticks and the notes live only in Jotform |
| Attach Certificate and Socials | The only email in the workflow whose subject does not carry the Swim ID |

---

## Workflow configuration

| Where | What |
|---|---|
| Approval nodes | Require Login is on for attaching a certificate and off for recording a ratification determination |
| Swim Completed task | Typo in the description: "next stp" |
| Attach Certificate task | Asset list names the Print-Ready Certificate twice and asks for a Square 1:1 that has no Canva template |
| Builder | Two detached Email nodes addressed to seven recipients, greyed out, purpose unknown |
| Builder | HubSpot nodes at seven points, inert since HubSpot was dropped, being removed |

---

## Naming and consistency

| Where | What |
|---|---|
| Drive folder A | Naming is inconsistent in practice: both `SWIM-0050 Firstname Lastname` and `Swim-1028` styles exist |
| ZeroSixZero maps | Some map slugs carry the swim date, others do not |
| Asset lists | Three different descriptions of the certificate and social deliverables across the task field, the confirmation email and the Canva templates |

---

## Things that need a decision, not a fix

- **The distance figure.** Confirm whether the confirmation email should quote the straight-line tangent rather than the GPS track distance.
- **Publication consent.** The swimmer narrative requires consent to publish with no way to decline, and Not Ratified swims are now published as verified attempts too.
- **The committee scale.** Reviewers give four-point assessments, the page publishes ticks, and the workflow records a binary. Three shapes for the same thing.
- **Appeals.** The Not Ratified email offers 30 days to appeal, and the workflow ends at that email with no route back in.
- **Resubmission after a route denial.** A corrected plan returns as a new submission with a new SWIM ID, and nothing links it to the first.
