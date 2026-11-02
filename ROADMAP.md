# {{PROJECT_NAME}} roadmap

The plan and the **only** place for status: what's done, what's running, what's next, what needs the owner, decisions, risks and the log. Update it after every task. How features work is in [`GUIDE.md`](./GUIDE.md).

Status key: ✅ Done · 🚧 In progress · ⏳ Todo · ➖ Skipped · ⏸️ Paused

## Dashboard

Last updated **{{DATE}}**.

| | |
| --- | --- |
| **Now** | ⏳ M0-1 Fill in the placeholders and rename the project |
| **Next** | M0-2 brand and design check → M0-3 first real feature brief → M0-4 CI → M0-5 first deploy |
| **Waiting on the owner** | Product summary, roles and brand colour (M0-1) · Cloudflare account and go-ahead to deploy (M0-5) |
| **Released** | Nothing yet. **Goal: the skeleton live at the end of M0 (M0-5).** |

## Working rules

1. **One build task at a time**, top to bottom. The next starts only after the current one passes review and QA.
2. **Every build task ends with `code-reviewer`, then (UI) `ux-reviewer` and `qa-tester`**: phone, tablet, desktop, light and dark, every role that sees the change, and no console errors.
3. **One branch and one PR per milestone**, `dksk/<type>/<name>`, into `main`, with one commit per task.
4. **The owner approves** commits, pushes, PRs, remote migrations, secrets and deploys. Agents suggest the commands.
5. A task that needs design starts with `product-designer`, before any code.
6. New requests go to the end of the queue unless the owner says otherwise.
7. Quality first; cut wasted credits, not care. Never PNG: text checks first, then at most a few JPEGs (quality 60, CSS scale).
8. **The owner's progress page** is `docs/PROGRESS.html`: `progress-reporter` updates it after each step (task start, subtask done, review verdict, block, owner decision needed, task closed). It mirrors this file; if they disagree, this file wins.

## Milestones

| Status | Milestone | Branch | Outcome |
| --- | --- | --- | --- |
| ⏳ | **M0** Set up and ship the skeleton | `dksk/chore/project-setup` | The starter is renamed, branded, checked, in CI and live |
| ⏳ | **M1** {{FIRST_MILESTONE}} | `dksk/feat/{{name}}` | {{OUTCOME}} |

## Queue

Top to bottom. Feature IDs (F) match [`GUIDE.md`](./GUIDE.md).

| # | Status | ID | Task | Owner | Depends on | Size | F |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | ⏳ | M0-1 | Fill in the placeholders and rename the project | project-manager, owner | none | S | |
| 2 | ⏳ | M0-2 | Set the brand colour and logo; check contrast and the shell in both themes | product-designer | M0-1 | S | F01 |
| 3 | ⏳ | M0-3 | Brief and user flow for the first real feature | product-designer | M0-1 | M | F03 |
| 4 | ⏳ | M0-4 | CI on every PR: install, typecheck, lint, test, build | devops | M0-1 | S | |
| 5 | ⏳ | M0-5 | First deploy and smoke test | devops, then qa-tester | M0-2, M0-4, owner OK | S | F02 |
| 6 | ⏳ | M1-1 | {{TASK}} | {{AGENT}} | M0 | M | F03 |

**Small follow-ups** (picked up when their area is next touched): {{FOLLOW_UPS}}.

*Size: S (under half a day), M (about a day), L (several days; consider splitting).*

## Task details

### M0 Set up and ship the skeleton

| ID | Done when | Files |
| --- | --- | --- |
| M0-1 | Every `{{…}}` placeholder is filled in or deliberately left for later (listed here; find them with `grep -rnE "\{\{[A-Z0-9_]+\}\}" --exclude-dir=node_modules .`); the project slug is set everywhere; the Overview and States example pages are kept or replaced; typecheck, lint, test and build pass. | `CLAUDE.md`, `docs/*`, `.claude/agents/*`, `package.json`, `wrangler.jsonc`, `index.html`, `src/lib/app.ts` |
| M0-2 | `--primary`, `--primary-ink`, `--ring` and the washes use the brand colour in both themes; text 4.5:1 and UI parts 3:1 measured from the tokens and recorded in `DESIGN_SYSTEM.md` §2; logo and favicon in `public/`; checked at 375, 768 and 1280 px in light and dark with no console errors. | `src/index.css`, `index.html`, `public/`, `src/components/app-shell.tsx`, `docs/DESIGN_SYSTEM.md` |
| M0-3 | A one-paragraph brief (who, job, pain, success signal), the user flow, must-have vs later, and open questions for the owner; the feature added to `GUIDE.md` as Planned and broken into M1 tasks here. | `docs/GUIDE.md`, `docs/ROADMAP.md`, `docs/design/` |
| M0-4 | `.github/workflows/ci.yml` runs `pnpm install --frozen-lockfile`, `pnpm typecheck`, `pnpm lint`, `pnpm test` and `pnpm build` on every PR and passes on the M0 PR. | `.github/workflows/ci.yml` |
| M0-5 | `pnpm run deploy` succeeds; on the live URL `/` renders, `/api/health` answers `ok`, light and dark work, no console errors. The URL is in `CLAUDE.md` and `GUIDE.md`. | `wrangler.jsonc` |

### M1 {{FIRST_MILESTONE}}

| ID | Done when | Files |
| --- | --- | --- |
| M1-1 | {{ACCEPTANCE_CRITERIA}} | {{FILES}} |

## Owner decisions

| Date | Decision |
| --- | --- |
| {{DATE}} | One build task at a time; each ends with code review, then QA. |
| {{DATE}} | Never generate PNGs. Screenshots only as compressed JPEG (quality 60, CSS scale), few, after text checks. |
| {{DATE}} | Keep every shadcn component and brand file, even unused ones. |
| {{DATE}} | Seed and demo data come from real public content, never invented; one demo account per role, only when `DEMO_MODE` is on. |

## Risks

| Risk | Handling |
| --- | --- |
| Placeholders left in shipped text (`{{PROJECT_NAME}}` in the tab title) | M0-1 checks with `grep -rnE "\{\{[A-Z0-9_]+\}\}" --exclude-dir=node_modules .` |
| Brand colour fails contrast as text in dark mode | `--primary-ink` is a lighter tint in dark mode; measure it (M0-2) |
| Parallel build agents editing the same files | One build task at a time |
| A public site launches but can't be found (no Search Console, sitemap or meta tags; every URL the same `index.html`) | Record the two early decisions in `SEO_AND_DISCOVERABILITY.md` in `GUIDE.md` at the start; run the guide at launch with the compliance check |
| Public release without the compliance check (privacy, cookies, consents, deletion, accessibility, licences) | Before the first public release, the owner asks `project-manager` to turn `docs/PRE_LAUNCH_COMPLIANCE_CHECKLIST.md` into a release task; not earlier |
| The dev server sometimes stops answering on `127.0.0.1` | Use http://[::1]:5173 or restart `pnpm dev` |

## Log

| Date | Update |
| --- | --- |
| {{DATE}} | Project created from the project starter: app shell, design system, agents, Worker with `/api/health`; typecheck, lint, test and build pass. |
