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

### Custom events review

For each custom event:

1. **Does it duplicate autocapture?** — PostHog already captures `$pageview`, `$pageleave`, and clicks/forms/inputs. If a custom event tracks something autocapture handles, flag it as Critical. Use a PostHog Action instead.
2. **Does the code path exist?** — Find the actual file and function. If it doesn't exist, flag it.
3. **Is client/server correct?** — State changes must be server-side. UI-only interactions client-side.
4. **Are the properties capturable?** — Can they be read at the point of capture?
5. **Does it re-capture PostHog auto-properties?** — UTMs, referrer, browser, device are already captured. Flag any duplication.

### PostHog Actions review

1. **Are the Actions meaningful?** — Do they represent business-relevant interactions?
2. **Are the filters correct?** — Will the CSS selector / URL / element text actually match?
3. **Is anything missing?** — Are there important CTA clicks or page visits not covered by an Action?

### Identity & Groups review

1. **Is `posthog.identify()` called?** — Must be on login/signup and authenticated page loads. Critical if missing.
2. **Are person properties business-specific only?** — `$initial_referrer`, `$initial_utm_source` etc. are auto-set. Only add properties PostHog can't infer.
3. **Is Group Analytics defined?** — Required for multi-tenant / B2B. Critical if missing.
4. **Do server-side events specify `distinct_id` source?** — Critical if missing.

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
