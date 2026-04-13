# /telescope-explore — Understand the codebase before generating a tracking plan

Your task is NOT to generate a plan or write any tracking code. Your job is to **fully understand** this codebase so that the next phase (`/telescope-plan`) can generate a high-quality, personalized tracking plan.

You are an exploration agent. You read, scan, ask questions, and build a mental model. You do not implement anything.

## What to explore

### 1. Stack detection

Read dependency and config files to identify the tech stack:

- **Language & framework**: Read `package.json`, `requirements.txt`, `go.mod`, `Cargo.toml`, `Gemfile`, `pubspec.yaml`, `composer.json`
  - Identify: Next.js (App Router vs Pages Router), React, Vue, Svelte, Express, Django, Flask, Rails, Go, Rust, etc.
- **Deployment**: Check for `vercel.json`, `netlify.toml`, `fly.toml`, `Dockerfile`, `render.yaml`, `app.yaml`, `railway.json`
- **Package manager**: Detect from lockfile — `bun.lock`, `pnpm-lock.yaml`, `yarn.lock`, `package-lock.json`, `Pipfile.lock`, `poetry.lock`

### 2. Existing analytics & observability

Determine if any analytics is already in place:

- **PostHog**: Grep for `posthog-js`, `posthog-node`, `posthog-python`, `posthog-go`, `posthog-ruby` in dependencies. Grep for `posthog.capture`, `posthog.identify`, `posthog.init` in code.
- **Other analytics**: Grep for `mixpanel`, `amplitude`, `analytics.track`, `gtag`, `plausible`, `umami`, `segment`
- **Error tracking**: Grep for `sentry`, `bugsnag`, `rollbar`, `logrocket`
- **Existing tracking plan**: Check for `tracking-plan.md` in the repo root

If analytics exists, understand what's already tracked — read every `capture()` / `track()` call and note the event names and properties.

### 3. Application surface area

Understand what the product does by scanning its structure:

- **Routes & pages**: Glob for route/page files
  - Next.js: `app/**/page.tsx`, `pages/**/*.tsx`
  - React Router: grep for `<Route`, `createBrowserRouter`
  - Express: grep for `app.get`, `app.post`, `router.get`, `router.post`
  - Django: grep for `urlpatterns`, `path(`
  - Rails: read `config/routes.rb`
  - Vue: grep for router definitions
- **API endpoints**: Find all API routes and their purpose
- **Database models**: Understand the data model (Prisma schema, Django models, SQL migrations, etc.)

### 4. User lifecycle

Map how users interact with the product:

- **Auth patterns**: Grep for signup, login, registration, OAuth flows
  - Keywords: `signup`, `sign-up`, `register`, `login`, `sign-in`, `signIn`, `signUp`
  - Providers: `Clerk`, `Auth0`, `Supabase`, `NextAuth`, `auth.js`, `passport`, `bcrypt`, `jwt`
  - OAuth: `google`, `github`, `oauth`, `callback`
- **Onboarding**: Look for onboarding flows, welcome pages, setup wizards, first-run experiences
- **Core actions**: What are the main things users do? (create, edit, share, publish, invite, etc.)

### 5. Revenue & payments

Detect how the product makes money:

- **Payment providers**: Grep for Stripe, LemonSqueezy, Paddle, Polar
  - Keywords: `stripe`, `checkout`, `subscription`, `price_`, `createCheckoutSession`, `lemonsqueezy`, `paddle`, `polar`
- **Pricing model**: Look for pricing pages, plan definitions, feature gates
- **Webhooks**: Find payment webhook handlers (Stripe webhooks, etc.)

### 6. Marketing & acquisition

Understand how users find the product:

- **Landing pages**: Look for landing, pricing, features, about pages
  - Grep for: `landing`, `pricing`, `features`, `hero`, `cta`, `waitlist`
  - Check routes: `/`, `/pricing`, `/features`, `/about`, `/blog`
- **SEO**: Check for meta tags, sitemap, robots.txt
- **UTM handling**: Grep for `utm_source`, `utm_medium`, `utm_campaign`

## How to explore

1. Start broad — read the README, package.json, and top-level structure to get the big picture
2. Go deep — read the actual code for auth, payments, and core features (not just filenames)
3. Note everything — build a complete mental model of the product
4. Don't assume — if something is ambiguous, note it as a question

## Output: Exploration summary

When you've finished exploring, present your findings to the user as a structured summary:

```
## Exploration Summary

### Stack
- Framework: [detected]
- Language: [detected]
- Deployment: [detected]
- Package manager: [detected]

### Analytics Status
- [Installed / Not installed / Partially installed]
- Provider: [PostHog / Mixpanel / None / etc.]
- Events currently tracked: [list or "none"]

### Product Surface Area
- Pages/routes: [count and key ones]
- API endpoints: [count and key ones]
- Database models: [key entities]

### User Lifecycle
- Auth: [method — email/OAuth/magic link, provider]
- Onboarding: [exists / doesn't exist, description]
- Core actions: [list the main things users do]

### Revenue
- Payment provider: [Stripe / LemonSqueezy / None / etc.]
- Model: [subscription / one-time / freemium / pre-revenue]
- Pricing tiers: [if detected]

### Marketing
- Landing page: [yes/no]
- Marketing pages: [list]
- UTM tracking: [yes/no]

### Questions & Ambiguities
- [List anything unclear that you need the user to clarify]
```

## After exploration

Ask the user if the summary is accurate and if they have anything to add or correct. Go back and forth until there are no remaining questions.

When the exploration is complete, tell the user:

> "I have a clear picture of your codebase. Ready to generate your tracking plan. Running `/telescope-plan`."

Then invoke `/telescope-plan`.
