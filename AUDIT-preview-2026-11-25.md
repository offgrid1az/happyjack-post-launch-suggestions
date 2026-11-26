# Preview audit — `happyjack-site.replit.app`

Audited 25 Nov 2026 against tag `launch-ready-20261125-225205` (`5dd5b5eb`). Every line below is a
measurement taken from outside, not a restatement of the release notes.

## Verdict

- **Republishing the preview: GO.** Nothing found that damages the temporary host.
- **DNS cutover: NO-GO until one defect is fixed.** Six approved redirects return 404 on the running
  preview while four others work, which means the deployment is not serving the redirect set in the tag.

## What passes

| Check | Result |
|---|---|
| Pages served | 57/57 return 200, none thinner than 3 KB, 2,116,529 bytes total |
| Staging protection | 57/57 carry `X-Robots-Tag: noindex, nofollow, noarchive` **and** the same `meta robots` |
| `robots.txt` | 200, `Disallow: /` — correct for staging |
| `/sitemap.xml`, `/llms.txt` | 404 — correct for staging; these must return 200 only after cutover |
| `/moqui-drive`, `/moqui-drive/` | 404 — correct, the page is retired |
| `/_redirects` | 404 — hosting configuration is not exposed |
| `/financing` | 200 with `noindex, nofollow, noarchive`, absent from the sitemap (0 hits in the build's 57 `<loc>`) |
| Search by Map | The fourth nav option points at `https://happyjackrealestate.idxbroker.com/idx/map/mapsearch` on the homepage, in the footer, and in body copy — the owner's chosen variant |
| `/map-search` | Not served as a page; 302 to the IDX map search |
| Local map page | Gone: no `map-search/` directory in the build, nav entry rewritten to the IDX destination |
| Wix residue | 0 `wixstatic` / `parastorage` references in the homepage |
| CSP | Present on the homepage |

## Defect 1 — six approved redirects 404 (blocker for cutover)

The tag's `hosting/redirects.json` holds 10 rules plus the `/post/*` safety net. On the running preview:

| Source | Expected | Observed |
|---|---|---|
| `/copy-of-mortgage-calculator` | 301 `/selling-strategy` | **404** |
| `/about-sherrie-vrankovic` | 301 `/meet-the-team` | **404** |
| `/copy-of-my-selling-strategy` | 301 `/testimonials` | **404** |
| `/pages-sitemap.xml` | 301 `/sitemap.xml` | **404** |
| `/blog-posts-sitemap.xml` | 301 `/sitemap.xml` | **404** |
| `/blog-categories-sitemap.xml` | 301 `/sitemap.xml` | **404** |
| `/home` | 301 `/` | 301 ✓ |
| `/post/owning-vs-renting-…-arizona` | 301 `/posts/owning-vs-renting-a-cabin-in-happy-jack` | 301 ✓ |
| `/post/escape-to-happy-jack-…-bliss` | 301 (same target) | 301 ✓ |
| `/blog-feed.xml` | 301 `/blog` | 301 ✓ |
| `/post/some-random-wix-era-url` | 301 `/blog` (safety net) | 301 ✓ |

The redirect machinery is therefore **running correctly** — the safety net and the four older rules fire —
but the table it loaded is the pre-tag version with four rules. The six additions are in the tag and absent
from what is deployed.

Three of those six are redirects the **live Wix site serves today** (verified against `www` the same day:
all three 301), so shipping without them breaks URLs that currently work and are indexed. The other three
are Wix sitemap files Search Console reads.

**Fix:** confirm which commit the deployment was built from, and that the running process loaded
`hosting/redirects.json` with 10 rules + safety net. A rebuild from the tag should resolve it; if the
deployment is rebuilt and the six still 404, the fault is in how the deployment packaging carries
`hosting/`, and that is the thing to investigate.

## Defect 2 — the temporary host's protection is not the production configuration

Nothing to fix, but worth stating so it is not mistaken for a defect later: the preview intentionally
serves `Disallow: /`, no sitemap, no `llms.txt`, and `noindex` on every page. Every content assertion for
launch must therefore be made at application level before DNS, and at public level after DNS. Do not
"fix" the staging protection to make a check pass.

## Not tested

- `www` and apex routing, canonical tags against the production host, certificate behaviour, HSTS — none
  are observable before the records change.
- Browser-level map rendering: no embedded widget exists in this build, so the earlier MapTiler/CSP
  question is moot for launch. If the widget is ever remounted, the CSP and test repairs in comment
  `7176049308` become required again.
