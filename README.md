# 🎷🐋 Beluga

### Build your own store.

Beluga is open-source software for running your own ecommerce site. React, Node.js
and [Stripe](https://stripe.com/), on a server you control, in code you can change.

**[Read the docs](https://belugajs.com/start/what-beluga-is/)** ·
[Quickstart](https://belugajs.com/start/quickstart/) ·
[Examples](https://belugajs.com/examples/)

---

## What you get

- **Design your own store.** Theme it from the admin, or replace the storefront
  components outright.
- **Products, variants and collections.** Up to three priced axes per product,
  images per variant, CSV in and out.
- **Cart and checkout.** Stripe's hosted checkout page, so card fields never touch
  your server.
- **Orders, shipping, tax and refunds.** Zones and weight bands, Stripe Tax, refunds
  that restock.
- **Email and webhooks.** Order emails over any SMTP, and signed webhooks to whatever
  you run.
- **Accounts.** Staff invitations, customer logins, abandoned-cart reminders.

Underneath that is a **React 19 storefront and admin**, an **Express 5 API** in
TypeScript, a **SQLite or Postgres database**, and **Stripe Checkout** for payment.
[What Beluga is](https://belugajs.com/start/what-beluga-is/) has the full list, with
screenshots of a real store.

## A match made in heaven?

**🐋 Beluga will be your jam if**

- you want to sell products online
- you want full control over the look and feel of your store
- you want to save money by hosting your own site
- you are comfortable working with React and Node.js — the database is SQLite or
  Postgres, your pick
- you work with an AI assistant and want it to know the real rules — the docs ship as
  one Markdown file for exactly that

**You'd be better off with a paid service if**

- you don't want to write code to create your store — the admin covers a lot, but the
  landing page is a React component
- you want someone else to host it — there is no hosted Beluga, and there will not be
  one
- you need live carrier rates or shipping labels — Beluga ships zones and weight
  bands, not a carrier integration
- you want customer support while setting up — Beluga is open-source software, not a
  paid service

## Quickstart

You need **Node 22**. Node 18 fails with misleading "command not found" errors from
the build tooling, so run `nvm use` first; the version is pinned in `.nvmrc`.

```bash
nvm use
npm install
npm run setup       # generates secrets, checks your Stripe key, makes your admin account
npm run dev:all     # storefront on :5173, API on :4000
```

Then open <http://localhost:5173>, and <http://localhost:5173/admin> to sign in with
the account setup created. Neither needs a Stripe account or a database server:
SQLite is just a file, and a store without Stripe still browses, it just cannot take
money.

Prefer a browser? Skip `npm run setup` and run `npm run dev:all` on its own. The
server starts unconfigured on purpose, and <http://localhost:5173/setup> walks the
same steps.

Taking a test payment, setting up without a terminal, and deploying are all in the
[Quickstart](https://belugajs.com/start/quickstart/) and
[Deploy your store](https://belugajs.com/tutorials/deploy/).

## Scripts

| Command | What it does |
| --- | --- |
| `npm run setup` | Interactive first-run setup |
| `npm run dev:all` | Storefront and API together |
| `npm run dev` | Storefront only (Vite) |
| `npm run dev:server` | API only |
| `npm run build` | Typecheck, then compile the server to `dist-server/` and build the client to `dist/` |
| `npm start` | Run the compiled server (`node`, no TypeScript at runtime) |
| `npm run typecheck` | Types only |
| `npm run lint` | ESLint |
| `npm test` | Unit and component tests (Vitest) |
| `npm run test:e2e` | Browser tests (Playwright) |
| `npm run db:migrate` | Apply migrations |
| `npm run db:seed` | Migrate, then load the demo catalogue |
| `npm run db:generate` | Regenerate migrations after a schema change |

## Layout

```
src/       React 19 storefront (Vite)
src/admin/ Admin and setup wizard, loaded on demand
server/    Express 5 API (TypeScript)
scripts/   The `npm run setup` CLI
db/        Drizzle schema, migrations, repository, seed
shared/    zod schemas + helpers, imported by both sides
e2e/       Playwright specs
docs/      Task briefs, the roadmap, and guides for building on Beluga
legacy/    v1 code, kept for reference — not built
```

Two things live on disk and must persist across a redeploy: the SQLite file under
`data/`, and uploaded images under `ASSETS_DIR` (`public/assets` by default). That is
why Beluga is one long-lived Node process rather than a serverless app;
[The shape of a deployment](https://belugajs.com/deploying/shape/) says what that
means for where you host it.

## The rules worth knowing

Most of Beluga is ordinary React and Express, and you can change it the ordinary way.
A handful of rules are not, because breaking them produces code that compiles, passes
review, and is wrong:

- **Money never comes from the request.** Prices and totals are read from the
  database on every path, and are always integer cents.
- **The webhook is the only thing that marks an order paid.** The success redirect
  proves nothing — a buyer can close the tab, and the URL can be visited directly.
- **Publishing is the only thing that writes to Stripe.** The admin saves to your
  database as you type; nothing reaches Stripe until you press Publish.
- **Every admin route is behind `requireAdmin` and a CSRF check**, applied to the
  whole router, so a new route cannot be added unprotected by accident.

[The invariants](https://belugajs.com/start/invariants/) has all seven, and why each
one exists. If you are forking Beluga,
[`docs/building-on-beluga.md`](docs/building-on-beluga.md) maps which files are
cosmetic, which are a documented seam, and which hold one of those rules.

## Working with an AI assistant

The whole docs site is also one Markdown file at
[belugajs.com/docs/all.md](https://belugajs.com/docs/all.md), so an assistant can read
the real rules instead of guessing them. There is a
[CLAUDE.md for your fork](https://belugajs.com/examples/claude-md/) to start from, and
[For AI assistants](https://belugajs.com/reference/ai/) says what else is worth
pointing one at.

## Secrets

Never commit keys. Copy `.env.example` to `.env` for local development and use your
platform's environment variables in production. The Stripe **secret** key is
server-only and never reaches a browser; only the publishable key does.

## Contributing

Task briefs and the roadmap are in [`docs/tasks/`](docs/tasks/), and
[Conventions and contributing](https://belugajs.com/building/conventions/) summarises
how work lands. Accessibility is a build gate rather than a score:
`e2e/accessibility.spec.ts` runs axe against every public route at desktop and phone
widths, and a violation names the element and fails the build.

---

Looking for the original Beluga? [View the v1 docs](https://v1.belugajs.com/). v1
stopped working when Stripe removed the SKUs and Orders APIs it was built on; v2 is a
rebuild, and v1's code stays in [`legacy/`](legacy/) for reference. Before either,
there was [react-stripe-store](https://github.com/binx/react-stripe-store).

🎷🐋 Beluga is built by [rachel binx](https://rachelbinx.com/). If you build something
on it, please say so — and if it has saved you a platform fee or two, there is a
coffee link.

<a href="https://www.buymeacoffee.com/binx" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/lato-blue.png" alt="Buy Me A Coffee" height="51px" width="217px"></a>
