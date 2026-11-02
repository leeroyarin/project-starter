# SETUP: recreate the app skeleton

> **This is a recipe for Claude Code to execute.** When asked to "set up the project from SETUP.md", work through the steps below in order, in the project's root folder. Write every file exactly as given (code blocks are complete file contents unless a step says otherwise), then run the verify step. Don't commit, push or deploy; the owner does that. If a command prompts or fails, stop and report rather than improvising a different stack.

**Result:** a running skeleton. Vite + React 19 + TypeScript, Tailwind CSS v4, shadcn/ui (`base-nova` style, built on Base UI, every component added), a Hono API on Cloudflare Workers (`/api/health` with a test), an app shell with sidebar, phone menu, theme menu, page/list/state building blocks, toasts, a GET cache and a tiny router. Two example pages (Overview, States) show the building blocks; replace them once real pages exist.

**Before you start**
- Node 22 or newer and pnpm 10 or newer (`node -v`, `pnpm -v`).
- The other starter files should already be in place (see `00-START-HERE.md`): `CLAUDE.md` at the root; the reference docs (`GUIDE.md`, `ROADMAP.md`, `DESIGN_SYSTEM.md`, `DESIGN_INSPIRATION.md`, `WORKFLOW.md`, `RESEARCH.md`, `BACKEND_CLOUDFLARE.md`, `ENV_AND_SECRETS.md`, `SECURITY.md`, `AI_FEATURES.md`, `TESTING.md`, `DEPLOY_AND_CI.md`, `PRE_LAUNCH_COMPLIANCE_CHECKLIST.md`, `SEO_AND_DISCOVERABILITY.md`, `CLAUDE_PLUGINS.md`, `PROGRESS.html`, and `BROWSER_EXTENSION.md` if used) in `docs/`; `GITHUB_TEMPLATES.md` still at the root (step 11 uses it); `agent-*.md` in `.claude/agents/` as `<name>.md`. If they're still loose at the root, move them there first (step 0 in `00-START-HERE.md`). Don't overwrite them.
- Pick the project slug (lowercase kebab-case, e.g. `team-planner`). Below it's written `{{PROJECT_SLUG}}`; replace it everywhere you write it. Leave the other `{{…}}` placeholders as they are; the owner fills them in (roadmap task M0-1).

## Folder map (what you'll end up with)

```text
.
├── CLAUDE.md, docs/, .claude/agents/   From the starter files (already placed)
├── .github/workflows/ci.yml            CI on every PR (step 11)
├── .github/ (templates)                PR template, issue forms, CODEOWNERS (step 11)
├── index.html                          No-flash theme script, title, theme-color
├── public/favicon.svg, public/brand/   Logo files
├── src/
│   ├── main.tsx, App.tsx               Entry and page choice by route
│   ├── index.css                       Theme tokens (light, dark), type scale, radius, shadows, motion
│   ├── components/ui/                  Every shadcn component (base-nova); a few customized in step 7
│   ├── components/                     app-shell, page, states, form-field, form-select, copy-button,
│   │                                   live-region, theme-provider, chunk-boundary
│   ├── features/                       overview/, states/, items/ (examples)
│   ├── lib/                            api, resource, route, route-focus, nav, notify, motion, utils, app
│   └── hooks/use-mobile.ts             Added by shadcn (sidebar)
├── worker/                             Hono API (index.ts, http.ts, env.d.ts, test/health.test.ts)
├── shared/api.ts                       Types shared by the app and the API
├── migrations/0001_init.sql            D1 migrations (an empty example)
└── package.json, pnpm-workspace.yaml, vite.config.ts, vitest.config.ts, tsconfig*.json,
    eslint.config.js, .prettierrc, .prettierignore, components.json, wrangler.jsonc,
    .gitignore, .dev.vars.example
```

## Step 1. Scaffold Vite (React + TypeScript)

In the project folder (it may already hold `CLAUDE.md`, `docs/` and `.claude/`):

`create-vite` cancels in a non-empty folder unless told to delete it, so scaffold into a temporary subfolder (non-interactive, ESLint instead of the default Oxlint, no install) and copy it in. This keeps the starter docs:

```bash
pnpm create vite@latest .vite-tmp --template react-ts --eslint --no-interactive --no-immediate
cp -R .vite-tmp/. . && rm -rf .vite-tmp
```

Then remove the Vite demo files:

```bash
rm -rf src/assets src/App.css public/vite.svg public/icons.svg
```

## Step 2. Add the dependencies

First write `pnpm-workspace.yaml` (below) so pnpm may run the `esbuild` and `workerd` build scripts; without it `pnpm add` fails with "Ignored build scripts". Then add the packages:

```bash
# Tailwind CSS v4 (Vite plugin) and animation helpers
pnpm add tailwindcss @tailwindcss/vite tw-animate-css
# shadcn/ui on Base UI and what its components need
pnpm add shadcn @shadcn/react @base-ui/react class-variance-authority cn lucide-react \
  cmdk date-fns embla-carousel-react input-otp react-day-picker react-resizable-panels recharts
# API: Hono on Cloudflare Workers, zod for validation
pnpm add hono zod
# Cloudflare Vite plugin (runs the Worker inside `pnpm dev`) and Wrangler
pnpm add -D @cloudflare/vite-plugin wrangler
# Tests in the Workers runtime
pnpm add -D vitest @cloudflare/vitest-plugin
# Formatting, Node types, browser checks
pnpm add -D prettier prettier-plugin-tailwindcss @types/node playwright
```

Then write `package.json` exactly as below (the name is your slug; the version ranges are the ones the starter was verified with) and run `pnpm install`.

#### `package.json`

```json
{
  "name": "{{PROJECT_SLUG}}",
  "private": true,
  "version": "0.0.1",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "lint": "eslint .",
    "doctor": "react-doctor --yes --no-telemetry",
    "format": "prettier --write \"**/*.{ts,tsx}\"",
    "typecheck": "tsc -b",
    "preview": "vite preview",
    "deploy": "pnpm build && wrangler deploy",
    "test": "vitest run",
    "db:migrate:local": "wrangler d1 migrations apply DB --local",
    "db:migrate:remote": "wrangler d1 migrations apply DB --remote",
    "cf-typegen": "wrangler types worker/worker-configuration.d.ts"
  },
  "dependencies": {
    "@base-ui/react": "^1.9.0",
    "@shadcn/react": "^0.3.1",
    "@tailwindcss/vite": "^4",
    "class-variance-authority": "^0.7.1",
    "cmdk": "^1.1.1",
    "cn": "^0.4.0",
    "date-fns": "^4.4.0",
    "embla-carousel-react": "^8.6.0",
    "hono": "^4.13.13",
    "input-otp": "^1.5.0",
    "lucide-react": "^1.55.0",
    "react": "^19.3.0",
    "react-day-picker": "^10.0.2",
    "react-dom": "^19.3.0",
    "react-resizable-panels": "^4.14.4",
    "recharts": "^3.10.1",
    "shadcn": "^4.21.4",
    "tailwindcss": "^4",
    "tw-animate-css": "^1.4.0",
    "zod": "^4.6.6"
  },
  "devDependencies": {
    "@cloudflare/vite-plugin": "^1.63.1",
    "@cloudflare/vitest-plugin": "^1.4.0",
    "@eslint/js": "^10",
    "@types/node": "^24",
    "@types/react": "^19",
    "@types/react-dom": "^19",
    "@vitejs/plugin-react": "^6",
    "eslint": "^10",
    "eslint-plugin-react-hooks": "^7.1.1",
    "eslint-plugin-react-refresh": "^0.5.7",
    "globals": "^17",
    "playwright": "^1",
    "prettier": "^3.9.9",
    "prettier-plugin-tailwindcss": "^0.8.1",
    "react-doctor": "^0.9.18",
    "typescript": "~6.0.2",
    "typescript-eslint": "^8",
    "vite": "^8",
    "vitest": "^5.0.4",
    "wrangler": "^4.149.0"
  },
  "engines": {
    "node": ">=22"
  }
}
```

#### `pnpm-workspace.yaml` (write this before the `pnpm add` commands above)

```yaml
packages: []

allowBuilds:
  esbuild: true
  msw: false
  workerd: true
```

`allowBuilds` must allow `workerd` and `esbuild`, or install fails with "Ignored build scripts".

## Step 3. Config files

Write each file below (overwrite what Vite created).

#### `vite.config.ts`

```ts
import { resolve } from "node:path"
import { cloudflare } from "@cloudflare/vite-plugin"
import tailwindcss from "@tailwindcss/vite"
import react from "@vitejs/plugin-react"
import { defineConfig } from "vite"

// The Cloudflare plugin runs the API Worker (worker/index.ts) inside the dev
// server, so `pnpm dev` serves the app and /api/* together on one port.
export default defineConfig({
  plugins: [react(), tailwindcss(), cloudflare()],
  resolve: {
    alias: {
      "@": resolve(import.meta.dirname, "./src"),
      "@shared": resolve(import.meta.dirname, "./shared"),
    },
  },
})
```

#### `vitest.config.ts`

```ts
import { cloudflareTest } from "@cloudflare/vitest-plugin"
import { defineConfig } from "vitest/config"

// Worker tests run inside the Workers runtime (workerd via Miniflare) with the
// bindings from wrangler.jsonc. With D1 enabled, read the migrations here with
// readD1Migrations("./migrations") and apply them in the test (see docs/GUIDE.md).
export default defineConfig({
  plugins: [
    cloudflareTest(async () => ({
      wrangler: { configPath: "./wrangler.jsonc" },
      miniflare: {
        // Everything the API reads is set here, so tests never depend on a
        // local .dev.vars (it's gitignored; these values override it).
        bindings: { ENVIRONMENT: "test" },
      },
    })),
  ],
  test: {
    include: ["worker/**/*.test.ts"],
    // workerd can't read vitest's on-disk module cache.
    fsModuleCache: false,
    testTimeout: 30_000,
    hookTimeout: 60_000,
  },
})
```

#### `tsconfig.json`

```json
{
  "files": [],
  "references": [
    { "path": "./tsconfig.app.json" },
    { "path": "./tsconfig.node.json" },
    { "path": "./tsconfig.worker.json" }
  ],
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"],
      "@shared/*": ["./shared/*"]
    }
  }
}
```

#### `tsconfig.app.json`

```json
{
  "compilerOptions": {
    "tsBuildInfoFile": "./node_modules/.tmp/tsconfig.app.tsbuildinfo",
    "target": "es2023",
    "lib": ["ES2023", "DOM"],
    "module": "esnext",
    "types": ["vite/client"],
    "allowArbitraryExtensions": true,
    "skipLibCheck": true,

    /* Bundler mode */
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "verbatimModuleSyntax": true,
    "moduleDetection": "force",
    "noEmit": true,
    "jsx": "react-jsx",

    /* Linting */
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "erasableSyntaxOnly": true,
    "noFallthroughCasesInSwitch": true,
    "paths": {
      "@/*": ["./src/*"],
      "@shared/*": ["./shared/*"]
    }
  },
  "include": ["src", "shared"]
}
```

#### `tsconfig.node.json`

```json
{
  "compilerOptions": {
    "tsBuildInfoFile": "./node_modules/.tmp/tsconfig.node.tsbuildinfo",
    "target": "es2023",
    "lib": ["ES2023"],
    "module": "nodenext",
    "types": ["node"],
    "skipLibCheck": true,

    /* Bundler mode */
    "allowImportingTsExtensions": true,
    "verbatimModuleSyntax": true,
    "moduleDetection": "force",
    "noEmit": true,

    /* Linting */
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "erasableSyntaxOnly": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["vite.config.ts", "vitest.config.ts"]
}
```

#### `tsconfig.worker.json`

```json
{
  "compilerOptions": {
    "tsBuildInfoFile": "./node_modules/.tmp/tsconfig.worker.tsbuildinfo",
    "target": "es2023",
    "lib": ["ES2023"],
    "module": "esnext",
    "moduleResolution": "bundler",
    "types": [
      "./worker/worker-configuration.d.ts",
      "@cloudflare/vitest-plugin/types"
    ],
    "skipLibCheck": true,
    "verbatimModuleSyntax": true,
    "moduleDetection": "force",
    "noEmit": true,
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "paths": {
      "@shared/*": ["./shared/*"]
    }
  },
  "include": ["worker", "shared"]
}
```

#### `eslint.config.js`

```js
import js from '@eslint/js'
import globals from 'globals'
import reactHooks from 'eslint-plugin-react-hooks'
import reactRefresh from 'eslint-plugin-react-refresh'
import tseslint from 'typescript-eslint'
import { defineConfig, globalIgnores } from 'eslint/config'

export default defineConfig([
  globalIgnores(['dist', '.wrangler', 'worker/worker-configuration.d.ts']),
  {
    files: ['**/*.{ts,tsx}'],
    extends: [
      js.configs.recommended,
      tseslint.configs.recommended,
      reactHooks.configs.flat.recommended,
      reactRefresh.configs.vite,
    ],
    languageOptions: {
      globals: globals.browser,
    },
    rules: {
      'react-refresh/only-export-components': 'off',
    },
  },
  {
    // Generated by shadcn (carousel, use-mobile) and kept as shipped.
    files: ['src/components/ui/**', 'src/hooks/**'],
    rules: {
      'react-hooks/set-state-in-effect': 'off',
    },
  },
])
```

#### `.prettierrc`

```json
{
  "endOfLine": "lf",
  "semi": false,
  "singleQuote": false,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 80,
  "plugins": ["prettier-plugin-tailwindcss"],
  "tailwindStylesheet": "src/index.css",
  "tailwindFunctions": ["cn", "cva"]
}
```

#### `.prettierignore`

```text
node_modules/
dist/
coverage/
.wrangler/
.pnpm-store/
pnpm-lock.yaml
worker/worker-configuration.d.ts
```

#### `wrangler.jsonc`

```jsonc
{
  "$schema": "node_modules/wrangler/config-schema.json",
  // {{PROJECT_SLUG}}: the Worker name, lowercase kebab-case. Live at
  // https://<name>.<your-subdomain>.workers.dev after `pnpm run deploy`.
  "name": "{{PROJECT_SLUG}}",
  "main": "worker/index.ts",
  "compatibility_date": "2026-10-01",
  "compatibility_flags": ["nodejs_compat"],
  "workers_dev": true,
  "preview_urls": false,
  "assets": {
    "not_found_handling": "single-page-application",
    // API requests reach the Worker; everything else is the React app.
    "run_worker_first": ["/api/*"]
  },
  // Optional: a D1 database. Create it with
  //   npx wrangler d1 create {{PROJECT_SLUG}}
  // paste the ID below, uncomment, then run `pnpm cf-typegen`.
  // "d1_databases": [
  //   {
  //     "binding": "DB",
  //     "database_name": "{{PROJECT_SLUG}}",
  //     "database_id": "{{D1_DATABASE_ID}}",
  //     "migrations_dir": "migrations"
  //   }
  // ],
  // Optional: an R2 bucket for uploaded files (bytes never go in D1).
  //   npx wrangler r2 bucket create {{PROJECT_SLUG}}-files
  // "r2_buckets": [{ "binding": "FILES", "bucket_name": "{{PROJECT_SLUG}}-files" }],
  "vars": {
    "ENVIRONMENT": "production"
  },
  "observability": { "enabled": true }
}
```


#### `.gitignore`

```gitignore
# Logs
logs
*.log
npm-debug.log*
pnpm-debug.log*

# Dependencies and builds
node_modules
dist
dist-ssr
coverage
*.local

# Editor and OS
.vscode/*
!.vscode/extensions.json
.idea
.DS_Store
*.sw?

# Cloudflare, env and keys (never commit secrets)
.wrangler
.dev.vars
.env*
!.env.example
*.pem

# Test output
test-results
playwright-report

# Remote database exports (personal data)
backups
```

#### `.dev.vars.example`

```bash
# Copy to .dev.vars for local development. Never commit .dev.vars.
# Secrets in production go in with `npx wrangler secret put NAME`.
ENVIRONMENT=development
# Add local-only values and secrets here, e.g.
# ANTHROPIC_API_KEY=
```

Then copy it for local work: `cp .dev.vars.example .dev.vars`.

#### `index.html`

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/favicon.svg" />
    <!-- {{BRAND_COLOR}}: the same colour as --primary in src/index.css. -->
    <meta name="theme-color" content="#2563EB" />
    <meta name="description" content="{{PROJECT_DESCRIPTION}}" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>{{PROJECT_NAME}}</title>
    <!-- No-flash theme: apply the saved (or system) theme before first paint. -->
    <script>
      try {
        var t = localStorage.getItem("theme")
        var dark =
          t === "dark" ||
          ((!t || t === "system") &&
            matchMedia("(prefers-color-scheme: dark)").matches)
        document.documentElement.classList.add(dark ? "dark" : "light")
      } catch (e) {}
    </script>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

#### `public/favicon.svg`

```svg
<svg width="32" height="32" viewBox="0 0 32 32" fill="none" xmlns="http://www.w3.org/2000/svg">
  <!-- {{LOGO}}: replace with your brand mark. Fill = {{BRAND_COLOR}}. -->
  <rect width="32" height="32" rx="8" fill="#2563EB"/>
  <path d="M10 22V10h7a4 4 0 0 1 0 8h-7" stroke="#fff" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"/>
</svg>
```

#### `public/brand/README.md`

```md
# Brand files

Put the project's logos, marks and brand assets here (`{{LOGO}}`), with a dark-mode version of any logo that has dark text. Reference them as `/brand/<file>` in the app. Keep every brand file, even unused ones (owner's rule).
```

## Step 4. The Worker API, shared types and migrations

`/api/*` reaches the Worker (`run_worker_first` in `wrangler.jsonc`); everything else is the React app. Errors are JSON `{ error }`; every API response is `Cache-Control: private, no-store`.

#### `shared/api.ts`

```ts
// The contract between the React app (src/) and the Worker API (worker/).
// Both sides import these types; change them deliberately and on both sides.

// Every error response: { error: "What happened and what to do" }.
export type ApiError = { error: string }

// GET /api/health
export type HealthResponse = {
  ok: true
  // "production", "development" or "test" (wrangler.jsonc / .dev.vars).
  environment: string
  // ISO time on the server, so the app can show the API answered just now.
  time: string
}
```

#### `worker/http.ts`

```ts
// Errors the API means to send: thrown anywhere in a handler, turned into
// { error } JSON with the right status by app.onError (worker/index.ts).
export class HttpError extends Error {
  status: 400 | 401 | 403 | 404 | 409 | 413 | 429 | 503
  constructor(status: HttpError["status"], message: string) {
    super(message)
    this.status = status
  }
}

export type AppEnv = { Bindings: Env }
```

#### `worker/env.d.ts`

```ts
// Optional secrets that `wrangler types` can't see (set with
// `npx wrangler secret put NAME`, locally in .dev.vars). Add yours here.
interface Env {
  EXAMPLE_SECRET?: string
}
declare namespace Cloudflare {
  interface Env {
    EXAMPLE_SECRET?: string
  }
}
```

#### `worker/index.ts`

```ts
import { Hono } from "hono"
import type { HealthResponse } from "../shared/api"
import { HttpError, type AppEnv } from "./http"

// The API Worker. Requests to /api/* land here (wrangler.jsonc
// "run_worker_first"); everything else is the React app from dist/.
const app = new Hono<AppEnv>().basePath("/api")

// User data must never be cached by the browser or anything in between.
app.use("*", async (c, next) => {
  await next()
  c.res.headers.set("Cache-Control", "private, no-store")
})

app.get("/health", (c) =>
  c.json<HealthResponse>({
    ok: true,
    environment: c.env.ENVIRONMENT,
    time: new Date().toISOString(),
  }),
)

// Add routes here, e.g. app.route("/", items) from worker/routes/items.ts.
// Validate every request body with zod; never trust who the client says it is.

app.notFound((c) => c.json({ error: "There's no API endpoint here." }, 404))

app.onError((error, c) => {
  if (error instanceof HttpError) {
    return c.json({ error: error.message }, error.status)
  }
  // Log for the owner; never send internals (stack traces, SQL) to the client.
  console.error("Unhandled API error:", error instanceof Error ? error.message : "unknown")
  return c.json({ error: "Something went wrong on our side. Try again in a moment." }, 500)
})

export default app
```

#### `worker/test/health.test.ts`

```ts
import { env } from "cloudflare:test"
import { describe, expect, it } from "vitest"
import type { HealthResponse } from "../../shared/api"
import app from "../index"

describe("GET /api/health", () => {
  it("answers ok, uncached", async () => {
    const response = await app.fetch(new Request("https://app.test/api/health"), env)
    expect(response.status).toBe(200)
    expect(response.headers.get("Cache-Control")).toBe("private, no-store")
    const body = (await response.json()) as HealthResponse
    expect(body.ok).toBe(true)
    expect(body.environment).toBe("test")
  })

  it("answers unknown paths with a JSON 404", async () => {
    const response = await app.fetch(new Request("https://app.test/api/nope"), env)
    expect(response.status).toBe(404)
    expect(await response.json()).toEqual({ error: "There's no API endpoint here." })
  })
})
```

#### `migrations/0001_init.sql`

```sql
-- Migration 0001: the first schema. Applied with `pnpm db:migrate:local`
-- (and `pnpm db:migrate:remote` with the owner's go-ahead) once the D1
-- binding in wrangler.jsonc is enabled.
--
-- Rules: one numbered file per change (0002_<what>.sql, ...). Never edit a
-- migration that has been applied anywhere; add a new one instead.
--
-- Example (uncomment and adapt):
-- CREATE TABLE items (
--   id TEXT PRIMARY KEY,
--   title TEXT NOT NULL,
--   created_by TEXT NOT NULL,
--   created_at TEXT NOT NULL DEFAULT (strftime('%Y-%m-%dT%H:%M:%fZ', 'now')),
--   updated_at TEXT NOT NULL DEFAULT (strftime('%Y-%m-%dT%H:%M:%fZ', 'now'))
-- );
-- CREATE INDEX items_created_by ON items (created_by);
SELECT 1;
```

Generate the Worker types (creates `worker/worker-configuration.d.ts`, which `tsconfig.worker.json` needs; rerun after every `wrangler.jsonc` change):

```bash
pnpm cf-typegen
```

## Step 5. shadcn/ui: init and add every component

Write `components.json` (this is what `shadcn init` produces for Vite + Base UI + the Nova style + Neutral; writing it directly avoids the prompts):

#### `components.json`

```json
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "base-nova",
  "rsc": false,
  "tsx": true,
  "tailwind": {
    "config": "",
    "css": "src/index.css",
    "baseColor": "neutral",
    "cssVariables": true,
    "prefix": ""
  },
  "iconLibrary": "lucide",
  "rtl": false,
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils",
    "ui": "@/components/ui",
    "lib": "@/lib",
    "hooks": "@/hooks"
  },
  "menuColor": "default",
  "menuAccent": "subtle",
  "registries": {}
}
```

Make sure `src/index.css` exists and starts with `@import "tailwindcss";` (step 6 replaces it fully afterwards), then add **every** component. The owner keeps all of them, even unused ones.

```bash
printf '@import "tailwindcss";\n' > src/index.css
pnpm dlx shadcn@latest add --all --yes --overwrite
```

If `--all` isn't supported by the CLI version, add them by name:

```bash
pnpm dlx shadcn@latest add --yes --overwrite accordion alert alert-dialog aspect-ratio attachment avatar \
  badge breadcrumb bubble button button-group calendar card carousel chart checkbox collapsible combobox \
  command context-menu dialog direction drawer dropdown-menu empty field hover-card input input-group \
  input-otp item kbd label marker menubar message message-scroller native-select navigation-menu \
  pagination popover progress questionnaire radio-group resizable scroll-area select separator sheet \
  sidebar skeleton slider spinner switch table tabs textarea toast toggle toggle-group tooltip
```

This also creates `src/hooks/use-mobile.ts` (used by `sidebar`). Steps 6 to 8 overwrite `src/index.css` and `src/lib/utils.ts` that shadcn touched.

## Step 6. Theme tokens: `src/index.css`

The whole token block. `{{BRAND_COLOR}}`: every value marked `BRAND` is the brand colour in oklch (the default is a neutral blue, `#2563EB`); neutrals carry a faint tint of the brand hue (262). Form controls use the soft `--field` borders, not `--border`. Motion tokens: 150 / 200 / 300 ms, enter eases out, exit eases in, everything off with reduced motion.

#### `src/index.css`

```css
@import "tailwindcss";
@import "tw-animate-css";
@import "shadcn/tailwind.css";

@custom-variant dark (&:is(.dark *));

/* Design tokens. Rules and the reasons behind them: docs/DESIGN_SYSTEM.md.
   {{BRAND_COLOR}}: replace every "BRAND" value below with your brand colour in
   oklch (https://oklch.com). The default is a neutral blue (#2563EB). Neutrals
   carry a faint tint of the brand hue (262); change the hue to match.
   Check contrast numerically after any change (DESIGN_SYSTEM §2). */
:root,
.light {
  --background: oklch(1 0 0);
  --foreground: oklch(0.2 0.006 262);
  --card: oklch(1 0 0);
  --card-foreground: oklch(0.2 0.006 262);
  --popover: oklch(1 0 0);
  --popover-foreground: oklch(0.2 0.006 262);
  --primary: oklch(0.546 0.215 262.9); /* BRAND */
  --primary-foreground: oklch(1 0 0);
  /* Brand colour for text and links. Fills keep --primary. */
  --primary-ink: oklch(0.546 0.215 262.9); /* BRAND */
  --secondary: oklch(0.954 0.006 262);
  --secondary-foreground: oklch(0.2 0.006 262);
  --muted: oklch(0.965 0.003 262);
  /* Secondary text: about 5.5:1 on cards, 5:1 on --muted. */
  --muted-foreground: oklch(0.52 0.008 262);
  /* Decoration only (· separators, list markers): never for text to read. */
  --subtle-foreground: oklch(0.6 0.008 262);
  --accent: oklch(0.94 0.03 262); /* BRAND tint */
  --accent-foreground: oklch(0.546 0.215 262.9); /* BRAND */
  --destructive: oklch(0.53 0.2 27);
  --destructive-foreground: oklch(1 0 0);
  /* One stroke for every surface: cards, lists, dividers, sidebar, menus,
     dialogs. Form controls use --field. */
  --border: oklch(0.922 0.004 262);
  /* Only for dark-mode control fills (dark:bg-input/30), never borders. */
  --input: oklch(0.94 0 0);
  /* Form control borders: soft, near-neutral. Hover deepens towards the
     brand rather than to near-black. */
  --field: oklch(0.87 0.006 262);
  --field-hover: oklch(0.76 0.012 262);
  /* Small controls (checkbox, radio) need a stronger edge to be seen. */
  --field-strong: oklch(0.62 0.01 262);
  --ring: oklch(0.546 0.215 262.9); /* BRAND */
  /* Row and item feedback: a wash of the ink, so it works on any surface. */
  --hover: oklch(0.2 0.006 262 / 0.04);
  --selected: oklch(0.2 0.006 262 / 0.065);
  --chart-1: oklch(0.546 0.215 262.9);
  --chart-2: oklch(0.7459 0.1483 156.4499);
  --chart-3: oklch(0.58 0.13 236);
  --chart-4: oklch(0.68 0.16 60);
  --chart-5: oklch(0.559 0 0);
  /* The sidebar is white chrome, like the top bar. */
  --sidebar: var(--background);
  --sidebar-foreground: var(--foreground);
  --sidebar-primary: var(--foreground);
  --sidebar-primary-foreground: oklch(1 0 0);
  --sidebar-accent: var(--selected);
  --sidebar-accent-foreground: var(--foreground);
  --sidebar-border: var(--border);
  --sidebar-ring: var(--ring);
  --radius: 0.625rem;
  /* The page behind cards: a touch darker than cards so they stand out. */
  --canvas: oklch(0.982 0.002 262);
  /* Very soft washes on the canvas (keep them faint). */
  --wash-1: oklch(0.546 0.215 262.9 / 0.03); /* BRAND */
  --wash-2: oklch(0.58 0.13 236 / 0.03);
  /* Success green: dark enough for text on its light tint (AA). */
  --success: oklch(0.48 0.12 155);
  /* Elevation: cards barely lift; menus, popovers, ⌘K and toasts share one
     floating shadow with the surface stroke built in. */
  --elev-card: 0 1px 2px oklch(0.2 0.02 262 / 0.04);
  --elev-bar: 0 1px 6px oklch(0.2 0.02 262 / 0.06);
  --elev-pop:
    0 0 0 1px var(--border), 0 8px 24px -6px oklch(0.2 0.02 262 / 0.16);
}

/* Dark layers: canvas 0.18 < chrome 0.205 < card 0.228 < popover 0.255, so
   each step is visible and menus don't melt into cards. */
.dark {
  --background: oklch(0.205 0.005 262);
  --foreground: oklch(0.9551 0 0);
  --card: oklch(0.228 0.006 262);
  --card-foreground: oklch(0.9551 0 0);
  --popover: oklch(0.255 0.007 262);
  --popover-foreground: oklch(0.9551 0 0);
  --primary: oklch(0.546 0.215 262.9); /* BRAND */
  --primary-foreground: oklch(1 0 0);
  /* The brand fill is often too dark to read as text here; a lighter tint
     of it is (BRAND, lightness ~0.72). */
  --primary-ink: oklch(0.72 0.14 262);
  --secondary: oklch(0.294 0.013 262);
  --secondary-foreground: oklch(0.9551 0 0);
  --muted: oklch(0.27 0.008 262);
  --muted-foreground: oklch(0.72 0.006 262);
  --subtle-foreground: oklch(0.62 0.006 262);
  --accent: oklch(0.28 0.037 262);
  --accent-foreground: oklch(0.72 0.14 262);
  --destructive: oklch(0.8 0.11 22);
  --destructive-foreground: oklch(1 0 0);
  /* Alpha, so the one surface stroke works on every dark layer. */
  --border: oklch(1 0 0 / 0.08);
  --input: oklch(0.329 0.009 262);
  --field: oklch(0.4 0.01 262);
  --field-hover: oklch(0.5 0.014 262);
  --field-strong: oklch(0.56 0.01 262);
  --ring: oklch(0.72 0.14 262);
  --hover: oklch(1 0 0 / 0.04);
  --selected: oklch(1 0 0 / 0.07);
  --chart-1: oklch(0.72 0.14 262);
  --chart-2: oklch(0.8003 0.1821 151.711);
  --chart-3: oklch(0.7697 0.1218 229.8);
  --chart-4: oklch(0.78 0.14 70);
  --chart-5: oklch(0.7058 0 0);
  --sidebar: var(--background);
  --sidebar-foreground: var(--foreground);
  --sidebar-primary: var(--primary);
  --sidebar-primary-foreground: oklch(1 0 0);
  --sidebar-accent: var(--selected);
  --sidebar-accent-foreground: var(--foreground);
  --sidebar-border: var(--border);
  --sidebar-ring: var(--ring);
  --canvas: oklch(0.18 0.004 262);
  --wash-1: oklch(0.546 0.215 262.9 / 0.05);
  --wash-2: oklch(0.7697 0.1218 229.8 / 0.025);
  --success: oklch(0.78 0.15 155);
  /* The border does the work in dark mode. */
  --elev-card: none;
  --elev-bar: 0 1px 6px oklch(0 0 0 / 0.35);
  --elev-pop: 0 0 0 1px var(--border), 0 12px 32px -8px oklch(0 0 0 / 0.6);
}

/* Very wide screens (1920px and up). */
@theme {
  --breakpoint-3xl: 120rem;
}

@theme inline {
  /* {{FONT}}: the UI font. Brand display fonts are for brand material. */
  --font-sans: "Helvetica Neue", Helvetica, Arial, sans-serif;
  --font-heading: var(--font-sans);

  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-canvas: var(--canvas);
  --color-success: var(--success);
  --color-card: var(--card);
  --color-card-foreground: var(--card-foreground);
  --color-popover: var(--popover);
  --color-popover-foreground: var(--popover-foreground);
  --color-primary: var(--primary);
  --color-primary-foreground: var(--primary-foreground);
  --color-primary-ink: var(--primary-ink);
  --color-secondary: var(--secondary);
  --color-secondary-foreground: var(--secondary-foreground);
  --color-muted: var(--muted);
  --color-muted-foreground: var(--muted-foreground);
  --color-subtle-foreground: var(--subtle-foreground);
  --color-hover: var(--hover);
  --color-selected: var(--selected);
  --color-accent: var(--accent);
  --color-accent-foreground: var(--accent-foreground);
  --color-destructive: var(--destructive);
  --color-destructive-foreground: var(--destructive-foreground);
  --color-border: var(--border);
  --color-input: var(--input);
  --color-field: var(--field);
  --color-field-hover: var(--field-hover);
  --color-field-strong: var(--field-strong);
  --color-ring: var(--ring);
  --color-chart-1: var(--chart-1);
  --color-chart-2: var(--chart-2);
  --color-chart-3: var(--chart-3);
  --color-chart-4: var(--chart-4);
  --color-chart-5: var(--chart-5);
  --color-sidebar: var(--sidebar);
  --color-sidebar-foreground: var(--sidebar-foreground);
  --color-sidebar-primary: var(--sidebar-primary);
  --color-sidebar-primary-foreground: var(--sidebar-primary-foreground);
  --color-sidebar-accent: var(--sidebar-accent);
  --color-sidebar-accent-foreground: var(--sidebar-accent-foreground);
  --color-sidebar-border: var(--sidebar-border);
  --color-sidebar-ring: var(--sidebar-ring);

  /* Radius scale: kbd 4, badges and chips 6, buttons, inputs and nav items
     10, cards and menus 12, dialogs and sheets 16. Never pills. */
  --radius-sm: 4px;
  --radius-md: 6px;
  --radius-lg: 10px;
  --radius-xl: 12px;
  --radius-2xl: 16px;

  --shadow-card: var(--elev-card);
  --shadow-pop: var(--elev-pop);
  --shadow-bar: var(--elev-bar);

  /* Extra type sizes. Meta: list meta, help text, times. Reading: long-form
     text. Doc: a document's title. Registered with cn in src/lib/utils.ts. */
  --text-meta: 0.8125rem;
  --text-meta--line-height: 1.125rem;
  --text-reading: 0.9375rem;
  --text-reading--line-height: 1.625rem;
  --text-doc: 1.375rem;
  --text-doc--line-height: 1.875rem;

  /* iOS-style spring curve (UIKit sheet presentation). */
  --ease-ios: cubic-bezier(0.32, 0.72, 0, 1);

  /* Motion system (DESIGN_SYSTEM §15). Three durations: 150 ms for small
     feedback (hover, press, menus), 200 ms for fades and exits, 300 ms for
     anything that moves into place (dialogs, sheets, toasts, new content).
     Enter eases out; exit eases in. Only opacity and transform animate.
     Everything is off with reduced motion. */
  --ease-enter: cubic-bezier(0.32, 0.72, 0, 1);
  --ease-exit: cubic-bezier(0.4, 0, 1, 1);
  --animate-fade: app-fade 200ms var(--ease-enter) backwards;
  --animate-enter: app-enter 300ms var(--ease-enter) backwards;
  --animate-enter-up: app-enter-up 300ms var(--ease-enter) backwards;
}

/* Fade in: route changes, skeleton-to-content swaps. */
@keyframes app-fade {
  from {
    opacity: 0;
  }
}
/* Fade and settle 4 px: list rows, cards. */
@keyframes app-enter {
  from {
    opacity: 0;
    transform: translateY(4px);
  }
}
/* The same, rising 8 px: content that appears below the reader's eye. */
@keyframes app-enter-up {
  from {
    opacity: 0;
    transform: translateY(8px);
  }
}

/* Rows of a list settle in one after another when they first appear: 30 ms
   apart, and nothing waits longer than 150 ms. */
@media (prefers-reduced-motion: no-preference) {
  .list-enter > * {
    animation: app-enter 300ms var(--ease-enter) backwards;
  }
  .list-enter > :nth-child(2) {
    animation-delay: 30ms;
  }
  .list-enter > :nth-child(3) {
    animation-delay: 60ms;
  }
  .list-enter > :nth-child(4) {
    animation-delay: 90ms;
  }
  .list-enter > :nth-child(5) {
    animation-delay: 120ms;
  }
  .list-enter > :nth-child(n + 6) {
    animation-delay: 150ms;
  }
}

/* Safety net: with reduced motion nothing slides, scales or fades, even in
   third-party primitives. Colour changes still happen, just instantly.
   Delays are kept, so delayed skeletons still stay hidden on quick loads. */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}

/* The working area: the canvas colour with two very soft washes. Fixed, so
   it stays put while the page scrolls. */
@utility canvas-surface {
  background-color: var(--canvas);
  background-image:
    radial-gradient(70% 55% at 100% 0%, var(--wash-1), transparent 70%),
    radial-gradient(60% 50% at 0% 100%, var(--wash-2), transparent 70%);
  background-attachment: fixed;
}

@layer base {
  * {
    @apply border-border outline-ring/50;
  }
  /* Lighter icon strokes sit better beside 14 px text. */
  .lucide {
    stroke-width: 1.75;
  }
  html {
    @apply font-sans;
    /* Keep focused and scrolled-to elements clear of the sticky top bar
       (WCAG 2.4.11). */
    scroll-padding-top: 4.5rem;
    scroll-padding-bottom: 1rem;
  }
  /* Large monitors: scale the whole interface up a little. Everything is in
     rem, so text, spacing and controls grow together. */
  @media (min-width: 1920px) {
    html {
      font-size: 106.25%;
    }
  }
  @media (min-width: 2400px) {
    html {
      font-size: 118.75%;
    }
  }
  /* Anything clickable shows the hand, not the arrow. */
  button:not(:disabled),
  [role="button"]:not([aria-disabled="true"]),
  summary {
    cursor: pointer;
  }
  @media (prefers-reduced-motion: no-preference) {
    html {
      scroll-behavior: smooth;
    }
  }
  /* Headings that receive focus after a route change (tabindex=-1) are not
     controls, so no ring. */
  h1[tabindex="-1"]:focus {
    outline: none;
  }
  body {
    @apply bg-background text-foreground;
    line-height: 1.55;
  }
}
```

## Step 7. Customize the shadcn primitives

These follow `docs/DESIGN_SYSTEM.md`. Three files are replaced in full; the rest get the class changes in the table.

#### `src/components/ui/button.tsx`

```tsx
import { isValidElement } from "react"
import type * as React from "react"
import { Button as ButtonPrimitive } from "@base-ui/react/button"
import { useRender } from "@base-ui/react/use-render"
import { cva, type VariantProps } from "class-variance-authority"
import { cn } from "@/lib/utils"

// Shape and depth follow the owner's standard buttons: a 10px corner at every
// size (owner's rule), a subtle inset highlight on the primary fill, and a
// quick press that respects reduced motion.
const buttonVariants = cva(
  "group/button inline-flex shrink-0 items-center justify-center rounded-lg border border-transparent bg-clip-padding text-sm font-medium whitespace-nowrap transition-[background-color,color,box-shadow,transform] duration-150 ease-out outline-none select-none focus-visible:border-ring focus-visible:ring-3 focus-visible:ring-ring/50 active:not-aria-[haspopup]:scale-[0.98] disabled:pointer-events-none disabled:opacity-50 aria-invalid:border-destructive aria-invalid:ring-3 aria-invalid:ring-destructive/20 motion-reduce:active:scale-100 dark:aria-invalid:border-destructive/50 dark:aria-invalid:ring-destructive/40 [&_svg]:pointer-events-none [&_svg]:shrink-0 [&_svg:not([class*='size-'])]:size-4",
  {
    variants: {
      variant: {
        default:
          "border-black/10 bg-primary text-primary-foreground shadow-[inset_0_1px_0_rgb(255_255_255/0.3),0_1px_2px_rgb(16_16_32/0.1)] hover:bg-[color-mix(in_oklch,var(--primary),black_12%)]",
        outline:
          "border-border bg-card shadow-[0_1px_2px_rgb(16_16_32/0.04)] hover:bg-muted hover:text-foreground aria-expanded:bg-muted aria-expanded:text-foreground dark:bg-input/30 dark:hover:bg-input/50",
        secondary:
          "bg-secondary text-secondary-foreground hover:bg-[color-mix(in_oklch,var(--secondary),var(--foreground)_5%)] aria-expanded:bg-secondary aria-expanded:text-secondary-foreground",
        ghost:
          "hover:bg-muted hover:text-foreground aria-expanded:bg-muted aria-expanded:text-foreground dark:hover:bg-muted/50",
        destructive:
          "bg-destructive/10 text-destructive hover:bg-destructive/20 focus-visible:border-destructive/40 focus-visible:ring-destructive/20 dark:bg-destructive/20 dark:hover:bg-destructive/30 dark:focus-visible:ring-destructive/40",
        link: "text-primary-ink underline-offset-4 hover:underline",
      },
      size: {
        default:
          "h-9 gap-1.5 px-3.5 has-data-[icon=inline-end]:pr-3 has-data-[icon=inline-start]:pl-3",
        xs: "h-6 gap-1 rounded-lg px-2 text-xs has-data-[icon=inline-end]:pr-1.5 has-data-[icon=inline-start]:pl-1.5 [&_svg:not([class*='size-'])]:size-3",
        sm: "h-8 gap-1 rounded-lg px-3 text-[0.8rem] has-data-[icon=inline-end]:pr-1.5 has-data-[icon=inline-start]:pl-1.5 [&_svg:not([class*='size-'])]:size-3.5",
        lg: "h-11 gap-2 px-5 text-[0.95rem] has-data-[icon=inline-end]:pr-5 has-data-[icon=inline-start]:pl-5",
        icon: "size-9",
        "icon-xs": "size-6 rounded-lg [&_svg:not([class*='size-'])]:size-3",
        "icon-sm": "size-7 rounded-lg",
        "icon-lg": "size-11",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
)

type ButtonProps = ButtonPrimitive.Props & VariantProps<typeof buttonVariants>

function Button({
  className,
  variant = "default",
  size = "default",
  ...props
}: ButtonProps) {
  const classes = cn(buttonVariants({ variant, size, className }))
  // A link styled as a button (`render={<a href=… />}`) stays a link for
  // screen readers: Base UI's Button would add role="button" to it.
  if (isValidElement(props.render) && props.render.type === "a") {
    return <ButtonLink {...props} className={classes} />
  }
  return <ButtonPrimitive data-slot="button" className={classes} {...props} />
}

function ButtonLink({
  render,
  className,
  ...props
}: Omit<ButtonProps, "className" | "variant" | "size"> & {
  className: string
}) {
  // nativeButton only matters to Base UI's Button; keep it off the <a>.
  const rest: Record<string, unknown> = { ...props }
  delete rest.nativeButton
  return useRender({
    render: render as React.ReactElement,
    props: { "data-slot": "button", className, ...rest },
  })
}

export { Button, buttonVariants }
```

#### `src/components/ui/toast.tsx`

```tsx
"use client"

import * as React from "react"
import { Toast as ToastPrimitive } from "@base-ui/react/toast"
import { cn } from "@/lib/utils"

import { Button } from "@/components/ui/button"
import {
  XIcon,
  CircleCheckIcon,
  InfoIcon,
  TriangleAlertIcon,
  OctagonXIcon,
  Loader2Icon,
} from "lucide-react"

const toast = ToastPrimitive.createToastManager()

function ToastProvider({ ...props }: ToastPrimitive.Provider.Props) {
  return <ToastPrimitive.Provider {...props} />
}

function ToastPortal({ ...props }: ToastPrimitive.Portal.Props) {
  return <ToastPrimitive.Portal data-slot="toast-portal" {...props} />
}

function ToastViewport({ className, ...props }: ToastPrimitive.Viewport.Props) {
  return (
    <ToastPrimitive.Viewport
      data-slot="toast-viewport"
      className={cn(
        // Raised (bottom-28) so a toast never covers a sticky composer or action bar
        // at the bottom of the page.
        "pointer-events-none fixed inset-x-4 bottom-28 z-50 mx-auto w-auto max-w-sm outline-none sm:right-4 sm:left-auto sm:mx-0 sm:w-full",
        className
      )}
      {...props}
    />
  )
}

function Toast({ className, ...props }: ToastPrimitive.Root.Props) {
  return (
    <ToastPrimitive.Root
      data-slot="toast"
      className={cn(
        "group/toast pointer-events-auto absolute right-0 bottom-0 z-[calc(1000-var(--toast-index))] w-full origin-bottom rounded-xl bg-popover text-popover-foreground shadow-pop will-change-transform outline-none select-none focus-visible:border-ring focus-visible:ring-[3px] focus-visible:ring-ring/50",
        "[--gap:0.75rem] [--height:var(--toast-frontmost-height,var(--toast-height))] [--offset-y:calc(var(--toast-offset-y)*-1+calc(var(--toast-index)*var(--gap)*-1)+var(--toast-swipe-movement-y))] [--peek:0.75rem] [--scale:calc(max(0,1-(var(--toast-index)*0.1)))] [--shrink:calc(1-var(--scale))]",
        "h-(--height) [transform:translateX(var(--toast-swipe-movement-x))_translateY(calc(var(--toast-swipe-movement-y)-(var(--toast-index)*var(--peek))-(var(--shrink)*var(--height))))_scale(var(--scale))] [transition:transform_300ms_var(--ease-enter),opacity_200ms,height_150ms] data-ending-style:[transition:transform_200ms_var(--ease-exit),opacity_150ms]",
        "after:absolute after:top-full after:left-0 after:h-[calc(var(--gap)+1px)] after:w-full after:content-['']",
        "data-expanded:h-(--toast-height) data-expanded:[transform:translateX(var(--toast-swipe-movement-x))_translateY(var(--offset-y))]",
        "data-limited:opacity-0 data-starting-style:[transform:translateY(150%)] data-starting-style:opacity-0",
        "[&[data-ending-style]:not([data-limited]):not([data-swipe-direction])]:[transform:translateY(150%)]",
        "data-ending-style:data-[swipe-direction=down]:[transform:translateY(calc(var(--toast-swipe-movement-y)+150%))]",
        "data-ending-style:data-[swipe-direction=left]:[transform:translateX(calc(var(--toast-swipe-movement-x)-150%))_translateY(var(--offset-y))]",
        "data-ending-style:data-[swipe-direction=right]:[transform:translateX(calc(var(--toast-swipe-movement-x)+150%))_translateY(var(--offset-y))]",
        "data-ending-style:data-[swipe-direction=up]:[transform:translateY(calc(var(--toast-swipe-movement-y)-150%))]",
        "data-expanded:data-ending-style:data-[swipe-direction=down]:[transform:translateY(calc(var(--toast-swipe-movement-y)+150%))]",
        "data-expanded:data-ending-style:data-[swipe-direction=left]:[transform:translateX(calc(var(--toast-swipe-movement-x)-150%))_translateY(var(--offset-y))]",
        "data-expanded:data-ending-style:data-[swipe-direction=right]:[transform:translateX(calc(var(--toast-swipe-movement-x)+150%))_translateY(var(--offset-y))]",
        "data-expanded:data-ending-style:data-[swipe-direction=up]:[transform:translateY(calc(var(--toast-swipe-movement-y)-150%))]",
        className
      )}
      {...props}
    />
  )
}

function ToastContent({ className, ...props }: ToastPrimitive.Content.Props) {
  return (
    <ToastPrimitive.Content
      data-slot="toast-content"
      className={cn(
        "flex h-full items-center gap-2.5 overflow-hidden py-2.5 pr-2.5 pl-3.5 text-sm transition-opacity duration-200 ease-enter data-behind:opacity-0 data-expanded:opacity-100",
        className
      )}
      {...props}
    />
  )
}

function ToastTitle({ className, ...props }: ToastPrimitive.Title.Props) {
  return (
    <ToastPrimitive.Title
      data-slot="toast-title"
      className={cn("text-sm font-medium", className)}
      {...props}
    />
  )
}

function ToastDescription({
  className,
  ...props
}: ToastPrimitive.Description.Props) {
  return (
    <ToastPrimitive.Description
      data-slot="toast-description"
      className={cn("text-meta text-muted-foreground", className)}
      {...props}
    />
  )
}

function ToastAction({
  className,
  render = <Button variant="outline" size="sm" />,
  ...props
}: ToastPrimitive.Action.Props) {
  return (
    <ToastPrimitive.Action
      data-slot="toast-action"
      render={render}
      className={cn("shrink-0", className)}
      {...props}
    />
  )
}

function ToastClose({
  className,
  children,
  render = <Button variant="ghost" size="icon-xs" />,
  ...props
}: ToastPrimitive.Close.Props) {
  return (
    <ToastPrimitive.Close
      data-slot="toast-close"
      aria-label="Close toast"
      // Base UI hides the button from screen readers until the stack is
      // expanded, yet it stays focusable (axe aria-hidden-focus); it's
      // visible, so keep it announced.
      aria-hidden={undefined}
      render={render}
      className={cn(
        "relative shrink-0 text-muted-foreground after:absolute after:-inset-1.5 after:content-[''] hover:text-foreground",
        className
      )}
      {...props}
    >
      {children ?? <XIcon aria-hidden="true" />}
    </ToastPrimitive.Close>
  )
}

function ToastIcon({ type }: { type: string | undefined }) {
  let icon: React.ReactNode = null

  if (type === "success") {
    icon = <CircleCheckIcon className="text-success" aria-hidden="true" />
  }

  if (type === "info") {
    icon = <InfoIcon className="text-primary-ink" aria-hidden="true" />
  }

  if (type === "warning") {
    icon = <TriangleAlertIcon aria-hidden="true" />
  }

  if (type === "error") {
    icon = <OctagonXIcon className="text-destructive" aria-hidden="true" />
  }

  if (type === "loading") {
    icon = <Loader2Icon className="animate-spin" aria-hidden="true" />
  }

  if (!icon) {
    return null
  }

  return (
    <span
      data-slot="toast-icon"
      className="shrink-0 [&_svg]:pointer-events-none [&_svg:not([class*='size-'])]:size-4"
    >
      {icon}
    </span>
  )
}

function ToastList() {
  const { toasts } = ToastPrimitive.useToastManager()

  // Toasts are status messages, never dialogs (DESIGN_SYSTEM §13): polite
  // for completed actions, assertive for errors.
  return toasts.map((toastItem) => (
    <Toast
      key={toastItem.id}
      toast={toastItem}
      role={toastItem.type === "error" ? "alert" : "status"}
      aria-modal={undefined}
    >
      <ToastContent>
        <ToastIcon type={toastItem.type} />
        <div className="flex min-w-0 flex-1 flex-col gap-0.5">
          <ToastTitle />
          <ToastDescription />
        </div>
        <ToastAction />
        <ToastClose />
      </ToastContent>
    </Toast>
  ))
}

function Toaster({
  children,
  toastManager = toast,
  ...props
}: ToastPrimitive.Provider.Props) {
  return (
    <ToastProvider toastManager={toastManager} {...props}>
      {children}
      <ToastPortal>
        <ToastViewport>
          <ToastList />
        </ToastViewport>
      </ToastPortal>
    </ToastProvider>
  )
}

const createToastManager = ToastPrimitive.createToastManager
const useToastManager = ToastPrimitive.useToastManager

export {
  Toaster,
  Toast,
  ToastAction,
  ToastClose,
  ToastContent,
  ToastDescription,
  ToastPortal,
  ToastProvider,
  ToastTitle,
  ToastViewport,
  createToastManager,
  toast,
  useToastManager,
}
```

#### `src/components/ui/dialog.tsx`

```tsx
"use client"

import * as React from "react"
import { Dialog as DialogPrimitive } from "@base-ui/react/dialog"
import { cn } from "@/lib/utils"

import { Button } from "@/components/ui/button"
import { XIcon } from "lucide-react"

function Dialog({ ...props }: DialogPrimitive.Root.Props) {
  return <DialogPrimitive.Root data-slot="dialog" {...props} />
}

function DialogTrigger({ ...props }: DialogPrimitive.Trigger.Props) {
  return <DialogPrimitive.Trigger data-slot="dialog-trigger" {...props} />
}

function DialogPortal({ ...props }: DialogPrimitive.Portal.Props) {
  return <DialogPrimitive.Portal data-slot="dialog-portal" {...props} />
}

function DialogClose({ ...props }: DialogPrimitive.Close.Props) {
  return <DialogPrimitive.Close data-slot="dialog-close" {...props} />
}

function DialogOverlay({
  className,
  ...props
}: DialogPrimitive.Backdrop.Props) {
  return (
    <DialogPrimitive.Backdrop
      data-slot="dialog-overlay"
      className={cn(
        "fixed inset-0 isolate z-50 bg-black/10 duration-200 supports-backdrop-filter:backdrop-blur-xs data-open:animate-in data-open:fade-in-0 data-closed:animate-out data-closed:fade-out-0",
        className
      )}
      {...props}
    />
  )
}

function DialogContent({
  className,
  children,
  showCloseButton = true,
  sheetOnPhone = true,
  ...props
}: DialogPrimitive.Popup.Props & {
  showCloseButton?: boolean
  // Phones get a bottom sheet with a grab handle (DESIGN_SYSTEM, dialogs).
  // ⌘K search turns it off and stays a floating panel.
  sheetOnPhone?: boolean
}) {
  return (
    <DialogPortal>
      <DialogOverlay />
      <DialogPrimitive.Popup
        data-slot="dialog-content"
        data-sheet={sheetOnPhone || undefined}
        className={cn(
          "fixed z-50 grid w-full gap-4 bg-popover p-5 text-sm text-popover-foreground shadow-pop duration-300 ease-enter outline-none data-open:animate-in data-open:fade-in-0 data-closed:animate-out data-closed:duration-150 data-closed:ease-exit data-closed:fade-out-0",
          sheetOnPhone
            ? "max-sm:inset-x-0 max-sm:bottom-0 max-sm:max-h-[92svh] max-sm:overflow-y-auto max-sm:rounded-t-2xl max-sm:pt-7 sm:top-1/2 sm:left-1/2 sm:max-w-sm sm:-translate-x-1/2 sm:-translate-y-1/2 sm:rounded-2xl max-sm:data-open:slide-in-from-bottom-8 sm:data-open:zoom-in-95 max-sm:data-closed:slide-out-to-bottom-8 sm:data-closed:zoom-out-95"
            : "top-1/2 left-1/2 max-w-[calc(100%-2rem)] -translate-x-1/2 -translate-y-1/2 rounded-2xl sm:max-w-sm data-open:zoom-in-95 data-closed:zoom-out-95",
          className
        )}
        {...props}
      >
        {sheetOnPhone && (
          <span
            aria-hidden="true"
            className="absolute top-2.5 left-1/2 h-1 w-9 -translate-x-1/2 rounded-full bg-foreground/15 sm:hidden"
          />
        )}
        {children}
        {showCloseButton && (
          <DialogPrimitive.Close
            data-slot="dialog-close"
            render={
              <Button
                variant="ghost"
                className="absolute top-3 right-3"
                size="icon-sm"
              />
            }
          >
            <XIcon />
            <span className="sr-only">Close</span>
          </DialogPrimitive.Close>
        )}
      </DialogPrimitive.Popup>
    </DialogPortal>
  )
}

function DialogHeader({ className, ...props }: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="dialog-header"
      className={cn("flex flex-col gap-1 pr-8", className)}
      {...props}
    />
  )
}

function DialogFooter({
  className,
  showCloseButton = false,
  children,
  ...props
}: React.ComponentProps<"div"> & {
  showCloseButton?: boolean
}) {
  return (
    <div
      data-slot="dialog-footer"
      className={cn(
        "-mx-5 -mb-5 flex flex-col-reverse gap-2 border-t bg-muted/40 px-5 py-4 max-sm:pb-[max(1rem,env(safe-area-inset-bottom))] sm:flex-row sm:justify-end sm:rounded-b-2xl",
        className
      )}
      {...props}
    >
      {children}
      {showCloseButton && (
        <DialogPrimitive.Close render={<Button variant="outline" />}>
          Close
        </DialogPrimitive.Close>
      )}
    </div>
  )
}

function DialogTitle({ className, ...props }: DialogPrimitive.Title.Props) {
  return (
    <DialogPrimitive.Title
      data-slot="dialog-title"
      className={cn("font-heading text-base/6 font-medium", className)}
      {...props}
    />
  )
}

function DialogDescription({
  className,
  ...props
}: DialogPrimitive.Description.Props) {
  return (
    <DialogPrimitive.Description
      data-slot="dialog-description"
      className={cn(
        "text-meta text-muted-foreground *:[a]:underline *:[a]:underline-offset-3 *:[a]:hover:text-foreground",
        className
      )}
      {...props}
    />
  )
}

export {
  Dialog,
  DialogClose,
  DialogContent,
  DialogDescription,
  DialogFooter,
  DialogHeader,
  DialogOverlay,
  DialogPortal,
  DialogTitle,
  DialogTrigger,
}
```

#### Class changes in the other components

Edit only the class strings named here; keep the rest of each generated file.

| Component(s) | Change |
| --- | --- |
| `input.tsx` | Root class: `h-10 w-full min-w-0 rounded-lg border border-field bg-background px-3.5 py-2 text-base transition-[border-color,box-shadow] duration-200 outline-none file:inline-flex file:h-6 file:border-0 file:bg-transparent file:text-sm file:font-medium file:text-foreground placeholder:text-muted-foreground hover:border-field-hover focus-visible:border-ring focus-visible:ring-3 focus-visible:ring-ring/50 disabled:pointer-events-none disabled:cursor-not-allowed disabled:bg-input/50 disabled:opacity-50 aria-invalid:border-destructive aria-invalid:ring-3 aria-invalid:ring-destructive/20 md:text-sm dark:bg-input/30 dark:disabled:bg-input/80 dark:aria-invalid:border-destructive/50 dark:aria-invalid:ring-destructive/40` |
| `textarea.tsx` | Root class: `flex field-sizing-content min-h-16 w-full rounded-lg border border-field bg-background px-3.5 py-2.5 text-base transition-[border-color,box-shadow] duration-200 outline-none placeholder:text-muted-foreground hover:border-field-hover focus-visible:border-ring focus-visible:ring-3 focus-visible:ring-ring/50 disabled:cursor-not-allowed disabled:bg-input/50 disabled:opacity-50 aria-invalid:border-destructive aria-invalid:ring-3 aria-invalid:ring-destructive/20 md:text-sm dark:bg-input/30 dark:disabled:bg-input/80 dark:aria-invalid:border-destructive/50 dark:aria-invalid:ring-destructive/40` |
| `select.tsx` (trigger), `native-select.tsx`, `input-group.tsx` | `border-input` → `border-field`, add `hover:border-field-hover`, corners `rounded-lg` |
| `combobox.tsx` (chips), `input-otp.tsx` (slot), `toggle.tsx` (`outline` variant), `questionnaire.tsx` | `border-input` → `border-field` |
| `command.tsx` (search `InputGroup`), `combobox.tsx` (input inside the popup) | `border-input/30` → `border-field/30` |
| `checkbox.tsx`, `radio-group.tsx` | `border-input` → `border-field-strong` (small controls need the stronger edge, 3:1) |
| `tabs.tsx` (dark active trigger), `code-block.tsx`, and any other component `--all` added that still has `border-input` | `border-input` → `border-field` |
| `scroll-area.tsx` (not a class change) | Delete the unused `import * as React from "react"` line, or typecheck fails with TS6133 |
| `popover.tsx`, `hover-card.tsx`, `dropdown-menu.tsx`, `context-menu.tsx`, `select.tsx`, `combobox.tsx` (popups) | Surface: `rounded-xl bg-popover shadow-pop duration-150 ease-enter`; remove `ring-1 ring-foreground/10` and `shadow-md`; closing: `data-closed:duration-100 data-closed:ease-exit` |
| `dropdown-menu.tsx`, `context-menu.tsx`, `select.tsx` (items) | Focus/open state: `focus:bg-selected focus:text-foreground` (and `data-open:bg-selected`), items `min-h-8` |
| `alert-dialog.tsx` (content) | `rounded-2xl bg-popover p-5 shadow-pop duration-300 ease-enter`, closing `data-closed:duration-150 data-closed:ease-exit`; description `text-meta` |
| `sheet.tsx` (content) | `duration-300 ease-enter`; closing `data-ending-style:duration-200 data-ending-style:ease-exit` |
| `tooltip.tsx` | `duration-150 ease-enter`, closing `ease-exit` |
| `card.tsx` | `rounded-xl ring-1 ring-border` |
| `empty.tsx` | Root `rounded-xl`; description `text-meta text-muted-foreground` |
| `tabs.tsx` | Trigger transition `duration-150 ease-enter`. Replace `TabsContent` with the version below. |

```tsx
// Panels aren't a tab stop of their own (Base UI makes them focusable, but
// with `outline-none` that stop was invisible, WCAG 2.4.7). The controls
// inside are reachable as usual; pass tabIndex={0} for a panel with none.
function TabsContent({
  className,
  tabIndex = -1,
  ...props
}: TabsPrimitive.Panel.Props) {
  return (
    <TabsPrimitive.Panel
      data-slot="tabs-content"
      tabIndex={tabIndex}
      className={cn(
        // The new panel fades in (200 ms) when the tab changes.
        "flex-1 text-sm outline-none motion-safe:animate-fade",
        className
      )}
      {...props}
    />
  )
}
```

Check afterwards: `grep -rn "border-input" src/components/ui` returns nothing.

## Step 8. Lib helpers (`src/lib/`)

`cn` must come from `@/lib/utils` (it knows the custom type sizes), never from the `cn` package directly. The shadcn CLI writes `import { cn } from "cn"` into the generated components, so after writing `src/lib/utils.ts` point them at it (works with GNU and BSD sed):

```bash
grep -rl 'from "cn"' src --exclude=utils.ts | xargs sed -i.bak 's#from "cn"#from "@/lib/utils"#' && find src -name '*.bak' -delete
grep -rn 'from "cn"' src   # only src/lib/utils.ts may import from cn (cn/config)
```

#### `src/lib/utils.ts`

```ts
import { createCn } from "cn/config"

// `cn` (clsx + tailwind-merge in one) taught this app's custom type scale
// from src/index.css. Without it, `text-meta text-muted-foreground` looks
// like two colours and the size is dropped.
export const cn = createCn({
  extend: {
    classGroups: {
      "font-size": [{ text: ["meta", "reading", "doc"] }],
    },
  },
})
```

#### `src/lib/app.ts`

```ts
// {{PROJECT_NAME}}: the product name shown in tab titles and messages.
export const APP_NAME = "{{PROJECT_NAME}}"

// Prefix for localStorage and sessionStorage keys ({{PROJECT_SLUG}}).
export const STORAGE_PREFIX = "{{PROJECT_SLUG}}"
```

#### `src/lib/motion.ts`

```ts
import type { CSSProperties } from "react"

// Motion system: see DESIGN_SYSTEM §15 and the tokens in src/index.css.

// Cards and rows settle into place (fade + 4 px, 300 ms). Off with reduced
// motion; the animation's `backwards` fill keeps them hidden until their turn.
export const ENTER = "motion-safe:animate-enter"

// A page or a block of content swapping in (fade only, 200 ms).
export const FADE = "motion-safe:animate-fade"

// A short stagger: 30 ms apart, and nothing waits more than 150 ms, so long
// lists never feel slow.
export function enterDelay(index: number): CSSProperties {
  return { animationDelay: `${Math.min(index, 5) * 30}ms` }
}
```

#### `src/lib/route.ts`

```ts
import { useEffect, useSyncExternalStore } from "react"

import { APP_NAME, STORAGE_PREFIX } from "@/lib/app"

// A tiny router: most apps have a handful of pages, so the browser's own
// history is enough. The server sends every path to index.html
// (wrangler.jsonc: not_found_handling "single-page-application").
// Add a route: a member of `Route`, a line in STATIC (or a case in
// parseRoute for /section/:id), a page in App.tsx and, for a section, NAV.

export type Route =
  | { name: "overview" }
  | { name: "states" }
  | { name: "item"; id: string }
  | { name: "not-found" }

const STATIC: Record<string, Route> = {
  "/": { name: "overview" },
  "/states": { name: "states" },
}

export function parseRoute(pathname: string): Route {
  const path =
    pathname.length > 1 && pathname.endsWith("/")
      ? pathname.slice(0, -1)
      : pathname
  if (STATIC[path]) return STATIC[path]
  const [, section, param, extra] = path.split("/")
  if (!param || extra !== undefined) return { name: "not-found" }
  const value = decodeURIComponent(param)
  if (section === "items") return { name: "item", id: value }
  return { name: "not-found" }
}

const CHANGE = `${STORAGE_PREFIX}:navigate`

// Each history entry the app makes carries its position, so a refused
// Back/Forward can step back by the same amount instead of pushing a
// duplicate entry.
type HistoryState = { appIndex: number } | null
let index = (window.history.state as HistoryState)?.appIndex ?? 0
if (!(window.history.state as HistoryState)) {
  window.history.replaceState({ appIndex: index }, "")
}
// Set while we undo a refused move; the popstate it causes is ignored.
let undoing = false

// A page with unsaved changes registers a message here; leaving asks first.
let guard: string | null = null

function confirmLeave() {
  return !guard || window.confirm(guard)
}

window.addEventListener("popstate", (event) => {
  if (undoing) {
    undoing = false
    return
  }
  const next = (event.state as HistoryState)?.appIndex ?? 0
  if (next !== index && !confirmLeave()) {
    undoing = true
    window.history.go(index - next)
    return
  }
  index = next
  window.dispatchEvent(new Event(CHANGE))
})

function subscribe(listener: () => void) {
  window.addEventListener(CHANGE, listener)
  return () => window.removeEventListener(CHANGE, listener)
}

export function usePath(): string {
  return useSyncExternalStore(subscribe, () => window.location.pathname)
}

export function useRoute(): Route {
  // parseRoute returns new objects; callers compare fields, not identity.
  return parseRoute(usePath())
}

export function navigate(to: string, { replace = false } = {}) {
  const here = window.location.pathname + window.location.search
  if (to !== here) {
    if (!replace && !confirmLeave()) return
    if (replace) {
      window.history.replaceState({ appIndex: index }, "", to)
    } else {
      index += 1
      window.history.pushState({ appIndex: index }, "", to)
    }
    window.dispatchEvent(new Event(CHANGE))
  }
  if (!replace) window.scrollTo({ top: 0 })
}

// For <a href> links inside the app: navigate without a page reload, but let
// people open links in a new tab with a modifier key or middle click.
export function onLinkClick(event: React.MouseEvent<HTMLAnchorElement>) {
  if (
    event.defaultPrevented ||
    event.button !== 0 ||
    event.metaKey ||
    event.ctrlKey ||
    event.shiftKey ||
    event.altKey ||
    event.currentTarget.target === "_blank"
  )
    return
  event.preventDefault()
  navigate(event.currentTarget.pathname + event.currentTarget.search)
}

// While `dirty`, closing the tab or leaving the page asks first.
export function useLeaveGuard(dirty: boolean, message: string) {
  useEffect(() => {
    if (!dirty) return
    guard = message
    const onBeforeUnload = (event: BeforeUnloadEvent) => {
      event.preventDefault()
    }
    window.addEventListener("beforeunload", onBeforeUnload)
    return () => {
      if (guard === message) guard = null
      window.removeEventListener("beforeunload", onBeforeUnload)
    }
  }, [dirty, message])
}

export const TITLE_SUFFIX = ` · ${APP_NAME}`

// "Overview · {{PROJECT_NAME}}"
export function useDocumentTitle(title: string | null | undefined) {
  useEffect(() => {
    document.title = title ? `${title}${TITLE_SUFFIX}` : APP_NAME
  }, [title])
}
```

#### `src/lib/route-focus.ts`

```ts
import { useEffect, useRef } from "react"

import { announce } from "@/components/live-region"
import { TITLE_SUFFIX } from "@/lib/route"

// After an in-app navigation, focus would otherwise fall back to <body>
// (the link that was clicked is gone). Move it to the new page's h1 so
// keyboard and screen reader users start at the top of the new page
// (WCAG 2.4.3). Rules:
// - never on the first load;
// - only when focus was lost: if something already took it (a text box
//   after sending, a sidebar link that is
//   still there), leave it alone and just announce the page;
// - wait for the h1, since pages load their code and data first.
export function useRouteFocus(path: string) {
  const first = useRef(true)
  useEffect(() => {
    if (first.current) {
      first.current = false
      return
    }
    let frame = 0
    let tries = 0
    let placed: HTMLElement | null = null
    function lost() {
      const active = document.activeElement
      return !active || active === document.body
    }
    // Keeps watching for a few seconds: a loading page's h1 ("Loading
    // …") is replaced when the data arrives, which drops focus again.
    function tick() {
      const active = document.activeElement
      if (!lost() && active !== placed) {
        // Someone else moved focus (a text box, a click, Tab): stop. If we
        // never placed it, just say where we are.
        if (!placed) announcePage()
        return
      }
      const main = document.getElementById("main")
      if (lost()) {
        const heading = main?.querySelector("h1")
        if (heading) {
          if (!heading.hasAttribute("tabindex"))
            heading.setAttribute("tabindex", "-1")
          heading.focus({ preventScroll: true })
          placed = heading
        }
      }
      // Up to about four seconds (slow pages load code and data first).
      if (++tries < 240) {
        frame = requestAnimationFrame(tick)
      } else if (lost() && main) {
        main.setAttribute("tabindex", "-1")
        main.focus({ preventScroll: true })
      }
    }
    // Let the new page render (and anything that moves focus itself run).
    const timer = window.setTimeout(() => {
      frame = requestAnimationFrame(tick)
    }, 80)
    return () => {
      window.clearTimeout(timer)
      cancelAnimationFrame(frame)
    }
  }, [path])
}

function announcePage() {
  const title = document.title.endsWith(TITLE_SUFFIX)
    ? document.title.slice(0, -TITLE_SUFFIX.length)
    : document.title
  if (title) announce(title)
}
```

#### `src/lib/nav.ts`

```ts
import { LayoutGrid, Shapes, type LucideIcon } from "lucide-react"

import type { Route } from "@/lib/route"

// The sidebar sections, in order. Each one owns the routes that belong to it,
// so the sidebar can mark the current section on nested pages too.
export type NavItem = {
  id: string
  label: string
  icon: LucideIcon
  path: string
  routes: Route["name"][]
}

export const NAV: NavItem[] = [
  {
    id: "overview",
    label: "Overview",
    icon: LayoutGrid,
    path: "/",
    routes: ["overview", "item"],
  },
  {
    id: "states",
    label: "States",
    icon: Shapes,
    path: "/states",
    routes: ["states"],
  },
]

export function sectionOf(route: Route) {
  return NAV.find((item) => item.routes.includes(route.name))
}
```

#### `src/lib/notify.ts`

```ts
import { toast } from "@/components/ui/toast"
import { errorMessage } from "@/lib/api"

// The one toast helper: completed actions ("Saved", "Copied") and failures.
export function notify(
  title: string,
  {
    description,
    type = "success",
    action,
    onClose,
  }: {
    description?: string
    type?: "success" | "error" | "info"
    // A button in the toast, e.g. Undo.
    action?: { label: string; onClick: () => void }
    // Runs when the toast closes (timed out, dismissed or its action used).
    // Work that waits for an Undo belongs here, not on a separate timer: the
    // toast's own timer pauses while it's hovered or has keyboard focus
    // (WCAG 2.2.1), a separate one wouldn't.
    onClose?: () => void
  } = {}
) {
  toast.add({
    title,
    description,
    type,
    actionProps: action
      ? { children: action.label, onClick: action.onClick }
      : undefined,
    // "high" would make Base UI hide the toast from screen readers until it
    // is focused; errors get role="alert" in the Toaster instead.
    priority: "low",
    onClose,
  })
}

export function notifyError(title: string, error: unknown) {
  notify(title, { type: "error", description: errorMessage(error) })
}
```

#### `src/lib/api.ts`

```ts
import type { ApiError } from "@shared/api"

import { APP_NAME } from "@/lib/app"

// Typed calls to the Worker API (/api/*). Every response is JSON; errors
// come back as { error } with a non-2xx status and are thrown as ApiRequestError.

export class ApiRequestError extends Error {
  status: number
  // Seconds until a rate limit frees up (the Retry-After header), if sent.
  retryAfter: number | null
  constructor(
    message: string,
    status: number,
    retryAfter: number | null = null
  ) {
    super(message)
    this.name = "ApiRequestError"
    this.status = status
    this.retryAfter = retryAfter
  }
}

// Options for a single call. `signal` cancels it (a Cancel button on a slow call);
// a cancelled call rejects with the browser's AbortError, see isAbort().
export type RequestOptions = { signal?: AbortSignal }

export function isAbort(error: unknown) {
  return error instanceof DOMException && error.name === "AbortError"
}

const unauthorized = new Set<() => void>()

// Called when any API call (other than sign-in itself) answers 401.
export function onUnauthorized(listener: () => void) {
  unauthorized.add(listener)
  return () => {
    unauthorized.delete(listener)
  }
}

async function send<T>(
  method: string,
  path: string,
  body?: unknown,
  { signal }: RequestOptions = {}
): Promise<{ data: T; status: number }> {
  let response: Response
  try {
    response = await fetch(path, {
      method,
      signal,
      credentials: "same-origin",
      headers:
        body === undefined || body instanceof FormData
          ? { Accept: "application/json" }
          : { Accept: "application/json", "Content-Type": "application/json" },
      body:
        body === undefined
          ? undefined
          : body instanceof FormData
            ? body
            : JSON.stringify(body),
    })
  } catch (error) {
    if (isAbort(error)) throw error
    // fetch only rejects when the network is down, the request was blocked
    // or it was cancelled.
    throw new ApiRequestError(
      `Couldn't reach ${APP_NAME}. Check your connection and try again.`,
      0
    )
  }
  if (!response.ok) {
    let message = `Something went wrong (${response.status}). Try again.`
    try {
      const data = (await response.json()) as Partial<ApiError>
      if (typeof data.error === "string" && data.error) message = data.error
    } catch {
      // Not JSON (a proxy error page, for example); keep the generic message.
    }
    // A session that ended while the app was open. Sign-in calls answer 401
    // for a wrong password, which isn't the same thing.
    if (response.status === 401 && !path.startsWith("/api/auth/")) {
      for (const listener of unauthorized) listener()
    }
    const wait = Number(response.headers.get("Retry-After"))
    throw new ApiRequestError(
      message,
      response.status,
      Number.isFinite(wait) && wait > 0 ? wait : null
    )
  }
  return { data: (await response.json()) as T, status: response.status }
}

async function request<T>(
  method: string,
  path: string,
  body?: unknown,
  options?: RequestOptions
): Promise<T> {
  return (await send<T>(method, path, body, options)).data
}

export const get = <T>(path: string) => request<T>("GET", path)
export const post = <T>(
  path: string,
  body?: unknown,
  options?: RequestOptions
) => request<T>("POST", path, body ?? {}, options)
// Like post, with the status too, for routes where 200 and 201 mean
// different things (created, or already there).
export const postWithStatus = <T>(path: string, body?: unknown) =>
  send<T>("POST", path, body ?? {})
export const put = <T>(path: string, body: unknown) =>
  request<T>("PUT", path, body)
export const patch = <T>(path: string, body: unknown) =>
  request<T>("PATCH", path, body)
export const del = <T = { ok: true }>(path: string) =>
  request<T>("DELETE", path)

// The message to show for anything thrown by the helpers above.
export function errorMessage(error: unknown) {
  return error instanceof Error && error.message
    ? error.message
    : "Something went wrong. Try again."
}

export function isStatus(error: unknown, status: number) {
  return error instanceof ApiRequestError && error.status === status
}
```

#### `src/lib/resource.ts`

```ts
import { useCallback, useEffect, useState, useSyncExternalStore } from "react"

import { ApiRequestError, get } from "@/lib/api"

// A small shared cache for GET requests. Every component that reads the same
// path shares one entry, so a change made in one place (renaming an
// item, say) shows everywhere once it's invalidated.
// Data stays on screen while it reloads.

type Entry = {
  data?: unknown
  error?: ApiRequestError
  loading: boolean
}

const EMPTY: Entry = { loading: true }
const entries = new Map<string, Entry>()
const listeners = new Set<() => void>()
// How many mounted components read each path.
const readers = new Map<string, number>()
// The latest request per path, so a slow old response can't overwrite a newer one.
const inflight = new Map<string, Promise<void>>()

function emit() {
  for (const listener of listeners) listener()
}

function subscribe(listener: () => void) {
  listeners.add(listener)
  return () => listeners.delete(listener)
}

function load(path: string) {
  const previous = entries.get(path)
  entries.set(path, { ...(previous ?? {}), loading: true })
  emit()
  const promise = get<unknown>(path).then(
    (data) => {
      if (inflight.get(path) !== promise) return
      inflight.delete(path)
      entries.set(path, { data, loading: false })
      emit()
    },
    (error: unknown) => {
      if (inflight.get(path) !== promise) return
      inflight.delete(path)
      entries.set(path, {
        data: entries.get(path)?.data,
        error:
          error instanceof ApiRequestError
            ? error
            : new ApiRequestError("Something went wrong. Try again.", 0),
        loading: false,
      })
      emit()
    }
  )
  inflight.set(path, promise)
  return promise
}

// Reload every cached path under one of the prefixes ("/api/items"
// covers "/api/items?archived=1" and "/api/items/x"). Paths nobody
// is looking at are dropped and load fresh next time.
export function invalidate(...prefixes: string[]) {
  for (const path of [...entries.keys()]) {
    const matches = prefixes.some(
      (prefix) =>
        path === prefix ||
        path.startsWith(`${prefix}/`) ||
        path.startsWith(`${prefix}?`)
    )
    if (!matches) continue
    if (readers.get(path)) void load(path)
    else entries.delete(path)
  }
}

// Put data in the cache directly (after a write returned the new value).
// Any GET still in flight for the path started before the write, so its
// answer is older: forget it, and load() will ignore it when it lands.
export function setResource<T>(path: string, data: T) {
  inflight.delete(path)
  entries.set(path, { data, loading: false })
  emit()
}

// Forget a path entirely (after deleting the thing it points at), so nothing
// refetches it and gets a 404.
export function dropResource(path: string) {
  inflight.delete(path)
  entries.delete(path)
  emit()
}

// Forget everything (after signing in as someone else), so no page shows the
// previous person's data.
export function resetResources() {
  inflight.clear()
  entries.clear()
  emit()
}

// What the cache holds for a path right now, if anything.
export function peekResource<T>(path: string) {
  return entries.get(path)?.data as T | undefined
}

export function updateResource<T>(path: string, update: (data: T) => T) {
  const entry = entries.get(path)
  if (entry?.data === undefined) return
  setResource(path, update(entry.data as T))
}

export function useResource<T>(path: string | null) {
  const entry = useSyncExternalStore(subscribe, () =>
    path ? (entries.get(path) ?? EMPTY) : EMPTY
  )

  useEffect(() => {
    if (!path) return
    readers.set(path, (readers.get(path) ?? 0) + 1)
    // Cached data shows at once; it's refreshed in the background.
    if (!inflight.has(path)) void load(path)
    return () => {
      readers.set(path, (readers.get(path) ?? 1) - 1)
    }
  }, [path])

  const reload = useCallback(() => {
    if (path) void load(path)
  }, [path])

  return {
    data: entry.data as T | undefined,
    error: entry.error,
    // True only for the first load; reloads keep showing the old data.
    loading: entry.data === undefined && !entry.error,
    reloading: entry.loading,
    reload,
  }
}

// The last value that wasn't undefined: keeps old results on screen while a
// new search loads, instead of flashing a skeleton.
export function useLatest<T>(value: T | undefined) {
  const [latest, setLatest] = useState(value)
  if (value !== undefined && value !== latest) setLatest(value)
  return value ?? latest
}
```

## Step 9. Shared components (`src/components/`)

#### `src/components/live-region.tsx`

```tsx
import { useSyncExternalStore } from "react"

// One polite live region for the whole app, so short confirmations ("Copied")
// are announced without each button carrying its own status element.

let message = ""
let timer: number | undefined
const listeners = new Set<() => void>()

function emit() {
  for (const listener of listeners) listener()
}

export function announce(text: string) {
  // Clearing first makes a repeated message ("Copied" twice) announce again.
  message = ""
  emit()
  window.setTimeout(() => {
    message = text
    emit()
  }, 50)
  window.clearTimeout(timer)
  timer = window.setTimeout(() => {
    message = ""
    emit()
  }, 4000)
}

export function LiveRegion() {
  const text = useSyncExternalStore(
    (listener) => {
      listeners.add(listener)
      return () => listeners.delete(listener)
    },
    () => message
  )
  return (
    <p role="status" className="sr-only">
      {text}
    </p>
  )
}
```

#### `src/components/theme-provider.tsx`

```tsx
import * as React from "react"

type Theme = "dark" | "light" | "system"
type ResolvedTheme = "dark" | "light"

type ThemeProviderProps = {
  children: React.ReactNode
  defaultTheme?: Theme
  storageKey?: string
  disableTransitionOnChange?: boolean
}

type ThemeProviderState = {
  theme: Theme
  setTheme: (theme: Theme) => void
}

const COLOR_SCHEME_QUERY = "(prefers-color-scheme: dark)"
const THEME_VALUES: Theme[] = ["dark", "light", "system"]

const ThemeProviderContext = React.createContext<
  ThemeProviderState | undefined
>(undefined)

function isTheme(value: string | null): value is Theme {
  if (value === null) {
    return false
  }

  return THEME_VALUES.includes(value as Theme)
}

function getSystemTheme(): ResolvedTheme {
  if (window.matchMedia(COLOR_SCHEME_QUERY).matches) {
    return "dark"
  }

  return "light"
}

function disableTransitionsTemporarily() {
  const style = document.createElement("style")
  style.appendChild(
    document.createTextNode(
      "*,*::before,*::after{-webkit-transition:none!important;transition:none!important}"
    )
  )
  document.head.appendChild(style)

  return () => {
    window.getComputedStyle(document.body)
    requestAnimationFrame(() => {
      requestAnimationFrame(() => {
        style.remove()
      })
    })
  }
}

function isEditableTarget(target: EventTarget | null) {
  if (!(target instanceof HTMLElement)) {
    return false
  }

  if (target.isContentEditable) {
    return true
  }

  const editableParent = target.closest(
    "input, textarea, select, [contenteditable='true']"
  )
  if (editableParent) {
    return true
  }

  return false
}

export function ThemeProvider({
  children,
  defaultTheme = "system",
  storageKey = "theme",
  disableTransitionOnChange = true,
  ...props
}: ThemeProviderProps) {
  const [theme, setThemeState] = React.useState<Theme>(() => {
    const storedTheme = localStorage.getItem(storageKey)
    if (isTheme(storedTheme)) {
      return storedTheme
    }

    return defaultTheme
  })

  const setTheme = React.useCallback(
    (nextTheme: Theme) => {
      localStorage.setItem(storageKey, nextTheme)
      setThemeState(nextTheme)
    },
    [storageKey]
  )

  const applyTheme = React.useCallback(
    (nextTheme: Theme) => {
      const root = document.documentElement
      const resolvedTheme =
        nextTheme === "system" ? getSystemTheme() : nextTheme
      const restoreTransitions = disableTransitionOnChange
        ? disableTransitionsTemporarily()
        : null

      root.classList.remove("light", "dark")
      root.classList.add(resolvedTheme)

      if (restoreTransitions) {
        restoreTransitions()
      }
    },
    [disableTransitionOnChange]
  )

  React.useEffect(() => {
    applyTheme(theme)

    if (theme !== "system") {
      return undefined
    }

    const mediaQuery = window.matchMedia(COLOR_SCHEME_QUERY)
    const handleChange = () => {
      applyTheme("system")
    }

    mediaQuery.addEventListener("change", handleChange)

    return () => {
      mediaQuery.removeEventListener("change", handleChange)
    }
  }, [theme, applyTheme])

  React.useEffect(() => {
    const handleKeyDown = (event: KeyboardEvent) => {
      if (event.repeat) {
        return
      }

      if (event.metaKey || event.ctrlKey || event.altKey) {
        return
      }

      if (isEditableTarget(event.target)) {
        return
      }

      if (event.key.toLowerCase() !== "d") {
        return
      }

      setThemeState((currentTheme) => {
        const nextTheme =
          currentTheme === "dark"
            ? "light"
            : currentTheme === "light"
              ? "dark"
              : getSystemTheme() === "dark"
                ? "light"
                : "dark"

        localStorage.setItem(storageKey, nextTheme)
        return nextTheme
      })
    }

    window.addEventListener("keydown", handleKeyDown)

    return () => {
      window.removeEventListener("keydown", handleKeyDown)
    }
  }, [storageKey])

  React.useEffect(() => {
    const handleStorageChange = (event: StorageEvent) => {
      if (event.storageArea !== localStorage) {
        return
      }

      if (event.key !== storageKey) {
        return
      }

      if (isTheme(event.newValue)) {
        setThemeState(event.newValue)
        return
      }

      setThemeState(defaultTheme)
    }

    window.addEventListener("storage", handleStorageChange)

    return () => {
      window.removeEventListener("storage", handleStorageChange)
    }
  }, [defaultTheme, storageKey])

  const value = React.useMemo(
    () => ({
      theme,
      setTheme,
    }),
    [theme, setTheme]
  )

  return (
    <ThemeProviderContext.Provider {...props} value={value}>
      {children}
    </ThemeProviderContext.Provider>
  )
}

export const useTheme = () => {
  const context = React.useContext(ThemeProviderContext)

  if (context === undefined) {
    throw new Error("useTheme must be used within a ThemeProvider")
  }

  return context
}
```

#### `src/components/page.tsx`

```tsx
import type { ReactNode } from "react"
import { cn } from "@/lib/utils"

import { onLinkClick, useDocumentTitle } from "@/lib/route"

// The standard page frame: width, padding, a small title and optional actions
// on the right. Sets the tab title too ("Overview · {{PROJECT_NAME}}").
export function Page({
  title,
  count,
  documentTitle,
  actions,
  children,
  width = "wide",
  className,
  titleHidden = false,
  bare = false,
}: {
  // The page renders its own visible h1 (a document's title, say).
  bare?: boolean
  title: ReactNode
  // A quiet number after the title ("Items 11").
  count?: number
  // The page still has its h1, but only for screen readers.
  titleHidden?: boolean
  // Defaults to `title` when that is plain text.
  documentTitle?: string
  actions?: ReactNode
  children: ReactNode
  width?: "wide" | "narrow"
  className?: string
}) {
  useDocumentTitle(
    documentTitle ?? (typeof title === "string" ? title : undefined)
  )
  return (
    <div
      className={cn(
        "mx-auto w-full px-4 pt-7 pb-16 sm:px-6",
        // Rows wider than about 1000 px pull title and status too far apart.
        width === "wide" ? "max-w-260" : "max-w-3xl",
        className
      )}
    >
      {!bare && (
        <div
          className={cn(
            "mb-4 flex min-h-9 flex-wrap items-center justify-between gap-x-4 gap-y-2",
            titleHidden && !actions && "sr-only"
          )}
        >
          <h1 className="min-w-0 text-lg/[1.625rem] font-medium tracking-[-0.01em] wrap-break-word">
            {title}
            {count !== undefined && (
              <span className="ml-1.5 font-normal text-muted-foreground tabular-nums">
                <span className="sr-only">, </span>
                {count}
              </span>
            )}
          </h1>
          {actions && (
            <div className="flex flex-wrap items-center gap-2">{actions}</div>
          )}
        </div>
      )}
      {children}
    </div>
  )
}

// A bordered card list, used for every list in the app.
export function ListCard({
  className,
  children,
  label,
}: {
  className?: string
  children: ReactNode
  label?: string
}) {
  return (
    <ul
      aria-label={label}
      className={cn(
        "list-enter min-w-0 divide-y overflow-hidden rounded-xl border bg-card shadow-card",
        className
      )}
    >
      {children}
    </ul>
  )
}

// "Design · Kari Nordmann · Updated today". Separators are decoration. Each
// part carries its dot in front, and the row is shifted left by one dot
// width and clipped, so a wrapped line never starts (or ends) with a dot.
export function MetaLine({
  parts,
  className,
}: {
  parts: ReactNode[]
  className?: string
}) {
  const shown = parts.filter(
    (part) => part !== null && part !== false && part !== ""
  )
  if (shown.length === 0) return null
  return (
    <div
      className={cn(
        "overflow-x-clip text-meta text-muted-foreground",
        className
      )}
    >
      <p className="-ml-3 flex flex-wrap items-center">
        {shown.map((part, index) => (
          <span
            key={index}
            className="inline-flex min-w-0 items-center before:w-3 before:shrink-0 before:text-center before:text-subtle-foreground before:content-['·'_/_'']"
          >
            {part}
          </span>
        ))}
      </p>
    </div>
  )
}

// One row in a ListCard: a 14 px title that links (the whole row is the
// target), one meta line, and a trailing slot for a status or a hover
// action. On phones the trailing slot moves under the meta and the title
// can wrap to two lines.
export function ListRow({
  href,
  title,
  meta,
  trailing,
  actions,
  leading,
  className,
}: {
  href: string
  title: ReactNode
  meta?: ReactNode[]
  // Status, a chip or a quiet hint, read after the title.
  trailing?: ReactNode
  // Buttons that sit above the row link (they need their own target).
  actions?: ReactNode
  // An avatar or icon before the title.
  leading?: ReactNode
  className?: string
}) {
  return (
    <li
      className={cn(
        "group/row relative grid grid-cols-1 items-center gap-x-4 gap-y-1 px-4 py-2.5 transition-colors duration-150 hover:bg-hover motion-reduce:transition-none sm:grid-cols-[minmax(0,1fr)_auto]",
        // Two lines 56 px, one line 44 px.
        meta ? "min-h-14" : "min-h-11",
        className
      )}
    >
      <div className={cn("min-w-0", leading && "flex items-center gap-3")}>
        {leading}
        <div className="min-w-0 flex-1">
          <a
            href={href}
            onClick={onLinkClick}
            className="block text-sm font-medium outline-none after:absolute after:inset-0 after:content-[''] focus-visible:after:ring-3 focus-visible:after:ring-ring/50 focus-visible:after:ring-inset"
          >
            <span
              title={typeof title === "string" ? title : undefined}
              className="block truncate max-sm:line-clamp-2 max-sm:whitespace-normal"
            >
              {title}
            </span>
          </a>
          {meta && <MetaLine parts={meta} className="mt-0.5" />}
        </div>
      </div>
      {(trailing || actions) && (
        <div className="flex min-w-0 items-center gap-2 sm:justify-self-end">
          {trailing}
          {actions && (
            <div className="relative z-10 flex items-center">{actions}</div>
          )}
        </div>
      )}
    </li>
  )
}

// A label above a section of the page ("Due this week").
export function SectionLabel({
  id,
  children,
  className,
}: {
  id?: string
  children: ReactNode
  className?: string
}) {
  return (
    <h2
      id={id}
      className={cn(
        "mt-6 mb-2 text-meta font-medium text-muted-foreground",
        className
      )}
    >
      {children}
    </h2>
  )
}
```

#### `src/components/states.tsx`

```tsx
import type { ReactNode } from "react"
import { cn } from "@/lib/utils"
import {
  CircleAlert,
  Compass,
  Lock,
  RotateCw,
  type LucideIcon,
} from "lucide-react"

import { Button } from "@/components/ui/button"
import {
  Empty,
  EmptyContent,
  EmptyDescription,
  EmptyHeader,
  EmptyMedia,
  EmptyTitle,
} from "@/components/ui/empty"
import { Skeleton } from "@/components/ui/skeleton"
import { Page } from "@/components/page"
import { ENTER } from "@/lib/motion"
import { errorMessage } from "@/lib/api"
import { onLinkClick } from "@/lib/route"

// An icon, a title, one line and an action only if there's a real one.
// "page": the only thing on the page, so no card around it. "inset": inside
// a list's card (a filtered list with no matches), so the list keeps its
// place.
export function EmptyState({
  icon: Icon,
  title,
  description,
  action,
  className,
  variant = "page",
}: {
  icon: LucideIcon
  title: string
  description?: ReactNode
  action?: ReactNode
  className?: string
  variant?: "page" | "inset"
}) {
  return (
    <Empty
      className={cn(
        ENTER,
        "gap-3 rounded-xl",
        variant === "page"
          ? "border-0 py-16"
          : "border border-solid bg-card py-12 shadow-card",
        className
      )}
    >
      <EmptyHeader>
        <EmptyMedia variant="icon">
          <Icon aria-hidden="true" />
        </EmptyMedia>
        <EmptyTitle>{title}</EmptyTitle>
        {description && <EmptyDescription>{description}</EmptyDescription>}
      </EmptyHeader>
      {action && <EmptyContent>{action}</EmptyContent>}
    </Empty>
  )
}

// What happened and what to do, with a Retry.
export function ErrorState({
  title = "Couldn't load this",
  error,
  onRetry,
  className,
}: {
  title?: string
  error: unknown
  onRetry?: () => void
  className?: string
}) {
  return (
    <div role="alert">
      <EmptyState
        icon={CircleAlert}
        title={title}
        description={errorMessage(error)}
        className={className}
        action={
          onRetry && (
            <Button variant="outline" size="sm" onClick={onRetry}>
              <RotateCw aria-hidden="true" />
              Retry
            </Button>
          )
        }
      />
    </div>
  )
}

export function NoAccessPage({ title }: { title: string }) {
  return (
    <Page title={title}>
      <EmptyState
        icon={Lock}
        title="You don't have access"
        description="Ask an admin if you need this page."
      />
    </Page>
  )
}

export function NotFoundPage({
  title = "Page not found",
  description = "The link may be old or mistyped.",
  back = { label: "Go to overview", href: "/" },
}: {
  title?: string
  description?: string
  // Where to go instead.
  back?: { label: string; href: string }
}) {
  return (
    // The empty state says it; the h1 is for screen readers and the tab.
    <Page title={title} titleHidden>
      <EmptyState
        icon={Compass}
        title={title}
        description={description}
        action={
          <Button
            variant="outline"
            size="sm"
            nativeButton={false}
            render={<a href={back.href} onClick={onLinkClick} />}
          >
            {back.label}
          </Button>
        }
      />
    </Page>
  )
}

// Stays invisible for 250 ms, so quick loads don't flash a skeleton.
export function Delayed({
  children,
  className,
}: {
  children: ReactNode
  className?: string
}) {
  return (
    <div
      className={cn(
        "animate-in duration-200 fill-mode-backwards fade-in",
        className
      )}
      style={{ animationDelay: "250ms" }}
      aria-busy="true"
    >
      <span className="sr-only">Loading…</span>
      {children}
    </div>
  )
}

// Rows of a list while it loads.
export function ListSkeleton({ rows = 5 }: { rows?: number }) {
  return (
    <Delayed>
      <div className="divide-y overflow-hidden rounded-xl border bg-card shadow-card">
        {Array.from({ length: rows }, (_, index) => (
          // Shaped like a ListRow: title, meta, status on the right.
          <div
            key={index}
            className="flex min-h-14 items-center gap-4 px-4 py-3"
          >
            <div className="flex min-w-0 flex-1 flex-col gap-2">
              <Skeleton className="h-3.5 w-2/5" />
              <Skeleton className="h-3 w-1/4" />
            </div>
            <Skeleton className="h-3 w-16 max-sm:hidden" />
          </div>
        ))}
      </div>
    </Delayed>
  )
}
```

#### `src/components/app-shell.tsx`

```tsx
import { useEffect, useState, type ReactNode } from "react"
import { Menu, Monitor, Moon, Sun } from "lucide-react"

import { LiveRegion } from "@/components/live-region"
import { useTheme } from "@/components/theme-provider"
import { Button } from "@/components/ui/button"
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuGroup,
  DropdownMenuLabel,
  DropdownMenuRadioGroup,
  DropdownMenuRadioItem,
  DropdownMenuTrigger,
} from "@/components/ui/dropdown-menu"
import {
  Sheet,
  SheetContent,
  SheetTitle,
  SheetTrigger,
} from "@/components/ui/sheet"
import { APP_NAME } from "@/lib/app"
import { FADE } from "@/lib/motion"
import { NAV, sectionOf } from "@/lib/nav"
import { onLinkClick, usePath, useRoute } from "@/lib/route"
import { useRouteFocus } from "@/lib/route-focus"
import { cn } from "@/lib/utils"

// The app frame (DESIGN_SYSTEM §7): a white sidebar on wide screens, a
// slide-in menu on phones and tablets, a sticky white top bar, and the
// canvas behind the page. Pages fade in on route changes and focus moves to
// their h1 (src/lib/route-focus.ts).

// {{LOGO}}: swap the wordmark for the brand logo (public/brand/), with a
// dark-mode version if it has dark text.
function HomeLink({ className }: { className?: string }) {
  return (
    <a
      href="/"
      onClick={onLinkClick}
      className={cn(
        "flex items-center gap-2 rounded-md text-sm font-medium outline-none focus-visible:ring-3 focus-visible:ring-ring/50",
        className
      )}
    >
      <img src="/favicon.svg" alt="" className="size-5" />
      <span>{APP_NAME}</span>
    </a>
  )
}

function NavItems({ onNavigate }: { onNavigate?: () => void }) {
  const active = sectionOf(useRoute())
  return (
    <ul className="flex flex-col gap-0.5">
      {NAV.map(({ id, label, icon: Icon, path }) => {
        const current = active?.id === id
        return (
          <li key={id}>
            <a
              href={path}
              aria-current={current ? "page" : undefined}
              onClick={(event) => {
                onLinkClick(event)
                onNavigate?.()
              }}
              className={cn(
                "group/nav flex h-8 items-center gap-2.5 rounded-lg px-2.5 text-sm transition-colors duration-150 outline-none focus-visible:ring-3 focus-visible:ring-ring/50 motion-reduce:transition-none",
                current
                  ? "bg-selected font-medium text-foreground"
                  : "text-muted-foreground hover:bg-hover hover:text-foreground"
              )}
            >
              <Icon
                className="size-4 shrink-0 transition-transform duration-300 ease-ios motion-safe:group-hover/nav:-rotate-6"
                aria-hidden="true"
              />
              <span className="min-w-0 truncate">{label}</span>
            </a>
          </li>
        )
      })}
    </ul>
  )
}

// Phones and tablets: the navigation slides in from the left.
function MobileNav() {
  const [open, setOpen] = useState(false)
  return (
    <Sheet open={open} onOpenChange={setOpen}>
      <SheetTrigger
        render={
          <Button
            variant="ghost"
            size="icon"
            aria-label="Open menu"
            className="-ml-2 lg:hidden"
          >
            <Menu aria-hidden="true" />
          </Button>
        }
      />
      <SheetContent
        side="left"
        className="flex w-[min(18rem,85vw)] flex-col gap-0 bg-background p-0"
      >
        <SheetTitle className="sr-only">Menu</SheetTitle>
        <div className="flex h-13 shrink-0 items-center px-4">
          <HomeLink />
        </div>
        <nav aria-label="Main" className="px-2 pb-2">
          <NavItems onNavigate={() => setOpen(false)} />
        </nav>
      </SheetContent>
    </Sheet>
  )
}

const THEMES = [
  { value: "light", label: "Light", icon: Sun },
  { value: "dark", label: "Dark", icon: Moon },
  { value: "system", label: "System", icon: Monitor },
] as const

function ThemeMenu() {
  const { theme, setTheme } = useTheme()
  return (
    <DropdownMenu>
      <DropdownMenuTrigger
        render={
          <Button
            variant="ghost"
            size="icon"
            aria-label="Theme"
            title="Theme"
            className="group/theme"
          >
            <Sun
              className="transition-transform duration-300 ease-ios motion-safe:group-hover/theme:rotate-45 dark:hidden"
              aria-hidden="true"
            />
            <Moon
              className="hidden transition-transform duration-300 ease-ios motion-safe:group-hover/theme:-rotate-12 dark:block"
              aria-hidden="true"
            />
          </Button>
        }
      />
      <DropdownMenuContent align="end" className="w-40">
        <DropdownMenuGroup>
          <DropdownMenuLabel>Theme</DropdownMenuLabel>
          <DropdownMenuRadioGroup
            value={theme}
            onValueChange={(value) => setTheme(value as typeof theme)}
          >
            {THEMES.map(({ value, label, icon: Icon }) => (
              <DropdownMenuRadioItem key={value} value={value} closeOnClick>
                <Icon aria-hidden="true" />
                {label}
              </DropdownMenuRadioItem>
            ))}
          </DropdownMenuRadioGroup>
        </DropdownMenuGroup>
      </DropdownMenuContent>
    </DropdownMenu>
  )
}

// True once the page has scrolled, so the top bar can lift off the content.
function useScrolled() {
  const [scrolled, setScrolled] = useState(false)
  useEffect(() => {
    const update = () => setScrolled(window.scrollY > 4)
    update()
    window.addEventListener("scroll", update, { passive: true })
    return () => window.removeEventListener("scroll", update)
  }, [])
  return scrolled
}

export function AppShell({ children }: { children: ReactNode }) {
  const scrolled = useScrolled()
  const path = usePath()
  useRouteFocus(path)

  return (
    <div className="min-h-svh lg:grid lg:grid-cols-[15rem_minmax(0,1fr)]">
      <a
        href="#main"
        className="sr-only focus:not-sr-only focus:fixed focus:top-3 focus:left-3 focus:z-50 focus:rounded-md focus:bg-background focus:px-3 focus:py-2 focus:text-sm focus:ring-3 focus:ring-ring/50"
      >
        Skip to content
      </a>

      {/* Wide screens: a sidebar. */}
      <aside
        aria-label="Sidebar"
        className="sticky top-0 hidden h-svh min-w-0 flex-col overflow-hidden border-r bg-background lg:flex"
      >
        <div className="flex h-13 shrink-0 items-center px-4">
          <HomeLink />
        </div>
        <nav aria-label="Main" className="flex-1 px-2 pb-2">
          <NavItems />
        </nav>
      </aside>

      <div className="min-h-svh min-w-0 canvas-surface">
        <header
          data-scrolled={scrolled}
          className="sticky top-0 z-30 border-b bg-background/85 backdrop-blur-xl transition-shadow duration-300 ease-enter data-[scrolled=true]:shadow-bar"
        >
          <div className="flex h-13 items-center gap-2 px-4 sm:px-5">
            <MobileNav />
            <HomeLink className="lg:hidden" />
            <div className="ml-auto flex items-center gap-1.5">
              <ThemeMenu />
            </div>
          </div>
        </header>
        {/* Wide tap areas can poke out on narrow phones; clip them so the
            page never scrolls sideways. */}
        <main id="main" className="min-w-0 overflow-x-clip outline-none">
          {/* Each page fades in (200 ms, opacity only). */}
          <div key={path} className={FADE}>
            {children}
          </div>
        </main>
        <LiveRegion />
      </div>
    </div>
  )
}
```

#### `src/components/chunk-boundary.tsx`

```tsx
import { Component, type ReactNode } from "react"
import { RotateCw } from "lucide-react"

import { EmptyState } from "@/components/states"
import { Button } from "@/components/ui/button"
import { APP_NAME, STORAGE_PREFIX } from "@/lib/app"

// Pages load their code on first visit. After a deploy an open tab may ask
// for files that no longer exist; instead of a blank app, say what happened
// and offer a reload. Resets when the route changes.
export class ChunkBoundary extends Component<
  { resetKey: string; children: ReactNode },
  { failed: boolean; key: string }
> {
  state = { failed: false, key: this.props.resetKey }

  static getDerivedStateFromError() {
    return { failed: true }
  }

  static getDerivedStateFromProps(
    props: { resetKey: string },
    state: { failed: boolean; key: string }
  ) {
    return props.resetKey !== state.key
      ? { failed: false, key: props.resetKey }
      : null
  }

  render() {
    if (!this.state.failed) return this.props.children
    return (
      <div className="mx-auto w-full max-w-3xl px-4 pt-8 sm:px-6">
        <div role="alert">
          <EmptyState
            icon={RotateCw}
            title={`${APP_NAME} was updated. Reload to continue.`}
            action={
              <Button
                variant="outline"
                onClick={() => window.location.reload()}
              >
                <RotateCw aria-hidden="true" />
                Reload
              </Button>
            }
          />
        </div>
      </div>
    )
  }
}

const RELOADED_AT = `${STORAGE_PREFIX}:chunk-reload`

// Vite reports a failed dynamic import here. Reload once to pick up the new
// version; if that already happened in the last minute, let the boundary
// show its message instead of looping.
export function reloadOnStaleChunks() {
  window.addEventListener("vite:preloadError", (event) => {
    let last = 0
    try {
      last = Number(sessionStorage.getItem(RELOADED_AT) ?? 0)
    } catch {
      // Storage blocked: fall through to the boundary after one reload.
    }
    if (Date.now() - last < 60_000) return
    try {
      sessionStorage.setItem(RELOADED_AT, String(Date.now()))
    } catch {
      return
    }
    event.preventDefault()
    window.location.reload()
  })
}
```

#### `src/components/form-field.tsx`

```tsx
import { useState, type ComponentProps, type ReactNode } from "react"
import { cn } from "@/lib/utils"
import { CircleAlert, Eye, EyeOff } from "lucide-react"

import {
  Field,
  FieldDescription,
  FieldError,
  FieldLabel,
} from "@/components/ui/field"
import { Input } from "@/components/ui/input"

type InputProps = Omit<ComponentProps<"input">, "value" | "onChange" | "id">

// A labelled text field: label above, the input, then a hint or the error.
// Errors only appear after a submit attempt (the form decides when).
export function FormField({
  id,
  label,
  value,
  onChange,
  error,
  hint,
  optional = false,
  password = false,
  ...props
}: {
  id: string
  label: ReactNode
  value: string
  onChange: (value: string) => void
  error?: string
  hint?: ReactNode
  optional?: boolean
  // A password field with a Show password toggle. Paste and password
  // managers keep working (WCAG 3.3.8).
  password?: boolean
} & InputProps) {
  const [shown, setShown] = useState(false)
  // The error takes the hint's place while it shows.
  const showHint = Boolean(hint) && !error
  const describedBy = error
    ? `${id}-error`
    : showHint
      ? `${id}-hint`
      : undefined
  const input = (
    <Input
      id={id}
      value={value}
      onChange={(event) => onChange(event.target.value)}
      aria-invalid={Boolean(error) || undefined}
      aria-describedby={describedBy}
      {...props}
      type={password ? (shown ? "text" : "password") : props.type}
      className={cn(password && "pr-11", props.className)}
    />
  )
  return (
    <Field data-invalid={Boolean(error) || undefined}>
      <FieldLabel htmlFor={id}>
        {label}
        {optional && (
          <span className="font-normal text-muted-foreground">(optional)</span>
        )}
      </FieldLabel>
      {password ? (
        <div className="relative">
          {input}
          <span className="absolute inset-y-0 right-1 flex items-center">
            <button
              type="button"
              aria-label="Show password"
              aria-pressed={shown}
              aria-controls={id}
              onClick={() => setShown((value) => !value)}
              className="flex size-8 items-center justify-center rounded-[0.5rem] text-muted-foreground transition-colors duration-200 outline-none hover:bg-muted hover:text-foreground focus-visible:ring-3 focus-visible:ring-ring/50"
            >
              {shown ? (
                <EyeOff className="size-4" aria-hidden="true" />
              ) : (
                <Eye className="size-4" aria-hidden="true" />
              )}
            </button>
          </span>
        </div>
      ) : (
        input
      )}
      {showHint && (
        <FieldDescription id={`${id}-hint`}>{hint}</FieldDescription>
      )}
      <FieldError id={`${id}-error`}>{error}</FieldError>
    </Field>
  )
}

// A message about the whole form (the server's answer), above its button.
export function FormAlert({
  children,
  className,
}: {
  children: ReactNode
  className?: string
}) {
  return (
    <div
      role="alert"
      className={cn(
        "flex items-start gap-2.5 rounded-lg bg-destructive/10 px-3.5 py-3 text-sm text-destructive motion-safe:animate-in motion-safe:fade-in",
        className
      )}
    >
      <CircleAlert className="mt-0.5 size-4 shrink-0" aria-hidden="true" />
      <div className="min-w-0">{children}</div>
    </div>
  )
}
```

#### `src/components/form-select.tsx`

```tsx
import type { ComponentProps } from "react"
import { cn } from "@/lib/utils"

import { NativeSelect } from "@/components/ui/native-select"

// The native select at the app's control size: 40 px high with the 10 px
// corner that inputs and buttons use. Native, so it works well on phones and
// with screen readers.
export function FormSelect({
  className,
  compact = false,
  ...props
}: ComponentProps<typeof NativeSelect> & { compact?: boolean }) {
  return (
    <NativeSelect
      className={cn(
        "[&_select]:rounded-lg [&_select]:bg-background [&_select]:pl-3.5 dark:[&_select]:bg-input/30",
        compact ? "[&_select]:h-9" : "w-full [&_select]:h-10",
        className
      )}
      {...props}
    />
  )
}
```

#### `src/components/copy-button.tsx`

```tsx
import { useEffect, useRef, useState, type ComponentProps } from "react"
import { cn } from "@/lib/utils"
import { Check, Copy } from "lucide-react"

import { Button } from "@/components/ui/button"
import { announce } from "@/components/live-region"
import { notify } from "@/lib/notify"

export async function copyText(text: string) {
  try {
    await navigator.clipboard.writeText(text)
    return true
  } catch {
    notify("Couldn't copy", {
      type: "error",
      description: "Select the text and copy it instead.",
    })
    return false
  }
}

// Swaps to a green tick and "Copied" for about 1.6 s, announced politely.
export function CopyButton({
  text,
  label = "Copy",
  className,
  variant = "ghost",
  size = "sm",
}: {
  text: () => string
  label?: string
  className?: string
} & Pick<ComponentProps<typeof Button>, "variant" | "size">) {
  const [copied, setCopied] = useState(false)
  const timer = useRef<number | undefined>(undefined)
  useEffect(() => () => window.clearTimeout(timer.current), [])
  return (
    <>
      <Button
        type="button"
        variant={variant}
        size={size}
        className={cn(copied && "text-success hover:text-success", className)}
        onClick={async () => {
          if (!(await copyText(text()))) return
          setCopied(true)
          announce("Copied")
          window.clearTimeout(timer.current)
          timer.current = window.setTimeout(() => setCopied(false), 1600)
        }}
      >
        {copied ? <Check aria-hidden="true" /> : <Copy aria-hidden="true" />}
        {copied ? "Copied" : label}
      </Button>
    </>
  )
}
```

## Step 10. Example pages and the entry

The Overview page reads `/api/health` through `useResource`; the States page shows the empty, loading, error and toast states. Replace both once real pages exist (keep the pattern).

#### `src/features/items/items.ts`

```ts
// Example data so the starter has something to list. Replace with
// useResource("/api/…") once the API has real data.
export type ExampleItem = {
  id: string
  title: string
  owner: string
  updated: string
  status: "Draft" | "In review" | "Published"
}

export const EXAMPLE_ITEMS: ExampleItem[] = [
  {
    id: "welcome",
    title: "Welcome guide for new starters",
    owner: "Kari Nordmann",
    updated: "Updated today",
    status: "Published",
  },
  {
    id: "brand",
    title: "Brand colours and logo use",
    owner: "Ola Hansen",
    updated: "Updated 2 Oct",
    status: "In review",
  },
  {
    id: "travel",
    title: "Travel and expenses",
    owner: "Ingrid Berg",
    updated: "Updated 28 Sep",
    status: "Draft",
  },
]
```

#### `src/features/items/status.tsx`

```tsx
import { Badge } from "@/components/ui/badge"
import { cn } from "@/lib/utils"
import type { ExampleItem } from "@/features/items/items"

// Outline badges where the word carries the meaning; colour only supports it.
const TONE: Record<ExampleItem["status"], string> = {
  Draft: "text-muted-foreground",
  "In review": "border-primary/30 bg-primary/5 text-primary-ink",
  Published: "border-success/30 bg-success/8 text-success",
}

export function StatusBadge({ status }: { status: ExampleItem["status"] }) {
  return (
    <Badge variant="outline" className={cn(TONE[status])}>
      {status}
    </Badge>
  )
}
```

#### `src/features/items/item-page.tsx`

```tsx
import { MetaLine, Page } from "@/components/page"
import { NotFoundPage } from "@/components/states"
import { EXAMPLE_ITEMS } from "@/features/items/items"
import { StatusBadge } from "@/features/items/status"

// A detail page reached from a ListRow (/items/:id).
export function ItemPage({ id }: { id: string }) {
  const item = EXAMPLE_ITEMS.find((candidate) => candidate.id === id)
  if (!item)
    return (
      <NotFoundPage
        title="Item not found"
        description="It may have been deleted, or the link is old."
        back={{ label: "Back to overview", href: "/" }}
      />
    )
  return (
    <Page title={item.title} width="narrow">
      <div className="rounded-xl border bg-card p-5 shadow-card sm:p-6">
        <div className="flex flex-wrap items-center gap-2">
          <StatusBadge status={item.status} />
          <MetaLine parts={[item.owner, item.updated]} />
        </div>
        <p className="mt-4 text-reading text-foreground">
          This is an example detail page. Replace it with your own content.
        </p>
      </div>
    </Page>
  )
}
```

#### `src/features/overview/overview-page.tsx`

```tsx
import { Plus } from "lucide-react"
import type { HealthResponse } from "@shared/api"

import { ListCard, ListRow, Page, SectionLabel } from "@/components/page"
import { ErrorState } from "@/components/states"
import { Button } from "@/components/ui/button"
import { Skeleton } from "@/components/ui/skeleton"
import { EXAMPLE_ITEMS } from "@/features/items/items"
import { StatusBadge } from "@/features/items/status"
import { notify } from "@/lib/notify"
import { useResource } from "@/lib/resource"

// The first screen: an example list in a ListCard, and the API's health
// read through useResource (loading, error and data states).
export function OverviewPage() {
  return (
    <Page
      title="Overview"
      actions={
        <Button onClick={() => notify("Item created")}>
          <Plus aria-hidden="true" />
          New item
        </Button>
      }
    >
      <SectionLabel id="items-label" className="mt-0">
        Items
      </SectionLabel>
      <ListCard label="Items">
        {EXAMPLE_ITEMS.map((item) => (
          <ListRow
            key={item.id}
            href={`/items/${item.id}`}
            title={item.title}
            meta={[item.owner, item.updated]}
            trailing={<StatusBadge status={item.status} />}
          />
        ))}
      </ListCard>

      <SectionLabel>API</SectionLabel>
      <ApiHealth />
    </Page>
  )
}

function ApiHealth() {
  const { data, error, loading, reload } =
    useResource<HealthResponse>("/api/health")
  if (error) return <ErrorState error={error} onRetry={reload} />
  return (
    <div className="rounded-xl border bg-card p-5 shadow-card">
      {loading || !data ? (
        <div aria-busy="true" className="flex flex-col gap-2">
          <span className="sr-only">Loading…</span>
          <Skeleton className="h-3.5 w-1/3" />
          <Skeleton className="h-3 w-1/4" />
        </div>
      ) : (
        <div>
          <p className="text-sm font-medium" data-testid="api-status">
            API is up
          </p>
          <p className="text-meta text-muted-foreground">
            {data.environment} · answered at{" "}
            {new Date(data.time).toLocaleTimeString()}
          </p>
        </div>
      )}
    </div>
  )
}
```

#### `src/features/states/states-page.tsx`

```tsx
import { useState } from "react"
import { Inbox } from "lucide-react"

import { Page, SectionLabel } from "@/components/page"
import { EmptyState, ErrorState, ListSkeleton } from "@/components/states"
import { Button } from "@/components/ui/button"
import { ApiRequestError } from "@/lib/api"
import { notify } from "@/lib/notify"

// Every screen needs empty, loading and error states (DESIGN_SYSTEM §12).
// This page shows the shared ones side by side, plus toasts.
export function StatesPage() {
  const [loadingKey, setLoadingKey] = useState(0)
  return (
    <Page title="States">
      <SectionLabel className="mt-0">Empty</SectionLabel>
      <EmptyState
        variant="inset"
        icon={Inbox}
        title="No items yet"
        description="Items you create show up here."
        action={
          <Button variant="outline" size="sm" onClick={() => notify("Item created")}>
            New item
          </Button>
        }
      />

      <SectionLabel>
        Loading{" "}
        <button
          type="button"
          onClick={() => setLoadingKey((key) => key + 1)}
          className="ml-1 rounded-sm text-primary-ink underline-offset-4 outline-none hover:underline focus-visible:ring-3 focus-visible:ring-ring/50"
        >
          Replay
        </button>
      </SectionLabel>
      <ListSkeleton key={loadingKey} rows={3} />

      <SectionLabel>Error</SectionLabel>
      <ErrorState
        className="border border-solid bg-card shadow-card"
        error={
          new ApiRequestError(
            "The server didn't answer. Try again in a moment.",
            503
          )
        }
        onRetry={() => notify("Retried", { type: "info" })}
      />

      <SectionLabel>Toasts</SectionLabel>
      <div className="flex flex-wrap gap-2">
        <Button variant="outline" onClick={() => notify("Saved")}>
          Show success
        </Button>
        <Button
          variant="outline"
          onClick={() =>
            notify("Item deleted", {
              action: { label: "Undo", onClick: () => notify("Item restored") },
            })
          }
        >
          Show with Undo
        </Button>
        <Button
          variant="outline"
          onClick={() =>
            notify("Couldn't save", {
              type: "error",
              description: "Check your connection and try again.",
            })
          }
        >
          Show error
        </Button>
      </div>
    </Page>
  )
}
```

#### `src/App.tsx`

```tsx
import { AppShell } from "@/components/app-shell"
import { ChunkBoundary } from "@/components/chunk-boundary"
import { NotFoundPage } from "@/components/states"
import { ItemPage } from "@/features/items/item-page"
import { OverviewPage } from "@/features/overview/overview-page"
import { StatesPage } from "@/features/states/states-page"
import { usePath, useRoute, type Route } from "@/lib/route"

// Pages are chosen by path (src/lib/route.ts). Large pages can load on
// first visit with React.lazy + <Suspense>; ChunkBoundary covers a stale
// chunk after a deploy.
function page(route: Route) {
  switch (route.name) {
    case "overview":
      return <OverviewPage />
    case "states":
      return <StatesPage />
    case "item":
      return <ItemPage id={route.id} />
    case "not-found":
      return <NotFoundPage />
  }
}

export default function App() {
  const route = useRoute()
  const path = usePath()
  return (
    <AppShell>
      <ChunkBoundary resetKey={path}>{page(route)}</ChunkBoundary>
    </AppShell>
  )
}
```

#### `src/main.tsx`

```tsx
import { StrictMode } from "react"
import { createRoot } from "react-dom/client"

import "./index.css"
import App from "./App.tsx"
import { ThemeProvider } from "@/components/theme-provider.tsx"
import { Toaster } from "@/components/ui/toast"
import { reloadOnStaleChunks } from "@/components/chunk-boundary"

reloadOnStaleChunks()

createRoot(document.getElementById("root")!).render(
  <StrictMode>
    <ThemeProvider>
      <Toaster>
        <App />
      </Toaster>
    </ThemeProvider>
  </StrictMode>
)
```

Replace Vite's `README.md` with a short one: the project name, `pnpm install && pnpm dev`, and a pointer to `CLAUDE.md` and `docs/GUIDE.md`.

## Step 11. CI workflow and GitHub templates

Write `.github/workflows/ci.yml` (runs install, typecheck, lint, React Doctor, test and build on every PR; details and the optional deploy job in `docs/DEPLOY_AND_CI.md`):

```yaml
name: CI

on:
  pull_request:
  push:
    branches: [main]

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

permissions:
  contents: read

jobs:
  check:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4        # reads "packageManager" from package.json if set
        with:
          version: 10
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: pnpm
      - run: pnpm install --frozen-lockfile
      - run: pnpm typecheck
      - run: pnpm lint
      - run: pnpm doctor
      - run: pnpm test
      - run: pnpm build
```

Then write the GitHub templates: each block in `GITHUB_TEMPLATES.md` to the path in its heading (`.github/pull_request_template.md`, `.github/ISSUE_TEMPLATE/bug.yml`, `task.yml`, `config.yml`, `.github/CODEOWNERS`). Copy the contents exactly; leave `{{GITHUB_USERNAME}}` for the owner.

## Step 12. When the app needs D1, R2 or other bindings (not now)

The skeleton runs without any storage. When a feature needs it, `backend-developer` follows `docs/BACKEND_CLOUDFLARE.md` §10 (a `wrangler.jsonc` reference with every binding type) and turns on the matching pieces. For D1 that is:

1. `npx wrangler d1 create {{PROJECT_SLUG}}`, then uncomment `d1_databases` in `wrangler.jsonc` with the id; `pnpm cf-typegen`.
2. Write the schema in `migrations/0001_init.sql`; `pnpm db:migrate:local`.
3. Switch the tests to the in-memory D1 harness: replace `vitest.config.ts` and add `worker/test/harness.ts` and `worker/test/api.ts` exactly as in `docs/TESTING.md` §1 (migrations read with `readD1Migrations("./migrations")`, passed as the `TEST_MIGRATIONS` binding, applied per test file with `applyD1Migrations`).

Don't do this during the initial setup; the verify step below expects the storage-free skeleton.

## Step 13. Verify

All must pass before reporting done:

```bash
pnpm install
pnpm cf-typegen
pnpm format
pnpm typecheck
pnpm lint
pnpm doctor
pnpm test
pnpm build
```

Then a quick run: `pnpm dev`, open http://localhost:5173 and check that the Overview page renders with "API is up", `/states` renders, an unknown path shows "Page not found", the theme menu switches light and dark, and the console has no errors. `curl -s http://localhost:5173/api/health` answers `{"ok":true,...}`. Stop the dev server afterwards.

Finally list the placeholders left for the owner: `grep -rnE "\{\{[A-Z0-9_]+\}\}" --exclude-dir=node_modules .`

| Problem | Fix |
| --- | --- |
| `pnpm install` fails with "Ignored build scripts: workerd" | `allowBuilds` in `pnpm-workspace.yaml` must have `workerd: true` and `esbuild: true` |
| Typecheck can't find `worker-configuration.d.ts` | Run `pnpm cf-typegen` |
| Dev server stops answering on `127.0.0.1` | Use http://[::1]:5173 or restart `pnpm dev` |
| `pnpm deploy` doesn't deploy | Use `pnpm run deploy` (`pnpm deploy` is a different pnpm command), and only with the owner's go-ahead |

## Step 14. Hand over

1. If the folder isn't a git repo yet: `git init`, then `git checkout -b dksk/chore/project-setup`. Don't commit; the owner approves commits.
2. Report to the owner: the verify results, the placeholders still to fill (M0-1 in `docs/ROADMAP.md`), and that `00-START-HERE.md`, `SETUP.md` and `GITHUB_TEMPLATES.md` can be deleted from the repo now (or kept outside it for the next project).
