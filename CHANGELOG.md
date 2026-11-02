# Changelog

All notable changes to the starter. Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versions follow [Semantic Versioning](https://semver.org/) (major: a file moves, is renamed or changes its contract with SETUP.md; minor: new files or sections; patch: fixes and wording).

## [1.10.0] - 2026-10-30

### Added
- `PROGRESS.html` → `docs/`: the owner's live progress page. One self-contained file (no server, no network requests) that shows milestones, tasks and subtasks with progress bars, a build and review pipeline per task (code review, UX review, QA), filters by status and area, a "Waiting on you" box, an overall percentage weighted by task size, and recent activity. Seeded from the M0 queue; light and dark; phone to desktop; WCAG 2.2 AA checked with axe.
- `agent-progress-reporter.md` → `.claude/agents/progress-reporter.md`: a Haiku agent that is the only writer of the page. It edits only the JSON data block, mirrors `ROADMAP.md` (the roadmap wins), follows an event-to-change table, never marks work done without evidence, and validates its edit with a one-line check.
- Wired in: `CLAUDE.md` (team, docs table, objective table, rule), `WORKFLOW.md` (§1 team, §2 loop, §5 status files, §6 model), `ROADMAP.md` working rule 8, `README.md`, `00-START-HERE.md` (table, `mv` command, "Seeing progress"), `SETUP.md` prerequisites, and the `project-manager` and `credits-optimizer` agents.

## [1.9.0] - 2026-10-30

### Added
- `CLAUDE_PLUGINS.md`: React Doctor row in the "Writing code" table, pointing to the `pnpm doctor` setup in `TESTING.md` §4.
- `TESTING.md` §4: a telemetry note (`--no-telemetry`), the rule that our docs win over its findings, and a separate-approval note for React Doctor's `install` and `ci install` steps.

## [1.8.0] - 2026-10-29

### Added
- `CLAUDE.md`: a "Where to look for each objective" table (18 kinds of work, each with the docs to read, the agents, and the skills, plugins or libraries) and a cheapest-first order for finding a skill or plugin.
- `README.md`: a "Find what you need" section; `00-START-HERE.md`: a pointer to the table.

## [1.7.0] - 2026-10-29

### Added
- `CLAUDE_PLUGINS.md` → `docs/`: 30 community Claude plugins sorted into five jobs (writing code; research; huge projects and token limits; design and visuals; writing, marketing and video), each with repo, licence, what it runs (skills, hooks, CLI, MCP or proxy) and fit with this starter; one recommended pick per job; a five-minute repo check before installing; how agents recommend without installing.
- `DESIGN_INSPIRATION.md` Part E, premium interaction patterns (magnetic buttons, controlled parallax, staggered scroll reveal, Easter eggs, page transitions, cursor with context) with limits, a typechecked magnetic wrapper and View Transitions helper, and a review checklist; Part F, visual reference archives (Letterform Archive, The Met Open Access, Rijksmuseum, archives.design, NYPL) with what each allows.
- Listed in `00-START-HERE.md`, `README.md`, `CLAUDE.md` (docs table and the install rule) and `SETUP.md`.

## [1.6.0] - 2026-10-29

### Added
- `agent-skills-scout.md`: a read-only lookup agent on Haiku (`model: haiku`) that reads large skills and references and returns only the rules a task needs, plus first-pass checklist scans for a reviewer to confirm; it warns if it runs on an expensive model.
- `WORKFLOW.md` §8: the Vercel agent-skills repo (which skills fit this stack, approximate token cost, install one at a time, never `deploy-to-vercel` or `vercel-optimize`) and the **credit warning** rule: an agent on the strongest model stops and asks before loading a skill over about 20 KB or auditing more than about 10 files, and otherwise hands the lookup to `skills-scout`.
- Matching rules in `CLAUDE.md` and `agent-credits-optimizer.md`; the new agent listed in `WORKFLOW.md` §1, `00-START-HERE.md` and `README.md`.

## [1.5.0] - 2026-10-29

### Added
- `DESIGN_INSPIRATION.md` Part D, scroll and animation libraries: Lenis (smooth scroll, MIT), GSAP (GSAP Standard no-charge licence, all plugins free; not open source), Vanta (WebGL hero backgrounds, MIT; pinned three r134 under an alias) and React Bits (MIT + Commons Clause: use in projects, never copied into the starter or resold). Each with stack fit, rules and a typechecked example; one animation engine per surface (Motion for app UI, GSAP for scroll storytelling).
- Updated mentions in `00-START-HERE.md`, `README.md`, `CLAUDE.md`, `DESIGN_SYSTEM.md` §15 and `agent-product-designer.md`.

## [1.4.0] - 2026-10-29

### Added
- `SEO_AND_DISCOVERABILITY.md` → `docs/`: a portable launch guide built around five questions (Search Console, XML sitemap, SEO and GEO, meta titles and descriptions, Google Business Profile) plus crawlability first (no-JS HTML, real 404s, canonical URLs, Cloudflare AI bot policy, keeping private and `workers.dev` URLs out) and post-launch monitoring. Includes Google's May 2026 guidance that GEO builds on SEO, Business Profile eligibility (no online-only businesses), and an appendix for starter projects: `noindex` by default, and a typechecked Worker that serves per-page meta tags, `robots.txt` and `sitemap.xml` for listed public pages.
- Listed in `00-START-HERE.md`, `README.md`, `CLAUDE.md`, `SETUP.md`; a `ROADMAP.md` risk; a line in the compliance checklist's sign-off.

## [1.3.0] - 2026-10-29

### Added
- `DESIGN_INSPIRATION.md` → `docs/`: design inspiration and toolkit. Part A, expressive effects: React Three Fiber, Liquid Glass JS, Shader Gradient and Liquid Logo (with Paper Shaders `LiquidMetal` as the Apache-2.0 library, since the Liquid Logo app is PolyForm Shield). Part B, UI building blocks: Motion, Bklit UI charts, Kokonut UI components. Part C, outside tools: Manus. Each entry has its licence, stack fit, a typechecked example and rules; shared rules for lazy-loading, reduced motion, contrast, theme tokens, performance, CSP and licences; a review checklist.
- Listed in `00-START-HERE.md`, `README.md`, `CLAUDE.md`, `SETUP.md`; pointers in `DESIGN_SYSTEM.md` §15 and `agent-product-designer.md`.

## [1.2.0] - 2026-10-29

### Added
- `PRE_LAUNCH_COMPLIANCE_CHECKLIST.md` → `docs/`: a portable, end-of-production compliance gate with 20 items (privacy policy, terms, refunds, cookie policy and banner, business details, form consents, data minimisation, third-party SDK audit, age consent, data deletion, email unsubscribe, dark patterns, hidden fees, fake reviews, unsupported claims, alt text, contrast, keyboard navigation, licences), a rules-by-region table (EU/EEA incl. Norway, UK, US, app and extension stores, AI), a sign-off table and an appendix mapping items to the starter's docs. Agents don't act on it until the owner calls the end of production.
- Listed in `00-START-HERE.md` (table and step 0), `README.md`, `CLAUDE.md` docs table, `SETUP.md` prerequisites and a `ROADMAP.md` risk.

## [1.1.0] - 2026-10-30

### Added
- [React Doctor](https://www.npmjs.com/package/react-doctor) for React code health: `react-doctor` devDependency and a `pnpm doctor` script in `SETUP.md`, a CI step, the verify step, `CLAUDE.md` commands, `TESTING.md` §4, and the developer, code-reviewer and devops agents (changed files only with `--scope changed --base main`).
## [1.0.0] - 2026-10-27

First public release.

### Added
- `00-START-HERE.md`: how to use the starter, where each file goes, placeholders, conventions.
- `SETUP.md`: a tested recipe for Claude Code that scaffolds Vite + React 19 + TypeScript + Tailwind CSS v4 + shadcn/ui (Base UI) with a Hono API on Cloudflare Workers, `/api/health` and its test, CI, and a verify step.
- Docs templates: `CLAUDE.md`, `GUIDE.md`, `ROADMAP.md`, `DESIGN_SYSTEM.md` (UI principles, UX laws, the §23 review checklist), `WORKFLOW.md`.
- Reference docs: `BACKEND_CLOUDFLARE.md`, `SECURITY.md`, `AI_FEATURES.md`, `TESTING.md`, `DEPLOY_AND_CI.md`, optional `BROWSER_EXTENSION.md`.
- `ENV_AND_SECRETS.md`: every var and secret per environment, demo vs production, how to set each.
- `RESEARCH.md`: competitor and UX research template (questions, output format, what not to copy).
- `GITHUB_TEMPLATES.md`: PR template, issue forms, CODEOWNERS, written to `.github/` during setup.
- Nine Claude Code agents (`agent-*.md`): project-manager, credits-optimizer, product-designer, developer, backend-developer, ux-reviewer, code-reviewer, qa-tester, devops.
- `WORKFLOW.md` lessons: one build agent at a time, status only in the roadmap, commits that typecheck one by one, seed and demo data from real public content, demo accounts per role, research before big UI work, owner approval points.
- `BACKEND_CLOUDFLARE.md` §2: migration numbering rules and the idempotent seed pattern.
- Repo files for the starter itself: `README.md`, `LICENSE.md` (MIT), this changelog.

### Fixed
- `SETUP.md` `.gitignore` now ignores `backups/` (remote database exports hold personal data; `SECURITY.md` §8 already assumed it).
