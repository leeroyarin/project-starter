# Workflow

How work moves through the team of Claude Code agents (`.claude/agents/`), what the owner decides, and how status stays in one place. Short version: **one build task at a time; plan → cost plan → design → build → review → QA → roadmap update; the owner approves everything that leaves the laptop.**

## Contents
1. [The team](#1-the-team)
2. [The loop for one task](#2-the-loop-for-one-task)
3. [What the owner approves](#3-what-the-owner-approves)
4. [Branches, commits and PRs](#4-branches-commits-and-prs)
5. [Status: one place only](#5-status-one-place-only)
6. [Briefing an agent](#6-briefing-an-agent)
7. [Handoff after a context reset](#7-handoff-after-a-context-reset)
8. [Quality and credits](#8-quality-and-credits)
9. [Seed and demo data](#9-seed-and-demo-data)
10. [Lessons from earlier projects](#10-lessons-from-earlier-projects)

## 1. The team

| Agent | Does | Edits code? |
| --- | --- | --- |
| `project-manager` | Turns requests into ordered tasks with owners; keeps `GUIDE.md` and `ROADMAP.md` current; branch names and PR text | Docs only |
| `credits-optimizer` | Before a task: which agent and model, a tight brief, the checks. After: where credits were wasted | No |
| `skills-scout` | Runs on Haiku: reads large skills and references (Vercel agent-skills) and returns only the rules a task needs; first-pass checklist scans | No |
| `progress-reporter` | Runs on Haiku: the only agent that edits `docs/PROGRESS.html`, the owner's live progress page (phases, tasks, subtasks with progress, review steps, what needs the owner, activity) | The progress page only |
| `product-designer` | Brief, user flow, scope, then screens; owns `DESIGN_SYSTEM.md`; WCAG 2.2 AA and the UX laws | Visual changes |
| `developer` | Front end in `src/` | Yes |
| `backend-developer` | Worker API, D1, R2, sign-in, AI calls in `worker/`, `migrations/`, `shared/` | Yes |
| `ux-reviewer` | Audits screens and flows against the UI principles, UX laws and the §23 checklist in `DESIGN_SYSTEM.md` | No |
| `code-reviewer` | Reviews the diff for bugs, security, conventions, design-system drift | No |
| `qa-tester` | Tests in a real browser: widths, themes, keyboard, axe, console | No |
| `devops` | Builds, CI, environments, secrets, deploys (asks before production) | Config |

## 2. The loop for one task

```text
owner request
  → project-manager      plan: tasks in order, one owner each, decisions for the owner, row in ROADMAP
  → credits-optimizer    cost plan for big or risky tasks: agent + model, 3–8 files to read, criteria, checks
  → product-designer     big UI work (new section, core flow, redesign): research first (docs/RESEARCH.md)
  → product-designer     if the task has UI: brief → user flow → scope → screens (before any code)
  → developer / backend-developer    build (ONE build agent at a time)
  → code-reviewer        the diff: Blocker / Should fix / Nit  → fixes by the builder
  → ux-reviewer          UI changes: §1, §3 and the §23 checklist  → fixes by the builder
  → qa-tester            UI changes: browser at 5 widths, light/dark, keyboard, axe, console
  → project-manager      ROADMAP: ✅, dashboard, log line; GUIDE only if behaviour changed
  → owner                approves the commit (and later the PR, migration, deploy)
```

- **One build agent at a time**, top to bottom in the roadmap's queue. The next build starts only after the current one passes review and QA.
- `ux-reviewer` and `code-reviewer` are read-only, so they can run side by side after a build; fixes go back to the one builder.
- Backend-only tasks skip design, `ux-reviewer` and browser QA, but not `code-reviewer` and tests.
- New requests go to the end of the queue unless the owner says otherwise.
- **The owner's progress page.** `progress-reporter` updates `docs/PROGRESS.html` at each step: a task starts, a subtask finishes, a reviewer gives a verdict, work is blocked, the owner is needed, a task closes. Only the main session can call it (agents can't call other agents), so it runs it with a short event note after each of those moments and batches events from the same moment (two reviewers finishing together are one call). The page never replaces the roadmap: it mirrors it, and the roadmap wins any difference.
- A task is done when its "Done when" criteria in `ROADMAP.md` are met, all checks pass, and the roadmap is updated.

## 3. What the owner approves

Agents suggest the exact commands; the owner says go:

- Commits, pushes, PRs and merges.
- Deploys (production and demo), rollbacks.
- Remote migrations (after a backup), production data changes, resetting local data (`rm -rf .wrangler/state`, re-seeding).
- Secrets: adding, rotating, removing.
- New dependencies, and turning on a new paid service or Cloudflare resource.
- Anything irreversible: deleting files, branches, cloud resources; DNS; branch protection.

Never reset, check out, stash or clean uncommitted work. If a task conflicts with uncommitted changes, stop and ask.

## 4. Branches, commits and PRs

- Branches: `dksk/<type>/<name>`, lowercase kebab-case, type one of `feat fix bugfix refactor chore docs test update`. Check: `^dksk/(feat|fix|bugfix|refactor|chore|docs|test|update)/[a-z0-9]+(-[a-z0-9]+)*$`.
- **One branch and one PR per milestone**, into `main`, with **one commit per task**: imperative subject, a body that says why.
- When several tasks' changes share a file, split commits by hunk (`git add -p`) so each commit is one task; the owner approves each.
- **Every commit typechecks on its own**, so any commit can be checked out, bisected or reverted. Plan the split so each commit carries what it needs (a type with its first use, a migration with its query). After the owner's commits, check each one in a throwaway worktree, which leaves the working tree alone: `git worktree add ../check <sha>`, `ln -s "$PWD/node_modules" ../check/node_modules`, `pnpm --dir ../check typecheck`, then `git worktree remove ../check`. If a commit can't typecheck alone, say so before the push and suggest merging it with the next one.
- PR description: summary, a few JPEG screenshots from `qa-tester` (never PNG), what changed (one line per commit), how to verify (`pnpm typecheck`, `pnpm lint`, `pnpm doctor`, `pnpm test`, `pnpm build`), checklist, and the compare URL.
- `code-reviewer` before every commit.

## 5. Status: one place only

| File | Job | Updated |
| --- | --- | --- |
| `docs/ROADMAP.md` | **The only status source:** dashboard (Now, Next, Waiting on the owner, Released), queue (the **Status column comes right after #**, so status is the first thing read), task details, owner decisions, risks, log | After every task |
| `docs/PROGRESS.html` | **Live view for the owner:** phases, tasks, subtasks with progress bars, review steps, what needs the owner, recent activity. Open it in a browser. A view of the roadmap plus live detail; the roadmap wins any difference | At each step, by `progress-reporter` only |
| `docs/GUIDE.md` | **Reference:** what the product does, features with stable F-IDs and a State line, architecture, setup | Only when behaviour, permissions, routes or setup change |
| `docs/DESIGN_SYSTEM.md` | Design rules and the owner's feedback (§22) | When a rule changes or the owner gives feedback |
| `CLAUDE.md` | What every agent reads first | When conventions change |

Never copy status into the guide, `CLAUDE.md`, PR text or issues; link to the roadmap instead. The progress page is the one mirror, read-only for everyone but `progress-reporter`. Feature IDs are permanent. "Implemented" isn't "released": released means merged, migrated and deployed.

## 6. Briefing an agent

A good brief makes the agent right the first time (rework is the biggest waste):

- The task's roadmap row and its "Done when" criteria, copied.
- The 3–8 files to read, with line ranges where possible; point to docs instead of pasting them.
- What **not** to touch (other agents' areas, uncommitted work, `src/components/ui/`).
- Design notes from `product-designer` for UI work.
- The checks to run and what to report (file paths, results, anything left or risky).
- Model: the strongest for building, design, security, migrations, debugging; mid-size only for mechanical work (QA checklists, doc edits); `skills-scout` (Haiku) for skill and reference lookups; `progress-reporter` (Haiku) for the progress page; see `credits-optimizer` and §8.

## 7. Handoff after a context reset

Read in this order, then continue the active task:

1. `docs/GUIDE.md` (what the product is)
2. `docs/ROADMAP.md` (dashboard, the active task, waiting on the owner)
3. `CLAUDE.md` (conventions)
4. `docs/DESIGN_SYSTEM.md` before any UI work
5. `git status --short` and `git log --oneline -10`

Before stopping work, leave the roadmap current: what's done, what's half done (with file paths), what's next, what needs the owner. A future session must be able to continue from the roadmap alone.

## 8. Quality and credits

- **Quality first.** Credits efficiency means cutting waste, never quality. Keep: design first where UI needs it, research before big UI work, review and QA after every build, accessibility, tests, the strongest model for building.
- Waste to cut: rework from unclear briefs, re-reading whole trees, repeated full test runs (focused while working, full once at the end), oversized briefs, parallel edits that collide, screenshots.
- **Never PNG.** Text checks first (DOM text, roles, computed styles, overflow, axe, console). Screenshots only when text can't tell or the owner needs to see it (PR screenshots), as JPEG quality 60 with CSS scale, a few per task.
- No demo artifacts or long reports unless the owner asks.

### External agent skills (Vercel agent-skills)

[github.com/vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills) (MIT) packages Vercel Engineering's guidance as Agent Skills that Claude Code loads when a task matches. Useful, but **heavy**: some skills are large, and loading one on the strongest model in every task burns credits fast. Rough sizes (about 4 characters per token):

| Skill | For this stack | Size when loaded |
| --- | --- | --- |
| `web-design-guidelines` | Yes: a UI audit (accessibility, focus, forms, motion, typography, images, performance, dark mode). Our `DESIGN_SYSTEM.md` wins on conflicts. It fetches its rules live from GitHub on every run. | Small skill, but fetches about 2k tokens of rules and then reads every file it audits |
| `react-best-practices` | Partly: bundle size, client data fetching, re-renders and rendering rules apply; the Next.js server rules don't | About 27k tokens (`AGENTS.md`), plus its `rules/` folder |
| `composition-patterns` | Yes: compound components, lifting state, fewer boolean props | About 6k tokens (`AGENTS.md`) |
| `react-view-transitions` | Partly: check `<ViewTransition>` is in our React version; skip the Next.js parts | About 4k tokens, plus references |
| `writing-guidelines` | Optional, for docs; our copy rules (`DESIGN_SYSTEM.md` §20) win | Small skill plus fetched rules |
| `react-native-guidelines` | No (web stack), unless the project adds an Expo app | |
| `deploy-to-vercel`, `vercel-optimize` | **Don't install.** We deploy to Cloudflare, and the owner approves every deploy; `deploy-to-vercel` could fire on "deploy my app" and ship to the wrong host. | |

- **Install only the skills you need, one by one (owner OK first):** `npx skills add vercel-labs/agent-skills --skill web-design-guidelines -a claude-code`. Never install the whole pack.
- **Look up with the cheap agent.** Large skills are read by `skills-scout`, which runs on Haiku. It returns only the rules that apply to the task (under 40 lines with sources), and the building or reviewing agent works from that excerpt. A first-pass audit (for example the web interface guidelines over changed files) can also run on `skills-scout`; `ux-reviewer` or `code-reviewer` confirms findings before anything changes.
- **Credit warning on expensive models.** Before an agent running on the strongest (most expensive) model loads a skill or reference over about 20 KB (about 5k tokens), or runs a skill-based audit over more than about 10 files, it stops and shows this, then waits for the owner. When nobody is there to answer, it hands the lookup to `skills-scout` instead:

  > **Credit warning:** `<skill>` loads about `<N>`k tokens, and this agent runs on `<model>`, an expensive model. Cheaper: let `skills-scout` (Haiku) pull only the rules this task needs. Continue on `<model>` anyway? (yes / use skills-scout)

- This is the one place where the cheap model is the default. Building, design, security and final review still run on the strongest model, working from the scout's excerpt.

## 9. Seed and demo data

- **Real, public content only.** Seed and demo data come from real public sources that fit the product (public docs, handbooks, open data, published policies), with the source noted in the seed file. Never invented filler ("Lorem ipsum", "Test item 3", made-up companies or people): it hides layout problems, and the owner judges the app on it. Never real personal data.
- **One seed, safe to re-run.** Seeds are idempotent (`INSERT OR IGNORE` / `ON CONFLICT DO UPDATE` with fixed ids), so running them twice changes nothing ([`BACKEND_CLOUDFLARE.md` §2](./BACKEND_CLOUDFLARE.md#seeds-and-demo-accounts)). Local and demo only, never production.
- **A demo account per role**, pre-filled on the sign-in page as one-click buttons when `DEMO_MODE` is on (local and demo only). QA signs in as each role through them.
- Resetting local data and re-seeding needs the owner's OK (it wipes their test data).

## 10. Lessons from earlier projects

| What happened | Rule it produced |
| --- | --- |
| Several features (design pass, review queue, AI drafts, an extension, reports) were built by agents **in parallel**; simultaneous edits to shared files **broke the local app**. | One build agent at a time, in the roadmap's order. Read-only agents (reviewers) may run alongside. |
| Status was kept in **two docs**; they drifted, and nobody could tell what was released. | `ROADMAP.md` is the only status source; `GUIDE.md` is reference only, with a State line per feature. |
| A large uncommitted working tree built up across many tasks. | One commit per task, approved by the owner as tasks finish; split by hunk when files are shared. |
| PNG screenshots used up credits fast. | Text checks first; JPEG quality 60 only, few. |
| Migrations that rewrote data (a table becoming a view) were risky to run remotely. | Back up first, test on a production-like copy, keep migrations backward compatible. |
| Feature requests arrived mid-task and work got interleaved. | New requests go to the end of the queue unless the owner says otherwise. |
| The owner's visual feedback was given once and then forgotten by later sessions. | Record it in `DESIGN_SYSTEM.md` §22 the same day; every agent reads it. |
| Status was hard to scan because it sat at the end of each queue row. | The Status column comes right after # in the roadmap queue. |
| "Save credits" was read as "do less", and work had to be redone. | Credits efficiency cuts waste (rework, re-reads, repeated runs, images), never quality or the strongest model for building. |
| Commits split by hunk didn't build on their own, so one couldn't be reverted alone. | Every commit typechecks on its own; check each split before suggesting it (§4). |
| Invented demo data ("Test item 3") hid layout problems and looked fake to the owner. | Seed and demo data from real public content (§9). |
| Testing each role meant creating accounts by hand every time. | A pre-filled demo account per role behind `DEMO_MODE` (§9). |
| A big screen was designed from scratch, then redone after the owner compared it with familiar tools. | Research competitors and familiar tools before big UI work (`RESEARCH.md`). |
| Agents ran commits, deploys or data resets the owner hadn't approved. | The approval list in §3; agents suggest commands, the owner says go. |
