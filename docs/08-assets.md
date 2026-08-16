# Files, media and assets

## Google Drive

Two parents, two purposes. They are not two halves of one thing.

| Folder | Purpose | Created by | Naming |
|---|---|---|---|
| A, `[DRIVE_FOLDER_A]` | Upload space given to the swimmer and observer | Admin, manually, before Swim Completed can be marked | `SWIM-XXXX Firstname Lastname` |
| B, `[DRIVE_FOLDER_B]` | Files submitted through the forms. The working folder for building the ratification page | Jotform Send Files, automatically | `SWIMID_Name_SwimDate` |

Folder A is populated with a fixed set of empty template subfolders, copied from a local set kept
as "GDrive template folders" in the WOWSA folder on iCloud Drive.

Once a swim is ratified, a subfolder called **certificate and social cards** is created inside
folder A and the assets are uploaded there. Its link is pasted into the Jotform task.

Folder A naming is not fully consistent in practice. Both `SWIM-0050 Firstname Lastname` and
`Swim-1028` styles exist.

### What the Send Files node collects into folder B

| Field | Source form |
|---|---|
| Screenshot of your plotted course | Pre-Swim Planning |
| File Upload, observer log | Post-Swim Observer Report |
| File Upload, GPS data | Post-Swim Observer Report |
| Start and finish videos | Post-Swim Observer Report |
| Hourly photographs | Post-Swim Observer Report |
| Headshot for Certificate | Post-Swim Observer Report, hidden field |
| Headshot for the certificate | Post-Swim Swimmer Narrative |
| Additional images | Post-Swim Swimmer Narrative |

Submissions are also written as PDFs. Subfolders are created per submission.

### What is not verified

Jotform cannot see into folder A, so nothing checks whether anything was uploaded there. The
post-swim observer report can be submitted regardless. Every completeness check is a person
opening the folder and noticing what is missing.

---

## Photographs

The **metadata timestamp is the source of truth** for when a photograph was taken.

Each image is opened, its timestamp read, and the file renamed to that timestamp so it can be
uploaded to the gallery with the timestamp as its caption.

This is a persistent failure point. Photographs routinely arrive with metadata stripped, because
they were passed between devices, sent through messaging apps, or re-saved. Both the Next Steps
and Observer Confirmed emails warn against this in detail and it happens anyway. Forms for real
time photo submission during the swim were tried and did not produce consistent results.

The evidence standard therefore depends on metadata that the normal way of moving photographs
around destroys.

---

## Video

| Step | Tool |
|---|---|
| Assemble clips, add timestamps, apply the standing title template | Camtasia |
| Host, hidden but embeddable by link, player showing only play and volume | Vimeo |
| Embed | WordPress |

---

## GPS and maps

| Step | Where |
|---|---|
| Upload the GPX under Routes, then Create Routes | ZeroSixZero |
| Create a map and add the route to Live Tracks and Map Layers | ZeroSixZero |
| Embed as an iframe | WordPress |

URL pattern: `https://zerosixzero.com/worldopenwater/swim-XXXX-firstname-lastname`

Slugs are inconsistent. Some carry the swim date, some do not.

---

## Observer log

Three routes are offered to observers, and they choose one:

| Option | Where |
|---|---|
| Fillable PDF | Google Drive |
| Printable PDF | Google Drive |
| Online form, `253011354070038` | Jotform |

The online form is designed to be **submitted per timestamp**, one entry at the start and one
every hour until the finish, each potentially carrying photographs or video. A ten hour swim
therefore produces ten or more separate submissions on a standalone form that is not part of the
workflow.

Whichever route is used, the log is then transcribed entry by entry into the WordPress page.

---

## Canva

Six templates. All data is typed in manually per swim. The social post additionally uses a
**screenshot of the finished ratification page** as its imagery.

| Asset | Outcome | Design |
|---|---|---|
| Certificate | Ratified | `[CANVA_URL_CERT]` |
| Social card | Ratified | `[CANVA_URL_CARD_RATIFIED]` |
| Social post | Ratified | `[CANVA_URL_POST_RATIFIED]` |
| Social card | Verified attempt | `[CANVA_URL_CARD_ATTEMPT]` |
| Social post | Verified attempt | `[CANVA_URL_POST_ATTEMPT]` |
| Story format | Role unconfirmed | `[CANVA_URL_STORY]` |

**Both outcomes receive assets.** Swims that are not ratified are published as a verified attempt.
That was agreed after the workflow was built and could not be added to it, so it is done manually
over email.

Finished assets go to Quinn, who handles the posting.
