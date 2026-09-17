# Oliver's Products — Personal Portfolio Artifact

## What this is

A single personal page, "Oliver's Products," shown as one shareable Claude Artifact link. It's a portfolio of everything Oliver has shipped on the internet — his own products, client websites, and experiments — presented as one cohesive track record instead of scattered, unrelated domains.

Not a product for other people to use. It's for Oliver: to drop in a bio, cold outreach to local businesses, or a message to a collaborator. Optimize for three things: looking impressive, staying effortless to maintain, and letting a visitor understand + click through fast.

## Hard constraint: it's a static Claude Artifact

Artifacts run under a strict CSP — no external network calls (no fetch/XHR/WebSockets, no calling Vercel's API, no database, no favicon fetching). Everything must be self-contained in one HTML file. This shapes several decisions below — **do not design around a backend that doesn't exist.**

## Content model

Each entry is an object with these fields:

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

## Page structure

1. **Hero**: Oliver's name, a short identity line ("an indie developer in the Texas Hill Country who builds tools for developers and websites for local businesses" — placeholder, confirm final wording), a dynamic headline counting *live* entries ("Oliver has N sites live on the internet"), and a few contact links (email + socials).
2. **Search + filter**: instant client-side search over name/domain/description/type/stack. `/` focuses the search box. Filter chips: All / Products / Client / Experiments. No matches → friendly empty state suggesting clearing the search.
3. **Sections**, in order: Products (ZipSnap, VibeCheck, No Crickets, + others), Client Work (local business sites — double as sales proof), Experiments / In Progress (Niche Loop, etc.). Featured entries sort first within their section.
4. **Entry row/tile**: monogram icon (generated, since real favicons can't be fetched under CSP), name in large type, one-liner, domain, status dot, one-click "visit" link (opens in new tab).
5. **Details panel**: opens on click as a modal/slide-over (never a separate page — visitor doesn't lose their place). Shows full description, product type, tech stack, launch date, status, big "Visit" button, and for client work: business type + what was built. Closes on Escape or outside click.

## Owner mode (no real auth — it's a static page)

- Visiting with `?key=<secret>` unlocks owner mode; the key is stored in `localStorage` so it persists across visits on that device.
- The public link (shared with others) has no key param and never shows owner controls.
- **Caveat to remember**: anything in the page source — including hidden entries — is technically visible to anyone who inspects it. "Hidden" means hidden from the rendered UI, not private. Nothing truly confidential goes in this data.
- In owner mode: Add button, Edit/Delete on every entry, hidden entries shown dimmed (still owner-visible), a form with name required and everything else optional.

## Save/publish flow (the part that differs from a normal live app)

There is no server, so **client-side edits cannot make themselves live for visitors** — this was flagged and resolved during brainstorming. The agreed flow:

1. Owner edits (add/edit/delete/hide) update in-memory state immediately and cache to `localStorage`, so nothing is lost on refresh, and the owner sees live preview of their own changes right away.
2. A **Publish** button copies an update payload to the clipboard with instructions.
3. Owner pastes that into a Claude Code chat and asks to update the portfolio artifact; Claude edits the source file and redeploys to the **same artifact URL** (same link every time — this is the whole point).
4. A separate **Download backup** button always exports the full entry list as JSON, independent of publishing — so the data is never trapped in one place, and could later seed a Next.js version on Oliver's own domain.

## Design direction

- Generous white space, one or two typefaces, restrained/neutral color palette, no decorative filler.
- Mobile-first — most visitors will open this from a text or social bio on a phone.
- Theme-aware (light/dark), since it renders in the viewer's Artifact theme.

## Build approach (agreed in brainstorming)

- Classified as **architectural** (brand-new build, nothing existing to extend) but treated lightweight since the deliverable is a single self-contained artifact file — no formal spec-doc-in-git ceremony, no separate implementation plan document.
- **Phase 1 (current)**: scaffold the full page with obviously-fake placeholder entries so Oliver can click through search/filter/details/owner-mode/edit flow and confirm the design feels right.
- **Phase 2**: swap placeholders for real content once Oliver provides it (see below), either by telling Claude directly or using the in-page edit UI + publish flow himself.

## Content still needed from Oliver (blocking Phase 2, not Phase 1)

1. The other ~2 domains/products beyond ZipSnap (getzipsnap.com), VibeCheck (vibecheckhq.app), No Crickets (nocrickets.dev) — names, domains, which section (is Niche Loop one of these, under Experiments?).
2. One-line description for each of the 5 products, plus status (live/building/paused/retired) and which (if any) are featured.
3. Client sites Oliver is comfortable showing publicly — name, domain, business type, what was built.
4. Final intro line for the hero (draft above, confirm or tweak).
5. Contact links to show (email + socials: X/GitHub/LinkedIn/etc).
6. Oliver's chosen secret owner key for the `?key=` param.

## Known future upgrade path (out of scope now)

If this later moves to a real Next.js app on Vercel (e.g. `olivers.dev`), it can add: auto-detected domains via the Vercel API, real favicons/preview images, live uptime dots, a "recently shipped" feed, a shipped-this-year counter, and screenshots in the details panel. The JSON backup export exists specifically to make that migration easy.
