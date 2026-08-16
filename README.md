# WOWSA Ratifications: current system

A forensic record of how the WOWSA independent marathon swim ratification process works
**today**, recorded from a walkthrough of the live systems on 16 August 2026.

This is the source of truth. Any proposed change, rebuild or automation should be planned
against what is written here, not against a description of how the process is supposed to work.

Everything in `docs/` describes the system **as it is**. Forward looking analysis lives in one
place, [proposals/proposal-analysis.md](proposals/proposal-analysis.md).

---

## Start here

| If you want to | Read |
|---|---|
| Understand what ratification is and who is involved | [docs/01-overview.md](docs/01-overview.md) |
| Know which systems hold what, and their identifiers | [docs/02-systems.md](docs/02-systems.md) |
| Follow a swim from purchase to publication | [docs/03-flow.md](docs/03-flow.md) |
| See every form field that is collected | [docs/04-forms.md](docs/04-forms.md) |
| Read every email the process sends, verbatim | [docs/05-emails.md](docs/05-emails.md) |
| Inspect the Airtable base and its automation script | [docs/06-airtable.md](docs/06-airtable.md) |
| See how a ratification page is assembled | [docs/07-wordpress.md](docs/07-wordpress.md) |
| Find where files, media and assets live | [docs/08-assets.md](docs/08-assets.md) |
| Understand how the committee reviews and decides | [docs/09-committee.md](docs/09-committee.md) |
| See exactly what a person does, and how often | [docs/10-manual-work.md](docs/10-manual-work.md) |
| Know what the tooling genuinely cannot do | [docs/11-constraints.md](docs/11-constraints.md) |
| Review known defects | [docs/12-defects.md](docs/12-defects.md) and the [issue tracker](../../issues) |
| See what a proposed redesign would change | [proposals/proposal-analysis.md](proposals/proposal-analysis.md) |

Terminology matters in this domain and is easy to get wrong.
Read [reference/glossary.md](reference/glossary.md) before writing anything customer facing.

---

## The shape of it in one paragraph

A swimmer buys a ratification through WooCommerce. The order reaches Airtable, which emails
them a link to a Jotform planning form. Submitting that form mints their **SWIM ID** and starts
a Jotform workflow that runs the rest of the process through a sequence of assigned forms and
approvals. Along the way an admin measures the route manually, decides when the swim has
happened, assembles a WordPress ratification page out of photographs, video, a GPS map and a
transcribed observer log, clones a committee review form and embeds it in that page, gathers
three reviews, records the outcome, produces certificates and social assets in Canva, and hands
them over for posting. An Airtable tracker is updated manually at every stage and nothing writes
to it automatically.

Roughly **ten ratifications a month** at peak.

---

## What is automated and what is not

| Stage | Automated | Manual |
|---|---|---|
| Order to Airtable record and first email | Yes | |
| SWIM ID creation | Yes | |
| Calendar entry for the planned swim date | Yes | |
| Route measurement and approval | | Yes |
| Observer agreement assignment and emails | Yes | |
| Observer agreement review | | Yes |
| Detecting that a swim has happened | | Yes |
| Post-swim form assignment | Yes | |
| Collecting form files into Drive | Yes | |
| Photo, video, map and log preparation | | Yes |
| Ratification page assembly | | Yes |
| Committee form cloning, embedding, dispatch | | Yes |
| Recording the determination | | Yes |
| Certificate and social assets | | Yes |
| Status tracking | | Yes |

---

## Redactions

This repository is public. Three things are held back and replaced with placeholders:

| Placeholder | What it is |
|---|---|
| `[COMMITTEE_PASSWORD]` | The shared password on draft ratification pages |
| `[CANVA_URL_*]` | Canva template links, which are edit URLs carrying access tokens |
| `[DRIVE_FOLDER_*]` | Google Drive parent folder IDs, which are link shareable |

Real values live in `PRIVATE.md`, which is gitignored and never committed.
Everything else, including Airtable and Jotform identifiers, needs authentication to be useful
and is recorded in full.

Swimmer names, swimmer email addresses, order numbers and committee members' personal email
addresses are not recorded anywhere in this repository.

---

## How this was recorded

Step by step from the live systems: the Airtable automation builder, the Jotform workflow
builder, the Jotform API for exact form definitions, the WordPress block markup, and the Google
Drive structure, then checked line by line with the person who runs the process.
