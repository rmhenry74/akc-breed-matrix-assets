# Daisy Roo — Claude Code kickoff prompt (v1.0, 2026-10-16)

Paste everything below the line into Claude Code, run from an empty folder named `daisy-roo`.

---

You are building **Daisy Roo**, a small family web app. My daughter Rachel and her husband Ty rescued a dog named Daisy Roo and sent off an Embark DNA test. While results are pending, family members browse breed photos, heart the ones they think are in her, narrow those down, and submit final picks. When results arrive we score the picks.

## Read first

1. Clone `https://github.com/rmhenry74/akc-breed-matrix-assets` into `./assets-src` (temporary) and read `PROJECT_NOTES.md` end to end. It holds the data inventory, decisions already made, the design direction, and open items with defaults. Follow it; where it names a default, use the default.
2. The data you need from that repo: `img/*.jpg` (650 breed photos, 320×240, two per breed), `credits.json` (photo attribution — must be shown in the app), `manifest.json` and `done.json` (breed → group, AKC status, which two photos). Build a single `breeds.json` seed from these: `{slug, name, group, akc (boolean), photos:[{file, author, license, source}]}`. Display names: use the names in `manifest.json` keys except apply the rename map in `PROJECT_NOTES.md` §2 if present; otherwise keep as is. Treat `status === "akc"` as AKC; FSS, Misc and none are all "not AKC" and get no badge.

## Stack (don't deviate without asking)

- Next.js (App Router, TypeScript), Tailwind CSS. Mobile-first.
- Postgres on Neon via the Vercel Marketplace, Drizzle ORM, migrations checked in.
- Deploy on Vercel. Photos served as static files from `public/breeds/` (copy them in from `assets-src/img`), not from the database and not base64.
- No auth provider. Identity is a name the person types once, stored with a random device token in a cookie; the token keys their favorites and picks. Show "You're playing as {name} · change" in the header.

## Data model

- `players(id, name, token, created_at)`
- `favorites(player_id, breed_slug, tier, created_at)` where `tier` ∈ {`liked`, `shortlist`}; unique on (player_id, breed_slug)
- `picks(player_id, rank 1..3, breed_slug)`; unique on (player_id, rank)
- `settings(key, value)` for the About text and photo list, editable later by Rachel and Ty
- Breeds are static JSON, not a table.

## Pages

**/ (Home)** — several photos of Daisy Roo (placeholder images until provided, loaded from `settings`), an "About Daisy Roo" section rendered from markdown in `settings`, and two big buttons: Browse breeds / Make my picks. Keep it very plain; family of all ages.

**/breeds (Browse)** — the matrix: breed rows banded by group (Sporting, Hound, Working, Terrier, Toy, Non-Sporting, Herding, then "Wild canids & village dogs", then "Catch-all"). Each row: name, a small "AKC" tag only when AKC-recognized, two photos (tap to enlarge with credit line and source link), and a **heart** that toggles the breed in/out of favorites (tier `liked`), persisted via a server action immediately. Requirements:
- On a phone, **3–4 breed rows visible at once**. One compact sticky bar only: a search field and a single "Filter" button that opens a bottom sheet with the group list. No bottom tray, no second sticky row.
- Search matches breed names and the alias list in `PROJECT_NOTES.md` §2 ("pit" surfaces all pit-type breeds, "lab", "husky", "heeler", "cur", "mutt", etc.). Typing "doodle" shows a one-line note that Embark reports parent breeds.
- Heart count shown in the bar ("♥ 12") linking to /picks.

**/picks (Picking page)** — two panes; side by side on desktop, stacked on mobile with the favorites list first.
- **Right / favorites pane:** everything the person hearted, grouped into two tiers: **Shortlist** (top) and **Liked** (below). Each card has the breed's first photo, name, and three actions: promote to Shortlist (or demote back to Liked), remove entirely. Removing also unhearts it on /breeds.
- **Left / final picks pane:** three ranked slots. Fill a slot by choosing from a dropdown of the person's shortlist (fallback: all favorites if shortlist is empty) or by dragging a card in (dnd-kit; must also work with tap-to-assign on touch). Save on change. Show "Your picks are in" once all three are filled; picks stay editable until an admin locks them (a `settings` flag).

**/credits** — full photo attribution list generated from `credits.json`. Link to it from the footer of every page.

**/admin** — behind a single shared password in an env var: edit About text and Daisy Roo photo URLs, see every player's picks in a table, toggle the lock flag. Scoring comes later; leave a TODO.

## Build order

1. Scaffold, Tailwind, Drizzle + Neon wired up, `breeds.json` built, photos copied. Commit.
2. /breeds with hearts persisted. Verify on a 390px-wide viewport that 3+ rows are visible below the sticky bar. Commit.
3. /picks with tiers, remove, dropdown + drag. Commit.
4. Home, /credits, /admin. Commit.
5. Create the GitHub repo `rmhenry74/daisy-roo` with `gh`, push, link to Vercel with the Vercel CLI, provision Neon from the Marketplace, run migrations, deploy. Give me the production URL.

Work in small commits with clear messages. Keep a `CLAUDE.md` in the repo root summarizing the stack, data model, and the conventions above so future sessions start oriented. Ask me only when something in the notes is genuinely ambiguous; otherwise take the stated default and note it in your final summary.
