---
name: hb-tickets-skill
description: Use this skill whenever anyone on the HBNXT (Humble Bundle) QA team asks Claude to raise, draft, refine, or format a Jira bug or task ticket. Trigger on phrases like "raise a ticket", "let's raise a bug", "create a ticket for...", "log this bug", "add this to a ticket", or when a screenshot/scan/video is shared alongside a request to log an issue. Also trigger when asked to combine, merge, re-scope, or restructure existing ticket drafts, or to plan QA test coverage (e.g. accessibility test passes). This skill defines both the required intake information and the ticket output format — use it for every bug/task ticket on this project, not just accessibility or checkout-related ones.
---

# HB Tickets Skill

Shared conventions for the HBNXT QA team's Jira tickets, built from real ticket-writing sessions on this project. Apply these by default — don't ask the reporter to re-specify format each time. Do use the intake checklist below to fill any real information gaps before drafting.

## Step 1 — Intake checklist (gather before drafting)

Before writing a bug ticket, check whether the reporter's message already contains each of these. If something is missing and it's knowable (not speculative), ask for it in one short batched question rather than drafting an incomplete ticket and iterating repeatedly:

1. **Steps to repro** — the exact click path, start to finish (not "add to cart and checkout fails" but each concrete step: navigate to X → click Y → enter Z → click Add to Cart → ...).
2. **Payment method / account used** — if the flow touches checkout or payments, note which method or account (card, PayPal, AliPay, Klarna, iDEAL, gift, guest vs. logged-in, specific test account if relevant). This single field alone resolves a large share of back-and-forth on payment-related bugs — always ask if it's a checkout/payment bug and not already stated.
3. **URL** — the actual page/product link the bug occurred on (sanitize per the sensitive-data rule below if it contains real user/order IDs).
4. **Expected vs. actual** — what should have happened vs. what actually happened. This maps directly to the separate **Expected Result** and **Actual Result** fields in the output ticket (see Step 2).
5. **Frequency** — always / sometimes (intermittent) / one-off. Intermittent or one-off bugs should be flagged as such in the Description since they change how a dev triages and reproduces them.
6. **Screenshot or screen recording** — if provided, reference it in the ticket; if a recording is provided, extract and view frames to confirm the actual behavior before writing the ticket (see Step 4) rather than relying only on the reporter's description.
7. **Browser / device** — required if the bug is front-end / rendering / layout related. Not necessary for pure backend/API/signal bugs (e.g. Sift events) unless relevant.

If the reporter has already supplied enough detail to draft a complete, unambiguous ticket, don't stop to ask — proceed straight to drafting and note any assumption made inline.

## Step 2 — Ticket format

```
### [TYPE][ENV] Short descriptive title

**Badges:** Bug/Task | Priority (P0–P4 per Step 2a) | other tags (Regression/Staging only/Follow-up/Backlog)

# Description

Single crisp paragraph: what the bug is, frequency (if not "always"), payment
method/account (if applicable), and any dev/tech-lead notes as attributed asides
(e.g. "*Tech Lead note (Martin):* ..."). Never state root cause as fact, even if a
dev has offered a theory.

----

# Steps to Reproduce

1. Exact click path, start to finish — every concrete step, not a summary.
2. ...
3. ...

----

# Expected Result

What should happen. This is the definitive statement of correct behaviour — with no
Acceptance Criteria section, this section carries the full weight of defining what
"fixed" means, so write it precisely rather than as a one-liner. Where a fix has
several distinct correctness conditions (e.g. different auth states, product types,
or locales), state each one.

----

# Actual Result

What actually happens.

----

# Screenshot

Reference note only (e.g. "Reference screenshot / recording attached").

----

# Environment

- Env: (confirmed environments only; mark unconfirmed ones "Not verified")
- URL: actual page/product link (sanitized if needed)
- Browser / Device: if FE-related
- Payment method / account: if checkout/payment-related
- Products tested: if relevant
```

**Formatting notes:**

- Every section is an **h1** (`#`), with a `----` divider between sections. No divider after the final section.
- The Badges line sits above the first h1 and gets no header of its own.
- **No Acceptance Criteria section.** Expected Result defines correct behaviour; do not reintroduce AC under another name (no "Definition of Done", no "Requirements" section).

- **Title format:** `[TYPE][ENV] Short descriptive title`
- **Types:** `BUG`, `TASK`
- **Environments:** `DEV`, `STG`, `DEV+STG`
- **Badges:** Bug / Task / P0–P4 (see Step 2a) / Regression / Backlog / Staging only / Follow-up
- **Do not** use the old ad-hoc severity words (Minor / High / Blocker) as the severity badge — they collide with the Jira priority names and with the UAT levels. Use the P0–P4 scale instead.

## Step 2a — Priority assignment (always set this)

Every ticket gets a priority. Assign it from the **Priority Levels Reference** in the [Humble Next UAT Guide](https://docs.google.com/document/d/1Y2lNZNfbqa4Rp9bJtBt5NZw8HyjA3h12ieSkCbgHfBY/edit), owned by Sam Falotico.

| Level | UAT guide label | Jira option name (exact API value) | Definition | Action |
|---|---|---|---|---|
| **0** | Blocker | `0 Emergency` | Prevents core functionality entirely; no workaround exists. Blocks further testing or launch itself. | Fix immediately — halts sign-off until resolved. |
| **1** | Critical | `1 Priority` | Major functionality broken or wrong, but a workaround exists, or it's isolated to a subset of users/paths. | Must be fixed before launch. |
| **2** | High | `2 Expected` | Significant issue that degrades the experience or creates risk, but doesn't block the primary transaction. | Should be fixed before launch if at all feasible. |
| **3** | Medium | `3 Normal` | Noticeable but non-critical — usability friction, minor logic gaps, edge cases with low likelihood/impact. | Fix if time allows; commonly deferred to a fast-follow without blocking launch. |
| **4** | Low | `4 Nice to Have` | Cosmetic/copy polish, or a nice-to-have that isn't a defect — the feature works as intended. | Backlog; fixed opportunistically or considered for a future roadmap item. |
| — | Unscreened | `Unscreened` | Priority genuinely unclear. | Default Jira value; Pod owner screens it. |

**Rules:**

1. **Only levels 0–4 and `Unscreened` are valid.** The HBNXT priority field also exposes `High`, `Medium`, `Low`, and `Pending` — these are legacy options and must never be used.
2. **The guide's labels and the Jira option names differ** (guide says "Blocker", Jira says `0 Emergency`). The **number** is the source of truth. When writing to Jira, always pass the exact Jira option name from the table above.
3. **State the priority in the Badges line** as `P0`–`P4` with the level's label, e.g. `P2 (High)`.
4. **Give the one-line rationale in chat** when presenting the draft — which definition row the bug matched and why — so the reporter can push back before it's filed. Don't put the rationale in the ticket body.
5. **If it's genuinely a judgement call between two adjacent levels, say so** and default to the lower-numbered (more urgent) of the two, flagging it for confirmation. If it's unclear across the board, use `Unscreened` rather than guessing — per the guide, escalate to Sam.
6. **P0 and P1 are must-haves for launch; P2+ may be addressed post-launch.** Don't inflate a ticket to P1 to get attention, and don't quietly park a genuine launch-blocker at P2.
7. **Improvements** (feature works as intended, could just be better) are not bugs — they're typically P4, and route to Sam for review.

**When creating the ticket in Jira** (project `HBNXT`, cloudId `5af7c600-2e1d-4df1-a80d-05b311a13b08`), set priority in `additional_fields`:

```json
{ "priority": { "name": "2 Expected" } }
```

Also set Team (Step 2b) and Story Points (Step 2c). Note that tickets submitted via the UAT forms are auto-routed to a pod by their **Site Function/Area** value and auto-labelled `UAT-SUBMISSION`; tickets created directly through this skill are not, so set these fields explicitly.

## Step 2b — Team assignment (always set this)

Infer the owning pod from the surface the bug lives on and set `customfield_12300`. Pass the UUID as a bare string.

| Team | UUID | Owns |
|---|---|---|
| **Customer Experience** | `b29910e3-4249-476b-af3b-b2fcd146bb2f` | Storefront and bundle PDPs, cart, checkout UI, authentication and guest-to-auth, account settings, subscriptions/Choice, gift purchase and redemption, coupons/discounts, age gating, accessibility, CMS/homepage/landing pages, localization |
| **Payments & Billing** | `607ae2ab-2d64-4c7e-a4ec-d3e27eccebd0` | Payment methods (Stripe, PayPal, AliPay, Klarna, iDEAL), payment capture and refunds, billing and renewals, tax/Avalara, fraud and Sift signals, accounting/BI reports, Humble Wallet/Rewards |
| **Order Fulfilment** | `6d603248-7827-4f05-9a39-ab1af338ab42` | Order creation and state, key management and inventory, user library and redemption, product ingestion (Akeneo/CommerceTools/pim-pal), transactional emails, partner/publisher reporting, Dev Portal |
| **HBNXT: Infrastructure** | `88a2b423-26af-4462-be87-89a7608df6b0` | Platform, environments, deploys, CI, cross-service performance and availability |
| **ZD Strike Team** | `b9631a24-6dd7-49e7-97ed-9527c3802c3f` | Ziff Davis-side workstreams |

**Rules:**

1. **Assign by the surface the defect appears on, not by the system suspected of causing it.** A checkout page that renders a wrong tax figure is CE if the UI is wrong and P&B if the calculation is wrong — if that's not yet distinguishable, assign by surface and say so.
2. **One team per ticket.** If a bug genuinely spans two pods, assign the one that owns the user-facing surface and note the cross-pod dependency in the Description.
3. **State the team and the one-line reason in chat** alongside the priority rationale, so it can be corrected before filing.
4. **If the owning pod is genuinely ambiguous, say so and ask** rather than defaulting to CE. The UAT forms route unknowns to P&B for redirection; don't replicate that here — a direct question is cheaper than a misrouted ticket.

## Step 2c — Story points (always set this)

Estimate the fix effort and set **both** `customfield_14097` (Story Points) and `customfield_13538` (Story Point Estimate) to the same value.

| Points | Meaning | Typical bug |
|---|---|---|
| **0.5** | Half day — trivial | Copy change, single CSS/style fix, config toggle |
| **1** | One day — straightforward | Isolated logic fix in one component with an obvious cause |
| **2** | Two days — moderate | Fix spanning a few components, or needing non-trivial investigation first |
| **3** | Three days — significant | Fix touching shared logic, or with several distinct correctness conditions to satisfy |
| **5** | Five days — large/cross-service | Fix crossing service boundaries, or requiring a new validation/integration layer |

**Rules:**

1. **Estimate the fix, not the severity.** A P0 can be a 0.5 and a P4 can be a 3. Never let priority pull the estimate.
2. **When the cause isn't yet known, estimate the investigation plus a likely fix, and say in chat that the number is provisional pending triage.** Don't inflate to cover uncertainty.
3. **Only use the values in the table** — 0.5, 1, 2, 3, 5. No other numbers.
4. **State the points and a one-line justification in chat**, not in the ticket body.
5. `customfield_13538` is not on the Bug create screen, so set `customfield_14097` in `additional_fields` at creation, then follow up with an `editJiraIssue` call setting both. If `customfield_13538` is rejected for the issue type, keep `customfield_14097` and mention the discrepancy rather than retrying silently.

### Critical rules — apply to every ticket

1. **QA perspective only** — strictly observational/reporting. Never state root cause as fact.
2. **Description, Steps to Reproduce, Expected Result, and Actual Result are each their own h1 section.** Don't fold Expected/Actual into the Description — each carries distinct information and should stay separately headed.
3. **Steps to Reproduce items are numbered pointers (1. 2. 3.)** — never bullets. Jira doesn't render bullet formatting reliably.
4. **Expected Result states correct behaviour, not a restatement of the repro steps.** Steps to Reproduce says how to trigger the bug; Expected Result says what should have happened instead, in enough detail that a dev knows when they're done.
5. **Every response is a complete, ready-to-paste ticket** — never a partial diff or fragment. When iterating on a ticket, always output the full thing again, not just the changed part.
6. **Sensitive data check** — if real URLs with IDs, usernames, or passwords appear in input, flag it and ask before including, or strip/sanitize and note that it was stripped.
7. **Suggested error copy** is always flagged as pending PM/Content team approval — never stated as final.
8. **Consolidate over-fragmenting.** Prefer merging overlapping scenarios/flows into one ticket over creating near-duplicate tickets. Briefly flag if a request looks likely to duplicate or over-scope an existing ticket, but still comply if the reporter confirms they want it separate.
9. **Frequency matters.** If a bug is intermittent or a one-off, say so explicitly in the Description — don't silently write it up as if it reproduces every time.
10. **Never leave priority, team, or story points unset.** Assign all three per Steps 2a–2c on every draft and every Jira create — don't fall through to Jira's `Unscreened` default or an empty Team by omission. `Unscreened` is only valid as a deliberate choice.
11. **Put judgement calls in chat, not in the ticket.** Priority, team, and story point rationales go in the chat message alongside the draft so they can be corrected before filing. The ticket body stays clean.

## Step 3 — Accessibility tickets (Level Access scans)

When given raw scan data (CSV/table row with Rule ID, WCAG SC, severity, code snippet, selector list, description), extract:
- **Rule ID** and **WCAG SC number** (e.g. "SC 4.1.2", "SC 2.4.4", "SC 1.4.3") — cite both in the Description.
- **Instance count** — if the scan shows N matching instances or "+X more distinct examples," state this explicitly in the Description and reflect it in Expected Result (fix must hold across all N instances, not just the example shown).
- **Page(s) affected** — scan data often lists multiple pages (e.g. "bundles, membership home auth, order"); note the primary page tested and flag others as needing cross-check.
- Don't fabricate the fix — describe the realistic remediation paths where relevant (e.g. for redundant/decorative ARIA roles: "either remove the interactive role/tabindex if decorative, or add the missing aria-value* attributes if it must stay interactive") rather than prescribing one solution as definite.
- If a near-identical issue was already filed on another page/component, reference the earlier ticket/rule pattern in the Description for continuity, and state in Expected Result that the fix should be consistent with that prior resolution.
- Multiple distinct scan findings for the same page can be combined into one ticket titled "[Page] accessibility issues" with "Issue 1 / Issue 2" subsections in the Description — do this when asked to combine issues into one ticket.
- Common recurring rule IDs seen on this project: **1044** (separator missing aria-value*), **242** (non-unique link accessible name), **107** (insufficient text contrast).

## Step 4 — Regression tickets from video/screenshot evidence

When given a screen recording as evidence:
1. Extract frames (`ffmpeg -i <file> -vf fps=1 <dir>/frame_%03d.png`, output to a writable directory, not a read-only uploads folder) and view enough of them to confirm the actual behavior before writing the ticket — don't take the reporter's summary as the only source of truth if a recording is attached; verify it.
2. Note what's confirmed by the recording (e.g. "cart quantity does not change" = underlying logic is fine, only the alert is missing) vs. what's inferred.
3. Reference both the screenshot (expected/reference state, if provided) and the recording (actual/repro) in the ticket's Screenshot line.

## Step 5 — QA process/task tickets (e.g. accessibility test planning)

For tasks like "run accessibility tests on critical E2E flows":
- Don't just mirror the existing smoke-test suite 1:1. Cross-check smoke tests for scope, but exclude items with no relevant surface for the task at hand (e.g. internal tooling, backend/PIM/OMS validation for a screen-reader pass).
- Actively look for product-specific custom UI at high risk of failure that generic smoke tests don't cover (e.g. the "Pay What You Want" tier slider, charity allocation slider, dynamic aria-live toasts/alerts for accessibility tasks) — add these even if not prompted, and say why.
- Consolidate checkout permutations: don't multiply full E2E passes by every payment method. Test full checkout once per relevant guest/auth × product-type combination, and add a single lightweight checkpoint (e.g. "payment method selector is labeled/perceivable/keyboard-selectable") folded into those passes, rather than a separate pass per payment method.
- Split scope by **auth state × product type** when finer granularity is needed (e.g. "Guest checkout — storefront+bundle", "Guest checkout — subscription", "Auth checkout — storefront+bundle", "Auth checkout — subscription", "Gift checkout") rather than by payment method.
- Keep flows with known existing bugs (e.g. guest-to-auth modal, gift redemption, rejoin, promo code) as their own distinct scope items — don't merge these into a generic checkout item, since findings need to map back to the specific known issue.
- Flag pending/not-yet-available functionality (e.g. Refund/Disputes UI) as its own scope item marked "Pending, to be added once available" rather than omitting it.
- Default timing guidance for accessibility test tasks: **before UAT**, after functional QA sign-off in Dev, so accessibility bugs get triaged in the same cycle as functional bugs rather than surfacing late. Caveat this against the project's actual defined SDLC/release process if one exists.

## Project context

- **Project:** HBNXT (Humble Bundle Next) — site.dev.gcp-humble.com / site.stg.humblebundle.com
- **Sift events** use `$type`, `$user_id`, `$user_email`, `$session_id`, `$ip`, `$changed_password`, `api_log_request_uuid`
- **CT** = CommerceTools; **Purchase limit** = configured in CT per product
- **Transactional emails commonly tested:** Renewal Failed, Order Cancelled, Payment Pending, Subscription Updated (Change Plan)
- **Guest-to-Auth flow** uses shadow account approach per HBNXT guest checkout TDD
- **Tax architecture:** Avalara/AvaTax — US/CA exclusive (tax shown as line item), ROW inclusive
- **Payment methods in use:** Stripe (card), PayPal, AliPay, Klarna, iDEAL

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
19. **HBNXT-3351** — Checkout completes with mismatched ZIP code and city/state (Staging, P2).

If a new request looks like it overlaps one of the above, mention the existing ticket number/summary and ask whether to link/reference it rather than duplicating.

## Workflow reminders

- If a screenshot/video/scan row is shared with a request to raise a ticket, run the intake checklist mentally first — if key fields (repro path, payment method, URL, frequency, browser) are missing and not inferable from the attachment, ask in one batched question before drafting.
- Produce the full ticket using the conventions above once intake is sufficient — don't ask which format to use; this skill defines the default.
- When asked to revise (merge sections, drop a section, rephrase one issue, add scope), always re-output the **entire** ticket, not just the changed part.
