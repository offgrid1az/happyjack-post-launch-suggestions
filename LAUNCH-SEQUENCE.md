# Launch sequence — the order I would do it in, and why

Written 25 Nov 2026, after reviewing the prepared candidate (`f50d618`, source tree `bd862b34`) and measuring
the current DNS myself. This is a recommendation for the record-only cutover, in the order that leaves the
smallest window where a visitor sees something wrong.

It is deliberately short. Everything already agreed — the candidate, the redirects, the held page — is treated
as settled and is not re-argued here.

---

## Step 0 — Lower the TTLs first, tonight

Measured against public resolvers on 25 Nov:

| Name | Type | Current TTL | Value |
|---|---|---|---|
| `happyjackrealestate.com` | A ×3 | **3600** | `185.230.63.107`, `185.230.63.171`, `185.230.63.186` |
| `www.happyjackrealestate.com` | CNAME | **3600** | `cdn3.wixdns.net` → `cfd.wixdns.net` → Cloudflare |

At 3600 seconds, a mistake is live for up to an hour and so is the rollback. **Drop these records to 300 seconds
and let the old hour age out before anything else happens** — ideally overnight. This costs nothing, changes no
routing, and is the difference between "back in five minutes" and "back within the hour".

Do this **before** the deploy, not on the day. A TTL lowered at the same time as an address change has not been
aged out and buys nothing.

## Step 1 — Deploy from the tagged commit, and say which commit

- Tag the reviewed revision and deploy from the tag, not from a branch head that can move.
- State the deployed commit SHA and the source tree SHA in this thread.
- The deployment must not include the review-backup files; they are evidence, not application code.

I will re-run the test suite against that exact SHA and post the output, so from that point the numbers here are
measurements rather than quotations.

## Step 2 — Verify the deployment before DNS changes

Checks that do not need the domain: `/robots.txt` `Allow: /`; `/sitemap.xml` with 58 `<loc>`; `/llms.txt` 200;
the 11 redirects returning 301 with the correct destination and the query string preserved; `/moqui-drive` and
`/moqui-drive/` returning 404; `/financing` returning `noindex, nofollow` and absent from the sitemap; the
temporary host still `noindex` with `X-Robots-Tag` so it cannot compete with `www`; `_redirects` and internal
documents returning 404.

Anything failing here is fixed before the domain is touched, at no risk to the live site.

## Step 3 — One short DNS window

- Provide the record plan as **JSON**: one entry per delete and per add, with type, host, value and TTL. I will
  verify it against resolvers record by record rather than reading prose back.
- Make the change in one sitting. A half-applied change — apex moved, `www` still on Wix — is the one state that
  serves the wrong site on the canonical host.
- Keep Wix's nameservers. Nothing in this step requires a migration, and a migration adds a failure surface for
  no benefit today.

## Step 4 — Verify from the outside, immediately

`www` returns 200 with `index, follow` and exactly one canonical on `https://www.happyjackrealestate.com`; apex
returns 301 to `www` with path and query intact, and 308 for non-GET; the certificate is valid on both names; the
redirect set behaves as in step 2 but through the real domain; `replit.app` stays `noindex`.

I will run this within minutes of the change and post the raw output.

## Step 5 — Watch, then Search Console

Twenty-four hours of watching before anything else changes. Search Console submission and indexing requests come
after the site is verified live, and are not a prerequisite for the move.

---

## Rollback

Restore the four records: three apex A records to the addresses in step 0, `www` CNAME to `cdn3.wixdns.net`.
With TTLs at 300 the site is back within five minutes.

Nothing in steps 1 and 2 needs undoing to roll back — the deployment can stay up untouched.

## The one thing that must not be lost

Keeping Wix's nameservers keeps the domain's address book inside the Wix account. That is fine for the cutover
and **fatal at subscription cancellation**: cancelling Wix takes the domain down with it, however healthy the
deployment is. Moving DNS to the registrar belongs to the retirement track, on its own date, before the Wix
subscription ends.

## What is not a gate

Wix data exports and Search Console. Neither affects resolution, and either one, if allowed to gate the move,
becomes a reason to wait another day. Items already in progress with a third party are waiting, not blocking, and
are tracked on their own schedule.
