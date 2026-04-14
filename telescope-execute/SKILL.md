---
name: telescope-execute
description: Generate PostHog tracking code from the approved tracking-plan.md. Installs the SDK, wires identify / group / reset, implements each custom event at the correct client or server location, and verifies every event in the plan exists in the generated code. Final step in the telescope pipeline, runs after /telescope-review. Use when the user asks to implement tracking or generate tracking code.
---

# /telescope-execute — Generate tracking code from the approved plan

You are an implementation agent. Your job is to turn the approved `tracking-plan.md` into working PostHog code.

**Only implement what the plan specifies.** Do NOT add tracking for things PostHog handles automatically.

**Prerequisites:** `tracking-plan.md` must exist and be approved via `/telescope-review`.

## What PostHog already does — DO NOT implement

PostHog autocaptures these with zero code (after SDK init):
- `$pageview` on every page/route change (with URL, referrer, UTMs, scroll depth)
- `$pageleave` with scroll depth
- `$autocapture` — clicks on buttons, links, forms, inputs (with element text, CSS selector)
- UTM parameters, referrer, device info — all auto-captured as event properties
- `$initial_referrer`, `$initial_utm_source`, etc. — auto-set as person properties
- Session replay, heatmaps, web analytics — built-in, no code

**Do NOT write `posthog.capture()` for page views, button clicks, form submissions, or marketing attribution.** PostHog handles all of this.

## Step 1: PostHog SDK setup (if missing)

Check if PostHog SDK is installed. If not:

1. Tell user to sign up at posthog.com (free — 1M events/month)
2. Install SDK for the detected stack (posthog-js for frontend, posthog-python/posthog-node for backend)
3. Ask user for their PostHog **project token** (find it at Settings > General > "Project token & ID" — starts with `phc_`)
4. Add initialization code to the correct entry point

**Critical init options** (from the plan's PostHog Configuration section):
- `person_profiles: 'identified_only'`
- `capture_pageview: 'history_change'` — for SPAs
- `cross_subdomain_cookie: true` — if product spans subdomains
- `api_host: 'https://us.i.posthog.com'`

### Where to put the project token

PostHog project tokens (`phc_*`) are **public by design**. They're rate-limited per project and meant to ship in client bundles — this is how PostHog's own copy-paste install snippet works. They are NOT secrets.

Place the token differently depending on where the SDK runs:

**Backend (posthog-python, posthog-node):** read from `POSTHOG_API_KEY` env var at runtime. Standard pattern, add to `.env.example`, load from the server's env. No build step involved.

**Frontend (posthog-js in a build-compiled framework — Vite, Next.js, Remix, Astro, CRA, SvelteKit, Nuxt, etc.):** the token is inlined at **build time**, not runtime. This is the #1 way frontend tracking silently fails in production: the Dockerfile (or Cloudflare Workers / Vercel / Netlify build) runs `npm run build` in an environment that doesn't have `VITE_POSTHOG_KEY` / `NEXT_PUBLIC_POSTHOG_KEY` set, the bundle ships with an empty key, and every `posthog.capture` no-ops.

To avoid this, **hardcode the project token in the frontend init file as a fallback**, with an env-var override for local dev:

```ts
// analytics.ts — PostHog project tokens are public by design, safe to commit.
// Env var override lets local dev disable analytics (set to '') or swap projects.
const DEFAULT_KEY = 'phc_...';  // paste the user's token here
const ENV_KEY = import.meta.env.VITE_POSTHOG_KEY as string | undefined;  // or process.env.NEXT_PUBLIC_POSTHOG_KEY for Next.js
const KEY = ENV_KEY ?? DEFAULT_KEY;
```

Do this for every frontend app in the repo — marketing site and console/app frontend are separate builds and each needs its own init. Ask the user for the token once, paste it into all frontend init files.

Do NOT rely on runtime `.env` files (e.g. `.env.prod` mounted into a container) to set frontend build-time env vars — by the time the container starts, the bundle is already built and the key is already baked in (or not).

## Step 2: User identification

Implement BEFORE custom events. Without this, all events are anonymous.

**Client-side** — call on every authenticated page load and after login/signup:

```ts
posthog.identify(userId, {
  // $set — updated each time
  email: user.email,
  name: user.name,
  // add other $set properties from the plan
}, {
  // $set_once — immutable, only set first time
  signup_method: 'google',
  signup_date: new Date().toISOString(),
  // add other $set_once from the plan
})
```

Call `posthog.reset()` on logout.

**Server-side** — every `capture()` call needs `distinct_id` from the authenticated session.

## Step 3: Group analytics (for products with shared spaces)

Applies to any product with shared spaces between users — teams/workspaces in SaaS, stores in marketplaces, channels in community apps, organizations in B2B. Skip if the plan doesn't define group types.

If the plan defines group types:

```ts
// Client-side — on login and group/project switch
posthog.group('project', projectId, {
  name: projectName,
  // group properties from the plan
})
```

```python
# Server-side — include in capture calls
posthog.capture(
    distinct_id=user_id,
    event='event_name',
    properties={ ... },
    groups={ 'project': project_id }
)
```

## Step 4: Custom event tracking code

Read the plan's Funnel Events section. For each event marked `client` or `server`, generate a `posthog.capture()` call. **Skip all events marked `auto`** — PostHog handles those.

**Client-side:**
```ts
posthog.capture('event_name', { property: value })
```

**Server-side:**
```python
posthog.capture(
    distinct_id=request.user.id,
    event='event_name',
    properties={ 'property': value }
)
```

Place in the exact code paths the plan references. Match existing code style.

## Common emission bugs — check these before committing

These come from real dogfood incidents. Verify every generated capture call against each pattern before moving to verification.

### 1. Timezone mismatch on `duration_seconds` and other time-delta properties

Any property computed as `(datetime.now(...) - db_timestamp).total_seconds()` breaks if one side is naive and the other is aware. Python raises `TypeError: can't subtract offset-naive and offset-aware datetimes` — and because `posthog.capture()` is wrapped in a `try/except`, the error gets swallowed and **no event fires at all**. The user thinks tracking works. It doesn't.

Real incident: `investigation_completed` never reached PostHog for a week because `Message.created_at` was SQLAlchemy `DateTime` (naive UTC) but the emitter used `datetime.now(UTC)` (aware). The defensive guard hid it.

Before generating a time-delta property, inspect the source column's tz handling:

- **SQLAlchemy**: `DateTime` (default) = naive UTC; `DateTime(timezone=True)` = aware.
- **Raw SQL**: `TIMESTAMP` = naive; `TIMESTAMPTZ` = aware.
- **Django**: `DateTimeField` under `USE_TZ=True` = aware; otherwise naive.
- **Node ORMs (Prisma / Drizzle / TypeORM)**: JS `Date` is always tz-aware, but the DB column can still be naive — check the migration and the driver's conversion.

Generate tz-matched code. Defensive normalization works regardless of source:

```python
now = datetime.now(UTC)
if created_at.tzinfo is None:
    now = now.replace(tzinfo=None)
duration_seconds = max(0.0, (now - created_at).total_seconds())
```

### 2. Never bare-`except: pass` around `posthog.capture()`

Analytics failures must not break the product — but **silent** failures are worse. They hide real bugs (like the TZ mismatch above) for weeks while the user believes tracking is healthy.

**Wrong** — generates silent product data holes:

```python
try:
    posthog.capture(...)
except Exception:
    pass
```

**Right** — preserves the "analytics never crashes prod" property AND surfaces emission bugs on day one:

```python
try:
    posthog.capture(...)
except Exception:
    logger.warning("posthog capture failed for %s", event_name, exc_info=True)
```

Node / TypeScript:

```ts
try {
  posthog.capture(...)
} catch (err) {
  logger.warn({ err, event: eventName }, "posthog capture failed")
}
```

If the project has no obvious logger, use stdlib `logging.warning` or `console.warn` — anything but `pass`.

### 3. After deploy, ask the user to watch logs for 24h

Tell the user in Step 7's commit message or a follow-up:

> For the next 24h, grep your server logs for `posthog capture failed`. Zero occurrences + non-zero events in PostHog's Activity feed = tracking healthy. Any warnings = emission bug — fix immediately before the gap ossifies into "we assumed this event was firing."

This is the cheap bridge until a dashboard surfaces missing-event signals visually.

## Step 5: PostHog Actions — skip by default

The plan may define PostHog Actions (UI groupings of autocaptured events). **Skip them in the execute phase.** The raw autocaptured data is already tracked and queryable — Actions are just dashboard shortcuts users discover later in PostHog's UI.

Do NOT mention Actions to the user. Do NOT ask them to create anything. Do NOT present a list of manual steps. The user didn't ask about Actions, doesn't know what they are, and doesn't need to make a decision about them.

If the user later wants Actions, they can create them from the tracking plan manually in PostHog — it's a 30-second task they'll understand better once they're using the product.

## Step 6: Verify implementation matches the plan

Before committing, verify every custom event in the plan is actually implemented. Go through the plan's Funnel Events section and check each `client` or `server` event:

1. **Read the generated code** — find the actual `posthog.capture()` call for each event
2. **Event name matches exactly** — the code must use the exact snake_case name from the plan
3. **All properties are captured** — every property listed in the Event Properties section is present in the capture call
4. **`distinct_id` is correct** — server-side events use the source documented in the plan
5. **`$groups` is included** — project-scoped server events include the group parameter
6. **Placement is correct** — the capture call is in the right file/function (state changes server-side, UI interactions client-side)
7. **`posthog.identify()` is called** — with the correct person properties (`$set` and `$set_once`)
8. **`posthog.group()` is called** — on the correct triggers (page load, project switch)
9. **`posthog.reset()` is called** — in the logout handler
10. **PostHog init config matches** — compare the actual `posthog.init()` call against the plan's PostHog Configuration section

Present a verification checklist:

```
## Implementation Verification

| Event | In Plan | In Code | File | Status |
|-------|---------|---------|------|--------|
| user_signed_up | server | server | routes/auth.py:145 | OK |
| email_verified | server | server | routes/auth.py:210 | OK |
| ... | ... | ... | ... | ... |

### Missing
- [any events in plan but not in code]

### Extra
- [any capture calls in code but not in plan]

### Config
- [ ] posthog.init() matches plan config
- [ ] posthog.identify() called on login/signup/page load
- [ ] posthog.group() called on project load/switch
- [ ] posthog.reset() called on logout
```

If anything is missing or wrong, fix it before committing.

## Step 7: Commit

Show the user the verification results and:

> **Tracking code generated and verified.** All X events from the plan are implemented.
> - **N files** modified
> - PostHog SDK initialized on [apps]
> - Identity, groups, and reset configured
>
> Want me to commit?

Commit with: `feat: add PostHog tracking code (via /telescope-execute)`

Do NOT include `tracking-plan.md` in this commit.
