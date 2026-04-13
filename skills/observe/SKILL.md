# /observe — Set up PostHog tracking + generate a tracking plan

Set up analytics for your product in ~15 minutes. Scans your codebase, detects your stack, sets up PostHog, asks business questions, generates a personalized tracking plan, and commits tracking code.

## When to use

- User says `/observe`, "set up tracking", "add analytics", or "I want to know what my users are doing"
- First-time setup for a new product
- Product has no analytics or incomplete tracking

## Phase 1: Stack Detection

Detect the project's tech stack by reading dependency/config files:

1. **Language & framework**: Read `package.json`, `requirements.txt`, `go.mod`, `Cargo.toml`, `Gemfile`, `pubspec.yaml`, `composer.json`
   - Identify: Next.js, React, Vue, Svelte, Express, Django, Flask, Rails, Go, Rust, etc.
2. **PostHog SDK**: Check if already installed — look for `posthog-js`, `posthog-node`, `posthog-python`, `posthog-go`, `posthog-ruby`, `posthog-php` in dependencies
3. **Deployment**: Check for `vercel.json`, `netlify.toml`, `fly.toml`, `Dockerfile`, `render.yaml`, `app.yaml`, `railway.json`
4. **Existing tracking plan**: Check for `tracking-plan.md` in the repo root
5. **Existing analytics**: Grep for `posthog.capture`, `analytics.track`, `mixpanel.track`, `gtag`, `plausible` calls

Tell the user what you found:
> "I detected a **Next.js** app with **TypeScript**, deployed on **Vercel**. PostHog is **not installed**. No existing tracking plan found."

If `tracking-plan.md` already exists, ask the user if they want to regenerate it or update it.

## Phase 2: PostHog Setup (if missing)

If PostHog SDK is NOT in the project dependencies:

1. Tell the user: "You need a PostHog account (free — 1M events/month). Sign up at posthog.com if you haven't already."
2. Install the appropriate SDK for the detected stack:
   - **Next.js / React / Vue / Svelte**: `npm install posthog-js` (or yarn/pnpm/bun based on lockfile)
   - **Node.js backend**: `npm install posthog-node`
   - **Python**: `pip install posthog`
   - **Go**: `go get github.com/posthog/posthog-go`
   - **Ruby**: `gem install posthog-ruby`
3. Ask the user for their PostHog project API key using AskUserQuestion
4. Add PostHog initialization code to the correct entry point:

**Next.js App Router:**
```tsx
// app/providers.tsx
'use client'
import posthog from 'posthog-js'
import { PostHogProvider } from 'posthog-js/react'
import { useEffect } from 'react'

export function PHProvider({ children }: { children: React.ReactNode }) {
  useEffect(() => {
    posthog.init('<API_KEY>', {
      api_host: 'https://us.i.posthog.com', // or eu.i.posthog.com
      person_profiles: 'identified_only',
      capture_pageview: true,
      capture_pageleave: true,
    })
  }, [])
  return <PostHogProvider client={posthog}>{children}</PostHogProvider>
}
```

**Next.js Pages Router:**
```tsx
// pages/_app.tsx
import posthog from 'posthog-js'
import { PostHogProvider } from 'posthog-js/react'
import { useEffect } from 'react'

export default function App({ Component, pageProps }) {
  useEffect(() => {
    posthog.init('<API_KEY>', {
      api_host: 'https://us.i.posthog.com',
      person_profiles: 'identified_only',
    })
  }, [])
  return (
    <PostHogProvider client={posthog}>
      <Component {...pageProps} />
    </PostHogProvider>
  )
}
```

**React + Vite:**
```tsx
// src/main.tsx
import posthog from 'posthog-js'
posthog.init('<API_KEY>', {
  api_host: 'https://us.i.posthog.com',
  person_profiles: 'identified_only',
})
```

**Express / Node.js:**
```ts
// src/analytics.ts
import { PostHog } from 'posthog-node'
const posthog = new PostHog('<API_KEY>', { host: 'https://us.i.posthog.com' })
export default posthog
```

**Python / Django / Flask:**
```python
# analytics.py
from posthog import Posthog
posthog = Posthog('<API_KEY>', host='https://us.i.posthog.com')
```

Replace `<API_KEY>` with the user's actual key. Wrap it in an env var (`NEXT_PUBLIC_POSTHOG_KEY` or similar) for the detected framework.

If PostHog is already installed: skip this phase, confirm the existing setup looks correct.

## Phase 3: Codebase Scan

Scan the codebase to understand the product's surface area. This informs the tracking plan.

1. **Routes/Pages**: Glob for route files to understand what pages exist
   - Next.js: `app/**/page.tsx`, `pages/**/*.tsx`
   - React Router: grep for `<Route`, `createBrowserRouter`
   - Express: grep for `app.get`, `app.post`, `router.get`, `router.post`
   - Django: grep for `urlpatterns`, `path(`
   - Rails: read `config/routes.rb`

2. **Auth patterns**: Grep for signup/login/registration flows
   - Keywords: `signup`, `sign-up`, `register`, `login`, `sign-in`, `signIn`, `signUp`
   - Providers: `Clerk`, `Auth0`, `Supabase`, `NextAuth`, `auth.js`, `passport`, `bcrypt`, `jwt`
   - OAuth: `google`, `github`, `oauth`, `callback`

3. **Payment patterns**: Grep for payment/checkout flows
   - Stripe: `stripe`, `checkout`, `subscription`, `price_`, `createCheckoutSession`
   - LemonSqueezy: `lemonsqueezy`, `lemon_squeezy`
   - Paddle: `paddle`
   - Polar: `polar`
   - Keywords: `payment`, `billing`, `invoice`, `upgrade`, `plan`

4. **Existing analytics**: Grep for any existing tracking calls
   - `posthog.capture`, `posthog.identify`
   - `analytics.track`, `analytics.identify`
   - `mixpanel.track`, `gtag`, `umami`, `plausible`

5. **Marketing/landing pages**: Look for landing, pricing, features pages
   - Grep for: `landing`, `pricing`, `features`, `hero`, `cta`, `waitlist`
   - Check for marketing-style routes: `/`, `/pricing`, `/features`, `/about`

Build a mental model: what pages exist, what user actions are possible, what payment provider is used, what marketing pages exist, and what's already tracked. You'll use this to generate the tracking plan.

## Phase 4: Business Discovery

Ask the user 5 questions to understand their business context. Use AskUserQuestion with all 5 questions in a single call:

1. **Product basics**: "What's your product name and what does it do in one sentence?"
2. **Activation**: "What does a successful user do in their first session? What's the 'aha moment'?"
3. **Revenue**: "How do you make money (or plan to)? Which payment provider (Stripe, LemonSqueezy, Polar)?"
4. **Marketing channels**: "Where do your users come from? (e.g., Twitter/X, Reddit, Product Hunt, SEO, paid ads, newsletter)"
5. **Biggest unknown**: "What's the #1 thing you wish you knew about how people use your product or where they come from?"

Use the answers combined with the codebase scan to generate a personalized tracking plan.

## Phase 5: Generate tracking-plan.md

Generate `tracking-plan.md` at the repository root. Follow the tracking plan spec v0.1 format below.

### Tracking Plan Format

The file has YAML frontmatter followed by markdown sections:

```yaml
---
version: "0.1"
generated: "<ISO 8601 timestamp>"
generator: "foggy-observe/0.1.0"
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

Five funnel stages, each with a table. Every metric needs all columns filled with specific, non-generic content.

| Column | Type | Description |
|--------|------|-------------|
| Metric | string | Human-readable name (e.g., "Signup completed") |
| Event | string | PostHog event name in snake_case (e.g., `user_signed_up`) |
| Description | string | One sentence explaining what this measures |
| Why | string | Why this matters for the business |
| Normal | string | Expected range, specific to this product (e.g., "3-8% of visitors") |
| Red Flag | string | Problem condition with time dimension (e.g., "<2% for 3+ days") |

Stages: **Acquisition**, **Activation**, **Engagement**, **Retention**, **Revenue** (omit Revenue if `stage: pre_revenue`).

Rules:
- Every stage has at least 1 metric
- Event names in snake_case (PostHog convention)
- "Normal" ranges must be specific to the product type, never "varies" or "depends"
- "Red Flag" conditions must include a time dimension

### Section 2: Infrastructure Health (optional in v1)

| Column | Description |
|--------|-------------|
| Metric | Human-readable name |
| Source | Where it comes from (e.g., "server framework", "Vercel metrics") |
| Description | What it measures |
| Threshold | Normal operating range |
| Action | What to do when breached |

Standard entries: error rate (5xx), P95 latency, uptime. Tailor to the detected deployment platform.

### Section 3: Runbook (required)

Under `## When Something Breaks`, add a subsection for every red flag in the funnel metrics and marketing attribution. Each subsection has numbered, actionable steps.

Rules:
- Steps must be actionable by someone using AI coding tools, not a DevOps engineer
- Include specific PostHog queries, URLs, or commands where possible
- Include marketing red flags: "Top channel traffic dropped >50%", "Revenue per visitor declining"

### Section 4: Event Properties (recommended)

Schema for each event. Under `## Event Properties`, add a subsection per event with a table:

| Column | Description |
|--------|-------------|
| Property | Property name |
| Type | string, number, boolean |
| Description | What it captures |
| Example | Example value |

Include marketing-related properties on relevant events (UTM params, referrer, first-touch attribution).

### Section 5: Marketing Attribution (recommended)

Under `## Marketing Attribution`, add:

**Channel table:**

| Column | Description |
|--------|-------------|
| Channel | Marketing channel name (e.g., "Organic Search", "Twitter/X") |
| Tracking Method | How identified (e.g., "UTM params", "`$referrer` header") |
| Key Metrics | What to measure (e.g., "visitors, signups, revenue/visitor") |
| Attribution Event | PostHog event that captures the source |

Standard channels to include (customize based on user's answer to question 4):
- Organic Search, Paid Ads, Social Media, Direct, Referral, Email, Product Hunt

**Revenue Attribution Properties** (on conversion/payment events):

| Property | Type | Description |
|----------|------|-------------|
| first_touch_source | string | Original traffic source (first visit) |
| first_touch_medium | string | Original medium |
| first_touch_campaign | string | Original campaign (if UTM) |
| last_touch_source | string | Most recent source before conversion |
| revenue_amount | number | Transaction amount in cents |
| payment_provider | string | Payment processor used |

Red flags for this section:
- "Top channel drops >50% week-over-week"
- "Revenue per visitor drops below $X for 2+ weeks"
- "Unknown/direct traffic exceeds 70% (attribution is broken)"

### Quality Criteria

Before finalizing the plan, verify:
1. Every metric has all columns filled with specific, non-generic content
2. Event names are valid snake_case for PostHog
3. "Normal" ranges are specific to the product type
4. "Red Flag" conditions include a time dimension
5. The runbook has actionable steps for every red flag (including marketing red flags)
6. Marketing attribution covers the channels the user mentioned

## Phase 6: Generate Tracking Code

For each event in the tracking plan, generate PostHog tracking code and place it in the correct files.

### Event capture

For each event in the funnel metrics, add `posthog.capture()` calls in the appropriate location:

```ts
posthog.capture('event_name', {
  property_1: value1,
  property_2: value2,
})
```

Place these based on the codebase scan:
- Page view events: in page components or route handlers
- Button click events: in click handlers
- Form submission events: in form submit handlers
- API events: in API route handlers or server actions
- Payment events: in payment webhook handlers or success callbacks

### Marketing attribution setup

Add first-touch attribution capture. This runs once on the user's first visit and persists across sessions:

```ts
// First-touch attribution — set once on first visit, persists across sessions
const urlParams = new URLSearchParams(window.location.search)
const referrer = document.referrer

posthog.register_once({
  first_touch_source: urlParams.get('utm_source') || getReferrerSource(referrer),
  first_touch_medium: urlParams.get('utm_medium') || getReferrerMedium(referrer),
  first_touch_campaign: urlParams.get('utm_campaign') || '',
})

function getReferrerSource(referrer: string): string {
  if (!referrer) return 'direct'
  const domain = new URL(referrer).hostname
  if (domain.includes('google')) return 'google'
  if (domain.includes('bing')) return 'bing'
  if (domain.includes('twitter') || domain.includes('x.com')) return 'twitter'
  if (domain.includes('reddit')) return 'reddit'
  if (domain.includes('linkedin')) return 'linkedin'
  if (domain.includes('producthunt')) return 'producthunt'
  if (domain.includes('news.ycombinator')) return 'hackernews'
  return domain
}

function getReferrerMedium(referrer: string): string {
  if (!referrer) return 'none'
  const domain = new URL(referrer).hostname
  if (['google', 'bing', 'duckduckgo', 'yahoo'].some(s => domain.includes(s))) return 'organic'
  if (['twitter', 'x.com', 'reddit', 'linkedin', 'facebook'].some(s => domain.includes(s))) return 'social'
  return 'referral'
}
```

Place this code in the app's entry point (same file as PostHog init or the root layout/provider).

### User identification

Add `posthog.identify()` in the auth flow, after signup or login:

```ts
posthog.identify(userId, {
  email: user.email,
  name: user.name,
  // add relevant user properties from the signup flow
})
```

### Revenue tracking

On payment/conversion events, capture revenue with attribution:

```ts
posthog.capture('payment_completed', {
  revenue_amount: amountInCents,
  payment_provider: 'stripe', // or detected provider
  plan: planName,
  // first_touch_* properties are auto-included via register_once
})
```

### Framework-specific patterns

- **Next.js App Router**: Use `useEffect` for client-side captures, server actions for server-side
- **Next.js Pages Router**: Use `useEffect` or event handlers
- **React SPA**: Event handlers directly
- **Express/Node**: Capture in route handlers or middleware
- **Python/Django**: Capture in views or middleware

## Phase 7: Review + Commit

Before committing, show the user a summary:

> **Tracking plan generated.** Here's what I set up:
> - **X events** across **5 funnel stages** (Acquisition → Revenue)
> - **Y marketing channels** tracked (Organic Search, Twitter/X, ...)
> - **Z files** modified with tracking code
> - First-touch attribution enabled (persists across sessions)
> - Revenue attribution on payment events
>
> Review the plan at `tracking-plan.md` and the tracking code changes. Want me to commit?

Wait for user confirmation, then commit with message:

```
feat: add tracking plan and PostHog tracking (via /observe)
```

Include all modified files: `tracking-plan.md` + all files with tracking code changes.
