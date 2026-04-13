# Foggy Observe

Make your product observable in 15 minutes. AI skill that scans your codebase, generates a personalized tracking plan + tracking code, and commits it to your repo.

No analytics experience needed. Run `/observe`, deploy, see what your users actually do.

## Install — 30 seconds

Requirements: an AI coding tool (Claude Code, Cursor, Codex), Git

Open your AI coding tool and paste this. The agent does the rest.

**Claude Code:**

> Install foggy-observe: run `git clone --single-branch --depth 1 https://github.com/foggylabs/foggy-observe.git ~/.claude/skills/foggy-observe` then add a "Foggy Observe" section to CLAUDE.md that lists the available skill: `/observe` — set up analytics and generate a tracking plan for your product.

**Cursor:**

> Install foggy-observe: run `git clone --single-branch --depth 1 https://github.com/foggylabs/foggy-observe.git .cursor/skills/foggy-observe` then add a "Foggy Observe" section to your project instructions that lists the available skill: `/observe`.

**Codex:**

> Install foggy-observe: run `git clone --single-branch --depth 1 https://github.com/foggylabs/foggy-observe.git ~/.codex/foggy-observe` then add a "Foggy Observe" section to AGENTS.md that lists the available skill: `/observe`.

**Other AI coding tools:**

Copy `skills/observe/SKILL.md` into your tool's skill directory — or paste its contents as custom instructions.

## Usage

```
/observe
```

The skill walks you through the entire flow. Takes about 15 minutes for a typical project.

## What it does

`/observe` scans your codebase, understands your product, and generates two things:

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

## Roadmap

| Skill | What it does | Status |
|-------|-------------|--------|
| `/observe` | Full flow: scan codebase, generate tracking plan + code | Done |
| `/observe-improve` | Audit existing tracking vs plan, fill gaps | Planned |
| `/observe-diagnose` | AI data analyst: "why did activation drop?" | Planned |

## Updating

```bash
cd ~/.claude/skills/foggy-observe && git pull
```

## Uninstall

```bash
rm -rf ~/.claude/skills/foggy-observe
```

## License

MIT
