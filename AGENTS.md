<!-- BEGIN colby-ecosystem: managed by pull-agents, do not edit inside this block -->
## Machine-wide agent briefings (synced — do not edit here)

Pulled from `jmbish04/colby-ecosystem` by `.github/workflows/agents-sync.yml`, right-sized to this
repo's stack. Read them before your first load-bearing change; where they and the
notes below this block disagree, they win. Edit them in `jmbish04/colby-ecosystem`, never here.

@.agents/ecosystem/AGENTS.md
@.agents/ecosystem/AGENTS-maestro.md
@.agents/ecosystem/AGENTS-github.md
@.agents/ecosystem/AGENTS-cloudflare-workers.md
@.agents/ecosystem/AGENTS-frontend.md
@.agents/ecosystem/AGENTS-ai.md

The scripts in `scripts/` carrying a `colby-ecosystem: managed script` banner come from there too.
Use them rather than writing another: `scripts/gh.mjs` is the GitHub entry point
and wraps the `gh-tools` CLI, which already solves the auth this machine trips on.

Work is tracked in **colby-maestro** (`https://colby-maestro.hacolby.workers.dev/mcp`,
bearer `$(tokens show WORKER_API_KEY --value-only)`) — never in this repo and never
in colby-ecosystem, which only stores config. Refresh this block with
`.agents/ecosystem/pull-agents`.
<!-- END colby-ecosystem -->

# AGENTS.md — Agent Visibility Template

Notes for AI agents working on this template.

## What this template is

One enriched content store (`Resource[]`) projected onto many agent-discovery
surfaces. The data is enriched once by Workers AI and cached in KV; every
surface (`/llms.txt`, `/index.json`, `/<slug>.md`, `/robots.txt`, JSON-LD) is
just a different rendering of that same store.

## Architecture

```
src/
  worker/index.ts        Hono app: routes for every surface + JSON API
  enrichment/index.ts    Workers AI enrichment (raw page -> structured Resource)
  enrichment/surfaces.ts Pure render functions, one per surface
  lib/store.ts           KV-backed enriched store (get / upsert / clear)
  lib/content.ts         Sample content (zero-config demo data)
  lib/types.ts           Shared types (Resource, RawResource, Env, SiteConfig)
  lib/web-bot-auth.ts    OPTIONAL agent-identity module (off by default)
  react-app/             Surface-explorer UI
test/index.test.ts       Worker tests (vitest-pool-workers, via SELF.fetch)
```

## Conventions

- **`surfaces.ts` is pure.** Render functions take `RenderCtx` and return
  strings/objects. No I/O. This keeps surfaces easy to test and add to.
- **Enrichment must never hard-fail.** `enrichResource` falls back to
  `fallbackEnrichment` on any model/parse error so surfaces always render.
- **Keep readability and identity separate.** Web Bot Auth is about _who_ an
  agent is, not _what_ it can read. It lives in its own module and is gated by
  `ENABLE_WEB_BOT_AUTH`. Don't wire it into the core surfaces.

## Adding a surface

1. Add a pure renderer to `src/enrichment/surfaces.ts`.
2. Add a route in `src/worker/index.ts` (send the `Content-Signal` header for
   text/JSON surfaces; add `cors()` if agents fetch it cross-origin).
3. Add it to the `/api/site` `surfaces` list so the UI shows it.
4. Add a test in `test/index.test.ts`.

## Validating changes

```bash
npm run build   # tsc -b && vite build
npm test        # vitest (hits live Workers AI — needs credentials)
```

After editing `wrangler.jsonc`, rerun `npx wrangler types`.
