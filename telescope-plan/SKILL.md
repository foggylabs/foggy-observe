# /telescope-plan — Generate a personalized tracking plan

You are a product analytics expert. Your job is to generate a `tracking-plan.md` — a **semantic layer** that AI agents will use to understand this product's analytics data.

This is not documentation for humans to read occasionally. It is structured context that AI agents (data analysts, SRE bots, MCP servers) will parse to:
- Query PostHog with business context ("what happened to activation this week?")
- Detect anomalies ("signup rate is below the red flag threshold")
- Explain data ("this metric dropped because...")

Every field matters. Vague descriptions, generic ranges, or missing properties mean the AI agent fails. Be specific, be structured, be machine-parseable.

You should have a complete understanding of the codebase from `/telescope-explore`. If you don't, run `/telescope-explore` first.

**Do NOT generate any tracking code.** Only the plan.

## Step 1: Use what explore already found

The explore phase already discovered the product name, stack, routes, auth, payments, core features, and marketing pages. **Read the explore summary from the conversation above.** Do not re-ask anything that was already answered.

From the explore output you already know:
- Product name and what it does
- Revenue model and payment provider (or pre-revenue)
- All user actions worth tracking
- Marketing pages and acquisition surface

## Step 2: Ask 3 questions the code can't answer

Ask the user exactly 3 questions in a single call. These are business context that no amount of code reading can determine:

1. **Activation**: "What does a successful user do in their first session? What's the 'aha moment'?"
2. **Biggest unknown**: "What's the #1 thing you wish you knew about how people use your product?"
3. **Success metric**: "If you could only check one number every morning, what would it be?"

Do NOT ask about:
- **Product name** — you already know it from explore
- **What the product does** — you already scanned it
- **Revenue model** — you found it (or it's pre-revenue)
- **Marketing channels** — track all standard channels by default (Organic Search, Social Media, Direct, Referral, Email, Paid Ads). The user is installing analytics precisely because they don't know where users come from yet. Don't ask them to guess.

## Step 3: Generate tracking-plan.md

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

### Section 3: Event Properties (recommended)

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
5. Events map to real code paths found during exploration (not invented features)
6. Marketing attribution covers the channels the user mentioned

## After generating

Save `tracking-plan.md` to the repository root. Tell the user:

> "Tracking plan generated. Running `/telescope-review` to validate it against your codebase."

Then invoke `/telescope-review`.
