# Forms

Field level detail for every form in the process. Read from the Jotform API, so these are exact.

Full context for where each sits in the sequence is in [03-flow.md](03-flow.md).

---

## Pre-Swim Planning, `210815057331043`

The entry point. Submitting this starts the workflow and mints the SWIM ID.

| # | Field | Type | Required | Notes |
|---|---|---|---|---|
| 1 | Pre-Swim Planning | Header | | Subtitle: In Accordance with WOWSA Rules & Regulations |
| 2 | Swim ID | Calculation | No | Marked DO NOT EDIT |
| 3 | Auto ID | Auto increment | | **Hidden.** Sequential, 6 characters, 4 digits padding, no prefix. Index 1060 |
| 4 | Name | Full name | Yes | |
| 5 | Email | Email | Yes | |
| 6 | Birthday | Date | Yes | Minimum age 13 |
| 7 | My Gender is | Radio | Yes | Female, Male, Other |
| 8 | Address | Address | No | **Hidden** |
| 9 | What is the date of your planned swim? | Date | Yes | Drives the calendar entry |
| 10 | Body of Water | Text | Yes | |
| 11 | General Route Description | Text | Yes | Used in almost every downstream email |
| 12 | Route Type | Text | Yes | one-way, multi-way, circumnavigation, island loop, stage swim, combination |
| 13 | Swimming Stroke | Text | Yes | |
| 14 | Route can be done shore to shore | Radio | No | Yes / No |
| 15 | If no, justify your exception request | Text | No | |
| 16 | Route defined by permanent natural landmarks | Radio | No | Yes / No |
| 17 | If no, please explain | Text | No | No buoys, anchored boats, docks, flags or GPS waypoints as turn points |
| 18 | Start Location Name | Text | Yes | |
| 19 | Start Location Coordinates | Text | Yes | Decimal degrees |
| 20 | Finish Location Name | Text | Yes | |
| 21 | Finish Location Coordinates | Text | Yes | Decimal degrees |
| 22 | Straight line tangent distance | Text | Yes | Google Maps recommended |
| 23 | Screenshot of your plotted course | File upload | Yes | Multiple, 10 MB limit |
| 24 | Will this be an unassisted or assisted swim? | Textarea | No | **Hidden.** See defects |
| 25 | Primary independent observer | Full name | Yes | |
| 26 | Primary Observer Email Address | Email | Yes | Drives observer assignment |
| 27 | Observer's credentials | Textarea | Yes | |
| 28 | Who is your pilot? | Textarea | Yes | **Hidden and required** |
| 29 | Name | Full name | No | Hidden |
| 30 | GPS / tracking devices | Textarea | No | |
| 31 | Equipment beyond swimsuit, cap, goggles, earplugs | Textarea | Yes | |
| 32 | Will your swim be a record attempt? | Textarea | No | Asks for category, organisation, current holder |
| 33 | Anything else about your swim | Textarea | No | |

**Relay Pre-Swim Planning**, `251746582157161`, exists and is connected to nothing.

---

## Independent Observer Agreement, `251242794026152`

Three pages, three separate signatures.

### Page 1, Roles and Responsibilities

| Field | Type | Required |
|---|---|---|
| Swim ID | Text, masked `@@@@-####` | Yes |
| Name | Full name | Yes |
| E-mail | Email | Yes |
| Any other responsibilities besides observer? | Textarea | Yes |
| Swims previously observed, with swimmer, date, ratifying body | Textarea | Yes |
| Embedded PDF, Observer Pilot and Crew Roles and Responsibilities | Widget | No |
| Signature | Signature | Yes |

Six undertakings: remain impartial; document accurately using the Observer Log Template;
prioritise swimmer safety while enforcing compliance; disclose conflicts before the swim; submit
a complete and truthful report; accept the role is essential to the credibility of the swim.

### Page 2, Rules of Engagement

| Field | Type | Required |
|---|---|---|
| Category of marathon swim | Assisted / Unassisted | Yes |
| If assisted, equipment used | Textarea | No |
| Type of marathon swim | One-Way Solo / Solo Multi-Leg / Solo Stage / Relay | Yes |
| Link to the marathon swimming rules | Text | |
| Any reason you cannot enforce the rules? | Yes / No | Yes |
| If yes, which rules and why | Textarea | No |
| Signature | Signature | Yes |

Five undertakings, including the observer's authority to pause or terminate the swim.

### Page 3, Conflict of Interest Disclosure

| Field | Type | Required |
|---|---|---|
| Relationship with swimmer, crew, organising party or outside entity? | Yes / No | Yes |
| If yes, describe | Textarea | No |
| Receiving compensation or benefits? | Yes / No | Yes |
| If yes, amount and other forms | Textarea | No |
| Any other circumstance affecting impartiality? | Yes / No | Yes |
| If yes, describe | Textarea | No |
| Signature | Signature | Yes |

Non-disclosure may result in disqualification of the swim, withdrawal of ratification, or loss of
standing as an impartial observer.

---

## Post-Swim Observer Report, `252602743229455`

Titled "Post-Swim Ratification". Sections: Swimmer Information, Swim Location Time and Distance,
Route and Swim Details, Support Crew and Vessels, Upload Observer Log, Upload GPS Data, Upload
Photos and Videos, Observer Reflections and Narrative.

**Only one field on this entire form is required, and it is the Swim ID.**

Everything below is optional:

- Observer name, email, phone, previous experience
- Swimmer name, date of birth, gender
- Date and time of swim start and finish, with timezones
- Start and finish location names and GPS coordinates
- Exact GPS swim distance covered
- Shortest straight-line tangent distance, labelled "for Record Setting Purposes"
- Exact elapsed swim time, and stage swim per-day times
- Body of water name and type, route description, route type, stroke
- Equipment used
- Escort pilot name, email, experience, boat name
- Support vessel name, type, home port
- **Observer log file upload**
- **GPS data file upload**
- **Start and finish videos**
- **Hourly photographs**
- Rules difficult to interpret or enforce
- Observer narrative
- **The observer's attestation signature**

The attestation reads:

> I personally witnessed the attempt from start to finish and that, to the best of my knowledge
> and belief, the swim was conducted in accordance with WOWSA Marathon Swimming Rules (or as
> otherwise disclosed in this report). The times, locations, and supporting materials submitted
> are complete and accurate, and any exceptions, deviations, or uncertainties are noted in this
> documentation.

Uploads are capped near 10 MB. The GPS section instructs observers to zip GPX files, and to email
them directly if that fails.

---

## Post-Swim Swimmer Narrative, `252386318465161`

Titled "Submit Your Swim Report".

| Field | Required |
|---|---|
| Name | Yes |
| E-mail | Yes |
| Swim ID | **No**, and no input mask |
| What you accomplished and why it matters | No |
| Narrative Overview | No |
| Hardest moment and how you responded | No |
| Post-swim reflections | No |
| What you would do differently | No |
| Lessons for other swimmers | No |
| Who you want to thank | No |
| Anything else for the community | No |
| Headshot for the certificate | **Yes** |
| Additional images | No |
| Or share an album link | No |
| WOWSA may publish selected quotes and media | **Yes**, single option, cannot be declined |
| Captcha | Yes |

Prefill is switched **off** on this node in the workflow.

---

## Committee Determination and Certification, template `252415408134147`

Cloned per swim. See [09-committee.md](09-committee.md).

| Field | Type | Required |
|---|---|---|
| Your Name | Full name | Yes |
| Your Email | Email | Yes, and the unique field |
| Swim ID | Masked text, pre-filled per clone | Yes |
| Swimmer Name | Text, pre-filled per clone | Yes |
| General Route Description | Textarea | Hidden |
| Date of swim finish | Date | Hidden |
| Photo & video evidence acknowledgement | Four-point scale | Yes |
| Qualifications / Notes | Textarea | No |
| Observer logs & report acknowledgement | Four-point scale | Yes |
| Qualifications / Notes | Textarea | No |
| GPS swim tracks acknowledgement | Four-point scale | Yes |
| Qualifications / Notes | Textarea | No |
| Overall recommendation | Four-point scale | Yes |
| Qualifications / Notes | Textarea | No |
| Three conflict of interest questions | Yes / No | Yes |
| Signature | Signature | Yes |

The four-point scale, identical throughout:

- Approve (no issues)
- Approve with qualifications
- Insufficient, needs follow-up
- Not Approved, see notes
