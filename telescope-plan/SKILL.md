# /telescope-plan — Generate a personalized tracking plan

You are a product analytics expert. Your job is to generate a `tracking-plan.md` — a **semantic layer** that AI agents will use to understand this product's analytics data.

This is not documentation for humans to read occasionally. It is structured context that AI agents (data analysts, SRE bots, MCP servers) will parse to:
- Query PostHog with business context ("what happened to activation this week?")
- Detect anomalies by comparing current data against expected patterns
- Explain data ("this metric dropped because...")

You should have a complete understanding of the codebase from `/telescope-explore`. If you don't, run `/telescope-explore` first.

**Do NOT generate any tracking code.** Only the plan.

## What PostHog already does — DO NOT re-implement

PostHog captures these automatically with zero custom code. Do NOT create custom events for any of these:

**Autocaptured events:**
- `$pageview` — every page/route change with URL, referrer, UTMs, scroll depth
- `$pageleave` — with scroll depth
- `$autocapture` — clicks on `a`, `button`, `form`, `input`, `select`, `textarea`, `label` tags (with element text, CSS selector, tag name)

**Auto-captured event properties:**
- UTMs: `$utm_source`, `$utm_medium`, `$utm_campaign`, `$utm_content`, `$utm_term`
- Click IDs: `gclid`, `fbclid`, `msclkid`
- Referrer: `$referrer`, `$referring_domain`
- Device: `$browser`, `$device_type`, `$os`
- Session: `$session_id`, `$session_duration`

**Auto-set person properties (first-touch attribution already handled):**
- `$initial_referrer`, `$initial_referring_domain`
- `$initial_utm_source`, `$initial_utm_medium`, `$initial_utm_campaign`

**Built-in features (no code needed):**
- Session replay (recordings with event timeline, network, console, errors)
- Heatmaps (clicks, scrolls, mouse movement)
- Web analytics dashboard (traffic, sources, pages)
- Actions (combine autocaptured events into meaningful groups — e.g., all CTA clicks → "Homepage CTA")

**This means:** page views, CTA clicks, button clicks, section scrolls, marketing channel identification, first-touch attribution, device/browser info — all already handled. The tracking plan should ONLY contain custom events for business logic PostHog can't see.

## Step 1: Use what explore already found

The explore phase already discovered the product name, stack, routes, auth, payments, core features, and marketing pages. **Read the explore summary from the conversation above.** Do not re-ask anything that was already answered.

## Step 2: Ask 3 questions the code can't answer

Ask the user exactly 3 questions using AskUserQuestion. For each question, **offer selectable choices based on what you found during explore** — don't ask open-ended free-text questions.

1. **Activation**: "What's the aha moment for a new user?"
   - Generate 3-4 choices from the core actions found during explore
   - Always include a "Something else" option

2. **Biggest unknown**: "What's the #1 thing you wish you knew?"
   - Generate 3-4 choices from common analytics gaps
   - Always include a "Something else" option

3. **Success metric**: "If you could only check one number every morning, what would it be?"
   - Generate 3-4 choices from the product's funnel
   - Always include a "Something else" option

Do NOT ask about product name, revenue model, or marketing channels — you already know or PostHog handles it.

## Step 3: Generate tracking-plan.md

Create `tracking-plan.md` in the repository root.

### Format

```yaml
---
version: "0.1"
generated: "<ISO 8601 timestamp>"
generator: "telescope/0.1.0"
product:
  name: "<from explore>"
  type: "saas"           # saas | marketplace | api | mobile | ecommerce | other
  business_model: "subscription"  # subscription | freemium | transactional | ad_supported | free
  stage: "pre_revenue"   # pre_revenue | has_users | has_revenue
analytics:
  provider: "posthog"
  project_id: ""
stack:
  framework: "<detected>"
  language: "<detected>"
  deployment: "<detected>"
---
```

### Section 1: Custom Events (required)

These are events PostHog CANNOT autocapture — business logic, server-side state changes, domain-specific milestones. Organized by funnel stage.

| Column | Description |
|--------|-------------|
| Metric | Human-readable name (e.g., "Signup completed") |
| Event | PostHog event name in snake_case |
| Capture | `client` or `server` |
| Description | What this measures and why it matters for the business |

Stages: **Acquisition**, **Activation**, **Engagement**, **Retention**, **Revenue** (omit Revenue if `stage: pre_revenue`).

Rules:
- Only include events that need custom `posthog.capture()` calls
- Do NOT include: page views, button clicks, form submissions, or anything PostHog autocapture handles
- If you need to track a specific button click (e.g., "Book a demo" CTA), note it as a **PostHog Action** to create in the UI — not a custom event
- `server` for state changes (user created, payment completed, resource deleted) — the source of truth
- `client` only for business-specific UI events that autocapture can't distinguish (e.g., onboarding choice between two paths)
- Events must map to actual code paths found during exploration

### Section 2: PostHog Actions (recommended)

Actions let you combine autocaptured events into meaningful groups without writing code. Define them here so the execute phase can create them in PostHog or document them for manual setup.

| Column | Description |
|--------|-------------|
| Action Name | Human-readable name (e.g., "Homepage CTA clicked") |
| Based On | What autocaptured event(s) to match |
| Filter | How to identify this specific action (URL, element text, CSS selector) |
| Description | What this action represents for the business |

Example:
```markdown
| Action Name | Based On | Filter | Description |
|-------------|----------|--------|-------------|
| Signup CTA clicked | $autocapture | Button text contains "Request access" OR "Get started" | Visitor clicks any signup CTA on marketing site |
| Pricing page viewed | $pageview | URL contains /pricing | Visitor views pricing page |
```

Use Actions for: CTA clicks, specific page visits, form submissions, navigation events. These are things PostHog already captures — you're just giving them business-meaningful names.

### Section 3: Identity & Groups (required)

**User identification:**
- When to call `posthog.identify()` (on login, on signup, on page load when authenticated)
- What person properties to set via `$set` (updateable) and `$set_once` (immutable)
- Note: `$initial_referrer`, `$initial_utm_source` etc. are already set by PostHog — only add business-specific person properties

**Group analytics** (for B2B / multi-tenant products):
- Define group types (e.g., `project`, `company`, `workspace`)
- When to call `posthog.group()`
- What group properties to set

### Section 4: Event Properties (recommended)

Schema for each **custom event** only. Under `## Event Properties`, add a subsection per event with a table:

| Column | Description |
|--------|-------------|
| Property | Property name |
| Type | string, number, boolean (no arrays — use comma-separated strings) |
| Description | What it captures |
| Example | Example value |

Rules:
- Server-side events must document where `distinct_id` comes from
- Do NOT document properties for autocaptured events — PostHog handles those
- Do NOT re-capture UTMs, referrer, browser, device — PostHog does this automatically

### Section 5: PostHog Configuration (required)

SDK configuration the execute phase needs:

- **SPA pageview tracking**: `capture_pageview: 'history_change'` for React/Vue/Svelte SPAs
- **Cross-subdomain cookies**: `cross_subdomain_cookie: true` if product spans subdomains
- **Person profiles**: `person_profiles: 'identified_only'`
- **Autocapture**: Confirm it should be enabled (default). List any elements to exclude if needed.
- **Session replay**: Confirm it should be enabled. Note any pages to exclude (e.g., admin panels with sensitive data).

## Quality check

Before saving, verify:
1. No custom events duplicate what PostHog autocapture already handles
2. Every custom event has a unique snake_case name
3. Every event specifies `client` or `server` capture location
4. PostHog Actions cover CTA clicks, specific page visits, and other autocapturable interactions
5. Identity section defines `identify()` and business-specific person properties only
6. Server-side events document `distinct_id` source
7. No properties re-capture what PostHog auto-captures (UTMs, referrer, browser, device)

## After generating

Save `tracking-plan.md` to the repository root. Tell the user:

> "Tracking plan generated. Running `/telescope-review` to validate it against your codebase."

Then invoke `/telescope-review`.
