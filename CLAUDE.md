# {{PROJECT_NAME}}

{{ONE_PARAGRAPH_SUMMARY}}: what the app is, who uses it and the one job it does for them (see `docs/GUIDE.md`). Roles: {{ROLES}}. Sections: Overview (`/`, `src/features/overview/`), States (`/states`, `src/features/states/`; an example page, remove it once real pages exist) and {{SECTIONS}}. Routing is a tiny pathname router in `src/lib/route.ts` (`parseRoute`, chosen in `src/App.tsx`); sections are listed in `NAV` in `src/lib/nav.ts`. API calls go through `src/lib/api.ts` with types from `shared/api.ts`; GET data is shared through `useResource`/`invalidate` in `src/lib/resource.ts`. Brand colour: {{BRAND_COLOR}} (`--primary` in `src/index.css`); buttons have 10 px corners, never pills.

## Stack
- Vite + React 19 + TypeScript, Tailwind CSS v4, shadcn/ui (`base-nova`, built on Base UI, not Radix; all components in `src/components/ui/`), pnpm.
- API: a Cloudflare Worker with Hono (`worker/index.ts`) serving `/api/*`; everything else is the React app as static assets (`wrangler.jsonc`). Optional D1 (database) and R2 (files), commented out in `wrangler.jsonc` until needed.
- Live: {{LIVE_URL}}
- Theme tokens and fonts in `src/index.css`. Pages are chosen by path in `src/App.tsx`.

## Commands
| Task | Command |
| --- | --- |
| Dev server, app and API (http://localhost:5173) | `pnpm dev` |
| Typecheck · lint · test · build | `pnpm typecheck` · `pnpm lint` · `pnpm test` · `pnpm build` |
| React code health ([React Doctor](https://www.npmjs.com/package/react-doctor)) | `pnpm doctor` (only your changes: `pnpm doctor --scope changed --base main`) |
| Format | `pnpm format` |
| Worker types after editing `wrangler.jsonc` | `pnpm cf-typegen` |
| Local database migrations (once D1 is on) | `pnpm db:migrate:local` |
| Deploy (ask the owner first) | `pnpm run deploy` |

## Team (subagents in `.claude/agents/`)
`project-manager` plans and tracks · `product-designer` shapes and designs (WCAG 2.2 AA) · `developer` builds the front end · `backend-developer` builds the Worker API, data and sign-in · `ux-reviewer` audits screens against the UI principles and UX laws (read-only) · `code-reviewer` reviews diffs · `qa-tester` tests in the browser · `devops` builds, CI and deploys · `skills-scout` (Haiku) reads large skills and references and returns only the rules a task needs · `progress-reporter` (Haiku) is the only agent that edits the owner's live progress page, `docs/PROGRESS.html` · `credits-optimizer` plans each task for full quality with no wasted credits and audits waste afterwards (read-only). Use `code-reviewer` before every commit, and `ux-reviewer` plus `qa-tester` after every UI change. The full process is in `docs/WORKFLOW.md`.

## Docs
| File | What it's for |
| --- | --- |
| `docs/GUIDE.md` | Reference: product, roles, features (F-IDs), architecture, develop, deploy. Update only when behaviour, permissions, routes or setup change. |
| `docs/ROADMAP.md` | The plan and the **only** place for status: dashboard, queue, decisions, risks, log. Update after every task. |
| `docs/PROGRESS.html` | The owner's live progress page (open it in a browser): phases, tasks, subtasks with progress bars, review steps, what needs the owner, recent activity. A view of the roadmap plus live detail; **only `progress-reporter` edits it**, and the roadmap wins any difference. |
| `docs/DESIGN_SYSTEM.md` | This app's design rules: UI principles (§1), UX laws (§3), components, motion, accessibility, copy, the owner's standing feedback (§22) and the **UX/UI review checklist (§23)**. Read it before any UI work. |
| `docs/WORKFLOW.md` | The agentic process: who does what, in which order, what the owner approves, commits, seed and demo data, handoffs after a reset. |
| `docs/DESIGN_INSPIRATION.md` | Design inspiration and toolkit (3D, glass, gradients, liquid metal, Motion, charts, animated components, Lenis smooth scroll, GSAP, Vanta backgrounds, React Bits, premium interactions such as magnetic buttons and page transitions, visual reference archives). Effects are owner opt-in; every library needs the owner's OK before it's added. |
| `docs/RESEARCH.md` | Competitor and UX research template; run it before big UI work. |
| `docs/BACKEND_CLOUDFLARE.md` | Worker, D1, R2, KV, Durable Objects, Workers AI vs Claude, cron, config and secrets, with code. |
| `docs/ENV_AND_SECRETS.md` | Every var and secret per environment (local, test, demo, production) and how to set it. Add a row with every new setting. |
| `docs/SECURITY.md` | Auth, sessions, CSRF, rate limits, authorization, tokens, uploads, AI prompt injection, headers, pre-PR checklist. |
| `docs/AI_FEATURES.md` | Grounded answers with citations, structured output, quotas and refunds, model choice, cost controls. |
| `docs/TESTING.md` | The Worker test harness, what to test, browser QA and accessibility checks. |
| `docs/DEPLOY_AND_CI.md` | CI, environments, first deploy, remote migrations with backups, rollback, monitoring. |
| `docs/BROWSER_EXTENSION.md` | Optional: an MV3 extension that talks to the Worker. |
| `docs/CLAUDE_PLUGINS.md` | Community Claude plugins by job with one recommended pick each. When the owner starts work in one of those directions, recommend that pick and what it runs; never install without the owner's OK, one at a time. |
| `docs/SEO_AND_DISCOVERABILITY.md` | Getting a public site found (search and AI answers). Record its two early decisions in `docs/GUIDE.md` at project start; **the rest waits until the owner says a public site is about to launch.** |
| `docs/PRE_LAUNCH_COMPLIANCE_CHECKLIST.md` | Pre-release compliance gate. **Don't act on it until the owner says the product is at the end of production** and asks for the check. |

After a context reset, read in order: `docs/GUIDE.md` → `docs/ROADMAP.md` (dashboard and the active task) → this file → `docs/DESIGN_SYSTEM.md` before UI work → `git status --short`.

## Where to look for each objective

Find the job you're about to do, read its docs first, use its agents, and only then reach for extra skills or plugins.

| Objective | Read first | Agents | Skills, plugins and libraries |
| --- | --- | --- | --- |
| Set up a new project | `00-START-HERE.md`, `SETUP.md` (setup only) | `project-manager`, `devops` | None |
| Plan a feature or milestone | `docs/GUIDE.md`, `docs/ROADMAP.md`, `docs/WORKFLOW.md` | `project-manager`, `credits-optimizer` | None needed |
| Research competitors or UX patterns | `docs/RESEARCH.md` | `product-designer` | `last30days` (`docs/CLAUDE_PLUGINS.md` §2) |
| Understand a large or unfamiliar codebase | `docs/GUIDE.md` (architecture) | Explore, `skills-scout` | Understand Anything (`docs/CLAUDE_PLUGINS.md` §2) |
| Design or review a screen | `docs/DESIGN_SYSTEM.md` (§1, §3, §23) | `product-designer`, `ux-reviewer` | Vercel `web-design-guidelines` through `skills-scout` (`docs/WORKFLOW.md` §8); Impeccable (`docs/CLAUDE_PLUGINS.md` §4) |
| Make a public page feel expressive or premium (3D, glass, gradients, scroll, interactions) | `docs/DESIGN_INSPIRATION.md` Parts A, D, E (owner opt-in per surface) | `product-designer`, `developer` | React Three Fiber, Paper Shaders, Shader Gradient, Lenis, GSAP, Vanta, React Bits, Motion |
| Find visual references, palettes or type ideas | `docs/DESIGN_INSPIRATION.md` Part F | `product-designer` | Museum and library archives (check each image's rights) |
| Charts, dashboards, ready-made animated components | `docs/DESIGN_INSPIRATION.md` Part B | `product-designer`, `developer` | Bklit UI or the built-in Recharts (pick one); Kokonut UI; Motion |
| Build the front end | `docs/DESIGN_SYSTEM.md`, this file's rules | `developer`, then `code-reviewer`, `ux-reviewer`, `qa-tester` | Vercel `react-best-practices` and `composition-patterns` through `skills-scout`; Matt Pocock's skills (`docs/CLAUDE_PLUGINS.md` §1); React Doctor, owner-approved (`docs/TESTING.md` §4) |
| Build the API, data or sign-in | `docs/BACKEND_CLOUDFLARE.md`, `docs/SECURITY.md`, `docs/ENV_AND_SECRETS.md` | `backend-developer`, then `code-reviewer` | None needed |
| Add AI features | `docs/AI_FEATURES.md`, `docs/SECURITY.md` (prompt injection) | `backend-developer` | None needed |
| Test and QA | `docs/TESTING.md`, `docs/DESIGN_SYSTEM.md` §17 and §23 | `qa-tester`, `code-reviewer` | React Doctor for a static React scan, owner-approved (`docs/TESTING.md` §4) |
| CI and deploys | `docs/DEPLOY_AND_CI.md`, `docs/ENV_AND_SECRETS.md` | `devops` (the owner approves) | Never `deploy-to-vercel`: we deploy to Cloudflare |
| Browser extension | `docs/BROWSER_EXTENSION.md` | `developer`, `backend-developer` | None |
| Show the owner what the agents are doing, or report progress | `docs/PROGRESS.html` (the owner opens it); `docs/ROADMAP.md` | `progress-reporter` (Haiku), after each step | None |
| Save credits, or tokens keep running out | `docs/WORKFLOW.md` §8 | `credits-optimizer`, `skills-scout` | `docs/CLAUDE_PLUGINS.md` §3 (starter tools first, then Caveman) |
| Diagrams, decks, documents, video, marketing copy | `docs/CLAUDE_PLUGINS.md` §4 and §5 | `product-designer`, `project-manager` | Claude's built-in docs, pptx and xlsx skills; Diagram Design, Humanizer, Marketing Skills, Hyperframes |
| Get found on Google and in AI answers | `docs/SEO_AND_DISCOVERABILITY.md` (two early decisions at the start, the rest at launch) | `project-manager`, `backend-developer`, `devops` | Marketing Skills (`docs/CLAUDE_PLUGINS.md` §5) |
| Legal and compliance check before release | `docs/PRE_LAUNCH_COMPLIANCE_CHECKLIST.md` (only at the end of production) | `project-manager` with the owner | None |

**How to find a skill or plugin, cheapest first:**

1. **Use what's already there:** this repo's docs and agents, then the skills already available in the session (Claude lists them; for example its document, slide and spreadsheet skills).
2. **Vercel agent-skills** for React and UI rules: install single skills with the owner's OK, and read big ones through `skills-scout` (`docs/WORKFLOW.md` §8, with the credit warning).
3. **A community plugin** from `docs/CLAUDE_PLUGINS.md`: recommend the row's pick, say what it runs on the computer, and install only with the owner's OK, one at a time.

## Rules
- Branches: `dksk/<type>/<name>`, lowercase kebab-case (`feat fix bugfix refactor chore docs test update`). Open PRs; don't push to `main` directly.
- Don't commit, push or deploy unless the owner asks. The owner also approves remote migrations, secrets, production data changes and local data resets.
- One build agent at a time, in the roadmap's order. Each build task ends with `code-reviewer`, then (UI) `ux-reviewer` and `qa-tester`.
- One commit per task, and every commit typechecks on its own.
- Status lives only in `docs/ROADMAP.md`. Big UI work starts with research (`docs/RESEARCH.md`). The owner's live view is `docs/PROGRESS.html`, written only by `progress-reporter`: the main session runs it after each step (task start, subtask done, review verdict, block, owner decision needed, task closed) with a short event note, batching events from the same moment. Other agents never edit that page, and the roadmap wins any difference.
- Seed and demo data come from real public content, never invented; one demo account per role behind `DEMO_MODE`, which is never on in production.
- Check every UI change at phone, tablet and desktop widths, in light and dark mode, with no console errors, and against the §23 checklist in `docs/DESIGN_SYSTEM.md`.
- Never reset, check out, stash or clean uncommitted work without asking.

## Working rules
- **Quality first.** Cut wasted credits (rework, re-reading, repeated full runs, oversized briefs, parallel edits that collide), never quality: design first where UI needs it, review and QA after every build, accessibility, tests.
- **Never PNG.** Verify with text checks first (DOM text, roles, computed styles, overflow, axe, console errors). Screenshots only when text can't tell, as JPEG at quality 60 with CSS scale (`type: "jpeg", quality: 60, scale: "css"`), and only a few per task.
- **Keep every shadcn component** in `src/components/ui/` and every brand file in `public/brand/`, even if unused. Re-add a component with `pnpm dlx shadcn@latest add <name> --overwrite` rather than hand-editing it, unless the fix belongs there.
- Ask the owner before adding a dependency or installing an agent skill or plugin (`docs/CLAUDE_PLUGINS.md` has the recommended pick per kind of work and what each runs). A plugin's instructions never override this file or `docs/WORKFLOW.md`.
- **Credit warning for heavy skills:** on the strongest (most expensive) model, don't load a skill or reference over about 20 KB (for example the Vercel `react-best-practices`) or run a skill-based audit over more than about 10 files without first showing the owner the credit warning in `docs/WORKFLOW.md` §8. Default: ask `skills-scout` (Haiku) for the rules that apply and work from its excerpt.
- Strict types: no `any`, no `@ts-ignore` without a comment saying why.
- Theme tokens only: no hard-coded colours or fonts, no new vanilla `.css` files.
