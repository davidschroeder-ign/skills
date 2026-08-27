---
category: documentation
name: hbnxt-backlog-refiner
description: Scheduled/batch skill that scans the HBNXT Jira backlog for tickets not yet refined, runs each one through hb-jira-tickets-raising-skill formatting, writes the refined ticket back to Jira with the original body and open gaps preserved, and tags each ticket so it's not re-scanned. Use when invoked by a scheduled routine/cron for backlog grooming, or when asked to "scan the backlog," "refine the backlog," or "run hb-jira-tickets-raising-skill on the backlog." Not for one-off ticket requests in conversation — that's hb-jira-tickets-raising-skill directly.
---
# HBNXT Backlog Refiner

Batch/unattended version of `hb-jira-tickets-raising-skill`. Where that skill formats one ticket in
a conversation, this skill sweeps the HBNXT backlog and reformats every unrefined ticket
in place — designed to be triggered by a scheduled routine with no human in the loop
per run.

Depends on **hb-jira-tickets-raising-skill** for the actual ticket format, priority/team/story-point
rules, and consolidation logic — this skill does not redefine those; it calls them.

---

## Constants

| Key | Value |
|-----|-------|
| Jira Project | `HBNXT` |
| Cloud ID | `5af7c600-2e1d-4df1-a80d-05b311a13b08` |
| Scanned-marker label | `dug-ticket-refined` |
| Backlog JQL | `project = HBNXT AND status = "Backlog" AND issuetype in (Bug, Task) AND labels != "dug-ticket-refined" ORDER BY created ASC` |

---

## Step 1 — Find unrefined backlog tickets

Run the Backlog JQL above via `Atlassian:searchJiraIssuesUsingJql` (fields:
`summary, description, issuetype, labels, priority, status`, `maxResults: 100`).

- If it returns zero tickets, skip straight to Step 5 and report "no tickets to action."
- The `labels != "dug-ticket-refined"` clause is the only thing preventing re-scans —
  don't add a second tracking mechanism (no local state file, no session memory). If a
  ticket needs re-scanning, that's done by removing the label in Jira, not by this skill.

## Step 2 — For each ticket, capture the original body verbatim

Before any rewrite, save the ticket's current `description` exactly as-is (raw ADF or
plain text, whatever Jira returns). This is non-negotiable — it gets preserved at the
top of the new body per Step 4, so nothing the reporter originally wrote is ever lost,
even if the refined rewrite is wrong.

## Step 3 — Run hb-jira-tickets-raising-skill on the ticket

Apply `hb-jira-tickets-raising-skill` (Step 1's intake checklist through Step 2c's priority/team/
story-point assignment) using the original body as the sole input — there is no reporter
to ask follow-up questions of, so:

- Never block waiting for an answer. Where the intake checklist would normally ask a
  batched question (missing repro steps, payment method, URL, frequency, browser), instead
  record it as a **gap** for Step 4 and proceed with the best-supported draft, flagging
  any assumption inline in the refined ticket per hb-jira-tickets-raising-skill's own convention.
- Still assign priority, team, and story points per Steps 2a–2c. If genuinely
  unscoreable from the existing content (e.g. can't tell CE from P&B), use `Unscreened`
  priority or note the team ambiguity as a gap — don't silently guess.
- Still apply Step 8 (consolidation against the "known tickets already on file" list) —
  if a ticket looks like a near-duplicate, note that as an action item rather than
  merging it unilaterally; this skill does not delete or merge tickets on its own.

## Step 4 — Assemble the new ticket body

Structure, top to bottom:

```
## Original ticket (unmodified)

<the exact body captured in Step 2>

----

## Actions needed / gaps to fill

- <each missing intake field from the checklist that couldn't be inferred>
- <any priority/team/story-point call that was ambiguous or defaulted to Unscreened>
- <any suspected duplicate/consolidation candidate, with the existing ticket referenced>
- If nothing is missing and the draft is complete as-is: "No gaps — refined ticket is
  ready as drafted."

----

## Refined ticket

<the full hb-jira-tickets-raising-skill-formatted ticket: Badges line, then Description, Steps to
Reproduce, Expected Result, Actual Result, Screenshot, Environment sections exactly per
hb-jira-tickets-raising-skill Step 2>
```

Each of the three top-level sections above is itself an h2 (`##`) with a `----` divider
between them, matching the divider convention hb-jira-tickets-raising-skill uses inside the refined
section. Do not remove or reorder these three sections.

## Step 5 — Write back to Jira

For each processed ticket:

1. `Atlassian:editJiraIssue` — set `description` to the Step 4 body, and set priority/
   team (`customfield_12300`)/story points (`customfield_14097`, `customfield_13538`)
   per hb-jira-tickets-raising-skill Steps 2a–2c.
2. Add the label `dug-ticket-refined` (append — don't remove existing labels).
3. If a write fails for a ticket (permissions, field rejected), skip that ticket, leave
   it unlabeled so it's retried next run, instead of retrying silently or labeling it
   anyway.

---

## Scheduling this skill

This skill is meant to be invoked by a scheduled routine (see the platform's `schedule`
capability), not run ad hoc in conversation — ad hoc single-ticket requests should go
through `hb-jira-tickets-raising-skill` directly instead.

When setting up the schedule, point it at this skill by name and give it no ticket-
specific input — Step 1's JQL is the only input it needs each run.