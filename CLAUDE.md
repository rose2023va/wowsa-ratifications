# Working in this repository

This repo documents an existing, live process. It contains no application code.

## What this repo is for

It is the reference an engineer or agent reads before proposing any change to how WOWSA
ratifications work. The process spans six systems that do not know about each other, and most
of the work is done by one person. Changes that look small often are not, because the thing
being changed is usually load bearing for something three systems away.

## Rules for anyone editing these documents

**Record what is, not what should be.** `docs/` is a description of a live system. Proposals,
opinions and improvements belong in issues, or in `PROPOSAL-ANALYSIS.md`, which is the only
forward looking document in this repository.

**Ask rather than infer.** If something cannot be verified against the live system, ask the person
who runs the process. Do not fill a gap with a plausible guess. An earlier draft of this material described the two Google Drive folders as an
unlinked defect; they are in fact two folders with two different purposes. That error came from
inferring rather than asking.

**Keep the redactions.** See the table in `README.md`. Never commit the committee password, a
Canva edit URL, or a Google Drive folder ID. Never add a swimmer's name, a swimmer's email
address, an order number, or a committee member's personal email address.

**Quote email and form copy verbatim.** These are the actual words sent to swimmers and
observers. If copy has a defect, record the defect in `docs/12-defects.md` and leave the quote
intact.

## House style

- No em dashes and no en dashes anywhere. Use commas, colons or a rewrite.
- Do not describe the tooling used to produce documents.
- Say "manual" or "manually", not "by hand".
- Use the vocabulary in `reference/glossary.md`. In particular: never write "rejected", and note
  that "Not Ratified" and "verified attempt" are used in different places for the same outcome.

## Before proposing a change

Read `docs/11-constraints.md` first. Three of the most obvious improvements are blocked by
platform limits that have already been tested, and proposals that ignore them waste everyone's
time.

Then check `docs/10-manual-work.md`. The highest value changes are the ones that remove
repeated manual work at ten cases a month, not the ones that make the output look better.
