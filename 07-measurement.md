# 07 — Measurement: baseline it, then judge it by phone calls

**Type:** process · **Effort:** S · **When:** launch week

---

## The problem this solves

Every item in this repo is an assumption about what people search. Once the site is live and
verified in Search Console, that stops being an assumption — Search Console reports the exact
queries that produced impressions on *this* site, free, at the query level, without sampling. No
keyword tool can do that; it only sees the whole web, and it cannot resolve communities this small.

So the first three months are a measurement period, not a ranking campaign.

## Do this in the first week

1. **Search Console — Domain property**, verified by DNS TXT (see `00-launch-gate.md` item 5).
   Submit `sitemap.xml`. Confirm the coverage report shows pages as *Indexed*, not *Excluded —
   noindex*. If pages sit in "Excluded", the launch gate is not actually finished.
2. **Record a baseline on day 7.** One screenshot or export: total impressions, total clicks,
   queries with ≥1 impression. It will be small and that is fine — it is the number everything
   later is compared against.
3. **Note the query set that appears.** Within a few weeks this file will say which of the terms in
   items 01–05 people actually arrive on. That is the ranking of what to build next, and it costs
   nothing.
4. **Bing Webmaster Tools** as a second free source — its keyword report is sometimes less
   suppressed than Google's for small sites.

## What not to spend money on

- **No keyword-volume subscription.** Google Trends cannot resolve Happy Jack, Pine or Strawberry
  as geographies, and every paid tool's floor is around 10 searches/month — above the resolution
  of this market. A number that says "10" for both a phrase nobody types and a phrase that
  converts is worse than no number.
- **No ranking tracker** until there is something to track. Search Console impressions by query
  answer the same question for free.
- **A SERP API is optional**, and only if someone wants weekly position checks on a fixed list of
  phrases (SerpApi or DataForSEO, roughly $50–100/month — confirm current pricing before buying).
  Nothing in this repo depends on it.

## How to judge success

The measure is **phone calls from people who found the site**, not positions.

- Count inbound calls that start with "I found you online" or "I saw your site" — a line in the
  existing call log is enough.
- Count form/IDX inquiries that originate from the site, per Search Console's landing-page report.
- A page that gets 40 impressions a month and produces two calls is doing its job. A page that
  ranks third for a phrase with no buyers is not.

Report the two numbers monthly: calls attributed to the site, and the query phrases that produced
the most sessions. Everything else is noise at this scale.
