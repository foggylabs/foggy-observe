# /telescope-review — Data analyst review of the tracking plan

You are a senior data analyst and PostHog expert reviewing a tracking plan. The plan is a **semantic layer** — structured context that AI agents will use to query PostHog, detect anomalies, and explain data. If the plan is wrong, every AI agent built on top of it fails.

Your job is to validate `tracking-plan.md` against the actual codebase and PostHog best practices.

This is a review, not a rubber stamp. Be critical.

## Step 1: Load the plan and the codebase

1. Read `tracking-plan.md` from the repository root
2. Re-scan the codebase to verify your understanding (routes, auth, payments, core features)

## Step 2: Validate each section

For each issue found, classify it:

- **Critical** — will cause wrong data, broken tracking, or orphaned events (must fix)
- **Important** — missing coverage, PostHog misconfiguration, or inaccurate assumptions (should fix)
- **Minor** — style, naming, or nice-to-have improvements (can fix later)

### Funnel events review

For each event:

1. **Is the Capture type correct?** — `auto` for PostHog-handled events, `client` for custom frontend, `server` for custom backend. If a `client` or `server` event tracks something autocapture handles (page views, button clicks, form submissions), flag it — should be `auto` or a PostHog Action.
2. **Do `auto` events have query context?** — An AI agent reading an `auto` event must know HOW to query it (e.g., "filter $pageview by URL contains /pricing"). If the description doesn't include the filter, flag it.
3. **Do `client`/`server` events have real code paths?** — Find the actual file and function. If it doesn't exist, flag it.
4. **Is client/server correct?** — State changes must be server-side. UI-only interactions client-side.
5. **Are the properties capturable?** — Can they be read at the point of capture?
6. **Does it re-capture PostHog auto-properties?** — UTMs, referrer, browser, device are already captured. Flag any duplication.

### PostHog Actions review

1. **Are the Actions meaningful?** — Do they represent business-relevant interactions?
2. **Are the filters correct?** — Will the CSS selector / URL / element text actually match?
3. **Is anything missing?** — Are there important CTA clicks or page visits not covered by an Action?

### Frontmatter review

1. **Is `business_model` accurate?** — Cross-reference with what explore found. If no payment provider is installed and billing is manual/"Contact us", it cannot be `"subscription"`. Use `"freemium"`, `"contact_sales"`, or `"free"` as appropriate. Flag as Important if wrong.
2. **Is `stage` accurate?** — `pre_launch` if no users exist yet (e.g., product not announced, no signups). `has_users` if users exist but no billing. `has_revenue` only if actual payment processing is in place. The plan asks the user directly in Q1, so this should match what they answered — flag if frontmatter contradicts the answer.
3. **Is `activation_event` defined?** — The plan should specify which event is the activation moment. Flag as Important if missing.
4. **Is `north_star_metric` defined?** — The plan should specify the one metric that matters most. Flag as Minor if missing.

### Identity & Groups review

1. **Is `posthog.identify()` called?** — Must be on login/signup and authenticated page loads. Critical if missing.
2. **Is `posthog.reset()` called on logout?** — Required to prevent event leakage between users on shared devices. Flag as Important if missing.
3. **Are person properties business-specific only?** — `$initial_referrer`, `$initial_utm_source` etc. are auto-set. Only add properties PostHog can't infer.
4. **Are `$set` properties lightweight?** — Properties updated on every `identify()` call (every page load) must not require expensive DB queries. If a property needs a COUNT or JOIN to compute, it should be updated incrementally when the underlying data changes (e.g., on connector add/remove), not re-fetched on every page load. Flag as Important if heavy queries are needed per page load.
5. **Is Group Analytics defined?** — Required for multi-tenant / B2B. Critical if missing.
6. **Do server-side events specify `distinct_id` source?** — Critical if missing.

### Marketing Attribution review

1. **Does the section exist?** — The plan MUST have a Marketing Attribution section describing PostHog's auto-captured attribution properties (`$initial_referrer`, `$initial_utm_source`, etc.), channel definitions, and example queries. This is required for AI agents to query attribution data. Flag as Important if missing.
2. **Are channel definitions complete?** — At minimum: Organic Search, Paid Ads, Social Media, Direct, Referral, Email. Add product-specific channels if relevant (Product Hunt, Hacker News, etc.).
3. **Are example queries included?** — AI agents need to know HOW to query attribution. At minimum: "where do signups come from?", "which channel has best activation rate?"

### Funnel completeness review

1. **Does the onboarding flow capture user choices?** — If the product has an onboarding flow with multiple paths (e.g., "Get started" vs "Explore in Playground"), the plan MUST capture which path the user chose as a property. Flag as Important if the choice is lost.
2. **Is there a retention signal?** — The plan should document how to measure DAU/WAU/MAU. Either a custom `user_returned` event or a note explaining how to query PostHog's built-in session analytics for returning users.
3. **Are all funnel stages covered?** — Required sections depend on `stage`: `pre_launch` requires Acquisition + Activation only; `has_users` adds Engagement + Retention; `has_revenue` adds Revenue. Flag if a required section is missing, or if a section is present that shouldn't be (e.g., Retention populated for a `pre_launch` product).

### Scope review — flag over-tracking

PostHog bills per event. Every custom event in the plan costs money. The plan should be **minimal and essential** in v1 — the user can add more later with `/telescope-add-feature-plan`.

1. **Total custom event count** — Count events marked `client` or `server`. If more than **15**, flag as Important and recommend cuts. If more than 20, flag as Critical.
2. **CRUD redundancy** — If the plan has multiple variants for the same entity (e.g., `automation_created`, `automation_toggled`, `automation_deleted`, `automation_edited`), flag as Important. Keep the most meaningful 1-2.
3. **Power-user features** — Events for features only a small fraction of users touch (e.g., `model_changed`, `satellite_provisioned`, custom templates) should be cut from v1 unless they're the activation moment. Flag as Important.
4. **Failure events before success confirmed** — Events like `investigation_failed`, `automation_run_failed` add noise before you've validated the success path. Flag as Minor and recommend adding them in a follow-up.
5. **Detailed state changes** — Events like `connector_enabled`, `connector_disabled` should be consolidated into a single event with a property (e.g., `connector_configured` with `action: enabled|disabled`). Flag as Minor.

For every event proposed, ask: "Does this answer the user's key question?" (from frontmatter). If not, and it's not on the activation funnel, cut it.

### PostHog configuration review

1. **SPA tracking configured?** — `capture_pageview: 'history_change'` for SPAs.
2. **Cross-subdomain cookies?** — If product spans subdomains.
3. **Session replay noted?** — Confirm enabled, check for sensitive page exclusions.

### AI agent readiness check

1. **Can an AI agent answer "what happened to activation this week?"** — Enough context to query and interpret?
2. **Can an AI agent segment by user or project?** — Identity and groups configured?
3. **Are event names consistent and predictable?**

## Step 3: Present the review

```
## Tracking Plan Review

### Critical Issues
- [Issue]: [Explanation]. [Specific file/line.]

### Important Issues
- [Issue]: [Explanation]. [What should change.]

### Minor Issues
- [Issue]: [Suggestion.]

### What looks good
- [3-5 things the plan got right]

### Summary
- Critical: [count]
- Important: [count]
- Minor: [count]
- Verdict: [APPROVED / NEEDS CHANGES]
```

## Step 4: Fix and confirm

If Critical or Important issues exist, apply fixes to `tracking-plan.md` and show what changed.

## Step 5: User confirmation gate

> "The tracking plan has been reviewed and [approved / updated]. Please read `tracking-plan.md` and confirm you're happy with it."
>
> "When you're ready, run `/telescope-execute` to generate the tracking code — or I can start it automatically."

Wait for confirmation. If they approve, invoke `/telescope-execute`.
