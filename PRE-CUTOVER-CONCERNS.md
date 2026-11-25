# Before cutover — what I found checking the backup against the live site

**Date:** 25 November 2026 (MST) · **From:** the owner's side (Hermes, bot 1) · **For:** the Replit build team
**Check performed:** `offgrid1az/happyjack-site`, branch `backup/current-20261125-154950` (commit `9f378709`,
pushed 25 Nov 08:50 MST) against the running deployment at `https://happyjack-site.replit.app`

This is not a design review. It answers one question — *can we switch DNS without the site breaking?* — and
then lists what still has to be true when we do. Nothing in this report needs a reply unless we disagree;
items are written so they can be worked straight through.

---

## 1. Bottom line

**The backup is the live site.** Not "close to" it — measurably the same bytes.

| Check | Result |
|---|---|
| HTML pages compared | **62/62 identical** after removing the two scripts Replit injects at serve time |
| Assets (CSS, JS, images) | **29/29 identical** |
| `robots.txt`, `humans.txt` | identical |
| The site's own 404 page | identical to the deployed error body |
| Internal links/assets referenced by pages | 66 distinct; **0 point at a file missing from the backup** |
| The backup's own `verify.py` | **exit 0 — 94 checks pass**, "ALL CHECKS PASSED" |
| Rebuilding the backup from its own source | 114 files rewritten, **0 differences beyond a date stamp** |

Method: shallow-cloned the backup branch, walked every file, fetched the same path from
`happyjack-site.replit.app`, stripped the two injected `<script>` blocks, and compared bytes. The 404 page is
the one exception — it is served as the error body for unknown paths, so it is compared against
`build/staging-noindex/404.html`. The rebuild test means the backup is not just a snapshot of output: its
source reproduces that output.

**Conclusion: nothing is missing from the backup and nothing is stale in it. The risk in this cutover is not
the content — it is the deployment configuration, covered below.**

---

## 2. Concerns, in the order I would fix them

### C1 — The deployment serves the *staging* tree. Removing `noindex` is a config change, not a find-and-replace.

`deploymentTarget = "autoscale"` and the build command is:

```
test -s build/staging-noindex/index.html && python3 public_root.py build/staging-noindex && python3 -m unittest test_chat.py
```

So the running site is `build/staging-noindex`, which by construction has `noindex, nofollow, noarchive` on
every page, no canonical tags anywhere, and a `robots.txt` that disallows everything.

`build/happyjack-mirror-hosted` is the tree that is meant to go live: **62/62 pages carry a canonical and
`index, follow, max-image-preview:large`**, plus it ships `sitemap.xml`, `llms.txt`, `_redirects` and
`.htaccess`.

**What to do:** cut the deployment over to `build/happyjack-mirror-hosted` rather than editing the staging
tree. One change, and items 1–4 of `00-launch-gate.md` are satisfied by content that has already been
reviewed. **Check afterwards:** every page has exactly one absolute canonical on `www` and no `noindex`.

> **Correction, 25 Nov, measured — the build line alone does nothing.** `server.py:57` hard-codes
> `public = Path(directory or ROOT / "build/staging-noindex")` and `server.py:58` passes the profile as a
> literal: `violations(str(public), "staging")`. The server, not the build command, chooses the folder, so a
> deployment whose build line points at the hosted tree would still serve the staging tree. Two files must
> change, three strings in total: the build line in `.replit`, line 57 and line 58 of `server.py`.
>
> Also wrong in the earlier version of this note: `public_root.py` does **not** infer the profile. `violations(tree,
> profile="hosted")` defaults to hosted and `server.py` overrides it with `"staging"` explicitly. Only the
> `public_root.py` command-line entry point infers from the directory name. Passing the hosted tree with the
> literal `"staging"` profile is therefore accepted — it is the staging allowlist, which is the smaller one,
> so the mistake fails silently instead of loudly.
>
> Measured: applying all three string changes to a clone and booting the app gives `/robots.txt` `Allow: /`,
> `/sitemap.xml` 200 with 58 `<loc>`, `/llms.txt` 200, homepage canonical + `index, follow`. The repo's own
> suite then reports **5 failures**, all of them tests that assert the staging behaviour being removed.

### C2 — The sitemap already exists. Staging drops it on purpose, so `/sitemap.xml` 404s today.

`stage_noindex.py` deliberately removes four root files from the staging copy:
`sitemap.xml`, `llms.txt`, `_redirects`, `.htaccess` (see `public_root.STAGING_ROOT_FILES`).

The real sitemap is fine and current:

- **58 URLs**, every one `https://www.happyjackrealestate.com`, **no trailing slashes**
- `<lastmod>2026-11-20</lastmod>` on all 58
- Excludes `/404.html`, `/404`, `/thank-you` and `/financing` — correct, all four are either an error page or
  noindex
- Cross-checked against the deployed tree: **every sitemap URL exists as a staged page; no staged page that
  should be indexed is missing from it**

`llms.txt` (27.6 KB) is in the same position — present in the repo, absent from the deployment. **Check
after cutover:** `/sitemap.xml` and `/llms.txt` both return 200 and `/robots.txt` references the sitemap.

### C3 — Three old Wix URLs 404 today. Five already answer 301, `/moqui-drive` 404s on purpose. *(corrected 25 Nov, measured)*

`hosting/redirects.json` holds **9 rules** (8 exact + a `/post/*` net), and `make_redirects.py` generates
`_redirects` from it. That file is Netlify syntax, and Replit does not read it.

**What I got wrong in the first version of this report, and what is actually true.** I wrote that "a rewrite
is not a 301". In this app it usually is: `server.py` canonicalises any request that resolves to an existing
`index.html`, and answers 301. So every rewrite in `.replit` that lands on an `index.html` already returns
301. Measured on the live host today:

| Old URL | Live answer |
|---|---|
| `/home`, `/home/` | 301 → `/` |
| `/post/owning-vs-renting-the-cabin-lifestyle-in-happy-jack-arizona` | 301 → `/posts/owning-vs-renting-a-cabin-in-happy-jack` |
| `/post/escape-to-happy-jack-arizona-…` | 301 → same article |
| `/post/*` (anything else) | 301 → `/blog` |
| `/blog-feed.xml` | 301 → `/blog` |
| `/moqui-drive` | **404** |
| `/pages-sitemap.xml` | **404** |
| `/blog-posts-sitemap.xml` | **404** |
| `/blog-categories-sitemap.xml` | **404** |

The four 404s are the three Wix sitemap children and one old listing page — exactly the addresses Search
Console has already seen from the Wix years.

**What to do:** read the eight actionable rules from `hosting/redirects.json` in `server.py` and answer 301
before file resolution, keeping the query string and the deployment's security headers. A drop-in patch is in
`CUTOVER-INSTRUCTIONS-2026-11-25.md` §1.3 (`patches/happyjack-cutover-server.patch`). It also adds the piece
Replit does not provide at all: **bare domain → `www` 301**.

**`/moqui-drive` is deliberately excluded, and this report previously got that wrong.** It maps to `/` in
`hosting/redirects.json`, but `AUDIT-DECISIONS-2026-11-17.md` F5 records the opposite decision — the sold
listing was deleted 2026-10-29 and *"No generic homepage 301 was invented; any launch redirect should follow
the approved legacy map."* The live host follows the decision, not the artifact: 404. The patch holds that
one URL at 404 and carries the reason in a comment, so the contradiction cannot be closed silently by a
deploy. Reconciling `redirects.json`, `test_redirects.py` and the decision is an owner call, in one commit.

**Checked after the patch:** all nine rules answer 301 with query strings preserved, and
`Host: happyjackrealestate.com` on `/blog` answers `301 https://www.happyjackrealestate.com/blog`, while
`www` and the `replit.app` host still answer 200.

### C4 — Two scripts are injected at serve time, and one of them is in the CSP on purpose.

Every served page carries, and the repo does not:

```html
<script src="https://replit-cdn.com/feedback-widget/widget.global.js" ...></script>
<script src="https://i.replit.com/script.js" async data-website-id="d0d445ca-..."></script>
```

They account for the only byte-difference on every page (homepage 45,512 B served vs 45,176 B in the repo).
`script-src` in `.replit` explicitly allows `https://i.replit.com`. The feedback widget is a staging tool and
the "Made with Replit" badge should not appear on a client's domain.

**What to do:** switch both off in deployment settings and drop `https://i.replit.com` from `script-src` when
nothing else needs it. **Check afterwards:** a fetched page is byte-identical to its file in
`build/happyjack-mirror-hosted`.

### C5 — `main` is not the release source. Use the backup commit.

The backup branch is 67 commits **ahead** of `main` and 1 **behind** it. The one commit it lacks is a *revert*
from 16 November (`15b11391`, "Revert 'content: fix the six live defects the 15 Nov audits found'"), which
still flips `assets/site.css (+6/−14)`, `meet-the-team`, `happy-jack-lifestyle`, `selling-strategy` and
`posts/septic-systems-happy-jack` back to older copy.

**What to do:** deploy and archive `9f378709`. Tag it. If anyone later builds from `main`, the site silently
regains content that was deliberately removed.

### C6 — A rebuild re-dates all 58 sitemap entries.

`lastmod` is generated, not stored. Rebuilding today stamps **today** on all 58 URLs, which tells Google that
58 pages changed when one did not. Harmless to ranking, but it makes the one signal we would use to spot a
real content change useless for months.

**What to do:** decide now — either ship the committed sitemap as-is (all 2026-11-20), or accept the re-date
knowing the first `lastmod` sweep is noise. I would ship the committed one and let the next real content
change move it. Do not hand-edit `lastmod` afterwards.

### C7 — `/financing` is deployed, noindex, and has zero inbound links.

It is the only page in the hosted tree still carrying `noindex`, and no page on the site links to it. It is
reachable only by typing the URL. That is either intentional (held content) or a page that was meant to be
linked and was not — worth one sentence of confirmation before it becomes invisible on launch day.

### C8 — `happy-jack.replit.app` is not this site.

That hostname returns Replit's "This app isn't live yet" page (8,417 B). The deployment we verified is
`happyjack-site.replit.app`. If any document, email or card mentions the other hostname, it is dead — fix it
before it goes out under the new domain.

### C9 — The verification records have to exist *before* DNS moves, and one must never be deleted.

Already item 5 of `00-launch-gate.md`, restated here because it is the only irreversible ordering constraint:
add the Google Search Console **DNS TXT** now, while DNS is still with the current provider, and keep
Replit's `replit-verify` TXT permanently — it is used for certificate renewal, and deleting it later breaks
the certificate, not just the verification.

---

## 3. What I could not verify from outside

- **Deployment settings** — whether the injected scripts and badge can be switched off per-deployment, and
  what the custom-domain configuration will be. Visible only in the Replit dashboard.
- **Certificate and apex/www behaviour after cutover** — depends on where DNS terminates. Replit's static
  rewrites cannot issue a 301 for the apex, so this has to be decided at the DNS layer.
- **Whether Replit's own backup captures source or only output** — the branch contains both, so this is not
  blocking, but it decides what a future restore would actually restore.
- **Anything behind a login** — the chat server (`server:app`, gunicorn) and its Postgres dependency build
  and pass `test_chat.py` in CI; I did not exercise it against the live host beyond page fetches.

---

## 4. Cutover checklist, in order

1. Tag the backup commit `9f378709` as the release.
2. Add the GSC DNS TXT · confirm the `replit-verify` TXT is present and stays. **(before DNS)**
3. Point the deployment at `build/happyjack-mirror-hosted` — not the staging tree.
4. Disable the injected scripts + badge; drop `i.replit.com` from `script-src`.
5. Wire the nine legacy URLs into the server as real 301s (C3) and configure apex → `www` there too —
   Replit's Domains tool issues A records only and does no redirecting of its own.
6. Publish and attach the custom domain; wait for the certificate.
7. Verify, from outside:
   - `/` returns 200 with `index, follow` and **no** `noindex`, exactly one canonical on `https://www.happyjackrealestate.com`
   - `/robots.txt` returns 200 and references the sitemap; `/sitemap.xml` and `/llms.txt` return 200
   - a legacy Wix URL answers **301**, not 200 and not a rewrite
   - a fetched page is byte-identical to its file in `build/happyjack-mirror-hosted` (proves no injection)
   - `apex → www` redirects once, not twice
8. Submit the sitemap in Search Console the same day.

---

## 5. One correction to what we published on 24 November

`00-launch-gate.md` item 3 says *"The build has no sitemap at all."* **That is wrong, and I have corrected it
in this repo.** The build generates a complete 58-URL sitemap; `stage_noindex.py` is what removes it from the
staging copy. The gate item still stands — `/sitemap.xml` is 404 today — but the work is "deploy the sitemap
you already have", not "write one".

---

*Evidence for every figure above is reproducible from the commands in this report against the backup branch;
the comparison scripts and outputs are held on the owner's machine. If a number here does not reconcile with
what you see, say so — I would rather be corrected than make the cutover on a claim I cannot re-run.*
