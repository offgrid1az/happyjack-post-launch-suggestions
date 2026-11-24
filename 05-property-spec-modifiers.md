# 05 — Property-spec modifiers: tiny homes, mobile/manufactured, well depth

**Type:** content · **Effort:** S–M · **When:** after launch
**Where:** `/homes-for-sale`, `/cabins-for-sale`, `/land`, and the existing guides

---

## The queries

Autocomplete, 24 November 2026:

- `pine az tiny homes for sale` — appears in the top ten for `pine az homes for sale`
- `mobile homes for sale in strawberry az` — appears in the top ten for `strawberry az homes for sale`
- `wellhouse happy jack az`, `clint's well resort happy jack az` — the only two completions for
  `happy jack az well` (both are a business name, not a property query — but they show the word
  "well" is how people orient themselves here)

## What the site says today

- "tiny home" — **0 of 14** pages
- "mobile home" / "manufactured" — not present as a buying angle
- "square footage" — **0 of 14**
- "well depth" — **0 of 14**, though `/guides` includes a guide on "Wells, depth and hauling water"

So the site has the *knowledge* (there is a well-depth guide) and not the *phrasing* that people
type, and the guides are not surfaced from the pages where a buyer would be looking.

## What to add

### (a) Tiny homes and manufactured/mobile homes are a real segment here

People buy land on the Rim specifically to put something small on it, and manufactured homes are
common in several of these communities. The site should say, per area or per subdivision where it
is known:

- whether a tiny home / RV / manufactured home is permitted (and by whom — county vs covenants)
- minimum dwelling size requirements, where covenants set them
- whether the parcel already has power, water and septic, since that is what usually decides it

**This is the single most useful thing an agent can answer in this market**, and no portal can.
Whether it becomes a section on `/land` or a page at `/land/tiny-homes` or `/land/building-on-a-lot`
is the build team's call — the phrase just has to appear.

### (b) Surface the well answer where the buyer is

`/guides` links to a well-depth guide, but `/land-for-sale` and `/homes-for-sale` do not. A single
sentence with a link is enough:

> Wells on the Rim typically run deep and the report matters more than the depth — see
> [what a well report tells you] before you make an offer.

Same for square footage: the site never says what a typical Happy Jack cabin actually measures,
which is the first thing a buyer from the Valley wants to know, because it is not what they expect.

## Numbers rule

Well depths, minimum square footages and permit thresholds are all facts that vary by subdivision
and change. **Use only figures already verified in the existing guides**, cite the subdivision, and
do not generalise a single example into "typically". If a figure is not in hand, describe the
question and link the guide — that still satisfies the query.

## Acceptance criteria

- [ ] "tiny home" appears in visible body text on at least one land page
- [ ] "manufactured" and "mobile home" appear where the build can state the rules accurately
- [ ] "square footage" or "square feet" appears on `/homes-for-sale` or `/cabins-for-sale`
- [ ] "well depth" appears on `/land-for-sale` with a link to the existing well guide
- [ ] Every numeric threshold names the subdivision or county rule it comes from
- [ ] No figure is stated that is not already verified in the site's own guides
