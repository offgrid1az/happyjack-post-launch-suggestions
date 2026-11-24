# 06 — Schema: one real change, and a correction to an earlier claim

**Type:** schema · **Effort:** S (much smaller than originally reported) · **When:** after launch
**Where:** the shared `@graph` emitted on every page

---

## Correction first

An earlier draft of this research said the build was missing `BreadcrumbList`, `ItemList` and
article markup. **That was wrong, and it came from reading only the first two JSON-LD blocks on
each page.** Re-checked on 24 November 2026 against the served HTML:

| Page | Types actually present |
|---|---|
| `/` | `Organization`, `WebSite`, `WebPage`, `FAQPage` (12 Q&A), `Place` ×5, `City` ×8, `ImageObject`, `PostalAddress`, `PropertyValue`, `EducationalOccupationalCredential` |
| `/land-for-sale` | the above plus `BreadcrumbList`, `ItemList` (9 `ListItem`), `WebPage` ×2 |
| `/posts/how-far-is-happy-jack` | the above plus `BlogPosting`, `Person`, `BreadcrumbList` |

The structured data on this build is **better than most agent sites** — breadcrumbs, item lists,
article typing, FAQ blocks, area served with contained `Place` nodes, licence credentials. No work
needed on any of those. Two small things remain.

---

## 1. Type the practice as `RealEstateAgent`, not `Organization`

**Today**, on every page:

```json
{
  "@type": "Organization",
  "@id": "https://www.happyjackrealestate.com/#practice",
  "name": "The Thresher Team",
  ...
}
```

`RealEstateAgent` is a subtype of `LocalBusiness` and is the type search engines and AI assistants
use to identify an agent as a *local business in a place* rather than a company name. Everything
else in the node is already correct — keep it all, change the type and add the two fields that
type expects:

```json
{
  "@type": "RealEstateAgent",
  "@id": "https://www.happyjackrealestate.com/#practice",
  "name": "The Thresher Team",
  "priceRange": "$$",
  "geo": { "@type": "GeoCoordinates", "latitude": ..., "longitude": ... },
  "openingHoursSpecification": [ ... ],
  ...
}
```

Use the brokerage office coordinates, not a residence — the address in the node is already the
Flagstaff office and must stay that way for advertising compliance.

**Check:** no page emits `"@type": "Organization"` for `#practice`; `RealEstateAgent` appears; the
address is unchanged.

## 2. Put Jackie in the graph on the homepage

Posts already carry a `Person` node. The homepage — the page that answers "who is this and why
should I call her" — does not. Add a `Person` with `@id` `…#jackie`, `worksFor` the practice, and
`sameAs` pointing at her public profiles (RateMyAgent, Google Business Profile, Zillow agent page,
Facebook). Wire it into the existing `#practice` node as `founder`/`employee` rather than creating
a second competing entity.

**Check:** one `Person` node on `/`, referenced by the practice node, `sameAs` resolves.

## Explicitly not recommended

- **No `AggregateRating` or `Review` markup built from the site's own testimonials.** Google's
  policy on self-serving reviews means it is ignored at best and a manual action at worst. The
  RateMyAgent profile is the correct place for that signal.
- **No `Offer` markup on listing pages.** The listings come from the IDX feed under someone else's
  terms; marking them up as offers the site is making is a claim the site cannot support.
- **No `LocalBusiness` for a second address.** One entity, one address.

## Acceptance criteria

- [ ] `RealEstateAgent` on every page, replacing `Organization` for `#practice`
- [ ] `geo` and `openingHoursSpecification` present and correct for the Flagstaff office
- [ ] `Person` for Jackie on `/`, linked to the practice, with `sameAs`
- [ ] `@id` values unchanged so existing internal references still resolve
- [ ] Validates in the Schema Markup Validator with zero errors
- [ ] No review, rating or offer markup added
