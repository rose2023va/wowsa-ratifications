# Proposals

Forward looking material. Everything in [`../docs/`](../docs/) describes the system **as it is**;
this folder is the only place that discusses changing it.

That separation is deliberate. A description of a live system stops being useful the moment
opinion is mixed into it, so proposals live here and the record stays clean.

| Document | What it covers |
|---|---|
| [proposal-analysis.md](proposal-analysis.md) | The proposed redesign assessed item by item against the current system, a combined Jotform, Airtable and Lovable target architecture, what that would automate and how, what would still need a person, and eight recommendations in neither plan |

## How to read a proposal document here

Each one is written against the current system rather than in the abstract, and marks every item
as one of:

| Status | Meaning |
|---|---|
| **New** | No equivalent exists today |
| **Replaces** | Something exists and would be swapped out |
| **Already exists** | The current system already does this, whatever the proposal says |
| **Conflicts** | The proposal describes the current system incorrectly, so adopting it would be a change rather than a description |

That last one matters most. A proposal that misreads how something works today will produce a
plan that quietly changes it.

## Before adding anything here

Read [`../docs/11-constraints.md`](../docs/11-constraints.md). Three of the most obvious
improvements are blocked by platform limits that have already been tested.
