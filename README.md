# Foggy Observe

**Set up analytics for your product in 15 minutes.** One AI skill that scans your codebase, understands your business, and generates a personalized tracking plan + PostHog tracking code — committed directly to your repo.

No analytics experience needed. No dashboard configuration. Just run `/observe` and answer 5 questions.

## What it does

`/observe` is a Claude Code skill that:

1. **Detects your stack** — framework, language, deployment, existing analytics
2. **Sets up PostHog** — installs the SDK, adds initialization code (or detects existing setup)
3. **Scans your codebase** — finds routes, auth flows, payment integrations, marketing pages
4. **Asks 5 business questions** — product name, activation moment, revenue model, traffic sources, biggest unknown
5. **Generates `tracking-plan.md`** — a personalized semantic layer with funnel metrics, marketing attribution, infrastructure health, runbook, and event properties
6. **Generates tracking code** — `posthog.capture()` calls, first-touch attribution, revenue tracking, user identification — placed in the correct files
7. **Commits everything** — tracking plan + code, ready to deploy

### The tracking plan

The generated `tracking-plan.md` is both human-readable and machine-parseable. It covers:

- **Funnel metrics** — Acquisition, Activation, Engagement, Retention, Revenue (with normal ranges and red flags)
- **Marketing attribution** — which channels bring users, revenue per visitor, first-touch/last-touch tracking
- **Infrastructure health** — error rates, latency, uptime
- **Runbook** — actionable steps for every red flag
- **Event properties** — schema for each tracked event

## Install

### Claude Code (30 seconds)

```bash
git clone --single-branch --depth 1 https://github.com/foggylabs/foggy-observe.git ~/.claude/skills/foggy-observe
```

That's it. The skill is now available as `/observe` in any Claude Code session.

### Agent setup instructions

If you're an AI agent setting up Foggy Observe for a user, follow these steps:

1. Clone the skill repo into the user's Claude Code skills directory:
   ```bash
   git clone --single-branch --depth 1 https://github.com/foggylabs/foggy-observe.git ~/.claude/skills/foggy-observe
   ```

2. Verify the skill file exists:
   ```bash
   cat ~/.claude/skills/foggy-observe/.claude/skills/observe.md | head -5
   ```
   You should see: `# /observe — Set up PostHog tracking + generate a tracking plan`

3. Add the following to the project's `CLAUDE.md` (create it if it doesn't exist):
   ```markdown
   ## Analytics

   This project uses Foggy Observe for analytics. Run `/observe` to set up or update tracking.
   ```

4. Done. The user can now run `/observe` in their project to set up analytics.

### Cursor

Copy `.claude/skills/observe.md` from this repo into your project's `.cursor/skills/` directory (or equivalent skill location for your Cursor setup).

### Manual install

If you prefer not to clone, copy the raw skill file directly:

```bash
mkdir -p .claude/skills
curl -o .claude/skills/observe.md https://raw.githubusercontent.com/foggylabs/foggy-observe/main/.claude/skills/observe.md
```

## Usage

In any project with Claude Code:

```
/observe
```

The skill walks you through the entire flow interactively. It takes about 15 minutes for a typical project.

### Prerequisites

- [Claude Code](https://claude.ai/code) (latest)
- A [PostHog](https://posthog.com) account (free tier: 1M events/month) — the skill will guide you through signup if you don't have one

## How it works

```
Your project                          What /observe generates
┌─────────────┐                       ┌──────────────────────────┐
│ package.json │─── stack detection ──>│ tracking-plan.md         │
│ routes/      │                      │  - funnel metrics        │
│ auth/        │─── codebase scan ───>│  - marketing attribution │
│ payments/    │                      │  - infrastructure health │
│              │                      │  - runbook               │
│              │<── tracking code ────│  - event properties      │
│              │    posthog.capture()  │                          │
│              │    posthog.identify() │ PostHog SDK + init code  │
│              │    register_once()    │ First-touch attribution  │
│              │    revenue tracking   │ Revenue per channel      │
└─────────────┘                       └──────────────────────────┘
```

## Supported stacks

| Framework | Language | PostHog SDK |
|-----------|----------|-------------|
| Next.js (App Router & Pages Router) | TypeScript/JavaScript | posthog-js |
| React + Vite | TypeScript/JavaScript | posthog-js |
| Express / Node.js | TypeScript/JavaScript | posthog-node |
| Django / Flask | Python | posthog-python |
| Rails | Ruby | posthog-ruby |
| Go | Go | posthog-go |

Payment providers: Stripe, LemonSqueezy, Paddle, Polar

## Roadmap

| Skill | Description | Status |
|-------|-------------|--------|
| `/observe` | Full flow: stack detection, PostHog setup, tracking plan, tracking code | Done |
| `/observe-improve` | Audit existing tracking vs plan, fill gaps | Planned |
| `/observe-dashboard` | Generate a dashboard from the plan + PostHog API | Planned |
| `/observe-diagnose` | AI data analyst: reads plan + queries PostHog | Planned |
| `/observe-infra` | OTel + Grafana infrastructure observability | Planned |
| `/observe-alert` | Set up alerts from plan thresholds | Planned |

## Updating

```bash
cd ~/.claude/skills/foggy-observe && git pull
```

## Uninstall

```bash
rm -rf ~/.claude/skills/foggy-observe
```

## Philosophy

The vibe-coder doesn't need a dashboard. They need a colleague who watches their app and tells them what matters.

The bottleneck isn't installing an SDK — PostHog's wizard does that in 90 seconds. The bottleneck is the **thinking work**: deciding what to track, why it matters, what normal looks like, and what to do when something breaks. That thinking work takes even an experienced PM about a week. `/observe` does it in 15 minutes.

## License

MIT
