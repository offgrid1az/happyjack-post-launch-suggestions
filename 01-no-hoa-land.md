# 01 — Answer "land for sale … no HOA"

**Type:** content · **Effort:** M · **Priority:** highest in this repo · **When:** after launch
**Where:** `/land-for-sale` (primary), `/land` (secondary), one new page if you prefer

---

## The query

Google Autocomplete, 24 November 2026, seed `happy jack az land for sale`:

```
happy jack az land for sale
happy jack az lots for sale
land for sale happy jack az no hoa
land for sale happy jack az no hoa by owner
rural land for sale happy jack az no hoa by owner
```

Seed `happy jack no hoa` returns **exactly two** completions, and both are that phrase:

```
land for sale happy jack az no hoa by owner
land for sale happy jack az no hoa
```

This is the most specific commercial phrase in the market. Someone typing it has decided the
area, decided the property type and named the one constraint that decides the purchase. There is
no broader query it rolls up into. It is worth a page of its own.

## What the site says today

**Zero occurrences** of "no hoa" on any of the 14 pages checked (24 Nov 2026), including
`/land-for-sale`, `/homes-for-sale` and `/land`. The word "HOA" appears on `/blog` only, as prose.

The site does hold the raw material: `/land` already summarises lot sizes and zoning for seven
Happy Jack subdivisions "read from Coconino County's own parcel records — plus what only the CC&Rs
answer". That is the knowledge this query needs. It just never says it in the words people use.

## What to add

An H2 on `/land-for-sale` — or a dedicated page at `/land-for-sale/no-hoa`, whichever fits the
site's URL shape better — answering the question directly, in the first sentence, with the
subdivision-level detail underneath.

**Suggested opening (adapt, do not paste blind):**

> **Land for sale in Happy Jack with no HOA**
>
> Whether a Happy Jack parcel has a homeowners association depends on the subdivision, not on the
> town. Of the seven subdivisions we track, [n] have recorded covenants and [n] have none. Here is
> which is which, and what the recorded covenants actually restrict.

Then a table: subdivision · HOA/dues yes/no · recorded CC&Rs yes/no · architectural committee
yes/no · minimum dwelling size if specified · short-term rental restriction if specified.

Close with what to check before buying — because the honest answer in this market is usually *"no
association, but yes, recorded covenants"*, and saying that plainly is what builds the call.

## Accuracy rule — this one matters

**Do not print a blanket "no HOA" claim.** Arizona subdivisions frequently have recorded CC&Rs
with no dues-paying association; some have an architectural committee that must approve plans.
A page that promises "no HOA" and then has a buyer discover a covenant is a complaint, not a lead.

State it per subdivision, sourced to "Coconino County recorded documents", and let the page be
the one that explains the distinction — that is more useful than a clean answer, and it is the
reason someone calls an agent instead of a portal.

## Acceptance criteria

- [ ] The literal phrase "no HOA" appears in an H2 or the page title of the target page
- [ ] The answer appears in the first 100 words, not after three paragraphs of scene-setting
- [ ] Every subdivision claim is tied to a source already used on `/land` (county records)
- [ ] No subdivision is described as HOA-free where recorded CC&Rs exist
- [ ] The page has a visible phone number within the first screen
- [ ] Internal links from `/land` and `/buying-land-in-happy-jack` to the new section
