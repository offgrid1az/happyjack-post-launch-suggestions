# 04 — Cover Forest Lakes and Christopher Creek

**Type:** content · **Effort:** M · **When:** after launch
**Where:** new sections/pages; cross-links from `/pine-and-strawberry`

---

## The query

Autocomplete, 24 November 2026:

**`forest lakes az`** — top ten include `forest lakes az homes for sale`, `forest lakes az zip code`,
`forest lakes az elevation`, `forest lakes az map`, `forest lakes az weather`.

**`christopher creek`** / `christopher creek az` — `christopher creek cabins`,
`christopher creek lodge`, `christopher creek campground`, `christopher creek restaurant`,
`christopher creek weather`.

Both are real-estate queries inside a tourist query set, which is exactly the pattern the site
already handles well for Pine and Strawberry.

## What the site says today

- "forest lakes" — **0 of 14** pages
- "christopher creek" — **0 of 14** pages

The site covers Happy Jack, Pine, Strawberry, Blue Ridge. Those four are the core, and the
coverage is good. These two sit on the same stretch of SR-260 and SR-260/SR-377 and are the two
obvious remaining neighbours.

## Why these two and not others

- **Forest Lakes** is a genuine second-home market with listings, a homeowners' character, and a
  distinct buyer (cooler, higher, less snow than the Rim proper in some pockets). It already has
  `homes for sale` demand in autocomplete.
- **Christopher Creek** is smaller and more cabin/rental-oriented, but "christopher creek cabins"
  is a live query, and the drive-time relationship to Happy Jack is a natural fit for the
  existing content.

Neither needs a 2,000-word guide to start. One page each, in the shape that already works on
`/pine-and-strawberry`: what the place is, what the property stock looks like, what to check
before buying, and how to search listings.

## Suggested shape (same as `/pine-and-strawberry`)

1. One-sentence answer to "what is this place" — elevation, drive time, what it is known for
2. What the property stock is (cabins / full-time homes / land / rental-heavy)
3. What to verify before buying here specifically — access, wells, septic, snow, insurance
4. Link into the property search
5. Phone number

**Do not** invent brokerages, price ranges, HOA details or water districts for these areas. If a
fact is not verified, leave it out — an empty section is better than a wrong one, and the build
team should not be generating local facts from a model.

## Acceptance criteria

- [ ] A page exists for each community, reachable from navigation or `/pine-and-strawberry`
- [ ] Each answers "what is it" in the first 100 words
- [ ] Each links to the property search with the right area pre-selected, if the IDX supports it
- [ ] No unverified local facts (dues, prices, utility providers, restrictions)
- [ ] Both are added to the sitemap and carry canonical + `Place` schema
