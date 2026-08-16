# Systems

Every system a single ratification touches, what it holds, and what it is identified by.

---

## WooCommerce

Runs on the main WordPress site at `openwaterswimming.com`.

| Item | Value |
|---|---|
| Product | Swim Ratification |
| Product URL | https://www.openwaterswimming.com/product/swim-ratification/ |
| Product ID | `70664` |
| Type | Variable product |
| Variation, Solo | `102119` |
| Variation, Relay | `102120` |
| Default variation | None set, so the buyer must choose |

The product listing carries a **purchase note** containing links to Jotform.

Add-ons offered: Record Attempt, Live Map Tracking, Blog Recap and Social Sharing.
See [12-defects.md](12-defects.md) for the fact that none of them are captured.

---

## Airtable

| Item | Value |
|---|---|
| Base | Ratifications Pipeline |
| Base ID | `appgUjmgd0K8WWp31` |
| Table, Workflow | `tblulYhWIpP0E3eov` |
| Table, Table 1 | `tblAPf6E4uzRS0oCE` |
| Table, Pre-Swim Planning | Present, contents not yet documented |
| Automation | Ratification Order, `wac1dXCCGGJzzisj8` |

Full detail in [06-airtable.md](06-airtable.md).

**Workflow** is the tracker. It is updated manually at every stage and nothing writes to it.
**Table 1** is where orders land from the automation.

---

## Jotform

The workflow engine, the identity layer and the form layer.

| Item | Value |
|---|---|
| Workflow | Ratifications Workflow |
| Workflow ID | `252603334494456` |
| Builder | https://www.jotform.com/workflow/252603334494456/build |

### Forms inside the workflow

| Form | ID |
|---|---|
| Pre-Swim Planning 043 on Product | `210815057331043` |
| Relay Pre-Swim Planning 043 on Product | `251746582157161` |
| Independent Observer Agreement Form | `251242794026152` |
| [Workflow] Post-Swim Observer Report | `252602743229455` |
| Post-Swim Swimmer Narrative | `252386318465161` |

### Forms used alongside the workflow

| Form | ID | Role |
|---|---|---|
| Observer Log | `253011354070038` | Offered to observers as the online logging option |
| WOWSA Ratification Committee Determination And Certification | `252415408134147` | Master template, cloned per swim |

### Committee clones, examples

| Form | ID |
|---|---|
| Chillon Castle Relay | `261867450870465` |
| SWIM-1033 | `261868143746467` |
| SWIM-1035 | `261878390228467` |
| SWIM-0050 | `261868157468473` |

### Other ratification forms in the account

Not confirmed as part of the current flow. Recorded so nothing is assumed dead.

| Form | ID |
|---|---|
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

**Note for anyone reading programmatically.** The Jotform REST API exposes form definitions in
full. The workflow definition itself returns HTTP 401 and cannot be read through the API, so the
workflow structure in [03-flow.md](03-flow.md) was recorded from the builder interface.

---

## Google Calendar

| Item | Value |
|---|---|
| Calendar | `contact@openwaterswimming.com` |
| Written by | Jotform, on every Pre-Swim Planning submission |

Holds the planned swim date as declared at planning. It is never updated afterwards. It is also
the only signal that a swim has happened, and is checked manually.

---

## Google Drive

Two parent folders, serving two different purposes.

| Folder | Purpose | Created by | Naming | ID |
|---|---|---|---|---|
| A | Upload space given to the swimmer and observer | Admin, manually | `SWIM-XXXX Firstname Lastname` | `[DRIVE_FOLDER_A]` |
| B | Files submitted through the forms, the working folder for building the page | Jotform, automatically | `SWIMID_Name_SwimDate` | `[DRIVE_FOLDER_B]` |

Folder A also receives a subfolder called **certificate and social cards** once a swim is
ratified.

Folder A is populated with a fixed set of empty template subfolders, copied from a local set
kept as "GDrive template folders" in the WOWSA folder on iCloud Drive.

---

## WordPress, ratifications site

A **separate WordPress installation** from the main site.

| Item | Value |
|---|---|
| Site | https://openwaterswimming.com/ratifications/ |
| Admin | https://openwaterswimming.com/ratifications/wp-admin/ |
| URL pattern | `/ratifications/swim-XXXX-firstname-lastname/` |
| Example | https://openwaterswimming.com/ratifications/swim-1039-nicholas-bufton/ |

Plugins in use: WPBakery Page Builder, Code Snippets, Post Views, Yoast SEO, League Table,
Speed Optimizer, Security Optimizer, Envato Market.

**League Table** holds two tables per swim, named `SWIM-XXXX Name - Evaluation` and
`SWIM-XXXX Name - Recommendations`. They live in the plugin's own store, not in the post.

---

## ZeroSixZero

Holds GPS routes and produces the embedded map on each ratification page.

| Item | Value |
|---|---|
| URL pattern | `https://zerosixzero.com/worldopenwater/swim-XXXX-firstname-lastname` |
| Example | https://zerosixzero.com/worldopenwater/swim-1055-hector-pardoe-08-10-2026 |

Routes are created under Routes then Create Routes, then added to a map's
**Live Tracks and Map Layers**.

Map slugs are not consistent. Some carry the swim date, some do not.

---

## Camtasia

Local application. Holds the standing title screen template so every swim's video opens
consistently.

---

## Vimeo

Video hosting. Each video is set to hidden but embeddable by link, with the player configured to
show only the play button and volume.

---

## Canva

Six templates. All content is typed in manually for each swim.

| Asset | Outcome | Design |
|---|---|---|
| Certificate | Ratified | `[CANVA_URL_CERT]` |
| Social card | Ratified | `[CANVA_URL_CARD_RATIFIED]` |
| Social post | Ratified | `[CANVA_URL_POST_RATIFIED]` |
| Social card | Verified attempt | `[CANVA_URL_CARD_ATTEMPT]` |
| Social post | Verified attempt | `[CANVA_URL_POST_ATTEMPT]` |
| Story format | Role unconfirmed | `[CANVA_URL_STORY]` |

---

## Gmail

| Address | Role |
|---|---|
| `contact@openwaterswimming.com` | Shared admin mailbox. Receives every workflow notification, is the approver on every approval node, and is the sender of all outbound process email. |
| `quinn@openwaterswimming.com` | Appears as a direct recipient on the Observer Confirmed email and on committee dispatch. |

Gmail carries several parts of the process that exist nowhere else: the route approval thread
with Quinn, the committee dispatch, chasing incomplete evidence, appeals, and the certificate and
social assets for swims that were not ratified.

---

## Removed

**HubSpot** appeared at seven points across the Jotform workflow. It was dropped from the WOWSA
stack, leaving those nodes inert, and they are being removed from the workflow as of
16 August 2026.
