---
name: hb-jira-tickets-raising-skill
description: Use this skill whenever anyone on the HBNXT (Humble Bundle) QA team asks Claude to raise, draft, refine, or format a Jira bug or task ticket. Trigger on phrases like "raise a ticket", "let's raise a bug", "create a ticket for...", "log this bug", "add this to a ticket", or when a screenshot/scan/video is shared alongside a request to log an issue. Also trigger when asked to combine, merge, re-scope, or restructure existing ticket drafts, or to plan QA test coverage (e.g. accessibility test passes). This skill defines both the required intake information and the ticket output format — use it for every bug/task ticket on this project, not just accessibility, checkout, wallet, or order-fulfillment-related ones. Combines conventions from the Payments/Billing, Customer Experience, and Order Fulfillment QA sub-teams.
---

# HB JIRA Tickets Raising Skill

Shared conventions for the HBNXT QA team's Jira tickets, combining ticket-writing sessions and standing checklists from three sub-teams: Payments/Billing (base format + intake checklist), Customer Experience (test account / subscription / product-link / Wallet fields), and Order Fulfillment (direct Jira-API filing conventions). Apply these by default — don't ask the reporter to re-specify format each time. Do use the intake checklist below to fill any real information gaps before drafting.

## Step 1 — Intake checklist (gather before drafting)

Before writing a bug ticket, check whether the reporter's message already contains each of these. If something is missing and it's knowable (not speculative), ask for it in one short batched question rather than drafting an incomplete ticket and iterating repeatedly:

1. **Steps to repro** — the exact click path, start to finish (not "add to cart and checkout fails" but each concrete step: navigate to X → click Y → enter Z → click Add to Cart → ...).
2. **Payment method / account used** — if the flow touches checkout or payments, note which method or account (card, PayPal, AliPay, Klarna, iDEAL, gift, guest vs. logged-in, specific test account if relevant). This single field alone resolves a large share of back-and-forth on payment-related bugs — always ask if it's a checkout/payment bug and not already stated.
3. **Test account used (ID)** — note the test account identifier used for repro. **Do not include the account password in the ticket itself** — reference that credentials are available via the team's shared test-account store; pasting a plaintext password into a Jira ticket is a sensitive-data risk regardless of environment.
4. **Subscription status of the test account** — whether the account used is an active subscriber or not; if active, what plan type (Monthly/Annual) and current status. This applies to **any bug involving a logged-in/authenticated test account**, not just subscription-management flows (Change Plan, Rejoin, gift redemption) — auth checkout, order history, account settings, and any other logged-in-state testing should also capture this, since subscriber state can affect what the account is entitled to see or do even when the flow being tested isn't subscription-specific.
5. **Test product used, with reference links** — the Akeneo and/or CommerceTools (CT) links for the product if applicable, plus the storefront UI link. Sanitize per the sensitive-data rule if any link embeds real customer/order identifiers.
6. **Wallet-specific bug fields** — if the bug involves Humble Wallet/Cash, additionally collect: the user ID, the user's location (region relevant to wallet/currency behavior), and the UserHub link for that account.
7. **URL** — the actual page/product link the bug occurred on (sanitize per the sensitive-data rule below if it contains real user/order IDs).
8. **Expected vs. actual** — what should have happened vs. what actually happened. This maps directly to the separate **Expected Result** and **Actual Result** fields in the output ticket (see Step 2).
9. **Frequency** — always / sometimes (intermittent) / one-off. Intermittent or one-off bugs should be flagged as such in the Description since they change how a dev triages and reproduces them.
10. **Screenshot or screen recording** — if provided, reference it in the ticket; if a recording is provided, extract and view frames to confirm the actual behavior before writing the ticket (see Step 4) rather than relying only on the reporter's description.
11. **Browser / device** — required if the bug is front-end / rendering / layout related. Not necessary for pure backend/API/signal bugs (e.g. Sift events) unless relevant.

Items 3–6 are usually only relevant to specific bug categories (checkout/subscription-state bugs, product-catalog bugs, Wallet bugs, or any test performed on a logged-in account) — don't force them into every ticket, but don't silently drop them either. If the reporter has already supplied enough detail to draft a complete, unambiguous ticket, don't stop to ask — proceed straight to drafting and note any assumption made inline.

### Mandatory pre-draft checklist gate

Before outputting any ticket, run through items 1–11 explicitly, one by one — not just as a mental skim. For each item, do one of:
- **Include it** in the ticket (Description or Test Data section, per Step 2), or
- **Mark it explicitly not applicable** to this bug category (e.g., Wallet fields on a non-Wallet bug), or
- **Flag it as missing and unconfirmed** if it's relevant but wasn't stated by the reporter and isn't inferable from evidence provided — state this as an open item in the ticket (e.g., "Subscription status: Not stated by reporter — confirm status of `patest134@mailinator.com`") rather than omitting it silently.

The failure mode to avoid: a field is relevant, the reporter didn't mention it, and it quietly disappears from the ticket instead of being asked about or flagged. If a ticket is about to be output and any of items 3–6 apply to the bug category but weren't addressed, stop and either ask the reporter or add the "Not stated — confirm" flag before presenting the ticket as ready-to-paste.

## Step 2 — Ticket format

```
### [TYPE][ENV] Short descriptive title

**Badges:** Bug/Task | Severity (Minor/High/Blocker) | other tags (Regression/Staging only/Follow-up/Backlog)

**Description:**
Single crisp paragraph: what the bug is, frequency (if not "always"), payment
method/account (if applicable), subscription status (if relevant), and any
dev/tech-lead notes as attributed asides (e.g. "*Tech Lead note (Martin):* ...").
Never state root cause as fact, even if a dev has offered a theory.

**Steps to Reproduce:**
1. Exact click path, start to finish — every concrete step, not a summary.
2. ...
3. ...

**Expected Result:** What should happen.
**Actual Result:** What actually happens.

**Evidence:** Reference note only (e.g. "Reference screenshot / recording attached", or "Not provided" if none).

**Test Data:**
- Env: (confirmed environments only; mark unconfirmed ones "Not verified")
- URL: actual page/product link (sanitized if needed)
- Browser / Device: if FE-related
- Payment method / account: if checkout/payment-related
- Test account: account ID only (never include password — reference the shared credential store)
- Subscription status: plan type + status, whenever a test account is used to reproduce the bug — mark "Not stated — confirm" if unknown, don't drop the field entirely
- Products tested: with Akeneo / CT / UI links if applicable
- Wallet fields: User ID / Location / UserHub link — only for Wallet bugs

**Acceptance Criteria:**
1. (Testable requirement derived from the fix — not a repeat of the repro steps)
2. ...
3. ...
```

- **Title format:** `[TYPE][ENV] Short descriptive title`
- **Types:** `BUG`, `TASK`
- **Environments:** `DEV`, `STG`, `DEV+STG`
- **Badges:** Bug / Task / Minor / High / Blocker / Regression / Backlog / Staging only / Follow-up

Only include the Test Data sub-fields that are actually relevant to the bug category (per Step 1) — don't pad every ticket with all of them.

### Critical rules — apply to every ticket

1. **QA perspective only** — strictly observational/reporting. Never state root cause as fact.
2. **Steps to Reproduce, Expected Result, Actual Result, and Acceptance Criteria are each their own separate section.** Don't merge Steps to Reproduce into the AC, and don't fold Expected/Actual into the Description — each carries distinct information and should stay separately labeled.
3. **Steps to Reproduce and AC items are numbered pointers (1. 2. 3.)** — never bullets. Jira doesn't render bullet formatting reliably.
4. **AC describes the fix's requirements, not a restatement of the repro steps.** Steps to Reproduce says how to trigger the bug; AC says what must be true once it's fixed.
5. **Every response is a complete, ready-to-paste ticket** — never a partial diff or fragment. When iterating on a ticket, always output the full thing again, not just the changed part.
6. **Sensitive data check** — if real URLs with IDs, usernames, or passwords appear in input, flag it and ask before including, or strip/sanitize and note that it was stripped. **Test account passwords are never included in ticket text, full stop** — even sanitization doesn't apply here; just omit and reference the shared credential store.
7. **Suggested error copy** is always flagged as pending PM/Content team approval — never stated as final.
8. **Consolidate over-fragmenting.** Prefer merging overlapping scenarios/flows into one ticket over creating near-duplicate tickets. Briefly flag if a request looks likely to duplicate or over-scope an existing ticket, but still comply if the reporter confirms they want it separate.
9. **Frequency matters.** If a bug is intermittent or a one-off, say so explicitly in the Description — don't silently write it up as if it reproduces every time.

## Step 3 — Accessibility tickets (Level Access scans)

When given raw scan data (CSV/table row with Rule ID, WCAG SC, severity, code snippet, selector list, description), extract:
- **Rule ID** and **WCAG SC number** (e.g. "SC 4.1.2", "SC 2.4.4", "SC 1.4.3") — cite both in the Description.
- **Instance count** — if the scan shows N matching instances or "+X more distinct examples," state this explicitly in the Description and reflect it in the AC (fix must be verified across all N instances, not just the example shown).
- **Page(s) affected** — scan data often lists multiple pages (e.g. "bundles, membership home auth, order"); note the primary page tested and flag others as needing cross-check.
- Don't fabricate the fix — describe the realistic remediation paths where relevant (e.g. for redundant/decorative ARIA roles: "either remove the interactive role/tabindex if decorative, or add the missing aria-value* attributes if it must stay interactive") rather than prescribing one solution as definite.
- If a near-identical issue was already filed on another page/component, reference the earlier ticket/rule pattern in the Description for continuity, and add an AC item confirming the fix is consistent with that prior resolution.
- Multiple distinct scan findings for the same page can be combined into one ticket titled "[Page] accessibility issues" with "Issue 1 / Issue 2" subsections in the Description — do this when asked to combine issues into one ticket.
- Common recurring rule IDs seen on this project: **1044** (separator missing aria-value*), **242** (non-unique link accessible name), **107** (insufficient text contrast).

## Step 4 — Regression tickets from video/screenshot evidence

When given a screen recording as evidence:
1. Extract frames (`ffmpeg -i <file> -vf fps=1 <dir>/frame_%03d.png`, output to a writable directory, not a read-only uploads folder) and view enough of them to confirm the actual behavior before writing the ticket — don't take the reporter's summary as the only source of truth if a recording is attached; verify it.
2. Note what's confirmed by the recording (e.g. "cart quantity does not change" = underlying logic is fine, only the alert is missing) vs. what's inferred.
3. Reference both the screenshot (expected/reference state, if provided) and the recording (actual/repro) in the ticket's Evidence line.

## Step 5 — QA process/task tickets (e.g. accessibility test planning)

For tasks like "run accessibility tests on critical E2E flows":
- Don't just mirror the existing smoke-test suite 1:1. Cross-check smoke tests for scope, but exclude items with no relevant surface for the task at hand (e.g. internal tooling, backend/PIM/OMS validation for a screen-reader pass).
- Actively look for product-specific custom UI at high risk of failure that generic smoke tests don't cover (e.g. the "Pay What You Want" tier slider, charity allocation slider, dynamic aria-live toasts/alerts for accessibility tasks) — add these even if not prompted, and say why.
- Consolidate checkout permutations: don't multiply full E2E passes by every payment method. Test full checkout once per relevant guest/auth × product-type combination, and add a single lightweight checkpoint (e.g. "payment method selector is labeled/perceivable/keyboard-selectable") folded into those passes, rather than a separate pass per payment method.
- Split scope by **auth state × product type** when finer granularity is needed (e.g. "Guest checkout — storefront+bundle", "Guest checkout — subscription", "Auth checkout — storefront+bundle", "Auth checkout — subscription", "Gift checkout") rather than by payment method.
- Keep flows with known existing bugs (e.g. guest-to-auth modal, gift redemption, rejoin, promo code) as their own distinct scope items — don't merge these into a generic checkout item, since findings need to map back to the specific known issue.
- Flag pending/not-yet-available functionality (e.g. Refund/Disputes UI) as its own scope item marked "Pending, to be added once available" rather than omitting it.
- Default timing guidance for accessibility test tasks: **before UAT**, after functional QA sign-off in Dev, so accessibility bugs get triaged in the same cycle as functional bugs rather than surfacing late. Caveat this against the project's actual defined SDLC/release process if one exists.

## Step 6 — Direct Jira-API filing (Order Fulfillment team convention — optional)

Everything above produces a **ready-to-paste draft**. Some Order Fulfillment QA prefer tickets filed directly via the Jira API rather than pasted manually. Only switch into this mode when the reporter explicitly asks to have the ticket *created/filed* (not just drafted) and identifies themselves as being on that workflow — don't assume it as the default for the whole team, since the standing field values below are specific to that team's setup and go stale.

**Standing fields for direct filing (snapshotted from real HBNXT tickets — reverify if stale):**

| Field | Value | Jira field | How to set it |
|---|---|---|---|
| **Team** | Order Fulfilment | `customfield_12300` | plain string team id: `"6d603248-7827-4f05-9a39-ab1af338ab42"`. Do **not** wrap in `{"id": ...}` — this field type rejects that shape on create. |
| **Story Points** | best t-shirt estimate, ~0.25–3 | `customfield_14097` | plain number, e.g. `0.5`, `1`, `2`, `3` — state your estimate in the draft so it can be adjusted, don't set it silently |
| **Sprint** | current active sprint | `customfield_10006` | numeric sprint id. **This value goes stale every ~2 weeks.** Don't reuse an old id from memory — confirm the current sprint id first (e.g. via `getJiraIssue` on a very recent HBNXT ticket) or ask the reporter |
| **Epic** | link only when clearly related to an existing Epic | `parent` | `{"key": "HBNXT-XX"}` — only set when the connection is obvious from context (e.g. found while testing a ticket that's a child of a known Epic); don't guess |
| **Priority** | defaults to `0 Emergency` pre-UAT | `priority` | `{"id": "10100"}` — see rule below before downgrading |
| **Assignee** | team's standing assignee for this workflow | `assignee_account_id` | confirm the current assignee with the reporter if unspecified — don't assume it's unchanged from a prior session |

**Priority discipline:** default every new ticket to `0 Emergency`. Only use a lower priority if there's a clear reason it can wait (cosmetic nit, acknowledged non-blocking gap) — and even then, **ask the reporter first** rather than silently downgrading. Never reason your way to a lower priority unilaterally.

**Filing workflow:**
1. Gather the content (summary, description, issue type — Bug for a defect found in testing, Task/Story for new work) from what's already been discussed; don't ask the reporter to restate it.
2. Decide the optional fields (Epic link only if obvious, Story Points as your best estimate, Priority per the rule above, Assignee confirmed if unspecified).
3. **Show the draft — including every field you're defaulting — and get explicit confirmation before filing.** Don't file on the first pass.
4. Once confirmed, call the Jira ticket-creation tool with the base params plus an `additional_fields` object carrying the custom fields above (team, story points, sprint) and a top-level `parent` key only when an Epic link applies.
5. Report back the new ticket's key and link — no extra commentary about fields already shown in the draft.

**If a field rejects a previously-working value** (e.g. team id, sprint id, priority scheme), treat that as a signal something changed upstream — don't keep retrying the same shape; ask the reporter or re-derive the current value from a fresh ticket rather than trusting a snapshotted value indefinitely.

## Project context

- **Project:** HBNXT (Humble Bundle Next) — site.dev.gcp-humble.com / site.stg.humblebundle.com
- **Sift events** use `$type`, `$user_id`, `$user_email`, `$session_id`, `$ip`, `$changed_password`, `api_log_request_uuid`
- **CT** = CommerceTools; **Purchase limit** = configured in CT per product
- **Akeneo** = product/PIM catalog source referenced alongside CT and UI links for product-related bugs
- **Transactional emails commonly tested:** Renewal Failed, Order Cancelled, Payment Pending, Subscription Updated (Change Plan)
- **Guest-to-Auth flow** uses shadow account approach per HBNXT guest checkout TDD
- **Tax architecture:** Avalara/AvaTax — US/CA exclusive (tax shown as line item), ROW inclusive
- **Payment methods in use:** Stripe (card), PayPal, AliPay, Klarna, iDEAL
- **Wallet/Cash bugs** need User ID, user location, and UserHub link in addition to standard fields — see Step 1, item 6

### Key Jira references

| Ticket | Summary |
|--------|---------|
| HBNXT-1900 | Gift Orders: Sift Risk Signal (Store Products) — tested, passed |
| HBNXT-1727 | Stripe duplicate account creation — blocks gift subscription status bug |
| HBNXT-2392 | Subscription Management: Change Plan failure — blocks transactional email testing for Change Plan scenario |
| HBNXT-5 | Epic — Gift |

### Known tickets already on file (check before raising a new one — don't duplicate)

1. AliPay — Payment option missing at checkout (Staging, Blocker).
2. Zero-value payment subtitle remains visible on $0.00 orders (Dev, Minor, Backlog).
3. Sift `is_gift_order` returns `false` for gift subscription orders (Task, Dev, Backlog, Follow-up).
4. Change Plan fails with "Invalid time value" / `RangeError` (Dev, Blocker) — **HBNXT-2392**.
5. PayPal — Payment Method section missing in transactional emails.
6. AliPay — Payment Method section missing in transactional emails.
7. Sift `$update_account` event not triggered on password change (Dev, High).
8. Promo code clears payment card details at checkout (Dev, High).
9. Zero-value tax line items missing from Order Summary — Regression (Dev+Staging, High).
10. Purchase limit enabled in CT causes endless loading + "Unable to Process Request" (Staging, High).
11. Guest-to-auth checkout shows wrong modal for active subscribers (Dev, High).
12. Gift subscription redeemed by registered giftee doesn't show "Active" (Dev, High) — same root cause as **HBNXT-1727**.
13. Rejoin error toast doesn't explain 24-credit limit restriction (Task, Dev, Minor).
14. Accessibility — Separator elements missing aria-value* on Store product tiles (Rule 1044).
15. Accessibility — Countdown timer links non-unique accessible names + bundle title contrast, Bundles PLP (Rules 242, 107).
16. Accessibility — Separator elements missing aria-value* on Bundle PDP platform icons (Rule 1044).
17. Regression — "Already in cart" toast no longer displays on duplicate Add to Cart (Dev, High).
18. QA Task — Run accessibility tests on critical E2E flows using screen reader.
19. Line item price shows $0.00 on Order Confirmation for 100% charity split PWYW bundle purchase (Dev, Minor) — pending Design/Product spec.

If a new request looks like it overlaps one of the above, mention the existing ticket number/summary and ask whether to link/reference it rather than duplicating.

## Workflow reminders

- If a screenshot/video/scan row is shared with a request to raise a ticket, run the intake checklist mentally first — if key fields (repro path, payment method, URL, frequency, browser, and any category-specific fields from items 3–6) are missing and not inferable from the attachment, ask in one batched question before drafting.
- Produce the full draft ticket using the conventions above once intake is sufficient — don't ask which format to use; this skill defines the default. Only move to direct Jira-API filing (Step 6) if explicitly requested.
- When asked to revise (merge sections, drop a section, rephrase one issue, add scope), always re-output the **entire** ticket, not just the changed part.
- Never include a test account password in ticket text under any circumstances, even if the reporter pastes one in — omit it and note that credentials live in the shared store.
