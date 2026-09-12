# ClarionX

Complete source for the ClarionX website: Home, About, six Solutions, and Contact.

Live site: https://clarionx-ai.reneshkumar-pk.chatgpt.site

## Requirements

- Node.js 24 LTS recommended (minimum 22.13.0).
- npm, bundled with Node.js.
- Git to clone and update the repository.

The project runs on Windows, macOS, and Linux. You do not need the Sites plugin, a Cloudflare account, or API keys to develop locally.

## Run locally

Clone the public repository and open its root:

```sh
git clone https://github.com/renesh06/ClarionX-WebSite.git
cd ClarionX-WebSite
npm ci
npm run db:migrate
npm run dev
```

Open http://localhost:5173. Use the URL printed in the terminal if you specify another port:

```sh
npm run dev -- --port 5174
```

`npm ci` installs all dependencies exactly as recorded in `package-lock.json`. Do not copy `node_modules` between computers.

`npm run db:migrate` creates the local contact database and applies pending SQL migrations. It is safe to rerun: Wrangler records migrations that have already run. Local data is stored in `.wrangler/state` and is never committed.

## Project structure

```text
app/
  page.tsx                   Home and landing page
  about/page.tsx             Company story and metrics
  solutions/page.tsx         All six solutions
  contact/page.tsx           Contact page
  api/contact/route.ts       Validated contact submission endpoint
  layout.tsx                Shared header, footer, and metadata
  globals.css               Theme, layouts, and responsive styles
  chatgpt-auth.ts            Optional Sites authentication helpers
components/
  site-header.tsx            Desktop and mobile navigation
  contact-form.tsx           Form state, validation, and confirmation
  ui/                       Included UI primitives
public/
  hero.webp                 Optimized custom hero artwork
  favicon.svg               ClarionX icon
db/
  index.ts                  Cloudflare D1 binding helper
  schema.ts                 Inquiry table definition
drizzle/                    SQL migrations and Drizzle metadata
scripts/
  migrate-local.mjs         Repeatable local database initialization
  run-framework.mjs         Development/build entry point
  sites-env.mjs             Local runtime defaults
  execution-profile.mjs     Defaults clean checkouts to portable mode
  ...                       Retained build and installation helpers
build/sites-vite-plugin.ts  Sites build integration
.openai/hosting.json         Existing Site identity and DB binding
.env.example                Environment configuration notes
vite.config.ts              Framework and local Worker bindings
cloudflare-env.d.ts          Cloudflare binding types
drizzle.config.ts           Schema migration configuration
next.config.ts              Framework compatibility configuration
tsconfig.json               TypeScript configuration
package.json                Commands and dependency declarations
package-lock.json           Exact resolved dependency versions
```

## Commands

| Command | Purpose |
| --- | --- |
| `npm ci` | Install the locked dependency set |
| `npm run dev` | Development server with hot reload |
| `npm run typecheck` | Check TypeScript without emitting files |
| `npm run build` | Create the production Worker and browser assets |
| `npm start` | Preview the production build locally; run build first |
| `npm run db:migrate` | Apply pending migrations to local D1 |
| `npm run db:generate` | Generate a new migration after changing the schema |
| `npm run lint` | Run the configured ESLint checks |

To preview a production build:

```sh
npm run build
npm run db:migrate
npm start
```

Open the URL printed by Wrangler. A static file server alone cannot run the Contact API.

## Stack and dependencies

React 19, TypeScript, Vinext/Vite, Tailwind CSS 4, Radix-based UI primitives, Lucide icons, Cloudflare Workers/D1, and Drizzle migrations.

Every runtime and development dependency is declared in `package.json` and pinned by `package-lock.json`. Keep both files when copying the project. Google Fonts are loaded from Google's font service; the page falls back to sans-serif when offline.

## Contact form behavior

`POST /api/contact` validates name, email, message, and a UUID request identifier. It saves accepted inquiries in D1. Repeated submissions with the same identifier do not create duplicate rows.

The endpoint has input length limits, a honeypot field, and a same-origin check. Failed submissions keep the user's draft. There is no public endpoint for reading inquiries.

**Submissions are stored in the database. Email notifications are not configured.** The site owner can inspect hosted inquiries through Sites database tools.

## Configuration

No `.env` file is required for local use. The logical `DB` binding is declared in `.openai/hosting.json`; `vite.config.ts` supplies a local D1 instance. Keep the local database ID in sync with `scripts/migrate-local.mjs` if you change it.

Secrets, local databases, generated build output, runtime profiles, and installed dependencies are ignored by Git. Never commit source-repository write tokens.

Normal Git clones default to the portable runtime. The optional managed Linux profile is an integration detail for Sites and is not needed to run this project.

## Hosting

This repository is attached to the existing ClarionX Site through `.openai/hosting.json`. Preserve that project ID when updating this website. Sites packages the built Worker and assets, applies production migrations, and publishes the selected source revision.

Building locally does not publish. Source-only documentation and setup updates can be pushed without changing the live website.

For another hosting provider, use a Cloudflare Workers-compatible runtime and provision its D1 binding and migrations. Do not replace the logical local placeholder database ID with a production database ID for routine development.

## Navigation

Page links use standard HTML anchors so Home, About, Solutions, and Contact do not depend on client-side routing. Preserve this behavior when editing navigation.

## Troubleshooting

- **Contact submission fails locally:** run `npm run db:migrate` in this checkout and restart the development server.
- **Port is occupied:** stop the server that owns it or choose another port with `npm run dev -- --port 5174`.
- **Windows blocks npm.ps1:** use `npm.cmd` instead of `npm`. Do not change system-wide execution policy.
- **npm shell launcher is broken:** invoke npm's JavaScript entry point directly using the path in your Node installation. This is an environment-specific workaround, not a project dependency.
- **Sandbox prevents production preview:** run the commands in a normal terminal with access to the checkout. Development, build, and database files must be writable.
- **A schema change is needed:** edit `db/schema.ts`, generate a new migration, inspect the SQL, then run local migrations. Do not rewrite already-published migrations.
