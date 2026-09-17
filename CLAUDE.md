# Oliver's Products — Personal Portfolio Site

## What this is

"Oliver's Products" — one shareable page listing everything Oliver has shipped: his own products, client websites, and experiments, presented as one cohesive track record instead of scattered, unrelated domains.

Not a product for other people to use. It's for Oliver: to drop in a bio, cold outreach to local businesses, or a message to a collaborator. Optimize for three things: looking impressive, staying effortless to maintain, and letting a visitor understand + click through fast.

## Current deployment

- Source of truth: GitHub repo `olivrrut-stack/Dep-Shelf` (public), `main` branch.
- Deployed on Vercel as a static site (imported directly from the GitHub repo — no build step, no framework).
- Files: `index.html` (page/styles/behavior) + `data.json` (all entry + contact-link data, fetched at page load) + this `CLAUDE.md`.
- It started as a Claude Artifact (hence the CSP-safe, no-backend design below) and was later deployed to Vercel for a real domain. The architecture stayed static/no-backend even after that move — see Save/publish flow.

## Content model

`data.json` has two top-level keys: `contactLinks` (array of `{ label, href }`) and `entries` (array of objects with these fields):

- `id` (stable string/slug)
- `name` (required — the only required field)
- `domain` (URL; bare domains like `example.com` get auto-prefixed to `https://` on save)
- `tagline` (one-line description, shown on the row/card)
- `description` (full description, shown in the details panel)
- `section`: `"products" | "client" | "experiments"`
- `status`: `"live" | "building" | "paused" | "retired"` — shown as a small status marker so visitors never click into a dead end
- `type` (e.g. "Chrome extension tool", "SaaS", "dev tool", "client website")
- `stack` (array of tech stack strings)
- `launchDate`
- `featured` (bool — featured entries sort first within their section)
- `hidden` (bool — pulled from public view but not deleted; owner still sees it, dimmed)
- Client-specific (when `section === "client"`): `businessType`, `builtWhat` (what was built for them)

Sections only render if they have at least one visible entry — there are no empty placeholder slots. Only add an entry once it's real; don't pad the list.

## Page structure

1. **Hero**: "Oliver Rutland" + "Products, client work, and experiments — everything I've built, listed here." (no personal bio/location copy — that was deliberately removed), a dynamic headline counting *live* entries ("N sites live on the internet"), and contact links.
2. **Search + filter**: instant client-side search over name/domain/description/type/stack. `/` focuses the search box. Filter chips: All / Products / Client work / Experiments. No matches → empty state suggesting clearing the search.
3. **Sections**, in order: Your products, Client work, Experiments & in progress. Featured entries sort first within their section (currently none are marked featured).
4. **Entry row**: monogram icon (generated initials, no real favicons — CSP/no-backend), name, one-liner, domain, status label, one-click "Visit ↗" (opens in new tab).
5. **Details panel**: opens on click as a right-side slide-over (never a separate page). Full description, type, stack, launch date, status, big "Visit" button, and for client work: business type + what was built. Closes on Escape or outside click.

## Visual design

Committed single palette (not light/dark adaptive — the user pinned exact colors): deep navy background (`#0A1730`), white text, warm gold accent (`#E8BB3D`) used sparingly (live-count number, links, primary buttons, focus rings). Semantic status colors are distinct from the accent (green/orange/grey/rust for live/building/paused/retired). Flat hairline rows, no rounded cards. Type system: a slab-serif display face (Rockwell stack) for name/headings, a plain sans for body copy, and a monospace face for domains/stack/dates/labels (manifest/ledger feel).

## Owner mode (no real auth — it's a static page)

- Visiting with `?key=<secret>` unlocks owner mode; the key is stored in `localStorage` so it persists across visits on that device.
- The public link (shared with others) has no key param and never shows owner controls.
- **Caveat to remember**: anything in `data.json`, including hidden entries, is visible to anyone who fetches that file directly. "Hidden" means hidden from the rendered UI, not private. Nothing truly confidential goes in this data.
- In owner mode: Add button, Edit/Delete on every entry, hidden entries shown dimmed (still owner-visible), a form with name required and everything else optional.

## Save/publish flow (the part that differs from a normal live app)

The site is still static with no backend/API routes — **client-side edits cannot make themselves live for visitors on their own.** The agreed flow:

1. Owner edits (add/edit/delete/hide) update in-memory state immediately and cache to `localStorage`, so nothing is lost on refresh, and the owner sees a live preview of their own changes right away.
2. A **Publish** button copies the updated `data.json` contents (as JSON) to the clipboard with instructions.
3. Owner pastes that into a Claude Code chat and asks to update `data.json`; Claude edits that file, commits, and pushes to `main`.
4. Vercel is connected to the GitHub repo, so that push **auto-redeploys the live site** — this is what actually makes edits go live for everyone, not any in-browser mechanism.
5. A separate **Download backup** button always exports the same JSON as a file, independent of publishing — so the data is never trapped in one place.

## Content still needed from Oliver

1. Real one-line + full descriptions for ZipSnap, VibeCheck, No Crickets, Claude Usage Widget, and Niche Loop (all currently marked `PLACEHOLDER` in `data.json`).
2. A domain for Niche Loop once it has one.
3. Any client sites Oliver is comfortable showing publicly — name, domain, business type, what was built (none added yet; the Client Work section won't appear until at least one exists).
4. Real contact links (email + any socials) — currently placeholder `mailto:you@example.com` / placeholder GitHub link in `data.json`.
5. Oliver's chosen secret owner key for the `?key=` param, if he wants a memorable one instead of whatever he's already used.

## Known future upgrade path (now easier, since it's on Vercel)

Because this is a real Vercel deployment (not just an Artifact), it could gain a small serverless API route later to auto-fetch real favicons/OG data per domain, or an uptime check — that wasn't possible under the Artifact CSP but is now just an added `api/` function away, if wanted. Other future ideas: a "recently shipped" feed, a shipped-this-year counter, screenshots in the details panel, a custom domain in place of the default Vercel one.
