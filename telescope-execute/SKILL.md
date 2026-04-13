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

Store API key in an environment variable. Add to `.env.example`.

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

## Step 3: Group analytics (for B2B / multi-tenant)

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

## Step 5: PostHog Actions (document for manual setup)

If the plan defines PostHog Actions, these are created in the PostHog UI — not in code. Add a comment or note in the tracking plan indicating which Actions need to be created manually:

```markdown
<!-- PostHog Actions to create in UI:
- "Signup CTA clicked" — $autocapture where button text contains "Request access" OR "Get started"
- "Pricing page viewed" — $pageview where URL contains /pricing
-->
```

## Step 6: Review and commit

Show the user:

> **Tracking code generated.** Here's what I set up:
> - PostHog SDK initialized with [config]
> - `posthog.identify()` on [where]
> - `posthog.group()` for [group type] (if applicable)
> - **X custom events** (only business logic — PostHog autocapture handles the rest)
> - **Y PostHog Actions** to create in the UI
> - **N files** modified
>
> Want me to commit?

Commit with: `feat: add PostHog tracking code (via /telescope-execute)`

Do NOT include `tracking-plan.md` in this commit.
