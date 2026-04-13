# /telescope-plan — Generate a personalized tracking plan

You are a product analytics expert. Your job is to generate a `tracking-plan.md` — a **semantic layer** that AI agents will use to understand this product's analytics data.

This is not documentation for humans to read occasionally. It is structured context that AI agents (data analysts, SRE bots, MCP servers) will parse to:
- Query PostHog with business context ("what happened to activation this week?")
- Detect anomalies by comparing current data against expected patterns
- Explain data ("this metric dropped because...")

Every field matters. Vague descriptions or missing properties mean the AI agent fails. Be specific, be structured, be machine-parseable.

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

Ask the user exactly 3 questions using AskUserQuestion. For each question, **offer selectable choices based on what you found during explore** — don't ask open-ended free-text questions. The user should be able to pick from options you derived from the codebase.

1. **Activation**: "What's the aha moment for a new user?"
   - Generate 3-4 choices from the core actions you found during explore (e.g., "First chat message gets a useful result", "Connects first data source", "Completes playground demo")
   - Always include a "Something else" option

2. **Biggest unknown**: "What's the #1 thing you wish you knew?"
   - Generate 3-4 choices from common analytics gaps (e.g., "Where users drop off in onboarding", "Which features drive retention", "Why users churn after first session", "Which traffic sources bring the best users")
   - Always include a "Something else" option

3. **Success metric**: "If you could only check one number every morning, what would it be?"
   - Generate 3-4 choices from the product's funnel (e.g., "New signups", "Daily active users", "First investigation completed rate", "Weekly returning users")
   - Always include a "Something else" option

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

Do NOT put SDK configuration (like `cross_subdomain_cookie`) in the frontmatter. SDK config belongs in the execute phase, not in the plan.

### Section 1: Funnel Metrics (required)

Five funnel stages, each with a table:

| Column | Description |
|--------|-------------|
| Metric | Human-readable name (e.g., "Signup completed") |
| Event | PostHog event name in snake_case (e.g., `user_signed_up`) |
| Capture | `client` or `server` — where this event is captured |
| Description | One sentence explaining what this measures and why it matters |

Stages: **Acquisition**, **Activation**, **Engagement**, **Retention**, **Revenue** (omit Revenue if `stage: pre_revenue`).

Rules:
- Every stage has at least 1 metric
- Event names in snake_case (PostHog convention)
- Every custom event must have a unique name — do NOT reuse `$pageview` for different metrics. If you need to track visits to specific pages, use PostHog autocapture (`$pageview` is automatic) and note the URL filter, or use a custom event name.
- Mark `client` for UI interactions (clicks, form fills, navigation). Mark `server` for state changes (user created, payment completed, resource deleted). Capture where authoritative.
- Events must map to actual code paths found during exploration

### Section 2: Identity & Groups (required)

Define how users are identified and grouped. This is critical for PostHog to link events across sessions and domains.

**User identification:**
- When to call `posthog.identify()` (on login, on signup, on page load when authenticated)
- What person properties to set via `$set` (updateable: email, name, plan, role) and `$set_once` (immutable: signup_date, signup_method, first_touch_source)

**Group analytics** (for B2B / multi-tenant products):
- Define group types (e.g., `project`, `company`, `workspace`)
- When to call `posthog.group()` (on login, on project switch)
- What group properties to set (project name, plan, member count)

Example:
```markdown
## Identity & Groups

### User Identification
Call `posthog.identify(userId)` on every authenticated page load and after login/signup.

Person properties ($set):
| Property | Type | Description |
|----------|------|-------------|
| email | string | User's email |
| name | string | Display name |

Person properties ($set_once):
| Property | Type | Description |
|----------|------|-------------|
| signup_method | string | How user signed up (email, google, github) |
| signup_date | string | ISO date of account creation |

### Groups
Group type: `project`
Call `posthog.group('project', projectId, { name: projectName })` on login and project switch.
```

### Section 3: Marketing Attribution (recommended)

**Channel table:**

| Column | Description |
|--------|-------------|
| Channel | Marketing channel name (e.g., "Organic Search", "Twitter/X") |
| Tracking Method | How identified (e.g., "UTM params", "`$referrer` header") |
| Key Metrics | What to measure (e.g., "visitors, signups, revenue/visitor") |

Include standard channels: Organic Search, Paid Ads, Social Media, Direct, Referral, Email.

**First-touch attribution:**
Specify that `posthog.register_once()` must be used on first visit to persist `first_touch_source`, `first_touch_medium`, `first_touch_campaign` across sessions. And `$set_once` on person properties so attribution survives across devices.

### Section 4: Event Properties (recommended)

Schema for each custom event. Under `## Event Properties`, add a subsection per event with a table:

| Column | Description |
|--------|-------------|
| Property | Property name |
| Type | string, number, boolean (no arrays — use comma-separated strings for PostHog filterability) |
| Description | What it captures |
| Example | Example value |

Rules:
- Server-side events must document where `distinct_id` comes from (e.g., "from authenticated session user ID", "from request JWT")
- Do NOT use `string[]` — PostHog can't filter on JSON arrays. Use comma-separated strings instead.
- Note which events PostHog autocapture already handles (e.g., `$pageview` captures URL, referrer, UTMs automatically — don't re-capture those)

### Section 5: PostHog Configuration Notes (required)

Document SDK configuration that the execute phase needs to implement:

- **SPA pageview tracking**: If the app is a single-page app, note that `capture_pageview: 'history_change'` is needed in `posthog.init()` to track client-side route changes
- **Cross-subdomain cookies**: If the product spans subdomains (e.g., website on `example.com`, app on `app.example.com`), note that `cross_subdomain_cookie: true` is needed
- **Person profiles mode**: Specify `person_profiles: 'identified_only'` to avoid creating profiles for anonymous visitors
- **Autocapture**: Note what PostHog autocapture already handles (`$pageview`, `$pageleave`, clicks, form submissions) so the execute phase doesn't duplicate it

## Quality check

Before saving, verify:
1. Every custom event has a unique snake_case name (no `$pageview` reuse)
2. Every event specifies `client` or `server` capture location
3. Identity section defines when to call `identify()` and what person properties to set
4. Group analytics is defined for multi-tenant products
5. Server-side events document where `distinct_id` comes from
6. No `string[]` types — comma-separated strings instead
7. PostHog config notes cover SPA tracking, cross-domain, and autocapture
8. Events map to real code paths found during exploration (not invented features)

## After generating

Save `tracking-plan.md` to the repository root. Tell the user:

> "Tracking plan generated. Running `/telescope-review` to validate it against your codebase."

Then invoke `/telescope-review`.
