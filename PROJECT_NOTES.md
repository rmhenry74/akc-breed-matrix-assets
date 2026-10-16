# Daisy Roo Breed Guess — Project Notes

**Version:** v1.0 · **Date:** 2026-10-16 · **Owner:** Matt Henry
**Purpose:** Capture everything built and decided so far, plus the next-version design direction, ahead of a rebuild in Claude Code → Vercel → database.

---

## 1. What this is

Rachel and Ty rescued a dog, **Daisy Roo**, and sent off an **Embark** DNA test. While the results are pending, the family gets a web page with breed photos and guesses which breeds are in her mix. When results arrive, guesses get scored.

---

## 2. What exists today (v0 — the Claude artifact)

**Published page:** `https://claude.ai/artifact/NxVV7Ff1Qh81xjeXjr6Zp3` (private artifact; a single self-contained HTML file, ~13 MB with all photos embedded as base64).

**Asset repo:** `github.com/rmhenry74/akc-breed-matrix-assets` (public). Contents:

| File / folder | What it is |
|---|---|
| `img/*.jpg` | 650 photos, 320×240 JPEG, two per breed, named `<breed-slug>-1.jpg` / `-2.jpg`. All from Wikimedia Commons. |
| `credits.json` | One row per photo: breed, group, AKC status, slot, path, Commons file name, author, license, source URL. **This is the attribution record; the rebuilt app must display it.** |
| `manifest.json` | Per breed: group, Wikipedia title, AKC status, and up to ~6 scored candidate images (with license/size/date) — use for swapping a photo without re-searching. |
| `done.json` | Per breed: which two files are in use; `missing` slots where Commons had nothing usable. |
| `titles.json` | Breed → Wikipedia article title, group, status. |
| `embark_breeds.json` | Embark's full 424-page breed directory (handle, name, village flag). |
| `rejected.json` | Images reviewed and rejected (keep so they aren't re-picked). |

**Breed list (325 rows):**
- 205 AKC-recognized breeds (as of 2026-01-01, incl. the three new 2026 breeds).
- 58 AKC Foundation Stock Service breeds, 11 AKC Miscellaneous class.
- 51 breeds the AKC doesn't register but Embark reports (APBT, American Bully, American Bulldog, Olde English Bulldogge, Alapaha Blue Blood Bulldog, Catahoula, Black Mouth Cur, Blue Lacy, Carolina Dog, Koolie, etc.).
- Gray Wolf, Coyote, Dingo, New Guinea Singing Dog, Wolfdog.
- One **Village Dog** row standing in for Embark's 59 regional village-dog populations.
- **Supermutt** (Embark's catch-all).
- **Excluded on purpose:** ~39 designer mixes (Goldendoodle, Cavapoo, Pomsky…). Embark reports parent breeds, not the mix.

**Group assignment:** the AKC's seven functional groups (Sporting, Hound, Working, Terrier, Toy, Non-Sporting, Herding) applied to every breed, AKC or not, plus two small special sections: "Wild canids & village dogs" and "Catch-all." Non-AKC breeds were placed by function (Catahoula → Herding, American Bulldog → Working, APBT and American Bully → Terrier so pit-types sit together).

**Photo pipeline (reusable):** Wikipedia article → image list + Commons metadata (license, size, date) → auto-reject (drawings, maps, logos, pre-1965, <500 px, non-free) → score (infobox image first, landscape, breed name in file, recent, large; penalize puppies/collages) → crop 320×240 → commit to repo. Ran in the Composio sandbox because this workspace can't reach Wikimedia. Two visual review passes replaced ~95 images. Known trap: template images that appear on many Wikipedia pages (a black Lab in snow, an elk) get picked for the wrong breed; blacklist them.

**Photo gaps:** Fonni's Dog and Pungsan have one photo each.

**Search aliases already defined** (carry forward): pit / pitbull / pit bull / pittie / bully / staffy / amstaff → the nine pit-type breeds; lab, golden, GSD, husky, heeler, cur, mutt, wiener/doxie, chi, frenchie, yorkie, westie, rottie, dobie, sheltie, aussie, corgi, pom, pyr, mastiff, wolf, coyote. "doodle" shows an explainer instead of results.

---

## 3. Decisions made (and why)

- **Reference list = Embark's panel, not AKC or UKC.** The DNA result is reported against Embark's list, so every possible result should be a pickable option. The AKC alone omits APBT; the UKC omits others.
- **AKC status is metadata, not structure.** Show it as a small badge, never as a grouping. See §4 for the revised rule.
- **Photos from Wikimedia Commons** for licensing clarity. Every photo carries author + license + source; CC BY-SA requires that attribution stay visible somewhere in the app.
- **Village dogs collapsed to one row; designer mixes excluded** (reasoning above).
- **Repo is the system of record for assets and metadata.** The artifact page is a build output.

---

## 4. Next version — design direction (from Matt, 2026-10-16)

### Remove
- **All "pick" mechanics** on the browse page: the Pick buttons, the bottom tray, the submit/copy flow.
- **"Not AKC" badge.** Show nothing for non-AKC breeds. Optionally a small "AKC" tag after AKC breed names so the AKC ones are identifiable. (FSS/Misc tags also go; they're detail nobody needs.)
- **"AKC only / Not AKC" filter.** Not needed.

### Browse page (the photo matrix)
- A **heart** next to each breed name. Tap = favorite. Favorites persist per person.
- **Mobile-first.** Current problems: the sticky header (search + group chips + filters) plus the bottom tray leave room for only one row of dogs on a phone. Target: **3–4 breed rows visible at once** on a phone.
  - Shrink or collapse the sticky area; consider a single compact search field with filters behind a button or a slide-in sheet.
  - No fixed bottom bar.
- **Simpler filtering for phone.** Fewer controls; group filter can live behind a "Filter" button. Keep search with aliases.
- Keep: two photos per breed, tap-to-enlarge with credit, group banding.

### Picking page (separate page)
- **Two-pane layout** (stack on mobile):
  - **Left:** where the final picks are made — a dropdown, or drag from the right pane. (Number of final picks: previously three; confirm.)
  - **Right:** everything the person hearted. Example: 15 favorites show here.
- **Two levels of favorite.** From the favorites list a person can:
  - **Remove** entirely ("for sure not it").
  - **Promote** to a second tier ("made the first cut / definitely keep"). Tier naming TBD (e.g., Favorites → Shortlist → Final picks).
- Final picks drawn from the shortlist (or any favorite — decide).

### Home page (or top of picking page — undecided)
- **Several photos of Daisy Roo.**
- **"About Daisy Roo"** section: notes from Rachel and Ty — habits, personality, anything that helps people guess. Needs to be editable by them (CMS field or markdown in the DB).
- Keep it very simple for family members.

### Platform
- Build in **Claude Code**, deploy on **Vercel**, back with a **database** (favorites, tiers, final picks, per-person; Daisy Roo bio; later: Embark results + scoring).
- Serve photos as static files from the repo (or Vercel Blob), not base64 in the page.

---

## 5. Open items

| Item | Owner / trigger | Default meanwhile |
|---|---|---|
| Number of final picks per person | Matt | 3 |
| Names of the favorite tiers | Matt | Favorites → Shortlist → Picks |
| Where "About Daisy Roo" lives (home vs. picking page) | Matt | Home page |
| Identity: how a family member is recognized (name field, magic link, no auth) | Matt, at build | Simple name entry, stored with a device token |
| Scoring rules when Embark results arrive (exact match only? partial credit for %?) | Matt, after results | Point per correct breed in final picks |
| Daisy Roo photos and bio text | Rachel & Ty | Placeholder until provided |
| Second photos for Fonni's Dog and Pungsan | Nice-to-have | One photo shown |
| DB choice (Vercel Postgres / Neon / Supabase / Turso) | Matt, at build | Whatever the Thanksgiving app already uses |

---

## 6. Sources worth keeping

- Embark breed directory: `https://embarkvet.com/pages/dog-breeds` (list loads client-side; full set recovered from `https://embarkvet.com/sitemap_metaobject_pages_1.xml`).
- AKC breed directory incl. FSS: `https://www.akc.org/dog-breeds/foundation-stock-service/`; Miscellaneous class: `https://www.akc.org/dog-breeds/miscellaneous-class/`.
- AKC 2026 additions: Basset Fauve de Bretagne, Teddy Roosevelt Terrier, Russian Tsvetnaya Bolonka (effective 2026-01-01).
