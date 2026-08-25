---
category: documentation
name: hbnxt-backlog-refiner
description: Scheduled/batch skill that scans HBNXT Jira backlog tickets assigned to David for ones not yet refined, runs each one through hb-tickets-skill formatting, writes the refined ticket back to Jira with the original body and open gaps preserved, tags each ticket so it's not re-scanned, and posts a Google Chat summary of what was actioned. Use when invoked by a scheduled routine/cron for backlog grooming, or when asked to "scan the backlog," "refine the backlog," or "run hb-tickets-skill on the backlog." Not for one-off ticket requests in conversation — that's hb-tickets-skill directly.
---
# HBNXT Backlog Refiner

Batch/unattended version of `hb-tickets-skill`. Where that skill formats one ticket in
a conversation, this skill sweeps David's assigned HBNXT backlog tickets, reformats every
unrefined one in place, and reports out — designed to be triggered by a scheduled routine
with no human in the loop per run.

Depends on **hb-tickets-skill** for the actual ticket format, priority/team/story-point
rules, and consolidation logic — this skill does not redefine those; it calls them.

**Priority and team are never written to Jira by this skill.** Screening those two
fields is a human call — this skill only recommends them (in the gaps section per
Step 4) for a person to confirm. Story points are still written, since effort estimation
doesn't carry the same triage judgment.

---

## Constants

| Key | Value |
|-----|-------|
| Jira Project | `HBNXT` |
| Cloud ID | `5af7c600-2e1d-4df1-a80d-05b311a13b08` |
| Scanned-marker label | `dug-ticket-refined` |
| Assignee filter | `david_schroeder@ign.com` |
| Backlog JQL | `project = HBNXT AND status = "Screening" AND issuetype in (Bug, Task) AND assignee = "david_schroeder@ign.com" AND (labels != "dug-ticket-refined" OR labels IS EMPTY) ORDER BY created ASC` |

---

## Step 1 — Find unrefined backlog tickets

Run the Backlog JQL above via `Atlassian:searchJiraIssuesUsingJql` (fields:
`summary, description, issuetype, labels, priority, status`, `maxResults: 100`).

- If it returns zero tickets, skip straight to Step 5 and report "no tickets to action."
- The `labels != "dug-ticket-refined"` clause is the only thing preventing re-scans —
  don't add a second tracking mechanism (no local state file, no session memory). If a
  ticket needs re-scanning, that's done by removing the label in Jira, not by this skill.
- **Verify the page is complete before trusting it.** This tool has been observed
  returning fewer issue nodes than `remainingCount`/`hasNextPage` implies (a pagination
  inconsistency, not a JQL problem). After the first fetch, compare the number of nodes
  returned against the query's total count (re-run with `searchResultMode: "count"`, or
  check `totalCount` if the response includes it). If they don't match, re-issue the
  search (optionally narrowing by `key in (...)` for the missing keys, or paging with
  `nextPageToken`) until every matching ticket has actually been retrieved — don't
  proceed to Step 2 on a partial list.

## Step 1a — Re-check for stragglers before reporting

Immediately before Step 6 (after every ticket found in Step 1 has been labeled), re-run
the exact Backlog JQL from Step 1 one more time. Because every ticket that matched
earlier is now labeled `dug-ticket-refined`, this second run should return zero results.
Any tickets it does return are stragglers — either from the pagination gap above, or
tickets filed after the initial scan but still within this run's window — and must be
processed (or at minimum labeled, per whatever mode this run is operating in) in the
same pass rather than left for the next scheduled run. Fold any stragglers found here
into the Step 6 report rather than silently absorbing them.

## Step 2 — For each ticket, capture the original body verbatim

Before any rewrite, save the ticket's current `description` exactly as-is (raw ADF or
plain text, whatever Jira returns). This is non-negotiable — it gets preserved at the
top of the new body per Step 4, so nothing the reporter originally wrote is ever lost,
even if the refined rewrite is wrong.

## Step 3 — Run hb-tickets-skill on the ticket

Apply `hb-tickets-skill` (Step 1's intake checklist through Step 2c's priority/team/
story-point assignment) using the original body as the sole input — there is no reporter
to ask follow-up questions of, so:

- Never block waiting for an answer. Where the intake checklist would normally ask a
  batched question (missing repro steps, payment method, URL, frequency, browser), instead
  record it as a **gap** for Step 4 and proceed with the best-supported draft, flagging
  any assumption inline in the refined ticket per hb-tickets-skill's own convention.
- Work out priority and team per Steps 2a–2b as normal, but treat them as
  **recommendations only** — record the call (and the one-line rationale hb-tickets-skill
  would otherwise put in chat) as a gap for Step 4 rather than writing either field to
  Jira. If genuinely unscoreable from the existing content (e.g. can't tell CE from
  P&B), say so in the gap instead of guessing.
- Still assign story points per Step 2c and write that field — effort estimation isn't
  a triage judgment call the way priority/team are.
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
- Recommended priority: <P0–P4 or Unscreened, with the one-line rationale> — not written
  to Jira, for a human to confirm and set.
- Recommended team: <CE / P&B / Order Fulfilment / etc., with the one-line rationale> —
  not written to Jira, for a human to confirm and set.
- <any suspected duplicate/consolidation candidate, with the existing ticket referenced>
- If nothing is missing beyond the priority/team confirmation above: "No other gaps —
  refined ticket is ready as drafted; priority/team above still need human sign-off."

----

## Refined ticket

<the full hb-tickets-skill-formatted ticket: Badges line, then Description, Steps to
Reproduce, Expected Result, Actual Result, Screenshot, Environment sections exactly per
hb-tickets-skill Step 2. Badges line shows Bug/Task and any Regression/Staging only/
Follow-up/Backlog tags as usual, but omits the P0–P4 badge — priority isn't set by this
skill, so don't print a badge value that wasn't actually written to the field.>
```

Each of the three top-level sections above is itself an h2 (`##`) with a `----` divider
between them, matching the divider convention hb-tickets-skill uses inside the refined
section. Do not remove or reorder these three sections.

## Step 5 — Write back to Jira

For each processed ticket:

1. `Atlassian:editJiraIssue` — set `description` to the Step 4 body, and set story points
   (`customfield_14097`, `customfield_13538`) per hb-tickets-skill Step 2c. **Do not set
   `priority` or team (`customfield_12300`)** — those stay whatever they already were on
   the ticket (typically `Unscreened`/unset); the recommended values live in the gaps
   section of the ticket body only, for a human to apply.
2. Add the label `dug-ticket-refined` (append — don't remove existing labels).
3. If a write fails for a ticket (permissions, field rejected), skip that ticket, leave
   it unlabeled so it's retried next run, and record the failure for the Step 6 report
   instead of retrying silently or labeling it anyway.

## Step 6 — Post the Google Chat summary

Send the summary to the Teleos DM space (`spaces/lRM_kKAAAAE` — the `singleUserBotDm`
space with David) via the Chat MCP (`chat_send_message_as_agent`, falling back to
`chat_send_message_as_user` if the agent send is unavailable), passing that space id
directly rather than resolving a DM by email — `dm_user` lookups by email have failed
against this workspace. Post once the whole sweep is done — one message per run, not
one per ticket.

Format:

```
**HBNXT backlog refiner — <date>**

Actioned <N> ticket(s):
- HBNXT-1234 — <short title> — <clean | N gaps flagged>
- HBNXT-1235 — <short title> — <clean | N gaps flagged>

<if any failed writes:>
Skipped (left unlabeled, will retry next run):
- HBNXT-1236 — <reason>

<if zero tickets found:>
No unrefined backlog tickets assigned to David found this run.
```

Keep it scannable — ticket key, title, and a one-word gap status per line. Full gap
detail lives in the ticket body (Step 4), not the chat message.

---

## Scheduling this skill

This skill is meant to be invoked by a scheduled routine (see the platform's `schedule`
capability), not run ad hoc in conversation — ad hoc single-ticket requests should go
through `hb-tickets-skill` directly instead.

When setting up the schedule, point it at this skill by name and give it no ticket-
specific input — Step 1's JQL is the only input it needs each run.