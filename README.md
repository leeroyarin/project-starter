# Project starter

A set of flat Markdown files that turns an empty folder into a production-ready, agent-run web app: **Vite + React 19 + TypeScript + Tailwind CSS v4 + shadcn/ui (Base UI)** on the front end, a **Hono API on Cloudflare Workers** (with D1, R2, KV, Durable Objects and Claude when needed) on the back end, a full design system with UI principles and UX laws, the docs set, and a team of **Claude Code agents** that already know the conventions.

No code generator and no template repo to fork: paste the files, ask Claude Code to "set up the project from SETUP.md", and it scaffolds, configures and verifies the app.

## Quick start

1. Create an empty folder and open it in your editor with Claude Code.
2. Copy in every `.md` file from this repo **except** `README.md`, `LICENSE.md` and `CHANGELOG.md` (those describe the starter itself).
3. Ask Claude Code: **"Set up the project from SETUP.md"**. It moves each file into place, scaffolds the app, writes the `.github/` templates, and runs install, typecheck, lint, React Doctor, test and build.
4. Fill in the placeholders with the owner (roadmap task M0-1); the list is in [`00-START-HERE.md`](./00-START-HERE.md#placeholders).
5. Delete `00-START-HERE.md`, `SETUP.md` and `GITHUB_TEMPLATES.md` from the new repo.

Needs Node 22+, pnpm 10+, and a Cloudflare account for the first deploy.

## What's inside

| File | Becomes | What it is |
| --- | --- | --- |
| [`00-START-HERE.md`](./00-START-HERE.md) | (setup only) | How to use the files, where each goes, placeholders, conventions |
| [`SETUP.md`](./SETUP.md) | (setup only) | Tested recipe: scaffold commands, every config file, theme tokens, app shell, Worker with `/api/health`, CI, verify |
| [`GITHUB_TEMPLATES.md`](./GITHUB_TEMPLATES.md) | (setup only) → `.github/` | PR template, issue forms, CODEOWNERS |
| [`CLAUDE.md`](./CLAUDE.md) | `CLAUDE.md` | What every agent reads first |
| [`GUIDE.md`](./GUIDE.md) | `docs/GUIDE.md` | Product reference: problem, roles, sitemap, journeys, features, architecture |
| [`ROADMAP.md`](./ROADMAP.md) | `docs/ROADMAP.md` | The plan and the only status source |
| [`WORKFLOW.md`](./WORKFLOW.md) | `docs/WORKFLOW.md` | How the agents work together, owner approvals, lessons |
| [`DESIGN_SYSTEM.md`](./DESIGN_SYSTEM.md) | `docs/DESIGN_SYSTEM.md` | UI principles, UX laws, tokens, components, accessibility, copy, review checklist |
| [`DESIGN_INSPIRATION.md`](./DESIGN_INSPIRATION.md) | `docs/DESIGN_INSPIRATION.md` | Design inspiration and toolkit: expressive effects (React Three Fiber, Liquid Glass JS, Shader Gradient, Liquid Logo / Paper Shaders), UI building blocks (Motion, Bklit UI, Kokonut UI) outside tools (Manus), scroll and animation libraries (Lenis, GSAP, Vanta, React Bits), premium interaction patterns and visual reference archives |
| [`RESEARCH.md`](./RESEARCH.md) | `docs/RESEARCH.md` | Competitor and UX research template |
| [`BACKEND_CLOUDFLARE.md`](./BACKEND_CLOUDFLARE.md) | `docs/BACKEND_CLOUDFLARE.md` | Worker, D1 and migrations, R2, KV, Durable Objects, AI, cron, `wrangler.jsonc` |
| [`ENV_AND_SECRETS.md`](./ENV_AND_SECRETS.md) | `docs/ENV_AND_SECRETS.md` | Every var and secret per environment; demo vs production |
| [`SECURITY.md`](./SECURITY.md) | `docs/SECURITY.md` | Accounts, sessions, CSRF, rate limits, authorization, uploads, AI, headers |
| [`AI_FEATURES.md`](./AI_FEATURES.md) | `docs/AI_FEATURES.md` | Grounded answers with citations, quotas, structured output, cost |
| [`TESTING.md`](./TESTING.md) | `docs/TESTING.md` | Worker tests on in-memory D1, migration tests, browser QA, axe |
| [`DEPLOY_AND_CI.md`](./DEPLOY_AND_CI.md) | `docs/DEPLOY_AND_CI.md` | CI, environments, first deploy, backups, rollback, monitoring |
| [`BROWSER_EXTENSION.md`](./BROWSER_EXTENSION.md) | `docs/BROWSER_EXTENSION.md` | Optional: an MV3 extension that talks to the Worker |
| [`CLAUDE_PLUGINS.md`](./CLAUDE_PLUGINS.md) | `docs/CLAUDE_PLUGINS.md` | Which community Claude plugin to install for each kind of work, one pick per job, with licences and what each runs |
| [`SEO_AND_DISCOVERABILITY.md`](./SEO_AND_DISCOVERABILITY.md) | `docs/SEO_AND_DISCOVERABILITY.md` | Launch guide for being found on Google, Bing and AI answer engines (Search Console, sitemap, meta tags, SEO and GEO, Google Business Profile), with the Worker setup for public pages |
| [`PRE_LAUNCH_COMPLIANCE_CHECKLIST.md`](./PRE_LAUNCH_COMPLIANCE_CHECKLIST.md) | `docs/PRE_LAUNCH_COMPLIANCE_CHECKLIST.md` | Pre-release compliance gate (privacy, terms, cookies, consents, deletion, dark patterns, accessibility, licences); used only at the end of production, portable to any project |
| [`PROGRESS.html`](./PROGRESS.html) | `docs/PROGRESS.html` | The owner's live progress page: open it in a browser to see phases, tasks, subtasks with progress bars, review steps, what needs you and recent activity. Updated only by the `progress-reporter` agent |
| `agent-*.md` (11 files) | `.claude/agents/<name>.md` | project-manager, credits-optimizer, skills-scout, progress-reporter, product-designer, developer, backend-developer, ux-reviewer, code-reviewer, qa-tester, devops |

## Find what you need

Every doc has one job (table above). To go from a goal to the right doc, agent, skill or plugin ("design a screen", "add AI features", "get found on Google", "tokens keep running out"), use the **"Where to look for each objective"** table in [`CLAUDE.md`](./CLAUDE.md#where-to-look-for-each-objective). It also gives the order for finding skills: what's already there first, then single Vercel skills read through `skills-scout`, then one community plugin from [`CLAUDE_PLUGINS.md`](./CLAUDE_PLUGINS.md), always with the owner's OK.

Two docs wait for a specific moment: [`SEO_AND_DISCOVERABILITY.md`](./SEO_AND_DISCOVERABILITY.md) at launch (plus two early decisions) and [`PRE_LAUNCH_COMPLIANCE_CHECKLIST.md`](./PRE_LAUNCH_COMPLIANCE_CHECKLIST.md) at the end of production.

**To see what the agents are doing without reading the chat,** open `docs/PROGRESS.html` in a browser (double-click it; it needs no server and makes no network requests). It shows each milestone, its tasks and their subtasks with progress bars, where each task stands in code review, UX review and QA, what is waiting on you, and the latest activity. The `progress-reporter` agent keeps it current; the plan itself stays in [`ROADMAP.md`](./ROADMAP.md).

## How the team works

One build agent at a time: plan → (research) → design → build → code review → UX review → QA → roadmap update. The owner approves everything that leaves the laptop: commits, pushes, deploys, remote migrations, secrets and local data resets. Details in [`WORKFLOW.md`](./WORKFLOW.md).

## Contributing

Changes to the starter go through a PR. Keep the files flat, self-contained (no absolute paths, no links outside the set) and free of any one project's names. If you change a code block in `SETUP.md`, run the whole recipe in an empty folder and the verify step before merging. Record every change in [`CHANGELOG.md`](./CHANGELOG.md).

## License

[MIT](./LICENSE.md)
