# popguia

[![popguia](guia/assets/banner-en.jpg)](https://inematds.github.io/popguia/guia/en/)

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

## What it is

popguia is an open source project for creating popups that talk with the visitor: instead of asking for the email right away, it asks two or three questions, teaches something useful and points to the right product, course or content. It is for anyone with a site (HTML, Next.js, Astro, WordPress) who uses a coding agent such as Claude Code, Codex or Cursor: the agent installs it, writes the journeys and tracks the results. It is free, MIT licensed, with no subscription and no server of ours. Today the repository has the research and the plan; the code is still to be written.

## 📖 User guide

Full guide (landing + step by step): **https://inematds.github.io/popguia/guia/en/**

**Smart popups and journeys that the site owner's coding agent installs and configures by itself.**

A site owner (HTML, Next.js, WordPress…) tells their Claude Code / Codex / Cursor: *"install popguia and create a quiz that recommends the right course"*. The agent installs the snippet, writes the journeys as JSON files in the repo, generates variants, validates accessibility/LGPD/SEO and publishes. Then the system itself measures with a control group, runs A/B and evolves into a bandit — with no mandatory backend of ours and no per-pageview charge.

Inspiration: Alia Popups (interactive journeys, zero-party, Prism AI). Difference: open source (MIT), multi-platform, state in files, agent-driven install, guardrails by default.

## What it delivers

- **Journeys**: diagnostic/educational quiz, Learn & Earn, micro-commitments, 2-step newsletter, catalog-based recommendation.
- **Smart triggering**: time, scroll, exit intent (desktop/mobile), inactivity, click, adaptive; frequency caps; global cooldown.
- **Context-based personalization**: page, UTM, device, new/returning, previous answers (zero-party in `localStorage`).
- **Optimization**: permanent holdout → A/B (SRM, sequential) → Thompson per segment → contextual.
- **AI-driven install**: `npx popguia init`, local MCP (`npx -y @popguia/mcp`), `SKILL.md`, `setup.md`, `llms.txt`, copyable prompt.
- **AI generation at no cost to us**: the MCP returns brief + schema; the client's agent writes; optional provider configured by the client (Ollama included).
- **Built-in compliance**: accessible `<dialog>`, CLS 0, ≤ 20 KB gz, consent modes, no dark patterns, no random roulette by default.

## Status

**Research and planning completed on 10/10/2026. No product code written.** Next step: Nei's decisions (`PLANO.md §3`) and Phase 0.

## Documents

| File | Contents |
|---|---|
| [docs/00-referencia-alia.md](docs/00-referencia-alia.md) (in Portuguese) | Initial input brought by Nei |
| [docs/01-pesquisa-mercado.md](docs/01-pesquisa-mercado.md) (in Portuguese) | Alia in depth (entities, triggers, re-display, Prism) + 9 competitors (table) + complaints |
| [docs/02-open-source-github.md](docs/02-open-source-github.md) (in Portuguese) | 37 repos with stars/license/last push; verdict: build from scratch with references |
| [docs/03-instalacao-pela-ia.md](docs/03-instalacao-pela-ia.md) (in Portuguese) | State of the art (shadcn, Sentry, PostHog, Supabase, Stripe, Vercel, Clerk…) and CLI + MCP + skill + llms.txt design |
| [docs/04-arquitetura.md](docs/04-arquitetura.md) (in Portuguese) | Components, mermaid diagram, data model, stack, target weight |
| [docs/05-algoritmos-e-metricas.md](docs/05-algoritmos-e-metricas.md) (in Portuguese) | Triggering, A/B with holdout, Thompson/LinUCB, minimum volume, metrics, guardrails |
| [docs/06-conformidade.md](docs/06-conformidade.md) (in Portuguese) | LGPD/GDPR, WCAG 2.2, Google interstitials, CWV, dark patterns, roulette (Lei 5.768/71); 25-item checklist |
| [PLANO.md](PLANO.md) (in Portuguese) | Phases 0–5 with `command → output` criteria, demo sites, closed decisions and 7 open ones |
| [CLAUDE.md](CLAUDE.md) (in Portuguese) | Repo instructions for agents |

## Demo

There is no pilot on a real site: popguia is for other people to install on their own sites. Validation uses 3 fictional sites in `exemplos/` (school in HTML, store in Next.js, blog in Astro), published on GitHub Pages. Details in `PLANO.md §2`.

## License

MIT, free, no SaaS: each user hosts their own collector (or uses the analytics they already have).
