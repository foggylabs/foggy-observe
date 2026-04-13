# Telescope

Make your product observable in 15 minutes. AI skills that scan your codebase, generate a personalized tracking plan + tracking code, and commit it to your repo.

No analytics experience needed. Run `/telescope-explore` and follow the flow.

Works with Claude Code, Cursor, Codex, and any AI coding tool that supports custom skills.

## Install — 30 seconds

Requirements: an AI coding tool (Claude Code, Cursor, Codex), Git

Open your AI coding tool and paste this. The agent does the rest.

**Claude Code:**

> Install telescope: run `git clone --single-branch --depth 1 https://github.com/foggylabs/telescope.git ~/.claude/skills/telescope` then add a "Telescope" section to CLAUDE.md that lists the available skills: `/telescope-explore` — scan and understand your codebase, `/telescope-plan` — generate a tracking plan, `/telescope-review` — validate the plan, `/telescope-execute` — generate tracking code. Start with `/telescope-explore`.

**Cursor:**

> Install telescope: run `git clone --single-branch --depth 1 https://github.com/foggylabs/telescope.git .cursor/skills/telescope` then add a "Telescope" section to your project instructions that lists the available skills: `/telescope-explore`, `/telescope-plan`, `/telescope-review`, `/telescope-execute`. Start with `/telescope-explore`.

**Codex:**

> Install telescope: run `git clone --single-branch --depth 1 https://github.com/foggylabs/telescope.git ~/.codex/telescope` then add a "Telescope" section to AGENTS.md that lists the available skills: `/telescope-explore`, `/telescope-plan`, `/telescope-review`, `/telescope-execute`. Start with `/telescope-explore`.

**Other AI coding tools:**

Copy the `skills/` directory into your tool's skill directory — or paste skill contents as custom instructions.

## Skills

| Skill | What it does |
|-------|-------------|
| `/telescope-explore` | Scan and understand your codebase — stack, routes, auth, payments, existing analytics. Builds a complete mental model before generating anything. |
| `/telescope-plan` | Generate `tracking-plan.md` — personalized funnel metrics, marketing attribution, infrastructure health, runbook, and event properties. |
| `/telescope-review` | Data analyst review — validates the plan against actual code paths, flags issues by severity, fixes problems before any code is written. |
| `/telescope-execute` | Generate tracking code — PostHog setup, event capture, first-touch attribution, revenue tracking, user identification. Commits to repo. |

## The flow

```
/telescope-explore → /telescope-plan → /telescope-review → /telescope-execute
    understand          generate           validate            implement
```

Each skill auto-triggers the next. You can also run any skill independently.

The only user gate is after `/telescope-review` — you read the tracking plan and confirm before code is generated.

## Usage

```
/telescope-explore
```

That's it. The pipeline runs from there. Takes about 15 minutes for a typical project.

### Prerequisites

- An AI coding tool (Claude Code, Cursor, Codex, or similar)
- A [PostHog](https://posthog.com) account (free tier: 1M events/month) — the skill guides you through signup if you don't have one

## What it generates

**`tracking-plan.md`** — a personalized semantic layer:
- Funnel metrics (Acquisition → Activation → Engagement → Retention → Revenue)
- Marketing attribution (which channels bring users, revenue per visitor)
- Infrastructure health (error rates, latency, uptime)
- Runbook (actionable steps for every red flag)
- Event properties (schema for each tracked event)

**Tracking code** — placed in the correct files:
- `posthog.capture()` calls for each event
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
cd ~/.claude/skills/telescope && git pull
```

## Uninstall

```bash
rm -rf ~/.claude/skills/telescope
```

## Philosophy

The vibe-coder doesn't need a dashboard. They need a colleague who watches their app and tells them what matters.

The bottleneck isn't installing an SDK — PostHog's wizard does that in 90 seconds. The bottleneck is the **thinking work**: deciding what to track, why it matters, what normal looks like, and what to do when something breaks. That thinking work takes even an experienced PM about a week. Telescope does it in 15 minutes.

## License

MIT
