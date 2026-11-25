# 00 — Launch gate

**Type:** technical · **When:** at cutover · **Do not skip any of these**

The build is correct and deliberately non-indexable while it is a staging copy. Six things have
to change at cutover, or the site goes live and Google never sees it.

Every claim below was checked against `https://happyjack-site.replit.app` on **24 November 2026**
unless marked otherwise.

---

## 1. Remove the site-wide `noindex`

**Today:** every page we fetched (14 checked, and all 25 in the 24 Nov crawl) serves

```html
<meta name="robots" content="noindex, nofollow, noarchive">
```

`noarchive` is harmless. `noindex` and `nofollow` are not — with them in place nothing ranks and
internal links are not followed. Remove the tag entirely rather than switching it to `index,follow`;
an absent robots meta is the normal state for a page that should rank.

**Check:** `curl -s https://happyjackrealestate.com/ | grep -i 'name="robots"'` returns nothing.

## 2. Replace `robots.txt`

**Today:**

```
# STAGING COPY — do not index, do not follow.
User-agent: *
Disallow: /
```

**At cutover:**

```
User-agent: *
Allow: /

Sitemap: https://www.happyjackrealestate.com/sitemap.xml
```

Keep a `Disallow:` line only for paths that must never be indexed (search result URLs with
parameters, if any). Do not block CSS or JS — Google renders the page and needs them.

## 3. Ship a sitemap

**Today:** `/sitemap.xml` returns **404** (21,230-byte error page).

**Correction (25 Nov):** the build does generate a sitemap — 58 URLs, all on `www`, no trailing slashes, all
`<lastmod>2026-11-20`. `stage_noindex.py` is what removes it from the deployed staging copy, along with
`llms.txt`. So this item is "deploy the sitemap you already have", not "write one". See
[`PRE-CUTOVER-CONCERNS.md`](PRE-CUTOVER-CONCERNS.md) C2 for the check.

Reference it from `robots.txt` as above and submit it in Search Console (item 5).

**Check:** `/sitemap.xml` returns 200, `Content-Type: application/xml`, and the URL count matches
the number of deployed pages.

## 4. Add a self-referencing canonical to every page

**Today:** zero canonical tags anywhere on the site.

Every page should carry:

```html
<link rel="canonical" href="https://www.happyjackrealestate.com<path>">
```

Use the `www` host and https, matching what the site actually serves, and keep trailing-slash
convention consistent with the deployed paths. This is what stops the `replit.app` hostname from
competing with the real domain for the same content — without it, both can be indexed separately
once `noindex` comes off.

**Check:** every page has exactly one canonical; it is absolute; it resolves 200.

## 5. Verify the domain in Google Search Console — before the DNS change

The domain currently has **no TXT records at all** and no HTML verification file on either host
(both `google-site-verification.html` paths 404). So verification has to be created, not carried
over.

- Add a **Domain** property (not a URL-prefix property) for `happyjackrealestate.com` in Search
  Console and use the **DNS TXT** method.
- Add that TXT record now, while DNS is still at the current provider. It survives a host
  migration; an HTML file does not.
- Replit also requires a `replit-verify=<token>` **TXT record that must stay permanently** for
  certificate renewal. Two TXT records can coexist — do not delete either one later.

Without this there is no free source of the exact queries people used to reach the site, and the
whole point of items 1–7 is unmeasurable.

## 6. Turn off the host's injected scripts and badge

**Today:** `i.replit.com/script.js` and `replit-cdn.com/feedback-widget/widget.global.js` are
injected into every served page (they are not in the build). They cost bytes, they run on a
client-facing page, and the feedback widget is a staging tool.

Switch both off in the deployment settings, and remove the "Made with Replit" badge, before a
client domain points at the host.

---

## Two things to confirm with Replit before cutover, not after

**(a) Apex redirect.** Replit's static deployment config supports `[[deployment.rewrites]]` but
**not true 301s**, and the www/apex redirect has to be handled at the DNS/registrar layer. Decide
now whether the canonical host is `www.happyjackrealestate.com` (which is what the build already
uses in `og:url` and JSON-LD `@id`) and make the apex 301 to it.

**(b) DNS shape.** Replit issues an **A record for the apex** plus the verification TXT. Current
DNS is at the incumbent registrar/Wix nameservers with a `www` CNAME. Moving nameservers to a
provider that supports apex flattening (Cloudflare, already in front of this domain) is usually
cleaner than trying to express it in a DNS panel that has no ALIAS/ANAME support. Whichever route
is chosen, the redirect map for old URLs has to exist somewhere that can issue real 301s — a
rewrite keeps the old URL in the address bar, which is not what a moved page needs.
