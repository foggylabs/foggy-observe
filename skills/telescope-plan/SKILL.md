# /telescope-plan — Generate a personalized tracking plan

You are a product analytics expert. Your job is to generate a `tracking-plan.md` — a personalized semantic layer that describes what this product should track, why, what normal looks like, and what to do when something breaks.

You should have a complete understanding of the codebase from `/telescope-explore`. If you don't, run `/telescope-explore` first.

**Do NOT generate any tracking code.** Only the plan.

## Step 1: Business discovery

Ask the user these 5 questions. Use AskUserQuestion with all 5 in a single call:

1. **Product basics**: "What's your product name and what does it do in one sentence?"
2. **Activation**: "What does a successful user do in their first session? What's the 'aha moment'?"
3. **Revenue**: "How do you make money (or plan to)? Which payment provider?"
4. **Marketing channels**: "Where do your users come from? (e.g., Twitter/X, Reddit, Product Hunt, SEO, paid ads, newsletter)"
5. **Biggest unknown**: "What's the #1 thing you wish you knew about how people use your product?"

Combine the answers with the codebase exploration to generate the plan.

## Step 2: Generate tracking-plan.md

Create `tracking-plan.md` in the repository root.

### Format

The file has YAML frontmatter followed by markdown sections:

```yaml
---
version: "0.1"
generated: "<ISO 8601 timestamp>"
generator: "telescope/0.1.0"
product:
  name: "<from user answer>"
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

### Section 1: Funnel Metrics (required)

Five funnel stages, each with a table:

| Column | Description |
|--------|-------------|
| Metric | Human-readable name (e.g., "Signup completed") |
| Event | PostHog event name in snake_case (e.g., `user_signed_up`) |
| Description | One sentence explaining what this measures |
| Why | Why this matters for the business |
| Normal | Expected range, specific to this product (e.g., "3-8% of visitors") |
| Red Flag | Problem condition with time dimension (e.g., "<2% for 3+ days") |

Stages: **Acquisition**, **Activation**, **Engagement**, **Retention**, **Revenue** (omit Revenue if `stage: pre_revenue`).

Rules:
- Every stage has at least 1 metric
- Event names in snake_case (PostHog convention)
- "Normal" ranges must be specific, never "varies" or "depends"
- "Red Flag" conditions must include a time dimension
- Events must map to actual code paths found during exploration

### Section 2: Marketing Attribution (recommended)

**Channel table:**

| Column | Description |
|--------|-------------|
| Channel | Marketing channel name (e.g., "Organic Search", "Twitter/X") |
| Tracking Method | How identified (e.g., "UTM params", "`$referrer` header") |
| Key Metrics | What to measure (e.g., "visitors, signups, revenue/visitor") |
| Attribution Event | PostHog event that captures the source |

Include channels the user mentioned + standard ones (Organic Search, Paid Ads, Social Media, Direct, Referral, Email).

**Revenue Attribution Properties** (on conversion/payment events):

| Property | Type | Description |
|----------|------|-------------|
| first_touch_source | string | Original traffic source (first visit) |
| first_touch_medium | string | Original medium |
| first_touch_campaign | string | Original campaign (if UTM) |
| last_touch_source | string | Most recent source before conversion |
| revenue_amount | number | Transaction amount in cents |
| payment_provider | string | Payment processor used |

### Section 3: Infrastructure Health (optional)

| Column | Description |
|--------|-------------|
| Metric | Human-readable name |
| Source | Where it comes from (e.g., "server framework", "Vercel metrics") |
| Description | What it measures |
| Threshold | Normal operating range |
| Action | What to do when breached |

Standard entries: error rate (5xx), P95 latency, uptime. Tailor to the detected deployment platform.

### Section 4: Runbook (required)

Under `## When Something Breaks`, add a subsection for every red flag in the plan. Each subsection has numbered, actionable steps.

Rules:
- Steps must be actionable by someone using AI coding tools, not a DevOps engineer
- Include specific PostHog queries, URLs, or commands where possible
- Include marketing red flags: "Top channel traffic dropped >50%", "Revenue per visitor declining"

### Section 5: Event Properties (recommended)

Schema for each event. Under `## Event Properties`, add a subsection per event with a table:

| Column | Description |
|--------|-------------|
| Property | Property name |
| Type | string, number, boolean |
| Description | What it captures |
| Example | Example value |

Include marketing-related properties on relevant events (UTM params, referrer, first-touch attribution).

## Quality check

Before saving, verify:
1. Every metric has all columns filled with specific, non-generic content
2. Event names are valid snake_case
3. "Normal" ranges are specific to this product type
4. "Red Flag" conditions include a time dimension
5. Every red flag has a runbook entry
6. Events map to real code paths found during exploration (not invented features)
7. Marketing attribution covers the channels the user mentioned

## After generating

Save `tracking-plan.md` to the repository root. Tell the user:

> "Tracking plan generated. Running `/telescope-review` to validate it against your codebase."

Then invoke `/telescope-review`.
