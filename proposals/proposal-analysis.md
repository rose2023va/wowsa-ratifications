# Proposal analysis

Forward looking. Everything in `docs/` describes the system **as it is**. This document is the
only place that discusses changing it.

Two things are analysed here:

1. **Quinn's proposed redesign**, and what each part of it would touch
2. **The combined plan**: Jotform keeps the forms, Airtable becomes the engine, Lovable renders the pages

Read [../docs/03-flow.md](../docs/03-flow.md) first. Nothing below makes sense without it.

---

## Part 1. What Quinn's proposal touches

Nine parts of the proposal land on an existing step. Five have no equivalent step at all.

| Mark | Proposal | Against the current system | Status |
|---|---|---|---|
| 1 | Three identifiers: Swim, Submission, Ratification | SWIM ID is the only identifier and is already labelled "RAT #" in the confirmation email. Adds two more namespaces and changes page URLs. | Replaces |
| 2 | Ratified Distance and GPS Track Distance shown separately, with a versioned methodology | One figure exists, measured manually in Google Maps and never recorded on any record. The confirmation email currently quotes the GPS figure. | New |
| 3 | Gate 1: four non-negotiable thresholds, auto-blocking | The route review is a single approval with no thresholds and no auto-check. | New |
| 4 | Gate 1 evidence check before committee | The observer report requires nothing but a Swim ID, and completeness is checked by a person opening a folder. | New |
| 5 | Page created at clearance, private draft through review, published on decision | This is already the design. The genuine delta is per-member authenticated access instead of one shared password. | Already exists |
| 6 | A live committee docket replacing the cloned form and the dispatch email | The form is cloned per swim only because submission caps are per form. Dispatch and reviewer counting are manual. | Replaces |
| 7 | Comments anchored to named parts of the record | Reviewer notes live inside form submissions and never reach the page. The proposal does not specify a mechanism. | New |
| 8 | Binary unanimous vote, 3 of 3, named reviewers, recusal and alternates | Reviewers give a four-point assessment on four questions. Twelve people are asked, whichever three respond first decide, so there is no assignment to recuse from. | **Conflicts** |
| 9 | WOWSA Decision, GWR Referral and GWR Status as separate lines, plus amendment versions | One outcome line. Amendments arrive as email replies with no record of what changed. | New |
| — | Public status page per submission | Nothing equivalent. Status is checked against a manually maintained Airtable table. | New |
| — | Public registry of every submission, permanent | Nothing equivalent. A permanent transparency commitment. | New |
| — | Notion as a precedent and policy repository | Nothing equivalent. Adds a tool and an ongoing writing workload. | New |
| — | Shared canonical data graph and stable join keys | Nothing equivalent. Forward planning for a consumer directory that does not exist yet. | New |
| — | Public protocol explainer page | Substantially exists already across the knowledgebase. | Already exists |
| — | "Not Ratified" replacing "Rejected" | "Rejected" has never been used. The terms in use are Not Ratified, attempted, and verified attempt for the public and social wording. | Already exists |
| — | Publishing Not Ratified decisions | Already happens, and those swims also receive a certificate and social assets as a verified attempt. | Already exists |

### The one real conflict

Mark 8. The proposal describes the committee decision as "always a unanimous 3 of 3 vote" of
Ratified or Not Ratified, with quorum, recusal and alternates.

That is not what happens. Reviewers submit a four-point assessment on each of three evidence
categories plus an overall recommendation. Twelve people are invited and whichever three respond
first decide. Nobody is assigned, so there is nothing to recuse from.

Adopting mark 8 would be a **change to how the committee works**, not a description of it, and it
would flatten a graded assessment into a binary one.

### WordPress

The proposal replaces the WordPress ratification pages with Lovable. Every public page section in
the document is labelled "PUBLIC PAGE · LOVABLE", and section 1 states it directly:

> Replaces: Manually built WordPress pages, a separate database-update step after publishing, and
> a page-build step that used to happen only after ratification.

The document does not address the separate WordPress install, the already indexed and already
circulated page URLs, the League Table plugin, or the embedded committee form that the entire
review depends on.

---

## Part 2. The combined plan

| Tool | Role | Change |
|---|---|---|
| **Jotform** | Forms and file uploads. Knows who each form belongs to, prefills and routes them | Keep. Stops running the process |
| **Airtable** | The single record for every swim. Runs every stage and every email. Acts on dates, chases, flags gaps | Build. Replaces the Jotform workflow |
| **Lovable** | Public ratification pages, private committee view and docket, public status pages | New. Replaces WordPress pages |

Jotform answers into Airtable. Airtable sends the next form when it is due. Lovable reads the record.

### Why this matters more than it looks

All three platform constraints in [../docs/11-constraints.md](../docs/11-constraints.md) exist because
**Jotform runs the process**. Moving the engine to Airtable removes all three at once:

| Constraint today | Consequence removed |
|---|---|
| Cannot act on a date | Post-swim forms go out on their own. Nobody checks a calendar |
| Cannot run steps in parallel | Observer and swimmer are asked at the same time |
| No CC or BCC | Internal copies stay internal |

Three defects also disappear structurally rather than being fixed:

- **Relay** becomes a field on one record instead of a second workflow
- **Add-ons** carried on the order trigger their own instructions
- **Rescheduling** moves everything after it, with no restart

### What becomes automatic

Most of this is ordinary automation rather than AI. Only four rows genuinely need a Claude agent:
reading the observer log, writing the introduction, drafting chase emails, and producing the Canva
assets. Everything else is Airtable or Lovable doing what they already do.


| Stage | Today | After | How, and what does it |
|---|---|---|---|
| Order and intake | Automatic, but add-ons and solo versus relay are lost | Automatic and complete | **Airtable automation.** The order webhook writes the full line item, including the variation and any add-ons, straight onto the record |
| Route distance | Measured in Google Maps, screenshotted, emailed | Calculated from the coordinates and written to the record | **Airtable automation calling the distance service.** No AI involved, it is a calculation |
| Route approval | Emailed to Quinn, reply, then recorded separately | Quinn approves in one place and it records itself | **Airtable interface.** Quinn sees the route, the distance and any flags on one screen and approves there |
| Knowing the swim happened | Manual calendar check | Automatic | **Airtable scheduled automation.** Runs daily, finds swim dates that have passed, moves those records on |
| Folders and file collection | One folder made manually, one automatic | Both automatic | **Airtable automation into Google Drive.** The swim folder is created from the template on clearance |
| Checking evidence is complete | Opening folders and noticing | Flagged automatically, with the chase email drafted | **Airtable formula and view** for what is missing. **A Claude agent** writes the chase email, because the wording depends on who is being chased and why |
| Observer log | Every entry retyped | Read from the submitted log, held as data | **A Claude agent.** Reads the paper or PDF log the observer actually produced and extracts each timestamped entry. Genuine AI work, not automation |
| The introduction | Generated, then placed manually | Generated onto the record | **A Claude agent** against the existing tone, flow and wording rules |
| Building the page | Assembled manually across six systems | The page is the record. Nothing to assemble | **Lovable.** Reads the record and renders it. There is no build step to perform |
| Committee form | Cloned, edited and embedded per swim | No form to clone | **Lovable.** The review sits on the record, with access granted per member rather than per form |
| Committee dispatch and chasing | Email written manually, reviews counted manually | Automatic, with reminders to whoever has not responded | **Airtable automation.** Counts reviews as they arrive and reminds only the people still outstanding |
| Publishing the result | Two tables built manually, form removed, names and signatures added | Renders from the reviews as they arrive | **Lovable.** The determination panel reads the review records directly |
| Certificates and social cards | Made manually for every ratification | Generated from the record | **A Claude agent driving Canva.** Fills the templates from the record and exports into the swim folder |
| Status tracking | Typed in at every stage | The record is the status | **Airtable.** Each stage is written by whatever step performed it, so there is nothing to keep in sync |

### What still needs a person

Five things. These are the parts where WOWSA's judgment is the product, not the overhead.

| | What | Why it cannot move |
|---|---|---|
| Judgment | **Approving a route** | Everything can be prepared, measured and checked. Whether the plan is sound is a person's call |
| Judgment | **The committee's assessment** | Three people deciding whether a swim meets the standard. This is what WOWSA is for |
| Judgment | **Whether evidence is credible** | A system can confirm a photograph exists and carries a timestamp. It cannot judge whether the photograph shows the swim |
| Judgment | **Borderline threshold calls** | Whether an observer's role genuinely overlaps a crew role is not a pass or fail check |
| Field conditions | **Capturing anything live during a swim** | Already tried, twice. An online observer log and a real time photo submission form were both built and tested. Crews are in open water with no internet and often no laptop. Nothing captured during a swim can depend on being connected |
| Craft | **The video, and choosing the photographs** | Which moments carry a swim is an editorial decision. The video is also cut in desktop software that nothing can reach |

**Turning the committee's assessments into a determination also stays with a person.** It is not a
rule waiting to be written down.

---

## Part 3. Recommendations, in neither plan

| # | Recommendation | Why |
|---|---|---|
| 1 | Extract the observer log from whatever the observer submits, rather than retyping it | The largest repeated task in the process. **Note:** an online observer log and a real time photo submission form were both built and tested already, and neither held up. Crews are on a boat in open water with no internet, often no laptop. The answer is not another live form, it is reading the paper or PDF log the observer actually produced |
| 2 | Generate certificates and social assets from the record | Certificates appear nowhere in the proposal |
| 3 | Make the distance service the single source for both distance figures | Removes the manual measurement, fixes the wrong figure currently quoted to swimmers, and is what a versioned methodology needs to sit on. **In progress.** The service is built; circumnavigation routes still need work with Anthony at ZeroSixZero. https://github.com/rose2023va/wowsa-distance-calculator |
| 4 | Score evidence completeness on the record | Answers "is this ready for the committee" without anyone opening a folder, and without changing what the forms ask for |
| 5 | Surface anything sitting too long at one stage | The most common failure today is silent. A case that has not moved is the only signal needed to catch it |
| 6 | Remind committee members who have not responded | Chasing reviews is currently manual and batched. It should be a nudge per person per swim |
| 7 | Send add-on instructions automatically on purchase | People are paying for add-ons today and receiving nothing |
| 8 | Keep one identifier, not three | SWIM ID is already public, already in every URL, and the confirmation email already presents it as the ratification number. Adding two more namespaces creates work without answering a question anyone is asking |

---

## Sequencing note

Recommendations 3 and 7, and the relay fix, do not depend on any of the above. They can be done
against the system as it stands today. Everything else assumes Airtable has become the engine.
