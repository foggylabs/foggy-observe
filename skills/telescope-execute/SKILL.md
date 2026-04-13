# /telescope-execute — Generate tracking code from the approved plan

You are an implementation agent. The tracking plan has been explored, generated, and reviewed. Your job is to turn it into working code.

**Prerequisites:** `tracking-plan.md` must exist in the repo root and must have been approved by the user (via `/observe-review`). If it doesn't exist, tell the user to run `/telescope-explore` first.

## Step 1: PostHog setup (if missing)

Check if PostHog SDK is installed in the project dependencies.

If **not installed**:

1. Tell the user: "You need a PostHog account (free — 1M events/month). Sign up at posthog.com if you haven't already."
2. Install the SDK for the detected stack:
   - **Next.js / React / Vue / Svelte**: `npm install posthog-js` (or yarn/pnpm/bun based on lockfile)
   - **Node.js backend**: `npm install posthog-node`
   - **Python**: `pip install posthog`
   - **Go**: `go get github.com/posthog/posthog-go`
   - **Ruby**: `gem install posthog-ruby`
3. Ask the user for their PostHog project API key
4. Add PostHog initialization code to the correct entry point:

**Next.js App Router** — `app/providers.tsx`:
```tsx
'use client'
import posthog from 'posthog-js'
import { PostHogProvider } from 'posthog-js/react'
import { useEffect } from 'react'

export function PHProvider({ children }: { children: React.ReactNode }) {
  useEffect(() => {
    posthog.init(process.env.NEXT_PUBLIC_POSTHOG_KEY!, {
      api_host: 'https://us.i.posthog.com',
      person_profiles: 'identified_only',
      capture_pageview: true,
      capture_pageleave: true,
    })
  }, [])
  return <PostHogProvider client={posthog}>{children}</PostHogProvider>
}
```

**Next.js Pages Router** — `pages/_app.tsx`:
```tsx
import posthog from 'posthog-js'
import { PostHogProvider } from 'posthog-js/react'
import { useEffect } from 'react'

export default function App({ Component, pageProps }) {
  useEffect(() => {
    posthog.init(process.env.NEXT_PUBLIC_POSTHOG_KEY!, {
      api_host: 'https://us.i.posthog.com',
      person_profiles: 'identified_only',
    })
  }, [])
  return <PostHogProvider client={posthog}><Component {...pageProps} /></PostHogProvider>
}
```

**React + Vite** — `src/main.tsx`:
```tsx
import posthog from 'posthog-js'
posthog.init(import.meta.env.VITE_POSTHOG_KEY, {
  api_host: 'https://us.i.posthog.com',
  person_profiles: 'identified_only',
})
```

**Express / Node.js** — `src/analytics.ts`:
```ts
import { PostHog } from 'posthog-node'
const posthog = new PostHog(process.env.POSTHOG_KEY!, { host: 'https://us.i.posthog.com' })
export default posthog
```

**Python / Django / Flask** — `analytics.py`:
```python
from posthog import Posthog
posthog = Posthog(os.environ['POSTHOG_KEY'], host='https://us.i.posthog.com')
```

Store the API key in an environment variable appropriate for the framework. Add the env var name to `.env.example` (create if needed).

If PostHog is **already installed**: skip this step, confirm the existing setup.

## Step 2: Generate event tracking code

Read `tracking-plan.md`. For each event in the funnel metrics, generate a `posthog.capture()` call and place it in the correct file.

```ts
posthog.capture('event_name', {
  property_1: value1,
  property_2: value2,
})
```

**Placement rules** (based on codebase exploration):
- Page view events → page components or route handlers
- Button/action events → click handlers or form submit handlers
- API events → API route handlers or server actions
- Payment events → payment webhook handlers or success callbacks
- Auth events → signup/login success handlers

**Framework-specific patterns:**
- **Next.js App Router**: `useEffect` for page-level, event handlers for actions, server actions for server-side
- **Next.js Pages Router**: `useEffect` or event handlers
- **React SPA**: Event handlers directly
- **Express/Node**: Capture in route handlers or middleware
- **Python/Django**: Capture in views or middleware

## Step 3: First-touch attribution

Add first-touch attribution capture to the app's entry point (same file as PostHog init or root layout):

```ts
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

This runs once on first visit and persists across sessions via `register_once`.

## Step 4: User identification

Add `posthog.identify()` in the auth flow — after signup or login:

```ts
posthog.identify(userId, {
  email: user.email,
  name: user.name,
})
```

Place this where the auth provider confirms the user is logged in.

## Step 5: Revenue tracking

On payment/conversion events, capture revenue with attribution:

```ts
posthog.capture('payment_completed', {
  revenue_amount: amountInCents,
  payment_provider: 'stripe',
  plan: planName,
  // first_touch_* properties are auto-included via register_once
})
```

## Step 6: Review and commit

Show the user a summary:

> **Tracking code generated.** Here's what I set up:
> - **X events** across **Y funnel stages**
> - **Z marketing channels** tracked (first-touch attribution)
> - **N files** modified
> - Revenue tracking on payment events
> - User identification in auth flow
>
> Want me to commit?

Wait for confirmation. Commit with message:

```
feat: add PostHog tracking code (via /telescope-execute)
```

Include all modified files. Do NOT include `tracking-plan.md` in this commit — it was already saved by `/telescope-plan`.
