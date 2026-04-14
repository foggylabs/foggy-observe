# /telescope-explore — Understand the codebase before generating a tracking plan

Your task is NOT to generate a plan or write any tracking code. Your job is to **fully understand** this codebase so that the next phase (`/telescope-plan`) can generate a high-quality, personalized tracking plan.

You are an exploration agent. You read, scan, and build a mental model. You do not implement anything.

## Why this matters

The tracking plan (`tracking-plan.md`) is not documentation for humans. It is a **semantic layer** — structured context that AI agents read to understand what PostHog events mean, what's normal, and what to do when something breaks. Without it, AI agents see raw numbers. With it, they can answer "why did activation drop this week?" with ~100% accuracy.

Your exploration directly determines the quality of that semantic layer. If you don't understand the product deeply, the plan will be generic and useless to AI agents.

## Key principles

- **PostHog is the analytics provider.** Always. Do not ask the user which provider to use. If another provider is installed, note it but plan for PostHog.
- **Track the full funnel.** Website/marketing pages (acquisition), app frontend (activation/engagement), backend API (core actions), payments (revenue). Do not ask "should I track the website?" or "should I track backend events?" — the answer is always yes.
- **Capture events where they're authoritative.** Client-side for UI interactions (page views, clicks, form fills) that the backend never sees. Server-side for state changes (user created, payment completed, resource deleted) where the backend is the source of truth. This is standard analytics best practice — don't ask, just follow it.
- **Read the actual code, not just file names.** Understanding what the product does requires reading auth flows, core feature handlers, payment logic, and onboarding flows — not just listing routes.
- **Only ask about genuine ambiguities.** Things you can't determine from the code — e.g., "I found two signup flows, which is the primary one?" Never ask about technical decisions that you should make yourself.

## What to explore

### 1. Stack detection

Read dependency and config files to identify the tech stack:

- **Language & framework**: Read `package.json`, `requirements.txt`, `go.mod`, `Cargo.toml`, `Gemfile`, `pubspec.yaml`, `composer.json`
  - Identify: Next.js (App Router vs Pages Router), React, Vue, Svelte, Express, Django, Flask, Rails, Go, Rust, etc.
- **Deployment**: Check for `vercel.json`, `netlify.toml`, `fly.toml`, `Dockerfile`, `render.yaml`, `app.yaml`, `railway.json`
- **Package manager**: Detect from lockfile — `bun.lock`, `pnpm-lock.yaml`, `yarn.lock`, `package-lock.json`, `Pipfile.lock`, `poetry.lock`
- **Monorepo structure**: If multiple apps exist (frontend + backend, marketing site + app), identify all of them. All will be tracked.

### 2. PostHog status

Check if PostHog is already installed:

- Grep for `posthog-js`, `posthog-node`, `posthog-python`, `posthog-go`, `posthog-ruby` in dependencies
- Grep for `posthog.capture`, `posthog.identify`, `posthog.init` in code
- Check for `tracking-plan.md` in the repo root

If PostHog is installed, note what's already tracked. If another analytics provider is installed (Mixpanel, Amplitude, GA), note it — we'll replace or supplement it with PostHog.

If PostHog is NOT installed, note it. The execute phase will handle installation.

### 3. Application surface area

Understand what the product does — read the code, not just the file tree:

- **All frontends**: marketing site, app, admin panel — all of them
  - Routes/pages, what each one does
  - Key UI components and user interactions
- **All API endpoints**: what they do, what data they handle
- **Database models**: the data model tells you what the product is about
- **Background jobs/workers**: automated processes, cron jobs, queue consumers

### 4. User lifecycle — read the actual code

Map the complete user journey by reading the implementation:

- **Auth flows**: Read the signup, login, OAuth handler code. Understand what happens step by step.
- **Onboarding**: Read the onboarding flow. What does a new user see? What steps do they complete?
- **Core actions**: Read the main feature code. What are the key things users do? What makes this product valuable?
- **Collaboration**: Can users invite others? Share things? Work together?
- **Settings/configuration**: What can users customize?

### 5. Revenue & payments

Detect how the product makes money:

- **Payment providers**: Grep for Stripe, LemonSqueezy, Paddle, Polar
- **Pricing model**: Look for pricing pages, plan definitions, feature gates, quota systems
- **Webhooks**: Find payment webhook handlers
- **If pre-revenue**: Note it. The plan will omit the Revenue funnel stage.

### 6. Marketing & acquisition

Understand the full acquisition surface:

- **Marketing site**: landing page, pricing page, features page, blog — all trackable
- **SEO**: meta tags, sitemap, robots.txt
- **UTM handling**: any existing UTM parameter processing
- **Signup sources**: where does the signup form live? Can users sign up from multiple places?

## How to explore

1. Start broad — read the README, package.json, and top-level structure to get the big picture
2. Go deep — read the actual code for auth, payments, core features, and onboarding. Read function bodies, not just names.
3. Build the mental model — understand the user journey from "lands on website" to "becomes a paying customer"
4. Note genuine ambiguities — things the code doesn't make clear

## Output: Exploration summary

Present your findings as a structured summary:

```
## Exploration Summary

### Stack
- Framework: [detected]
- Language: [detected]
- Deployment: [detected]
- Package manager: [detected]
- Apps: [list all frontends + backends found]

### PostHog Status
- [Installed / Not installed]
- Events currently tracked: [list or "none"]
- Other analytics: [list any other providers found, or "none"]

### Product Surface Area
- Frontend pages: [count and key ones — marketing + app]
- API endpoints: [count and key ones]
- Database models: [key entities and what they represent]
- Background jobs: [if any]

### User Journey
- Acquisition: [how users find the product — marketing site, pages]
- Signup: [how signup works — email/OAuth, what happens after]
- Onboarding: [what new users see and do]
- Core actions: [the main things users do that make the product valuable]
- Collaboration: [invite, share, team features — if any]

### Revenue
- Payment provider: [detected or "pre-revenue"]
- Model: [subscription / one-time / freemium / quota / pre-revenue]
- Pricing: [tiers if detected]

### Tracking surface (areas, NOT specific events)
- Marketing site: [yes/no — list key pages that exist, e.g., `/`, `/pricing`, `/integrations`]
- App frontend: [yes/no — list user-action areas, e.g., "auth flow", "onboarding", "core feature X", "settings"]
- Backend API: [yes/no — list state-change domains, e.g., "user lifecycle", "core feature X", "billing", "team/invites"]
- Payments: [yes/no/pre-revenue — note what payment integration exists, if any]
```

**Critical: do NOT enumerate specific event names in this section.** That is `/telescope-plan`'s job. Explore identifies areas/domains; plan picks the minimal essential events (10-15 max) after asking the user the activation question. If you list specific event names here (e.g., `user_signed_up`, `item_created`, `payment_completed`), you anchor the plan to your guesses before scope discipline can be applied — and you may pre-commit to events PostHog autocaptures (page views, CTA clicks, form submits) that should never be custom events.

Do NOT include a "Questions & Ambiguities" section unless there are genuine ambiguities that you cannot resolve by reading the code. If you have ambiguities, they should be specific: "I found two signup flows in `/auth/signup.ts` and `/api/register.ts` — which is the primary one?" Never ask open-ended questions like "should I track the backend?"

## After exploration

Present the summary. If it's accurate, tell the user:

> "I have a clear picture of your codebase. Ready to generate your tracking plan. Running `/telescope-plan`."

Then invoke `/telescope-plan`.
