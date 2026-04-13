# Telescope

Generate a semantic layer for your product — so AI agents can understand your analytics data.

Telescope scans your codebase, generates a personalized `tracking-plan.md` (the semantic layer), and commits PostHog tracking code. The plan is structured context that AI agents use to query data, detect anomalies, and explain what's happening — not just documentation for humans.

Works with Claude Code, Cursor, Codex, and any AI coding tool that supports custom skills.

## Why a semantic layer?

AI agents querying raw PostHog events fail ~80% of the time. With a semantic layer — event descriptions, business context, normal ranges, red flag conditions — accuracy goes to ~100%.

Without it, an AI agent sees `user_signed_up` and has no idea what it means for the business. With `tracking-plan.md`, it knows: "this is an Acquisition metric, normal is 3-8% of visitors, below 2% for 3 days is a red flag, and here's what to investigate."

## Install — 30 seconds

Requirements: an AI coding tool (Claude Code, Cursor, Codex), Git

Open your AI coding tool and paste this. The agent does the rest.

**Claude Code:**

> Install telescope: run `git clone --single-branch --depth 1 https://github.com/foggylabs/telescope.git ~/.claude/skills/telescope && cd ~/.claude/skills/telescope && ./setup` then add a "Telescope" section to CLAUDE.md that lists the available skills: `/telescope-explore` — scan and understand your codebase, `/telescope-plan` — generate a tracking plan, `/telescope-review` — validate the plan, `/telescope-execute` — generate tracking code, `/telescope-add-feature-plan` — add tracking for a new feature. Start with `/telescope-explore` for initial setup or `/telescope-add-feature-plan` for existing projects.

**Cursor:**

> Install telescope: run `git clone --single-branch --depth 1 https://github.com/foggylabs/telescope.git ~/.telescope && cd ~/.telescope && ./setup` then add a "Telescope" section to your project instructions that lists the available skills: `/telescope-explore`, `/telescope-plan`, `/telescope-review`, `/telescope-execute`, `/telescope-add-feature-plan`. Start with `/telescope-explore`.

**Codex:**

> Install telescope: run `git clone --single-branch --depth 1 https://github.com/foggylabs/telescope.git ~/.telescope && cd ~/.telescope && ./setup` then add a "Telescope" section to AGENTS.md that lists the available skills: `/telescope-explore`, `/telescope-plan`, `/telescope-review`, `/telescope-execute`, `/telescope-add-feature-plan`. Start with `/telescope-explore`.

**Other AI coding tools:**

Copy the `telescope-*/SKILL.md` files into your tool's skill directory — or paste skill contents as custom instructions.

## Skills

| Skill | What it does |
|-------|-------------|
| `/telescope-explore` | Scan and understand your codebase — stack, routes, auth, payments, existing analytics. Builds a complete mental model before generating anything. |
| `/telescope-plan` | Generate `tracking-plan.md` — the semantic layer. Funnel metrics, marketing attribution, event properties — all structured for AI agent consumption. |
| `/telescope-review` | Data analyst review — validates the plan against actual code paths, checks AI agent readiness, flags issues by severity. |
| `/telescope-execute` | Generate tracking code — PostHog setup, event capture, first-touch attribution, revenue tracking, user identification. Commits to repo. |
| `/telescope-add-feature-plan` | Add tracking for a new feature to an existing plan. Point it at your code, it proposes events, updates the semantic layer, and generates tracking code. |

## The flow

**Initial setup:**
```
/telescope-explore → /telescope-plan → /telescope-review → /telescope-execute
    understand          generate           validate            implement
```

**New feature:**
```
/telescope-add-feature-plan
```

Each skill auto-triggers the next. The only user gate is after `/telescope-review` — you read the tracking plan and confirm before code is generated.

## What it generates

**`tracking-plan.md`** — a semantic layer that AI agents parse to understand your data:
- Funnel metrics (Acquisition → Activation → Engagement → Retention → Revenue) with normal ranges and red flags
- Marketing attribution (channels, revenue per visitor, first-touch tracking)
- Event properties (schema for each tracked event)

**Tracking code** — placed in the correct files:
- `posthog.capture()` calls for each event (client-side for UI, server-side for state changes)
- `posthog.identify()` in auth flows
- First-touch attribution via `register_once()`
- Revenue tracking on payment events

## Supported stacks

| Framework | Language |
|-----------|----------|
| Next.js (App Router & Pages Router) | TypeScript/JavaScript |
| React + Vite | TypeScript/JavaScript |
| Express / Node.js | TypeScript/JavaScript |
| Django / Flask | Python |
| Rails | Ruby |
| Go | Go |

Payment providers: Stripe, LemonSqueezy, Paddle, Polar

## Updating

```bash
cd ~/.claude/skills/telescope && git pull && ./setup
```

## Uninstall

```bash
rm -rf ~/.claude/skills/telescope ~/.claude/skills/telescope-{explore,plan,review,execute,add-feature-plan}
```

## Philosophy

AI agents don't need better models. They need better infrastructure to operate on. The missing layer is context — a semantic layer that describes what each metric means, what's normal, and what to do when something breaks.

The bottleneck isn't installing an SDK — PostHog's wizard does that in 90 seconds. The bottleneck is the **thinking work**: deciding what to track, why it matters, what normal looks like, and what to do when something breaks. That thinking work takes even an experienced PM about a week. Telescope does it in 15 minutes.

## License

MIT
