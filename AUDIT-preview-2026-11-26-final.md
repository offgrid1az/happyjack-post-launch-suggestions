AGREED — 1144ffe1c1eb6b4a1fb0b1be88d86e3bfcdce532

I fetched the annotated tag, verified the commit, ran the suite and audited the republished address. **G1 through G4 are satisfied on my side.** Below is everything I measured, so you can check my numbers against your own rather than trust them.

**Source.** `launch-ready-20261126-0221` peels to `1144ffe1c1eb6b4a1fb0b1be88d86e3bfcdce532`, `1144ffe "Fix staging legacy redirects for frozen launch review"`. The correction is where you said it was:

```python
# Temporary hosts must exercise the same approved routing before cutover.
# Their indexing restrictions remain separate from redirect selection.
redirects = legacy_redirects()
```

`hosting/redirects.json` holds 10 rules plus the safety net.

**Tests, run by me on that commit:**

```
Ran 131 tests in 21.453s
OK (skipped=13)
```

131 = **118 passed + 13 skipped** — your figure exactly.

**Host-profile probe on that source** (application-level, both Host headers): staging and www now return identical status and destination for all 12 paths I probe — the six formerly-404ing rules, `/home`, `/blog-feed.xml`, `/post/*`, plus `/map-search` 302, `/moqui-drive` 404, `/_redirects` 404. The profile divergence is gone.

**External audit of the republished address `https://happyjack-site.replit.app/`:**

- **13/13 redirect checks pass** — 11 rules and the `/post/*` safety net, correct destinations, and query strings preserved (`/copy-of-mortgage-calculator?utm_source=old-wix&x=1` → `/selling-strategy?utm_source=old-wix&x=1`; same for `/pages-sitemap.xml`).
- **57/57 pages return 200**, none thinner than 3 KB, **57/57 carrying `noindex, nofollow, noarchive`**.
- `/map-search` → **302** to `https://happyjackrealestate.idxbroker.com/idx/map/mapsearch`. Corrected criterion confirmed and adopted; my earlier 404 expectation is withdrawn on the record.
- `/moqui-drive`, `/moqui-drive/`, `/_redirects` → 404. `/financing` → 200 with `noindex, nofollow, noarchive`. `/robots.txt` → 200 `Disallow: /`. `/sitemap.xml` and `/llms.txt` → 404, staging protection intact.
- CSP present; 0 Wix CDN references; 4 IDX map-search links on the homepage.

One factual note, since it affects how we both talk about builds: the `deployment-id` meta value on the preview is **unchanged** from before the republish (`b32a7d74-…`) while behaviour changed from 404 to 301. That field is therefore not a build identity, and I will not cite it as one. **The behavioural evidence — 13/13 — is what I certify, and I am certifying acceptance of the temporary deployment, not a SHA-to-preview mapping I cannot see from outside.** If you can expose the deployed SHA (a header or a `/__build` route) I will verify the mapping too.

## The agreement, stated plainly

**We agree the build is launch-ready.** Frozen source `1144ffe1`, tested, deployed, and externally verified: correct routing, correct map handling, correct staging protection, no Wix residue, owner's content decisions in place. Neither of us is asserting a DNS change.

**What is still missing before a domain switch — unchanged from your point 4, and I adopt it as written:**

1. The **verified Replit destination record set** as exact add/delete JSON with TTLs, conflict removal and rollback. Not supplied; I do not hold it; it will not be invented. The Wix backup is not a substitute.
2. The **owner's explicit approval to switch**. Passing G1–G4 is deployment acceptance, not authorization.
3. **G6**, ordinary link-accessibility checking on the two county URLs — which remain **unverified**: Cloudflare returns a "Just a moment…" challenge to curl and to a headless browser given 25 seconds. That is not a defect in the site and it does not reopen snow or septic work; it is one human click each.

**My status line, for the record: GO on the build. NO-GO on DNS until the record set and the owner's approval exist.** When they do, I will post the cutover checklist against the exact records, and I will not execute the change myself.

Sequence accepted in full: temporary republish → external checks (done, above) → verified record/rollback plan → explicit owner approval → cutover → post-cutover verification of canonical, `www`/apex routing, `sitemap.xml` 200 and certificate within the hour.

This is as far as two parties can get by agreement alone: the remaining two items are not things either of us can produce.
