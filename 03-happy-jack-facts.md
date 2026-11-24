# 03 — One plain facts block: zip code, population, elevation, distance

**Type:** content + schema · **Effort:** S · **When:** after launch
**Where:** `/` (a compact block) and/or `/happy-jack-lifestyle`

---

## The query

Autocomplete for the single seed **`happy jack az`** returns, in the top ten:

```
happy jack az weather
happy jack az homes for sale
happy jack az elevation
happy jack az weather forecast
happy jack az zip code
happy jack az map
happy jack az directions
happy jack az real estate
happy jack az population
```

Three of the ten are pure fact lookups. Repeat the exercise on `happy jack arizona` and the same
three appear again (`elevation`, `population`, plus `land for sale`). This is what the top of the
funnel looks like for a small mountain community: people hear the name — at a campground, from
family, from a listing — and their first search is *where is this and how big is it*.

## What the site says today

- "zip code" — **0 of 14** pages
- "population" — **0 of 14** pages
- "elevation" — present on `/happy-jack-lifestyle` and `/winter-in-happy-jack`, but as prose, not as an answer
- `/winter-in-happy-jack` already answers the weather version well (42 inches average, 94 in 2022-23)

So the site answers the weather question and skips the three next to it.

## What to add

A labelled block — definition-list or small table — near the top of `/happy-jack-lifestyle`, and a
short version on the homepage. Six lines, plainly written, each one a sentence a person could
read out loud:

> **Happy Jack, Arizona at a glance**
>
> - **ZIP code** 86024
> - **County** Coconino
> - **Elevation** — ft above sea level
> - **Population** — see note below
> - **From Phoenix** — miles, about — hours
> - **From Payson** — miles

### On the population figure

Happy Jack is an unincorporated community, not a city, so it does not have its own published
census count the way a town does. **Do not publish a number that cannot be cited.** The honest and
more useful answer is one sentence saying exactly that, plus the census geography the reader
should look at instead. A page that says "no separate census count exists, here is why, and here
is the county figure that covers it" is more credible than one quoting a number with no source —
and it is the version an AI assistant will quote.

### On distance

`/posts/how-far-is-happy-jack` already exists and answers this. Do not duplicate it: put the two
numbers in the facts block and link the post for the drive detail.

## Why bother with non-commercial queries

Nobody searching "happy jack az zip code" is buying this week. But they are the largest group of
people who will ever become aware of the community, they arrive with no agent yet, and the page
that answers the question in one line is the page whose phone number they remember eighteen
months later. On a site judged by phone calls, that is the point.

It is also the cheapest structured-data win available: a `FAQPage` block here is the shape that
direct-answer and AI surfaces quote.

## Acceptance criteria

- [ ] The words "zip code", "population" and "elevation" each appear in visible text on one page
- [ ] The zip code 86024 appears
- [ ] Any population figure is accompanied by a named source and a year — or is not given at all
- [ ] Distance figures link to the existing `/posts/how-far-is-happy-jack` rather than restating it
- [ ] The block is marked up as `FAQPage` (see `06-schema-upgrades.md`)
- [ ] Numbers are consistent with anywhere else on the site that states them
