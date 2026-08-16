# The flow, end to end

How the swim ratification workflow runs today, app by app, in the order a case moves through it.

Status: in progress. Captured from walkthrough, 16 August 2026.
Anything not yet confirmed is listed under Open questions rather than filled in.

Note on personal data: swimmer names, email addresses and order numbers are visible in the live systems but are deliberately not reproduced in this document.

---

## Step 1. The customer places an order

**Where:** https://www.openwaterswimming.com/product/swim-ratification/

WooCommerce **variable product**, ID `70664`, with two variations:

| Variation ID | Name |
|---|---|
| `102119` | Solo |
| `102120` | Relay |

No default variation is set, so the customer must choose Solo or Relay before buying.

The product listing carries a **purchase note** containing links to Jotform.

Add-ons offered on this product: Record Attempt, Live Map Tracking, Blog Recap and Social Sharing.

---

## Step 2. The order reaches Airtable

**Base:** Ratifications Pipeline, `appgUjmgd0K8WWp31`

**Automation:** "Ratification Order"
https://airtable.com/appgUjmgd0K8WWp31/wflxdCC6f3GFBBwHq/wac1dXCCGGJzzisj8

State: ON. 5 runs this month. Last updated by Quinn Fitzgerald.

### Shape of the automation

| Step | Type | What it does |
|---|---|---|
| Trigger | When webhook received | Receives the WooCommerce order payload |
| Action 1 | Run a script | Filters for the ratification product and extracts order fields |
| Action 2 | Conditional group, `If matchFound is true` | Create record in Table 1, then Gmail: Send email |

Orders that are not the ratification product fall through and nothing is created.

### The script

```javascript
let inputConfig = input.config();
let payload = inputConfig.payload;
let order = payload.body ? payload.body : payload;

let RATIFICATION_PRODUCT_ID = 70664;
let matchFound = false;
let lineItems = order.line_items || [];

for (let item of lineItems) {
    if (item.product_id === RATIFICATION_PRODUCT_ID) {
        matchFound = true;
        break;
    }
}

if (!matchFound) {
    output.set('matchFound', false);
} else {
    let orderId = order.id;
    let customerEmail = order.billing ? order.billing.email : '';
    let firstName = order.billing ? order.billing.first_name : '';
    let lastName = order.billing ? order.billing.last_name : '';
    let orderDate = order.date_created;
    let amountPaid = order.total;

    output.set('matchFound', true);
    output.set('orderId', String(orderId));
    output.set('customerEmail', customerEmail);
    output.set('firstName', firstName);
    output.set('lastName', lastName);
    output.set('orderDate', orderDate);
    output.set('amountPaid', amountPaid);
}
```

### Fields written on Create record

| Airtable field | Script output |
|---|---|
| Order ID | `orderId` |
| Email Address | `customerEmail` |
| Order Date | `orderDate` |
| Amount Paid | `amountPaid` |
| First Name | `firstName` |
| Last Name | `lastName` |

### The Ratification Email

Sent by Gmail immediately after the record is created.

**To:** `customerEmail`
**Subject:** Your swim ratification application has been received, `firstName`

> Hello `firstName`,
>
> Thank you for starting your swim ratification application. We've received your order and we're excited to help you work toward making this official.
>
> To get started, please complete the Pre-Swim Planning form at the link below. This needs to be submitted at least 30 days before your swim date, so we recommend filling it out as soon as you can.
>
> [Pre-Swim Planning Form - Click here](https://www.jotform.com/210815057331043)
>
> If you have any questions along the way, just reply to this email.
>
> Warm Regards,
> Team WOWSA Support

This email is the route into Jotform. The purchase note on the product page carries the same links.

**Process rule captured here:** the Pre-Swim Planning form must be submitted at least 30 days before the swim date.

### How entry actually works

The link is a plain form URL with no parameters attached. The swimmer clicks it, fills the form, and **that submission is what enters them into the Jotform workflow.** Nothing is carried across from the order automatically.

Two consequences follow from that:

**Relay was never wired up.** The email hardcodes the Solo Pre-Swim Planning form. A separate Relay Pre-Swim Planning form exists in the account but is not connected to anything, so **every ratification to date has gone through the workflow as a solo swim**, including relays.

**There is no automatic join between the order and the submission.** The swimmer re-enters their name and email on the form. If they use a different address from the billing address on the order, nothing connects the two records. This is likely why Table 1 carries Email Address and Swimmer's Email Address as separate fields.

Note also that prefill and identity routing do apply, but further along. The entry form is open; it is the forms the workflow sends out afterwards, to the observer and back to the swimmer, that carry context with them.

### Current fault

- "Fix configuration" on the Run a script step
- The `payload` input reads **Invalid value**
- **Trigger test failed**, shown twice

Two candidate causes, neither confirmed:

1. An unfinished attempt to add Solo and Relay variation handling. The live script has no variation logic in it.
2. The trigger has never had a sample webhook payload captured, so the script step has no payload to validate against. This condition was already present before the variation work began.

The automation reports 5 runs this month, so orders may still be flowing. Whether those runs succeeded is not yet established.

### Known gaps at this step

**Add-ons are not captured.** Fields for all three add-ons exist on Table 1 but nothing populates them. The consequence is customer facing: someone who buys Record Attempt, Live Map Tracking or Blog Recap and Social Sharing does not currently receive instructions for what they bought.

An extended version of this script covering product, add-ons and order link was drafted on 6 August 2026 but never applied, because Airtable's API refuses to edit an automation containing a script step and the change has to be pasted in manually. That draft was not saved anywhere and is no longer available.

---

## The Airtable base

**Ratifications Pipeline**, `appgUjmgd0K8WWp31`. Three tables:

| Tab | Table ID | Role |
|---|---|---|
| Workflow | `tblulYhWIpP0E3eov` | The tracker. Records are moved here manually. |
| Table 1 | `tblAPf6E4uzRS0oCE` | Where orders land from the automation. |
| Pre-Swim Planning | to confirm | Part of a paused effort to rebuild the pipeline inside Airtable. Not currently in use. |

### Table 1 fields

8 records at time of capture.

| Field | Type | Populated by |
|---|---|---|
| SWIM ID# | Formula | Matches the SWIM ID held in Jotform |
| SWIM ID | Text, currently hidden | Jotform |
| Email Address | Email | Order automation |
| Order ID | Text | Order automation |
| Order Date | Date | Order automation |
| Amount Paid | Currency | Order automation |
| First Name | Text | Order automation |
| Last Name | Text | Order automation |
| Swim Date | Date | Typed in from the Pre-Swim Planning submission |
| Swimmer's First Name | Text | Typed in from the Pre-Swim Planning submission |
| Swimmer's Last Name | Text | Typed in from the Pre-Swim Planning submission |
| Swimmer's Email Address | Email | Typed in from the Pre-Swim Planning submission |
| Auto ID | Autonumber | Airtable side sequence |
| Product | Text | Nothing |
| Add-ons | Long text | Nothing |
| Record Attempt | Checkbox | Nothing |
| Live Map Tracking | Checkbox | Nothing |
| Blog Recap & Social Sharing | Checkbox | Nothing |
| Order Link | URL | Nothing |

The buyer and the swimmer are held separately. First Name and Last Name come from billing; Swimmer's First Name, Last Name and Email Address are distinct fields, so the person paying is not assumed to be the person swimming.

SWIM IDs in the table currently run in the SWIM-10xx range.

---

## Step 3. The Jotform workflow

**Workflow:** Ratifications Workflow
https://www.jotform.com/workflow/252603334494456/build
Last updated 16 August 2026.

The workflow definition is not readable through the Jotform API, so its shape is recorded from the builder view. Individual form contents are readable through the API and are accurate.

### Where the SWIM ID comes from

The SWIM ID is minted by the **Pre-Swim Planning** form, not by Airtable and not by WooCommerce. Two fields work together on that form:

| Field | Type | Behaviour |
|---|---|---|
| Auto ID | Auto increment, hidden | Sequential, 6 characters, 4 digits of padding, no prefix. Current index `1060`. |
| Swim ID | Calculation, marked DO NOT EDIT | Derived from Auto ID and shown as the SWIM ID. |

So the identifier is created at the moment the swimmer submits the planning form, which is after purchase. Everything downstream in the workflow uses the details captured on this form.

### Entry form: Pre-Swim Planning

Form `210815057331043`, titled "Pre-Swim Planning 043 on Product".

Fields in form order:

| # | Field | Type | Required | Notes |
|---|---|---|---|---|
| 1 | Pre-Swim Planning | Header | | Subtitle: In Accordance with WOWSA Rules & Regulations |
| 2 | Swim ID | Calculation | No | DO NOT EDIT |
| 3 | Auto ID | Auto increment | | Hidden |
| 4 | Name | Full name | Yes | |
| 5 | Email | Email | Yes | |
| 6 | Birthday | Date | Yes | Minimum age 13 |
| 7 | My Gender is | Radio | Yes | Female, Male, Other |
| 8 | Address | Address | No | Hidden |
| 9 | What is the date of your planned swim? | Date | Yes | |
| 10 | Body of Water | Text | Yes | ocean, sea, channel, lake, river |
| 11 | General Route Description | Text | Yes | |
| 12 | Route Type | Text | Yes | one-way, multi-way, circumnavigation, island loop, stage swim, or combination |
| 13 | Swimming Stroke | Text | Yes | |
| 14 | Route can be done shore to shore, accessible entry and exit | Radio | No | Yes / No |
| 15 | If no, justify your exception request | Text | No | |
| 16 | Route defined by permanent natural landmarks, repeatable without temporary markers | Radio | No | Yes / No |
| 17 | If no, please explain | Text | No | No buoys, anchored boats, docks, temporary flags, or GPS waypoints as turn points |
| 18 | Start Location Name | Text | Yes | Landmark plus city, state, country |
| 19 | Start Location Coordinates | Text | Yes | Decimal degrees |
| 20 | Finish Location Name | Text | Yes | |
| 21 | Finish Location Coordinates | Text | Yes | Decimal degrees |
| 22 | Straight line tangent distance from start to finish | Text | Yes | Google Maps recommended |
| 23 | Screenshot of your plotted course | File upload | Yes | Multiple files, 10 MB limit |
| 24 | Will this be an unassisted or assisted swim? | Textarea | No | **Hidden** |
| 25 | Who will serve as your primary independent observer? | Full name | Yes | |
| 26 | Primary Observer Email Address | Email | Yes | Drives observer assignment downstream |
| 27 | What are your observer's credentials? | Textarea | Yes | |
| 28 | Who is your pilot? | Textarea | Yes | **Hidden** |
| 29 | Name | Full name | No | Hidden |
| 30 | What GPS/GPS Tracking Devices will you use? | Textarea | No | |
| 31 | All equipment beyond swimsuit, one cap, goggles, earplugs | Textarea | Yes | |
| 32 | Will your swim be a record attempt? | Textarea | No | Asks for record category, maintaining organisation, current holder |
| 33 | What else do you want to share about your swim | Textarea | No | |

### Workflow map

Read top to bottom. Green nodes are approvals, blue nodes are assigned tasks, orange nodes are forms sent out, dark nodes are emails.

```
START
  On Submission: Pre-Swim Planning
    |
    +--> Google Calendar --> END
    |
    +--> Email
           |
         APPROVAL (Quinn Fitzgerald)          <-- route review
           |
           +-- Deny --> Email --> HubSpot --> END
           |
           +-- Approve --> Email --> HubSpot
                             |
                    Independent Observer Agreement    (sent to Primary Observer Email)
                             |
                        APPROVAL (Quinn Fitzgerald)
                             |
                             +-- Deny --> END
                             |
                             +-- Approve --> Email (3 recipients) --> Email
                                                    |
                                            TASK: Swim Completed
                                                    |
                                    [Drive folder A, CREATED BY HAND]   <-- before the task can be completed
                                                    |
                                    [Workflow] Post-Swim Observer Report   (to Primary Observer Email)
                                                    |
                                       Post-Swim Swimmer Narrative          (to swimmer Email)
                                                    |
                                    Google Drive: Send Files            --> Drive folder B, automated
                                                    |
                                    TASK: Complete Ratification Page
                                                    |
                                                 HubSpot
                                                    |
                                        APPROVAL (Quinn Fitzgerald)         <-- final determination
                                                    |
                        +-- Ratified ---------------+-- Not Ratified --+
                        |                                              |
                    HubSpot                                        HubSpot
                        |                                              |
        TASK: Attach Certificate and Socials                        Email
                        |                                              |
                     Email                                            END
                        |
                    HubSpot
                        |
                       END
```

Two **Email** nodes addressed to **7 recipients** sit detached from the main path in the builder, greyed out. They are left over from the previous system, when committee dispatch ran inside the workflow. That changed when the review moved onto the ratification page itself and the reviewer pool was widened. Nothing depends on them.

---

### Node detail

Built up as the walkthrough covers each node.

#### Workflow start

| Setting | Value |
|---|---|
| Selected form | Pre-Swim Planning 043 on Product |
| Starts | For every submission |

No conditions are set, so **every** Pre-Swim Planning submission enters the workflow. This is also why relay cases run down the solo path: there is one trigger form and it is the solo one.

Two things fire immediately and in parallel off the trigger:

**Branch A: Google Calendar.** Every submission is written to the calendar on `contact@openwaterswimming.com`, then that branch ends. It does not rejoin the main path.

**Branch B: the Next Steps email** to the swimmer, which then continues into the first approval.

#### Google Calendar event

Creates an event on the WOWSA calendar so the planned swim date is visible internally.

| Setting | Value |
|---|---|
| Calendar | `contact@openwaterswimming.com` |
| Event title | `Swim ID` `Name` |
| Required attendees | None |
| Start Time | `What is the date of your planned swim?` |
| Duration | All Day |
| Event description | `General Route Description` |
| Create only if time slot available | Off |

The event is a marker for the planned date as declared at submission. It is not updated later, so if the swim date moves the calendar entry does not follow it.

**This branch is not decorative.** It dead-ends in the builder, but the calendar is what Rose checks to know a swim has happened, which is what unblocks the entire post-swim phase. See the Swim Completed task below. So the calendar is load bearing, and it holds the date as declared months earlier rather than the date the swim actually took place.

#### The Next Steps email

**To:** `Email`
**Subject:** Next Steps | `Swim ID` - `Name`
**Attachments:** 1, the observer log

> Hello `Name`,
>
> Thanks for your submission, and good luck preparing for your swim on `What is the date of your planned swim?`. The Ratification Committee will review your eligibility, planned route, and independent observer.
>
> **What happens now**
> **Pre-review:** We'll check the details you provided (route, category, observer). If anything needs clarification, we'll reach out.
> **Observer onboarding:** We'll email `Who will serve as your primary independent observer?` the Observer Roles & Responsibilities and acknowledgment form.
>
> **You can prep in parallel:**
> **Rules & category:** Ensure your team knows the declared rules (Unassisted/Assisted) and follows them consistently. [WOWSA Marathon Swimming Rules]
>
> **Safety plan:** Confirm pilot/boat details, comms (VHF/cell), extraction points, and any permits/harbor notices.
>
> **GPS evidence:** Arrange a primary recorder on the escort boat (1-5s logging, raw GPX/NMEA export) plus a backup. [GPS Devices for Independent Marathon Swims]
>
> **Documentation kit:** Print the observer log (attached), stroke-count/feed sheets; plan start/finish waypoints + local times; align device clocks. [Documentation Reference for Ratification]
>
> **Photo & video evidence:**
>
> **Before the swim:** Enable Location Services for the Camera app (iPhone: Settings > Privacy & Security > Location Services > Camera > While Using the App, with "Precise Location" ON. Android: Settings > Location > App permissions > Camera > Allow only while using).
>
> Set phone clock to automatic date & time and make sure it matches the GPS/logger.
>
> **During the swim:** Use the phone's native Camera app (not WhatsApp/Instagram). Capture start, feeds, crew hand-offs, finish, and other key moments. Confirm the phone's GPS has a satellite lock before shooting.
>
> **After the swim:** Do not crop, filter, or re-save files (this strips metadata). Upload original files in full resolution (JPEG, MOV, MP4, HEIC). Use AirDrop, iCloud/Google Drive, USB, or Dropbox to share raw files only.
>
> **How ratification is evaluated**
> After the swim, the Ratification Committee will review documentation along three key criteria:
> - Observer logs & observer report
> - Photo and video evidence with accurate time-stamps and geotags
> - GPS tracks (raw exports preferred)
>
> **After your swim**
> You'll receive links to submit the Observer Report (logs, GPX, media) and the Swimmer Report (narrative + notes). Keep all original files unedited.
> If any of your details change (date/window, route, observer, pilot), just reply to this email so we can update your file.
> Thanks again, and we're wishing you a smooth build-up to `General Route Description`.
> Kind regards,
> Team WOWSA Support

Three knowledgebase links are referenced: WOWSA Marathon Swimming Rules, GPS Devices for Independent Marathon Swims, Documentation Reference for Ratification.

**Process rules captured here.** This email is currently the only place several standards are stated to the swimmer:

- The three evaluation criteria: observer logs and report, photo and video evidence with accurate timestamps and geotags, GPS tracks with raw exports preferred
- GPS logging interval of 1 to 5 seconds, raw GPX or NMEA export, plus a backup recorder
- Photo and video must be unedited originals, since cropping or re-saving strips metadata
- Device clocks must be aligned to the GPS logger
- Changes to date, route, observer or pilot are handled by replying to this email

#### Approval 1: route review

The first manual stop. The Ratification Committee reviews eligibility, planned route and independent observer.

| Setting | Value |
|---|---|
| Outcomes | Approve, Deny |
| Approvers | `contact@openwaterswimming.com` |
| Approval Request Email | On |
| Require Login for Approver | On |

The approver is the shared WOWSA admin mailbox, not a named person. The builder displays "Quinn Fitzgerald" as the node owner, but the address that receives and acts on the request is the admin address.

**Approval Request Email**

**To:** `contact@openwaterswimming.com`
**Subject:** Action Required | Review `Swim ID`
**Attachments:** 1

> Hello there,
> Please review and approve this route. Kindly include reason. Thanks!

Below the message the email renders Approve, Deny and Go to Inbox buttons, followed by a full read-out of the submitted form: Swim ID, Auto ID, Name, Email, Birthday, Gender, Address, planned swim date, Body of Water, General Route Description, Route Type, Swimming Stroke, the shore to shore confirmation and its exception field, and the permanent landmarks confirmation.

**What actually happens at this stop**

The configured behaviour and the real process are different, and the difference matters.

1. The notification arrives at the admin mailbox.
2. Rose takes the start and finish coordinates off the submission manually.
3. Rose measures the route in **Google Maps**, using the measure distance tool.
4. Rose screenshots the measured route, showing the plotted path and the total distance.
5. Rose emails that screenshot to Quinn, subject "Route for Approval `SWIM ID` `swimmer name`", with the original Jotform notification forwarded underneath.
6. Quinn replies with his decision.
7. Rose records the decision by clicking Approve, either in the approval email or in Jotform.

So the Jotform approval step is **not where the decision is made**. It is where a decision already reached over email gets recorded. The instruction "Kindly include reason" is addressed to a mailbox rather than to the person actually deciding, and Quinn's reasoning lives in an email thread rather than against the record.

The route measurement is also fully manual and its output is an image. The measured distance exists as pixels in a screenshot and in an email thread, not as a value on any record.

#### Approve path: Route Approval email

Sent by Jotform if the route review is approved. The workflow continues.

**To:** `Email`
**Subject:** `Swim ID` | Route Approval - `Name` `What is the date of your planned swim?`
**Attachments:** none

> Hello `Name`,
> Thank you for submitting your pre-swim planning form for your upcoming swim `General Route Description` on `What is the date of your planned swim?`.
> The Ratification Committee has reviewed your materials, and we're pleased to confirm that your proposed route is approved under WOWSA's ratification standards. This approval means your swim plan aligns with the established rules and criteria for recognition.
> **As you move forward, please keep in mind:**
> - Any adjustments to the route or date must be communicated to us in advance.
> - Your assigned observer will be provided with the protocols and templates needed to document the swim.
> - After the swim, you and your observer will each complete the post-swim reports to support ratification.
> We're excited to follow your progress and wish you the very best with final preparations.
> Kind regards,
> Team WOWSA Support

**Worth noting.** WOWSA's own support guidance is explicit that route approval is not ratification, that swimmers conflate the two constantly, and that the distinction should be stated plainly whenever route approval is the news being delivered. This email says the plan "aligns with the established rules and criteria for recognition" but never states that approval is not ratification. It is the single most likely place for that misunderstanding to start.

#### Deny path: Route Review Outcome email

Sent by Jotform if the route review is denied. **This email ends the workflow.**

**To:** `Email`
**Subject:** `Swim ID` | Route Review Outcome - `Name` `What is the date of your planned swim?`
**Attachments:** none

> Hello `Name`,
> Thank you for submitting your pre-swim planning form for your upcoming swim `General Route Description` on `What is the date of your planned swim?`. We appreciate the time and care you've put into preparing your plan.
> After review, the Ratification Committee is unable to approve the proposed route at this time.
> The main reason(s) are:
> `{7_comment}`
> This does not mean the swim cannot move forward, only that adjustments are needed for it to meet WOWSA's ratification standards. Once these issues are addressed, you're welcome to resubmit your revised plan for review.
> We understand how much effort goes into preparing a marathon swim, and our goal is to ensure your achievement can be recognized with the transparency and consistency it deserves.
> Please don't hesitate to reach out with any questions or to discuss possible adjustments.
> We're here to support you in refining your plan.
> Kind regards,
> Team WOWSA Support

`{7_comment}` pulls the reason entered by the approver at the approval step. This is what "Kindly include reason" in the approval request email is feeding. Given that the real reasoning arrives from Quinn by email, whoever clicks Deny has to carry that reasoning across into the comment box manually.

**What happens after a denial**

The workflow ends. The email invites the swimmer to resubmit a revised plan, but there is no route back into the same workflow instance. A resubmission means a new Pre-Swim Planning submission, which mints a **new SWIM ID** for what is the same swim.

So a swim that is denied once and corrected ends up holding two SWIM IDs, and nothing links them. This is the clearest example so far of a real problem in the current system, and it is the exact gap Quinn's document is reaching for when it separates a Submission from a Swim.

#### Independent Observer Agreement

Assigned automatically once the route is approved.

| Setting | Value |
|---|---|
| Form | Independent Observer Agreement Form, `251242794026152` |
| Assignee | `Primary Observer Email Address`, taken from the pre-swim planning form |
| Prefill | **On.** Uses submission info from the pre-swim form to pre-populate the assigned form |
| User Notification Email | Custom, see below |

This is where the identity and prefill layer starts doing its work. The observer named on the swimmer's planning form is assigned directly, their copy of the agreement arrives already carrying the swim's details, and their submission comes back attached to the right case.

**Form Notification Email**

**Subject:** Action Required | `Swim ID` Complete Observer Form for `Name` Swim

> Hello `Who will serve as your primary independent observer?`,
> I hope this message finds you well. I'm reaching out regarding `Name`'s upcoming swim `General Route Description` on `What is the date of your planned swim?`, for which you are serving as the independent observer
> `Name` has submitted her pre-swim planning form `Swim ID`, and we'd like to connect with you to review [WOWSA's independent observer ratification protocols].
> Please complete the Independent Observer Agreement form at your earliest convenience:
>
> Click the button below to fill out this form.
> **Swim ID:** `Swim ID`
> [View Form]
>
> The agreement outlines your responsibilities as the official observer and the protocols you'll follow during the swim to ensure proper documentation for WOWSA's ratification committee.
> As the independent observer, you serve as the eyes and ears of the swim, ensuring transparency, accuracy, and integrity from start to finish.
> We truly appreciate your commitment to upholding the standards of our sport through careful and independent observation.
> Warm Regards,
> Team WOWSA Support

**Copy defect.** The line "`Name` has submitted **her** pre-swim planning form" hardcodes a pronoun. It goes to the observer of every swim regardless of who the swimmer is.

**The observer types the Swim ID manually.** On the agreement form, Swim ID is a required text box with an input mask of `@@@@-####` and the instruction "Enter your Swim ID from the Pre-Swim form (e.g. SWIM-0005)". Prefill is switched on at the workflow node, so this may be populated automatically for an observer arriving through the assigned link, but the field is built for a person to type into. Anyone arriving any other way has to know the ID and enter it correctly.

#### What the Independent Observer Agreement asks

Form `251242794026152`. Three pages, three separate signatures.

**Page 1: Roles and Responsibilities**

| Field | Type | Required |
|---|---|---|
| Swim ID | Text, masked `@@@@-####` | Yes |
| Name | Full name | Yes |
| E-mail | Email | Yes |
| Besides Independent Observer, will you have any other responsibilities? | Textarea | Yes |
| Swims previously observed (swimmer name, date, ratifying body) | Textarea | Yes |
| Embedded PDF: Observer, Pilot and Crew Roles and Responsibilities | Widget | No |
| Observer Acknowledgment and Commitment, six undertakings | Text | |
| Signature | Signature | Yes |

The six undertakings: remain impartial and uphold the integrity of the swim; accurately document all required observations using the Observer Log Template; prioritise swimmer safety while enforcing compliance; disclose any potential conflicts before the swim; submit a complete and truthful report including all logs, data and narrative; and accept that the role is essential to the credibility and fairness of the swim.

**Page 2: Rules of Engagement**

| Field | Type | Required |
|---|---|---|
| Which category of marathon swim is being attempted | Assisted / Unassisted | Yes |
| If assisted, equipment being used | Textarea | No |
| What type of marathon swim is being attempted | One-Way Solo / Solo Multi-Leg / Solo Stage / Relay | Yes |
| Link to the marathon swimming rules | Text | |
| Any reason you will be unable to enforce any aspect of the rules? | Yes / No | Yes |
| If yes, which rules and what reasons | Textarea | No |
| Rules and Regulations Observer Acknowledgment Agreement, five undertakings | Text | |
| Signature | Signature | Yes |

The five undertakings include the observer's authority to pause or terminate the swim if conditions present undue risk or rules are violated.

**Page 3: Conflict of Interest Disclosure**

| Field | Type | Required |
|---|---|---|
| Any personal, professional or financial relationship with the swimmer, crew, organising party or outside entity that could compromise objectivity? | Yes / No | Yes |
| If yes, describe the conflict | Textarea | No |
| Receiving compensation or other benefits for involvement? | Yes / No | Yes |
| If yes, total monetary compensation and any other forms | Textarea | No |
| Any other circumstance that could reasonably be perceived to affect impartiality? | Yes / No | Yes |
| If yes, describe | Textarea | No |
| Disclosure Acknowledgment and Agreement | Text | |
| Signature | Signature | Yes |

The closing agreement states that failure to disclose a relevant conflict may result in disqualification of the swim, withdrawal of ratification or certification, or loss of standing as an impartial observer.

**Defects found on this form**

- The link labelled "WOWSA Marathon Swimming Rules & Regulations" points at `https://form.jotform.com/251242794026152`, which is this form itself. Observers are told to read the rules and sent back to the page they are on.
- The category and type questions are checkboxes rather than radio buttons, so an observer can tick both Assisted and Unassisted, or several swim types at once.
- Typo in the assisted equipment sublabel: "pourous".
- Relay appears as a swim type here, even though there is no relay path into the workflow.
- The swimmer is never asked to declare assisted or unassisted, because that question is hidden on the pre-swim planning form. The observer declares it instead, which means the category is set by the observer rather than by the applicant.

#### Approval 2: Independent Observer Form review

| Setting | Value |
|---|---|
| Outcomes | Approve, Deny |
| Approvers | `contact@openwaterswimming.com` |
| Approval Request Email | On |
| Require Login for Approver | **Off** |

Note the difference from Approval 1, where Require Login is on.

**Approval Request Email**

**To:** `contact@openwaterswimming.com`
**Subject:** `Swim ID` For Approval | Independent Observer Form

> Hello there,
> Please review Independent Observer Form submission below. Kindly add comments. Thanks!

Followed by Approve, Deny and Go to Inbox buttons and a read-out of the submission.

**What actually happens at this stop**

Unlike the route review, there is no external consultation. Rose reviews the submission and decides directly, either from the email or in Jotform. Nothing is sent to Quinn.

So the two approvals look identical in the builder but work completely differently in practice. The first records a decision made by someone else. The second is a decision.

**Outcomes**

- **Deny** ends the workflow. No email is sent on this branch, unlike the route review deny. In practice this has never been used.
- **Approve** continues the workflow.

Since a denial here has never happened, this branch is untested in real use.

#### Observer Confirmed email

Sent on approval. This is the three recipient node in the map.

**To:** `quinn@openwaterswimming.com`, `Primary Observer Email Address`, `Email`
**Subject:** `Swim ID` | Observer Confirmed
**Attachments:** 1, the observer log

So Quinn, the observer and the swimmer all receive the same message. This is the first point in the workflow where a personal address rather than the shared admin mailbox appears.

**Why Quinn is a direct recipient.** Jotform has no CC or BCC field on email nodes. Copying someone means adding them to the recipient list, so Quinn sits alongside the observer and the swimmer as a full To recipient. All three see each other's addresses, and there is no way to copy someone quietly.

> Hello `Name`,
> Thank you for serving as the independent observer for `Name`'s swim `General Route Description` on `What is the date of your planned swim?`. Your role is central to WOWSA's ratification process, and we want to make sure you have everything you need.
>
> **Key Observer Resources**
> **Observer Log Template:** [WOWSA Observer Log Form] (Also attached)
> Please print several pages depending on the length of the swim. We recommend leaving blank lines between entries for legibility.
>
> **GPS Recommendations:** [Approved GPS Devices for WOWSA Ratified Swims]
>
> **Observer Role Guide:** [Observer, Pilot & Crew Roles Quick Guide]
>
> **Example Ratification Reports:**
> [Unassisted, Catherine Breed, Farallon to Golden Gate]
> [Assisted, Espiritu Santo Circumnavigation]
>
> **Policies & Guide:** [WOWSA Ratification Committee Policies & Guide]
>
> **Documentation You'll Oversee**
> The Ratification Committee evaluates swims on three key criteria:
> - Observer logs & observer report
> - Photo and video evidence with accurate time-stamps and geotags
> - GPS tracks (raw exports preferred)
>
> **Preparing in Parallel**
> - **Rules & category:** Confirm the declared swim category (Unassisted/Assisted) and ensure it's followed throughout.
> - **Safety plan:** Verify pilot/boat details, comms (VHF/cell), extraction points, and permits/harbor notices.
> - **GPS evidence:** Ensure a primary GPS recorder is running on the escort boat (1-5s logging, raw GPX/NMEA export), with a backup device if possible.
> - **Documentation kit:** Bring printed observer logs, stroke-count/feed sheets; plan start/finish waypoints + local times; align all device clocks.
>
> **Photo & video evidence:**
> **Before the swim:** Confirm that location services are enabled on the phone's Camera app (iPhone: Settings > Privacy & Security > Location Services > Camera > While Using the App with "Precise Location" ON. Android: Settings > Location > App permissions > Camera > Allow only while using). Check the phone clock is set to automatic date & time and matches the GPS/logger.
> **During the swim:** Use the phone's Camera app (not WhatsApp/Instagram) to capture key moments, start, feeds, crew hand-offs, finish. Make sure the GPS has a satellite lock before shooting.
> **After the swim:** Do not crop, filter, or re-save files. Upload or transfer the original files (JPEG, MOV, MP4, HEIC) in full resolution. Use AirDrop, iCloud/Google Drive, USB, or Dropbox to submit raw files only.
>
> **Observer Log Form**
> **Offline option:**
> You can download a fillable or printable form below:
> **Fillable PDF:** `https://drive.google.com/file/d/16giaPcBLVsjcByetPcxsJ-LevxLSpc8F/view?usp=sharing`
> *Once you've filled all the rows, simply save it as a new file and continue logging on the next form. You will need a PDF reader like Adobe Acrobat to use this.*
> **Printable PDF:** `https://drive.google.com/file/d/19s7pUCNOAuFFznvFnz7_EzIu3LZN9JhF/view?usp=sharing`
> *Print as many copies as you may need, and please write legibly.*
> **Online option:**
> You may also use the online Observer Log form: `https://form.jotform.com/253011354070038`
> To prevent data loss, this form is designed to be submitted per timestamp, for example, submit one entry at the start of the swim, then another each hour until the finish. Please also upload photos or videos as you log each entry.
>
> **After the Swim**
> You'll receive a link to complete the Post-Swim Observer Report, where you'll submit your logs, GPS files, and media. Please complete this as soon as possible while details are fresh.
> Your careful observation and documentation are what allow WOWSA to recognize swims with accuracy, transparency, and integrity. Thank you for playing such an important role in upholding these standards.
> Warm Regards,
> Team WOWSA Support

**The observer log has three routes.** Fillable PDF on Google Drive, printable PDF on Google Drive, and an online Jotform (`253011354070038`). The observer picks one. Two of the three produce a document the observer holds until the post-swim report; the third produces submissions in Jotform.

The online option is submitted **per timestamp**, one entry at the start and one every hour until the finish. A ten hour swim therefore produces ten or more separate submissions on a standalone form that is not part of this workflow, each potentially carrying photos or video.

**Defect to check.** The greeting and the possessive both use the same `Name` field: "Hello `Name`, Thank you for serving as the independent observer for `Name`'s swim". Those refer to two different people, so one of them is wrong. Either the observer is being addressed by the swimmer's name, or the swimmer's swim is being labelled with the observer's name.

#### Good luck email to the swimmer

Sent immediately after the Observer Confirmed email.

**To:** `Email`
**Subject:** `Swim ID` | Sending support for your `General Route Description` swim
**Attachments:** none

> Hello `Name`,
> On behalf of WOWSA, I wanted to send a quick note of encouragement as you head into your marathon swim. Taking on a challenge like this is never just about the miles, it's about the preparation, the patience, and the trust you've built with your crew.
> We'll be cheering you on from afar and can't wait to follow your progress. Most of all, we hope you enjoy the experience, every stretch of water, every exchange with your team, and the sense of achievement that comes with putting yourself fully into the swim.
> Wishing you steady conditions, strong strokes, and a safe and successful crossing.
> Best of luck,
> Team WOWSA Support

**Known weak point, raised by Rose.** This email is written to land shortly before the swim. It cannot. Jotform has no way to send an email a set number of days before a date field, so the message fires the moment the observer agreement is approved. That can be weeks or months ahead of the actual swim date.

A swimmer can therefore receive "as you head into your marathon swim" and "wishing you steady conditions" in, say, February for a swim in August.

#### Task: Swim Completed

| Setting | Value |
|---|---|
| Title | Swim Completed |
| Description | "Mark swim as completed to proceed to the next stp" |
| Outcome | Complete, single outcome, no alternative |
| Assignee | `contact@openwaterswimming.com` |
| Notification Emails | On |
| Require Login for Assignee | Off |

**Swim Completed Email**

**To:** `contact@openwaterswimming.com`
**Subject:** `Swim ID` | Mark Swim as Completed

> `19_taskName` is assigned to you. Please complete this task.

Followed by View Task and Go to Inbox buttons and a read-out of the submission. The task can be completed from the email or in Jotform.

**What actually happens at this stop, and why it matters**

This is the point where a human becomes the clock.

Nothing tells the workflow that a swim has happened. Rose checks the Google Calendar, sees that a swim date has passed, and marks the task complete manually.

**Until she does, the swimmer and the observer cannot submit their post-swim documentation.** The post-swim observer report and swimmer narrative are both assigned downstream of this task, so the entire post-swim phase is gated on someone noticing a date has gone by.

Three things compound here:

1. The gate is invisible to the swimmer. They have finished their swim, they were told they would receive links, and nothing arrives until Rose looks at a calendar.
2. The calendar holds the date as declared at planning, which may be months old and may not be the date the swim actually happened.
3. Nothing prompts the check. There is no queue of swims whose dates have passed, no reminder, and no list to work from.

This is the clearest single consequence of the scheduling limitation below. Jotform cannot act on a date, so a person has to, and that person is doing it for every case, from memory, against a calendar.

Typo in the task description: "stp".

**Marking the task complete is not the whole job**

Before the task can be completed, Rose has to build the swim's Google Drive folder manually.

**Parent folder:** `https://drive.google.com/drive/folders/[DRIVE_FOLDER_A]`

**Naming format:** `SWIM-XXXX Firstname Lastname`. One folder per swim.

**Contents:** a fixed set of empty subfolders, copied from a template set kept locally as "GDrive template folders" in the WOWSA folder on iCloud Drive.

The folder link is then what fills the `Google Drive URL for this Swim` field in the post-swim observer email, which is how the observer is offered a direct upload location instead of attaching everything to a form.

So the sequence at this stop is:

1. Notice the swim date has passed, from the calendar
2. Create the swim folder in Drive, named to format
3. Copy the template subfolders into it
4. Mark the Swim Completed task complete

Only then is the post-swim observer report assigned.

The folder naming is not fully consistent in practice. Existing folders include both `SWIM-0050 Firstname Lastname` and `Swim-1028` styles.

#### Post-Swim Observer Report

Assigned as soon as the Swim Completed task is marked complete.

| Setting | Value |
|---|---|
| Form | [Workflow] Post-Swim Observer Report, `252602743229455` |
| Assignee | `Primary Observer Email Address` |
| Prefill | **On**, sourced from the **Independent Observer Agreement** form |
| User Notification Email | Custom, see below |

Note the prefill source. Earlier nodes prefill from the pre-swim planning form; this one prefills from the observer's own agreement submission. That is also why the subject line uses `Swim ID_1` rather than `Swim ID`, since two forms in the chain now each carry a Swim ID field.

**Form Notification Email**

**Subject:** `Swim ID_1` | WOWSA Post-Swim Ratification Next Steps

> Hello `Name`,
> Thank you again for serving as the independent observer for `Name`'s swim `General Route Description` on `What is the date of your planned swim?`. Your role is essential to ensuring transparency and integrity in the ratification process.
> At this stage, we kindly ask you to complete the Post-Swim Observer Report so the WOWSA Ratification Committee has a full record of the swim. Please submit the form here:
>
> **SWIM ID:** `Swim ID`
> Click the button below to fill out this form.
> [View Form]
>
> The report should be completed as soon as possible while details are still fresh. It will capture your direct observations, confirm adherence to swim rules, and provide supporting documentation for the committee's review.
> **Optional file drop (if easier than attaching in the forms):**
> You can upload logs, videos and photos here if easier: `Google Drive URL for this Swim`
>
> We greatly appreciate the time and effort you contribute to safeguarding the standards of our sport.
> Warm Regards,
> Team WOWSA Support

The same `Name` field is used twice for two different people, as in the Observer Confirmed email.

A `Google Drive URL for this Swim` field is merged in here. It points at the folder Rose created manually during the Swim Completed step, which is why that folder has to exist before this form is assigned.

**Nothing verifies that anything was uploaded**

The Drive folder is offered as an optional file drop, and the form can be submitted whether or not files were put there. Jotform has no visibility into the Drive folder, so:

- An observer can submit the report having uploaded nothing
- An observer can upload some files and not others
- The workflow advances either way, showing the step as done

Every completeness check is therefore manual. Rose opens the folder, works out what is missing, and chases the observer for it. Missing GPS tracks, missing observer logs, missing photos, all of it is caught by a person looking, or not caught at all.

This is a second silent failure of the same kind as the broken link problem. The workflow reports success; the evidence packet may be empty.

#### What the Post-Swim Observer Report asks

Form `252602743229455`, titled "Post-Swim Ratification". Subtitle: "To be filled out by the designated Independent Observer in accordance with WOWSA Rules & Regulations".

Sections: Swimmer Information, Swim Location Time and Distance, Route and Swim Details, Support Crew and Vessels, Upload Observer Log, Upload GPS Data, Upload Photos and Videos, Observer Reflections and Narrative.

**Only one field on this entire form is required, and it is the Swim ID.**

Everything else is optional. That includes:

| Optional field | Section |
|---|---|
| Observer Name, Email, Phone, Previous Experience | Swimmer Information |
| Swimmer Name, Date of Birth, Gender | Swimmer Information |
| Date and Time of Swim Start, with timezone | Location, Time, Distance |
| Date and Time of Swim Finish, with timezone | Location, Time, Distance |
| Swim Start Location and GPS Coordinates | Location, Time, Distance |
| Swim Finish Location and GPS Coordinates | Location, Time, Distance |
| Exact GPS Swim Distance Covered in Kilometers | Location, Time, Distance |
| Shortest Straight-Line Tangent Distance for Record Setting Purposes | Location, Time, Distance |
| Exact Elapsed Swim Time | Location, Time, Distance |
| Stage swim elapsed time per day and cumulative | Location, Time, Distance |
| Body of Water Name and Type, Route Description, Route Type, Stroke | Route and Swim Details |
| Equipment Used | Route and Swim Details |
| Escort Pilot name, email, experience, boat name | Support Crew |
| Support Vessel name, type, home port | Support Crew |
| **Observer Log file upload** | Upload Observer Log |
| **GPS data file upload** | Upload GPS Data |
| **Start and finish videos** | Upload Photos and Videos |
| **Hourly photos** | Upload Photos and Videos |
| Were any rules difficult to interpret or enforce | Reflections |
| Observer Narrative | Reflections |
| **The observer's attestation signature** | Reflections |

The attestation reads:

> I personally witnessed the attempt from start to finish and that, to the best of my knowledge and belief, the swim was conducted in accordance with WOWSA Marathon Swimming Rules (or as otherwise disclosed in this report). The times, locations, and supporting materials submitted are complete and accurate, and any exceptions, deviations, or uncertainties are noted in this documentation.

It is not required to submit the form.

This is the mechanical explanation for the chasing. It is not only that Jotform cannot see the Drive folder. **The report itself can be submitted containing nothing but a Swim ID**, and the workflow will advance.

**Also on this form**

- The Swim ID is again a hand-typed masked text box, `@@@@-####`, hint `SWIM-0005`.
- The straight-line tangent distance "for Record Setting Purposes" lives here, entered by the observer, optional. This is the ratified distance, and it is currently the observer's arithmetic rather than a measurement.
- File uploads are capped at roughly 10 MB each. The GPS section instructs: "YOU MUST ZIP all GPX files before uploading. If you have an issue please send directly to contact@openwaterswimming.com", so oversized evidence arrives by email and lands outside the system entirely.
- Photos are requested "at least every hour", against that same 10 MB cap.
- Several fields are hidden: Email, Primary Observer name, Observer Email, Previous Observer Experience, and Headshot for Certificate.

#### Post-Swim Swimmer Narrative

Assigned only after the observer's report comes back.

| Setting | Value |
|---|---|
| Form | Post-Swim Swimmer Narrative, `252386318465161` |
| Assignee | `Email`, the swimmer |
| Prefill | **Off** |
| User Notification Email | Custom, see below |

**Form Notification Email**

**Subject:** `Swim ID` | Swimmer Narrative

> Hello `Name`,
> Congratulations again on your incredible swim `General Route Description` on `What is the date of your planned swim?`. As part of WOWSA's ratification process, we'd love to include your own perspective in the official record.
> Please take a few minutes to complete the Swim Report / Swimmer Narrative form here:
>
> **SWIM ID:** `Swim ID`
> Click the button below to fill out this form.
> [View Form]
>
> Your narrative doesn't need to be long, just a concise reflection covering:
>
> - Pre-swim preparation
> - Key moments on course
> - Feeding approach
> - Conditions and challenges
> - Lessons learned
> - Acknowledgements (crew, supporters, etc.)
>
> Optional: If you'd like, you can also upload a few representative photos or short clips that help tell the story.
> Google Drive link: `Google Drive URL for this Swim`
> This personal perspective adds depth to the ratification report and helps future swimmers and crews learn from your experience.
> Thank you again for your contribution to our sport, we're honored to record your achievement.
> Warm Regards,
> Team WOWSA Support

**What the form asks**

Titled "Submit Your Swim Report". Subtitle: "Every swim tells a story. Your report becomes part of the living history of open water swimming."

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
| WOWSA may publish selected quotes and media | **Yes** |
| Captcha | Yes |

**Two things stand out.**

**The Swim ID is not required here, and has no mask.** Prefill is also off on this node. So the one field that ties a swimmer's narrative back to their case is optional, free text, and unassisted. A swimmer can submit a complete narrative with no Swim ID at all, and there is nothing to match it against except their name and email.

**The publication consent has only one option.** The field is a required radio button whose single choice is "Consent to publish". A swimmer cannot submit the form without agreeing, and there is no way to decline. Given that this consent covers publishing their quotes and media, it is worth deciding whether that is intended.

#### Google Drive: Send Files

Runs after the swimmer narrative comes back. Collects everything uploaded through the forms into Drive.

| Setting | Value |
|---|---|
| Account | `contact@openwaterswimming.com` |
| Parent folder | `https://drive.google.com/drive/folders/[DRIVE_FOLDER_B]` |
| Create a subfolder per submission | On |
| Subfolder name | `Swim ID`_`Name`_`What is the date of your planned swim?` |
| Submissions as PDF | On, as a new document |

**File fields collected**

| Field | Source form |
|---|---|
| Screenshot of your plotted course | Pre-Swim Planning |
| File Upload (observer log) | Post-Swim Observer Report |
| File Upload (GPS data) | Post-Swim Observer Report |
| 1 video of the start and 1 video of the finish | Post-Swim Observer Report |
| Photos during the swim, at least every hour | Post-Swim Observer Report |
| Headshot for Certificate | Post-Swim Observer Report, hidden field |
| Headshot for the certificate | Post-Swim Swimmer Narrative |
| Additional images from your swim | Post-Swim Swimmer Narrative |

---

### Two Drive folders per swim, and they do not know about each other

This is the point where the evidence for a single swim splits in two.

| | Folder A | Folder B |
|---|---|---|
| Created by | Rose, manually | Jotform, automatically |
| Parent | `[DRIVE_FOLDER_A]` | `[DRIVE_FOLDER_B]` |
| Naming | `SWIM-XXXX Firstname Lastname` | `SWIMID_Name_SwimDate` |
| Created when | Before Swim Completed is marked done | After the swimmer narrative arrives |
| Contains | Whatever the observer and swimmer uploaded directly, plus the empty template subfolders | Whatever was attached to the forms, plus submission PDFs |

Different parents. Different naming conventions, one spaced and one underscored, one carrying the swim date. Created at different points in the process. Nothing links them.

So for any given swim the evidence may be in either folder, or split across both, depending on whether people attached files to forms or dropped them into the link they were sent. Assembling a complete picture means opening both and merging them mentally.

Combined with the earlier finding that the observer report requires nothing but a Swim ID, the position before committee review is: evidence in two unlinked locations, with no check that any of it exists.

---

## Step 4. Building the ratification page

From here the process is almost entirely manual.

### Photo review and the timestamp problem

Photographs are reviewed manually. The **metadata timestamp is the source of truth** for verifying when a photo was taken, so each image has to be checked for it.

This is a persistent failure point. Photos routinely arrive with metadata stripped, because they were passed between devices, sent through messaging apps, or re-saved along the way. The Next Steps and Observer Confirmed emails both warn against this in detail, and it happens anyway.

Rose has tried building forms for real time photo submission during the swim, to capture images before they can be degraded. That has not produced consistent results either.

So the evidence standard depends on metadata that the collection method routinely destroys, and every reminder written into the process has failed to change that.

### The ratification page

Built on a **separate WordPress site** from the main one: `https://openwaterswimming.com/ratifications/`

Admin: `https://openwaterswimming.com/ratifications/wp-admin/`

Example published page: `https://openwaterswimming.com/ratifications/swim-1039-nicholas-bufton/`

URL pattern: `/ratifications/swim-XXXX-firstname-lastname/`

There is a template, but in practice a page is made by **copying an existing post** and replacing its contents.

### Page structure

| Section | Contents | Where it comes from |
|---|---|---|
| Header | Site logo, navigation, social share buttons for Facebook, X, LinkedIn, WhatsApp | Fixed template |
| Hero | Image and standing tagline about the full swim story | Fixed template |
| Title block | Swimmer name, route name | Typed |
| Overview | Narrative paragraph describing the swim, plus a cover photo | Written manually from the swimmer and observer reports |
| Swim Details | Route, Distance, Exact Elapsed Swim Time, Start local time, Start GPS coordinates, Finish local time, Finish GPS coordinates, Category, Escort Boat, Equipment Used | Transcribed from the observer report |
| Swim Route and Observer Logs | ZeroSixZero map embedded as an iframe, plus the observer log | Map service, plus manual transcription |
| Swim Highlights | Video embed and an image gallery | Vimeo, plus uploaded photos |

### Preparing the media

Nothing arrives ready to publish. Each media type goes through its own manual pipeline first.

**Photographs**

1. Open and check every image individually
2. Read the capture timestamp from the metadata
3. Rename each file to its timestamp
4. Upload to the post's gallery block
5. Set the timestamp as the caption

The renaming step is what makes the gallery captions possible, so the metadata check and the file naming are the same job.

**Videos**

1. Assemble the clips in **Camtasia**
2. Add timestamps
3. Apply a Camtasia title screen template, kept so every swim's video opens consistently
4. Upload to **Vimeo**
5. Configure the Vimeo player to hide everything except the play button and volume, and set the video to hidden but embeddable by link
6. Embed in the WordPress post

**GPS track and map**

1. Upload the GPX file to **ZeroSixZero** under Routes, via Routes then Create Routes
2. Create a map
3. Add the route to the map's "Live Tracks and Map Layers"
4. Embed the resulting map in the post

Example map: `https://zerosixzero.com/worldopenwater/swim-1055-hector-pardoe-08-10-2026`

Note that this map URL carries the swim date while the earlier example, `swim-1039-nicholas-bufton`, does not. The naming is not consistent across maps.

**Observer log**

Transcribed manually into the post, working in the WordPress code editor, in the format:

```
06:12  Feed 1 - we have sunlight 150ml mix
```

**Crew photographs**

Downloaded manually if anything is available. Usually nothing is sent.

### Sections of the finished post

| Section | How it is produced |
|---|---|
| Title block | Swimmer name and route, at the top of every page |
| Introduction | Generated from the swim information against a tone, flow and wording ruleset that Rose defined |
| Swim Details | Entirely manual, transcribed from what the forms captured. Rose also selects the best photograph for this section |
| Swim Route and Observer Logs | Embedded ZeroSixZero map plus the transcribed observer log |
| Swim Highlights | Vimeo embed and the captioned photo gallery |
| Personal Narrative | Taken from the swimmer narrative form and placed manually. Rose selects which photographs accompany it |
| Pilot and Crew | Assembled manually from whatever the submissions reveal about who did what |
| Observer Report | The post-swim observer report, with the observer's photograph, name and credentials where available. Signatures added manually |
| Download Documents and Forms | Manual links to every document the swimmer and observer submitted, pointing into Google Drive |
| Committee Review | A new Jotform created manually for this swim, capped so only three people can submit |

### The two heaviest parts

**The observer log is transcribed manually.** Every timestamped entry from the observer's log becomes a list item on the page. The example page carries roughly seventy entries, each with a time and a note, typed out into a scrolling box.

**The gallery is captioned by timestamp.** Each photo is uploaded and captioned with the time it was taken, read off the metadata, in the format `May 2, 2026 05:41`. WordPress preserves the underlying EXIF data, including capture timestamp and camera model, in the block markup.

**The map comes from ZeroSixZero,** embedded as an iframe at `https://zerosixzero.com/worldopenwater/swim-XXXX-firstname-lastname`. So the route visualisation is a third external service holding part of the record, alongside the two Drive folders.

### Note on access

The ratifications site is a separate WordPress installation and is not currently connected to anything that can read it programmatically. Everything recorded here about page structure comes from the block markup Rose supplied, not from reading the site.

The full block template was captured in part. Worth saving the complete markup as its own file in this folder, since it is the specification for anything that would generate these pages rather than copy them.

**The observer blocks the swimmer**

The post-swim observer report and the post-swim swimmer narrative run in sequence, not side by side. The swimmer's narrative is only assigned after the observer's report comes back.

This is not a design decision. Rose tried building the two as parallel branches, and **Jotform ends the whole workflow as soon as one branch completes**. So everything has to be a single chain.

The consequence is that a swimmer who is ready to write up their swim waits on an observer who may be slow, unreachable, or simply travelling home. Nothing about the two tasks genuinely depends on the other.

---

### Platform limitations

Things the current tooling genuinely cannot do, as distinct from things that were simply not set up.

**Jotform cannot schedule anything relative to a date field.** Every email fires on an event, never on a date. Consequences that follow from this one limitation:

- The good luck email arrives when the observer agreement clears, not before the swim
- There is no reminder to the swimmer as the swim date approaches
- There is no nudge when the Pre-Swim Planning form is due 30 days out
- There is no chase when a case sits at the same stage for weeks
- Nothing detects the broken link failure described above, because detecting it means noticing that time has passed

Every one of those is the same missing capability, and it is the clearest thing in the current system that something outside Jotform would have to provide.

**Jotform cannot run steps in parallel.** Tested, not assumed. Rose built the post-swim observer report and the swimmer narrative as parallel branches, and the workflow terminated as soon as one branch completed. Everything therefore has to be a single sequential chain, which is why the observer's report blocks the swimmer's narrative even though neither depends on the other.

The one apparent exception is the Google Calendar branch off the trigger, which runs alongside the main path and ends on its own. It works because nothing downstream waits for it.

**Jotform email nodes have no CC or BCC.** Copying anyone means listing them as a direct recipient. On the Observer Confirmed email this puts Quinn, the observer and the swimmer in the same To line, all visible to each other. There is no way to keep an internal recipient off a customer facing message other than sending a second, separate email.

---

### The fragility everything else depends on

**Everyone must use the form link in the email they were sent.**

The assigned form links carry the workflow context with them. If a swimmer or an observer instead goes to the public form URL, bookmarks it, or fills in a link someone forwarded them, the submission lands **outside the workflow instance**. The workflow keeps waiting for a submission that will never arrive, and the case stalls silently.

When that happens, the orphaned submission has to be found manually and reconciled.

This is the single largest operational risk in the current system. It is not a bug in any one node. It is a property of how the whole thing is held together: the workflow tracks a case through assigned links, and the links are the only thread. Nothing detects a break, nothing raises an alert, and the failure looks identical to a person simply not having responded yet.

---

### Manual stops

The workflow does not run end to end on its own. It halts at every green and blue node and waits for a person:

| # | Stop | Type |
|---|---|---|
| 1 | Route review after Pre-Swim Planning | Approval |
| 2 | Independent Observer Agreement review | Approval |
| 3 | Swim Completed | Task |
| 4 | Complete Ratification Page | Task |
| 5 | Final determination, Ratified or Not Ratified | Approval |
| 6 | Attach Certificate and Socials | Task, ratified path only |

All six are assigned to **Quinn Fitzgerald** in the builder.

### Forms used inside the workflow

| Form | ID | Sent to |
|---|---|---|
| Pre-Swim Planning 043 on Product | `210815057331043` | Swimmer, entry point |
| Relay Pre-Swim Planning 043 on Product | `251746582157161` | Swimmer, relay entry point |
| Independent Observer Agreement Form | `251242794026152` | Primary Observer Email |
| [Workflow] Post-Swim Observer Report | `252602743229455` | Primary Observer Email |
| Post-Swim Swimmer Narrative | `252386318465161` | Swimmer |

### Connected apps

- **Google Calendar**, on a branch straight off submission
- **Google Drive**, after the post-swim narrative
- **Email**, throughout

**HubSpot is being removed.** It appeared at seven points across the workflow. HubSpot was already dropped from the WOWSA app stack, so those nodes were doing nothing, and they are being deleted from the workflow as of 16 August 2026. They remain drawn in the map above for now and should be read as gone.

---

## Step 5. The committee stage

The Ratification Committee reviews via a Jotform that is **cloned manually for every swim**. A master template exists and Rose creates a copy per case, titled with the swim.

| Form | ID |
|---|---|
| WOWSA Ratification Committee Determination And Certification (template) | `252415408134147` |
| Chillon Castle Relay | `261867450870465` |
| SWIM-1033 Daragh Morgan | `261868143746467` |
| SWIM-1035 John Charles Curley | `261878390228467` |
| SWIM-0050 Renee Parks | `261868157468473` |

Every clone carries identical content. Only the swim details change.

### Why it is cloned rather than reused

Submission limits are per form. To cap a review at three votes for one swim, that swim needs its own form.

| Setting | Value |
|---|---|
| Unique Submission (cookies or IP) | **No check** |
| Unique Field | `Your Email` |
| Closes at | 3 submissions |

**This has already failed once.** The SWIM-1035 form holds four submissions, because someone submitted more than once and the controls did not stop it. That case needs correcting manually.

### What each clone has to be edited to say

Two fields carry per-swim default values that Rose sets on each copy:

| Field | Example value |
|---|---|
| Swim ID | `SWIM-1035` |
| Swimmer Name | `John Charles Curley` |

Two further fields exist but are hidden: General Route Description, and Date of swim finish.

### What the committee is asked

**WOWSA Ratification Committee Determination & Certification**

| Field | Type | Required |
|---|---|---|
| Your Name | Full name | Yes |
| Your Email | Email | Yes, and the unique field |
| Swim ID | Masked text, pre-filled | Yes |
| Swimmer Name | Text, pre-filled | Yes |
| General Route Description | Textarea, hidden | No |
| Date of swim finish | Date, hidden | No |

Then three evidence sections, each headed "Review & advise", each with the same four-option scale and a free text notes box:

| Section | Question |
|---|---|
| Photo & Video Evidence | Photo & video evidence acknowledgement |
| Observer Logs & Report | Observer logs & report acknowledgement |
| GPS Swim Tracks | GPS swim tracks acknowledgement |

The four options, identical throughout:

- Approve (no issues)
- Approve with qualifications
- Insufficient, needs follow-up
- Not Approved, see notes

Then an overall recommendation on the same four-option scale:

> Based on my review of the submitted materials, I provide the following recommendation regarding the ratification of this swim

Then a Conflict of Interest Disclosure section, matching the observer agreement:

| Question | Required |
|---|---|
| Any personal, professional or financial relationship with the swimmer, crew, organising party or outside entity that could compromise objectivity? | Yes |
| Receiving compensation or other benefits? | Yes |
| Any other circumstance that could reasonably be perceived to affect impartiality? | Yes |

Closing with a Disclosure Acknowledgment and Agreement for committee review, and a required signature.

### The vote is graded, not binary

Worth recording clearly, because it differs from how the process is described elsewhere.

A committee member does not vote Ratified or Not Ratified. They give a four-point assessment on each of three evidence categories, plus a four-point overall recommendation, plus optional notes on each.

So a single reviewer can return "Approve with qualifications" on photos, "Insufficient, needs follow-up" on GPS, and "Approve (no issues)" overall. Three reviewers produce twelve graded assessments and up to twelve notes.

**How those twelve assessments become a decision is not defined anywhere in the system.** There is no rule recorded for what combination ratifies a swim, what triggers follow-up, or how a split is resolved. It resolves in the final approval step, by a person.

The three evidence categories match the three criteria stated to swimmers and observers in the workflow emails: observer logs and report, photo and video evidence, GPS tracks. That part is consistent throughout.

**Design issue carried over from the observer form.** The three conflict questions are checkboxes rather than radio buttons, so a reviewer can tick both Yes and No.

### How the committee actually reaches the form

**The Jotform link is never sent.** The cloned committee form is **embedded into the WordPress ratification page**, in its Committee Review section.

The post is then published as **private with a password**, and that page link is what goes to the committee.

So a reviewer opens one page, reads the complete ratification, the narrative, the swim details, the map, the observer log, the gallery, the documents, and submits their determination from the bottom of that same page. They are never sent a bare form detached from the swim.

This is why the page has to be built before the committee can review, and why the build sits where it does in the sequence. The page is not the output of the decision. It is the thing the decision is made against.

### Getting the review requested

There is no automated dispatch. Rose composes an email manually and sends it to the committee. Quinn sometimes sends it instead, and it usually goes out as a batch covering several pending swims at once.

**Example: "Follow-Up: Pending Swim Ratifications", 6 August 2026**

From `contact@openwaterswimming.com`, to **twelve** named committee members, with Quinn copied. The addresses are mostly personal ones and are not reproduced here.

Structure, repeated per swim:

> Ratification N: `SWIM-XXXX` `Swimmer Name`
> View here: `https://openwaterswimming.com/ratifications/swim-xxxx-name/`
> Password: `[COMMITTEE_PASSWORD]`
> Status: N reviews received, needs N more

Closing with "When you have a chance, please review each page and submit the form for each swim."

**Twelve people are asked, three reviews are needed.** Nobody is assigned. It is an open call, and whichever three respond first make the determination for that swim. There is no panel, no rotation, and no recusal mechanism, because there is no assignment to recuse from.

**The password is shared, static and circulated by email.** `[COMMITTEE_PASSWORD]`, the same string for every swim, sent to twelve people, in a message that also lists which swims are pending and where to find them.

**The reviewer counts in the email are counted manually.** Rose checks each swim's Jotform, works out how many submissions have come in, and writes the status line. Then she monitors for new submissions and repeats.

### Publishing the result

Once all three reviews are in, Rose converts them into two tables using the **League Table** WordPress plugin, one pair per swim, built manually.

**Table 1: `SWIM-XXXX Name - Evaluation`**

The conflict of interest disclosures. Three rows, one per reviewer, four columns.

| | RELATIONSHIP | COMPENSATION | IMPARTIALITY IMPACTED |
|---|---|---|---|
| Reviewer 1 | X | X | X |
| Reviewer 2 | X | X | X |
| Reviewer 3 | X | X | X |

**Table 2: `SWIM-XXXX Name - Recommendations`**

The assessments. Three rows, one per reviewer, five columns.

| | Logs | Photos/Videos | GPS | Overall |
|---|---|---|---|---|
| Reviewer 1 | check | check | check | check |
| Reviewer 2 | check | check | check | check |
| Reviewer 3 | check | check | check | check |

Both tables carry the swim's route description as their table description.

**The steps at this stage**

1. Wait for all three reviews
2. Build the Evaluation table manually in League Table
3. Build the Recommendations table manually in League Table
4. Update the Committee Review section of the post with the decision
5. Delete the embedded committee form from the page
6. Insert the two tables in its place

**Reviewer names are published.** The three reviewers who happened to respond first appear by name on the public ratification page.

**The graded scale does not survive.** The form collects a four-point assessment on each of the four questions: Approve (no issues), Approve with qualifications, Insufficient needs follow-up, Not Approved see notes. The published table shows a tick. So "Approve with qualifications" and "Approve (no issues)" look identical to a reader, and the qualifications and notes the reviewer wrote do not appear at all.

Everything a reviewer said beyond the top-line grade lives only in the Jotform submission.

**The rest of the Committee Review section is typed in manually too.** Rose adds the names of the committee members who reviewed, and uploads their signatures to the page.

**Two tables accumulate per swim**, named `SWIM-XXXX Name - Evaluation` and `SWIM-XXXX Name - Recommendations`, sitting in the League Table plugin's own store rather than in the post.

**Note on plugins.** The ratifications site runs WPBakery Page Builder, Code Snippets, Post Views, Yoast SEO, League Table, Speed Optimizer, Security Optimizer and Envato Market. Code Snippets is installed here, which is worth knowing because it was previously recorded as unavailable on the main site.

---

## Step 6. Certificate and social assets

Made manually in **Canva**, from saved templates. **Both outcomes receive assets.**

**Ratified**

| Asset | Template |
|---|---|
| Certificate | `[CANVA_URL_CERT]` |
| Social card | `[CANVA_URL_CARD_RATIFIED]` |

**Not Ratified, published as a "verified attempt"**

| Asset | Template |
|---|---|
| Social card | `[CANVA_URL_CARD_ATTEMPT]` |


**"Verified attempt" is the public term for a swim that was not ratified.** Not rejected, not failed. This is the language WOWSA actually uses on social assets, and it sits alongside "Not Ratified" and "attempted" in internal use.

This is what the headshot collected on the swimmer narrative form is for. That headshot is one of only two required fields on that entire form.

This corresponds to the **Attach Certificate and Socials** task in the Jotform workflow, which sits on the Ratified branch after the final approval and is assigned for a person to complete.

At ten ratifications a month, this is thirty assets built manually from templates, with the swim's details typed in each time from information that already exists in the forms.

The social card is then uploaded into the final section of the WordPress post, **Congratulations to the Swimmer & Crew!**

---

## Step 7. Task: Complete Ratification Page

Back in the Jotform workflow. This task fires after the Google Drive step, and it is the node that everything in steps 4 to 6 sits inside.

| Setting | Value |
|---|---|
| Title | Complete Ratification Page |
| Description | Empty |
| Outcome | Complete, single outcome |
| Assignee | `contact@openwaterswimming.com` |
| Notification Emails | On |
| Require Login for Assignee | Off |

**Complete Ratification Page Email**

**Subject:** `Swim ID` | Complete Ratification Page

> Hello there,
> Please complete ratification page for `Swim ID` - `Name`. Thanks!

Followed by the standard task assignment block.

**What the task asks**

| Field | Type |
|---|---|
| Ratification Page URL | Text |
| Any sections not completed, or any other abnormalities | Textarea |
| Your Response | Complete |

This is the first and only point where the published page URL enters the workflow. Rose pastes it in and marks the task complete.

### What this one checkbox actually represents

The description field on this task is empty. What it stands for is everything in steps 4 through 6:

- Checking every photograph for metadata, renaming each file to its timestamp
- Assembling and titling the video in Camtasia, uploading to Vimeo, configuring the player
- Uploading the GPX to ZeroSixZero, creating the route, building the map
- Transcribing the observer log entry by entry into the code editor
- Writing and assembling ten sections of the page
- Cloning the committee form, editing its defaults, embedding it
- Publishing the post privately with a password
- Composing and sending the committee dispatch email
- Monitoring for submissions and chasing
- Building two League Tables manually
- Adding committee names and uploading their signatures
- Producing three Canva assets and uploading the social card

Days of work across six systems, resolving to a single Complete.

The free text field, "Any sections not completed, or any other abnormalities", is the only structured place in the entire system where incompleteness gets recorded.

---

## Step 8. Final approval: Ratified or Not Ratified

| Setting | Value |
|---|---|
| Outcomes | **Ratified**, **Not Ratified** |
| Approvers | `contact@openwaterswimming.com` |
| Approval Request Email | On |
| Require Login for Approver | **Off** |

The outcomes are renamed from the default Approve and Deny, so the workflow labels match the language used publicly.

**Approval Request Email**

**Subject:** `Swim ID` | Update Ratification Status

> Hello there,
> Please update committee decision below for `Swim ID`, `Name`. Thank you!

Followed by Ratified, Not Ratified and Go to Inbox buttons.

**What the approver sees**

Titled "Your action required".

| Field | Type |
|---|---|
| Your Response | Ratified or Not Ratified |
| Your Comment | Textarea |

Rose usually ignores the email and goes straight into Jotform to set the outcome.

### This step records a decision, it does not make one

The email says it outright: "Please update committee decision below." The determination was made by the committee, on the ratification page, through the embedded form. This node is where that outcome is transcribed into the workflow so it can branch.

**And this is where the twelve assessments become one word.**

Three reviewers each returned four graded assessments on a four-point scale, plus notes. Rose reads all of it and converts it into a single Ratified or Not Ratified, using judgment. There is no rule, no threshold, and no record of how the conversion was reasoned, beyond an optional free text comment.

So the answer to "how do twelve graded assessments become a decision" is: a person reads them and decides.

**Require Login is off.** A ratification determination can be recorded by clicking a button in an email, with no authentication.

### The three approvals compared

| Approval | Who actually decides | What the node does |
|---|---|---|
| Route review | Quinn, by email, after Rose measures the route in Google Maps | Records his answer |
| Independent Observer Agreement | Rose | Is the decision |
| Ratified or Not Ratified | The committee, via the embedded form on the page | Records their outcome, collapsed to binary by Rose |

Two of the three approvals in this workflow are data entry. Only one is a judgment made at the point of clicking.

### Branches

- **Ratified** goes to the Attach Certificate and Socials task, then an email, then ends.
- **Not Ratified** goes straight to an email, then ends.

### The Not Ratified email

**To:** `Email`
**Subject:** Ratification Committee Decision - `General Route Description` - `What is the date of your planned swim?`
**Attachments:** none

Note that this subject line does not carry the Swim ID, unlike every other email in the workflow.

> Hello `Name`,
> On behalf of WOWSA's Ratification Committee, I want to thank you for submitting your swim for review. Every swim is an accomplishment, and we recognize the commitment and effort that went into your attempt `General Route Description` on `What is the date of your planned swim?`.
> After careful review, the committee was unable to verify the swim for ratification based on the documentation provided. This does not diminish the significance of your achievement, it simply means we cannot formally certify it under WOWSA's ratification standards at this time.
> You can review the committee's full notes here in the Ratification Report: `Ratification Page URL`.
>
> If you would like to appeal the decision, WOWSA's appeal process allows you to submit additional documentation or clarification within 30 days of this notice, in line with our published committee policies and guide. You can find the appeal procedures here: `https://www.openwaterswimming.com/docs/independent-marathon-swim-ratification/wowsa-ratification-committee-policies-guide/`
> We deeply appreciate your contribution to the sport and your willingness to share your swim with the open water community. Your effort inspires others, regardless of the ratification outcome.
> With respect and best wishes,
> WOWSA Ratification Committee Support

### Three things this email reveals

**There is an appeal process.** Thirty days to submit additional documentation or clarification, published in the committee policies and guide. This is the first appearance of appeals anywhere in the walkthrough.

**But the workflow ends here.** There is no path back in. An appeal arriving on day twenty has nowhere to go except a person's inbox, and whatever happens next happens entirely outside the system, exactly like a resubmission after a route review denial.

**The swimmer is pointed at the ratification page for "the committee's full notes".** The published page carries two League Tables of ticks and crosses. The reviewers' written qualifications and notes live only inside the Jotform submissions. So a swimmer following that sentence to understand why their swim was not ratified may find grades rather than reasons.

### Divergence: the workflow no longer matches the agreed process

After this workflow was built, **Quinn agreed that Not Ratified swims should still receive a certificate and social assets**, worded as an attempt or as not ratified rather than as a ratification.

Rose was unable to add that step to the workflow, so **it is done manually, over email, for every Not Ratified case.**

This matters beyond the extra work. The built workflow and the agreed process have separated, and the workflow is the thing anyone would read to understand how ratifications work. Anyone reviewing the builder today would conclude that a Not Ratified swim receives an email and nothing else.

---

## Step 9. Task: Attach Certificate and Socials

On the Ratified branch only.

| Setting | Value |
|---|---|
| Title | Attach Certificate and Socials |
| Description | Empty |
| Outcome | Complete, single outcome |
| Assignee | `contact@openwaterswimming.com` |
| Notification Emails | On |
| Require Login for Assignee | **On** |

This is the only task in the workflow with Require Login switched on. The others have it off.

**Notification email**

**Subject:** A task is assigned to you.

This is the only email in the entire workflow whose subject does not carry the Swim ID. Every other message is titled with it.

**What the task asks**

| Field | Type |
|---|---|
| Link to Socials files | Text |
| Your Response | Complete |

The sublabel on the link field lists the expected assets: Print-Ready Certificate (PDF), IG Post 4:5 (PNG), IG Story 9:16 (PNG), Square 1:1 (PNG). "Print-Ready Certificate (PDF)" is duplicated in that sublabel, and Square 1:1 does not correspond to any of the three Canva templates, so the list and the actual output do not match.

### What actually happens here

1. Create a new folder called **"certificate and social cards"** inside that swim's existing ratification folder in Drive
2. Upload the certificate and social cards into it
3. Copy the folder link
4. Paste the link into this task and mark it complete

So the assets land in **a subfolder of Drive folder A**, the one Rose made manually at the Swim Completed step. Folder B, the one Jotform creates automatically, does not receive them.

That gives a completed ratification three separate Drive locations: the manual folder, its certificates subfolder, and the automated submissions folder, with only the first two related to each other.

---

## Step 10. The ratification confirmation email

Sent by Jotform once the certificate task is complete. This is the last thing the swimmer receives, and then the workflow ends.

**To:** `Email`
**Subject:** WOWSA Ratification - `Name` (RAT #: `Swim ID`)

> Hello `Name`,
> On behalf of the World Open Water Swimming Association (WOWSA), I'm pleased to confirm that your `General Route Description` swim on `What is the date of your planned swim?` has been **ratified** (`Which category of marathon swim is being attempted`).
> I've copied the ratification committee, who join me in applauding this outstanding achievement.
>
> **RATIFICATION SUMMARY:**
> - Time: `Exact Elapsed Swim Time in Days, Hours...`
> - Distance: `Exact GPS Swim Distance Covered in Kilometers`
> - Date: `Date of Swim Start`
> - Route: `General Route Description`
> - Water: `Body of Water Name`
> - Category: `Which category of marathon swim is being attempted`
> - RAT #: `Swim ID`
>
> Full details & verification: `Ratification Page URL`
> Download social cards from this link: `Link to Socials files`
> Print-Ready Certificate (PDF)
> Social Card 1:1 (PNG)
>
> If you'd like any tweaks to the page or assets, just reply here. Huge congrats on an extraordinary achievement!
>
> Warm Regards,
> WOWSA Ratification Committee Support

### Four things to check on this email

**1. WOWSA already uses "RAT #", and populates it with the Swim ID.**

The confirmation a swimmer receives labels their identifier `RAT #` and fills it with `SWIM-XXXX`. So the concept of a ratification number already exists in customer-facing language, and the Swim ID is already serving as it. Directly relevant to any proposal to mint a separate ratification ID: the label is in circulation, the value is the Swim ID, and swimmers have been quoting it that way.

**2. The distance quoted is the GPS track distance, not the straight-line tangent.**

The summary line reads `Exact GPS Swim Distance Covered in Kilometers`. WOWSA's own standard is that the ratified distance is the shortest straight-line tangent between start and finish, not the distance actually swum. That tangent figure is collected on the observer report, in a field explicitly labelled "for Record Setting Purposes", and it is not the one used here.

So a swimmer's ratification confirmation gives them a distance larger than their ratified distance, in a message headed RATIFICATION SUMMARY. Worth confirming whether that is deliberate.

**3. The email says the committee is copied, and the recipient list is the swimmer only.**

"I've copied the ratification committee, who join me in applauding this outstanding achievement." The Recipients field contains one entry, `Email`. Nobody from the committee receives this.

**4. The asset list disagrees with itself again.**

This email names two assets, Print-Ready Certificate (PDF) and Social Card 1:1 (PNG). The task field sublabel named four. Canva holds three templates. Three descriptions of the same deliverable, none matching.

### Amendments happen by email

"If you'd like any tweaks to the page or assets, just reply here."

So corrections after publication arrive as replies and are made manually, with no record of what changed or when. Relevant to any proposal involving decision versions or amendment history: there is currently no amendment trail at all.

---

## Step 11. The social post

After everything above, one more asset is built in Canva for social publication. This one is outside the Jotform workflow entirely.

| Outcome | Template |
|---|---|
| Ratified | `[CANVA_URL_POST_RATIFIED]` |
| Not Ratified, published as a verified attempt | `[CANVA_URL_POST_ATTEMPT]` |

**How it is made.** All data is typed in manually, and the imagery is a **screenshot taken from the finished ratification page**.

So the last artefact in the process is assembled by photographing a page that was itself assembled manually from six systems, and retyping the details that are already on it.

**Not Ratified swims are published too.** They receive a social post as a "verified attempt", alongside the social card from step 6. This is part of the agreement Quinn made after the workflow was built, and like the certificate step it happens entirely outside the workflow, manually.

**Handover.** The finished assets go to Quinn, who handles the posting. That is the end of the process.

---

## Rescheduled swims

Swimmers reschedule reasonably often. It is handled without restarting anything from scratch:

1. Edit the date on the swimmer's existing Pre-Swim Planning submission
2. Restart the Jotform workflow

The SWIM ID is preserved. This is why rescheduling does not create the duplicate identifier problem that a route denial would.

## How often a swim comes through twice

| Case | Frequency |
|---|---|
| Rescheduled swim | Common, handled by editing the submission |
| Route review denial | Rare |
| Same swim resubmitted | Rare |
| Withdrawal | Does not happen |
| Second attempt at the same route | None in the last two years |

---

## The Workflow table: the manual spine

**Airtable, Ratifications Pipeline, `tblulYhWIpP0E3eov`**
`https://airtable.com/appgUjmgd0K8WWp31/tblulYhWIpP0E3eov/viwiwpHCyoiqn5lCC`

Throughout everything described above, from purchase to committee review, **Rose tracks and updates this table manually.**

Nothing writes to it automatically. The order automation writes to Table 1, not here. Jotform does not write to it. The Google Drive nodes do not write to it. Every status change, every stage advance, every note is typed in by a person after doing the underlying work somewhere else.

So the Workflow table is not a system of record that the process feeds. It is a manual account of a process happening in other tools, maintained in parallel, and accurate only to the extent that someone remembered to update it.

This is what a swimmer's status is checked against when they ask where their ratification stands.

---

## Other ratification forms in the account

Not confirmed as part of the current flow. Recorded so nothing is assumed dead.

| Form | ID |
|---|---|
| Observer Log, **live**, offered as the online logging option to observers | `253011354070038` |
| Post-Swim Observer Report | `210904229503145` |
| [Backlog] Post-Swim Swimmer Narrative | `252607081230447` |
| Relay Post-Swim Ratification | `222776254666163` |
| 2025 WOWSA Pre-approved Ratification | `251477157224458` |
| WOWSA Pre-approved Ratification NEW | `231733508184355` |
| Pre-Approved Association Ratification | `213625788071158` |
| WOWSA Swim Ratification NEW 049 on Product | `231613835073049` |
| Observer Verification Form | `211305281719047` |
| Ratification Review Final Vote | `232456879284370` |
| Triple Crown Ratification | `212464965316157` |
| Lisboa-Cascais 20k Ratification | `232675574708366` |
| Submit Your Swim Report | `251157201508145` |
| Swim Safety Debrief | `251220205260135` |

---
