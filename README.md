# Happy Jack site — post-launch additions for the Replit build

Suggestions for **`happyjack-site.replit.app`**, derived from what people actually type into
Google for Happy Jack, Pine, Strawberry and the surrounding Rim communities.

**Prepared:** 24 November 2026 · **Author:** Bot 1 (research bot) on behalf of Jackie Thresher
· **Target:** the Replit build, before and shortly after cutover to `happyjackrealestate.com`

---

## Read this first

The build is in good shape. All the page types we expected to be missing are already there —
land, homes, cabins, subdivisions, Pine & Strawberry, Blue Ridge, guides, agents, map search.
Nothing here asks anyone to rewrite a page that already exists.

Every item below is a **gap between a phrase people type and a phrase the site says**. They are
small, additive, and each one has a stated way to check it is done.

## What is in here

| File | What it asks for | Effort | When | Issue |
|---|---|---|---|---|
| [`PRE-CUTOVER-CONCERNS.md`](PRE-CUTOVER-CONCERNS.md) | **The backup was verified against the live site. Read before switching DNS** — deployment config, sitemap, redirects, posted 25 Nov | — | **Before cutover** | [#9](../../issues/9) |
| [`BACKUP-REVIEW-CHECKLIST.md`](BACKUP-REVIEW-CHECKLIST.md) | **What a Wix export must contain and how it is reviewed** — six items, each with its own pass condition; the DNS baseline to check against; what only the owner can do | — | **Before uploading a backup** | [#9](../../issues/9) |
| [`AUDIT-preview-2026-11-25.md`](AUDIT-preview-2026-11-25.md) | **External audit of the republished preview** — 57/57 pages 200, staging protection correct, and six approved 301s returning 404 | `launch-ready-20261125-225205` | **Before DNS** | [#9](../../issues/9) |
| [`LAUNCH-SEQUENCE.md`](LAUNCH-SEQUENCE.md) | **The order to do it in** — lower TTLs first, deploy from the tag, verify before DNS, one short window, rollback in five minutes | — | **Before changing any record** | [#9](../../issues/9) |
| [`CUTOVER-INSTRUCTIONS-2026-11-25.md`](CUTOVER-INSTRUCTIONS-2026-11-25.md) | **Step-by-step cutover, plain language** — the three code strings, the five DNS records, the checks, the undo card. Drop-in 301 patch in [`patches/`](patches/happyjack-cutover-server.patch) | — | **Cutover day** | [#9](../../issues/9) |
| [`00-launch-gate.md`](00-launch-gate.md) | Six things that must be true at cutover or the site is invisible to Google | — | **Before launch** | [#1](../../issues/1) |
| [`01-no-hoa-land.md`](01-no-hoa-land.md) | Answer "land for sale … no HOA" — the single most specific buying phrase in this market | M | After launch | [#2](../../issues/2) |
| [`02-for-sale-by-owner.md`](02-for-sale-by-owner.md) | Answer "by owner" / FSBO searches on the search pages | S | After launch | [#3](../../issues/3) |
| [`03-happy-jack-facts.md`](03-happy-jack-facts.md) | One block of plain facts: zip code, population, elevation, how far | S | After launch | [#4](../../issues/4) |
| [`04-forest-lakes-christopher-creek.md`](04-forest-lakes-christopher-creek.md) | Two neighbouring communities with real search demand and no coverage | M | After launch | [#5](../../issues/5) |
| [`05-property-spec-modifiers.md`](05-property-spec-modifiers.md) | Tiny homes, mobile/manufactured, square footage, well depth | S–M | After launch | [#6](../../issues/6) |
| [`06-schema-upgrades.md`](06-schema-upgrades.md) | `RealEstateAgent`, plus one correction to an earlier claim | S | After launch | [#7](../../issues/7) |
| [`07-measurement.md`](07-measurement.md) | Turn on Search Console, record a baseline, stop guessing | S | Launch week | [#8](../../issues/8) |

## How this was built (so it can be checked)

- **Query evidence** — Google Autocomplete, harvested 24 November 2026 from
  `suggestqueries.google.com` with `hl=en&gl=us`, 19 seed phrases. Autocomplete shows that a
  phrase is *searched by real people*; it does **not** show volume. No volume claims are made
  anywhere in this repo. Raw completions are in [`evidence/autocomplete-2026-11-24.md`](evidence/autocomplete-2026-11-24.md).
- **Site evidence** — each of the 14 highest-value pages fetched live from
  `https://happyjack-site.replit.app` on 24 November 2026 and searched for the phrase in
  question. Counts are exact substring counts on the served HTML, quoted per item.
  Matrix in [`evidence/term-matrix-2026-11-24.md`](evidence/term-matrix-2026-11-24.md).
- **Nothing here is a ranking prediction.** The success measure for this site is phone calls,
  not positions. Search visibility is how someone finds the number to call.

## How to review

Open an **Issue** for anything you disagree with, want to defer, or need a decision on — one
issue per file name is fine, or comment on the existing issues. Closing an issue is a valid
"done" signal. If something here contradicts something the build already does, the build wins;
say so in the issue and it gets corrected here.

## One correction worth stating

The staging build is currently **not indexable**: every page carries
`<meta name="robots" content="noindex, nofollow, noarchive">`, `robots.txt` says
`Disallow: /`, there is no `/sitemap.xml` (404) and no canonical tag anywhere. That is correct
for staging and must be reversed at cutover. (The sitemap is not missing from the build — staging
deliberately drops it; see [`PRE-CUTOVER-CONCERNS.md`](PRE-CUTOVER-CONCERNS.md) C2.) It is item one in
[`00-launch-gate.md`](00-launch-gate.md) and it is the only thing here that is genuinely urgent.
