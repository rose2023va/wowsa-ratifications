# Open questions

Things that could not be verified against the live systems during the walkthrough of
16 August 2026. Recorded rather than guessed at.

Answer them by checking the live system, then move the answer into the relevant document and
delete the entry here.

| # | Question | Where it matters |
|---|---|---|
| 1 | The Canva template recorded as a Story format. Is it still in use, and for which outcome? | [02-systems.md](02-systems.md) |
| 2 | The Google Drive node in the Jotform builder is a Send Files action. Is there a second Drive node doing anything else? | [08-assets.md](08-assets.md) |
| 3 | The order automation reports five runs this month. Did they complete, or are they failing at the script step? This decides whether new orders are landing at all. | [06-airtable.md](06-airtable.md) |
| 4 | Airtable holds both `SWIM ID#`, a visible formula, and `SWIM ID`, a hidden text field. Which is authoritative, and how does the value get from Jotform into Airtable? | [06-airtable.md](06-airtable.md) |
| 5 | Airtable's `Auto ID` field. Is it the same value as the Jotform auto increment, or a separate Airtable autonumber? | [06-airtable.md](06-airtable.md) |
| 6 | Two detached Email nodes addressed to seven recipients, greyed out in the builder. What were they for, and does anything depend on them? | [03-flow.md](03-flow.md) |
| 7 | The Pre-Swim Planning table in Airtable. Synced from Jotform, or maintained separately? | [06-airtable.md](06-airtable.md) |
| 8 | Airtable's Swim Date and Swimmer's Name and Email fields. Filled from Jotform, or manually? | [06-airtable.md](06-airtable.md) |
| 9 | Are Require Login settings on the remaining approval and task nodes deliberate, or incidental? | [11-constraints.md](11-constraints.md) |
| 10 | Which of the fourteen other ratification forms in the Jotform account are live and which are retired? | [02-systems.md](02-systems.md) |
