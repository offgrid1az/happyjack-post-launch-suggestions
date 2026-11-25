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

> Note for `public_root.py`: it infers the profile from the directory name (`"staging" in basename`). Serve
> the hosted tree from a path whose name does not contain "staging", or pass the profile explicitly, or the
> stricter staging allowlist will reject a legitimate hosted root.

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

### C3 — The redirect map is in a format Replit does not serve.

`build/happyjack-mirror-hosted/_redirects` holds **12 rules** (generated from `redirects.json`), including the
old Wix-era `/home` and `/post/owning-vs-renting-the-cabin-lifestyle-in-happy-jack-arizona` URLs. That is
Netlify syntax.

`.replit` currently expresses only a subset as `[[deployment.rewrites]]`. **A rewrite is not a 301** — the
browser keeps the old URL, so the moved page is not consolidated and the old URL stays in Google. For a
domain that has been on Wix for years, the redirected legacy URLs are exactly the ones carrying history.

**What to do:** confirm which of the 12 `_redirects` rules are *permanent moves* and make sure they answer
**301** at an edge that can emit status codes (Cloudflare in front of the domain, or the registrar), not only
as internal rewrites. Regex-style nets (`/post/*`) are fine as rewrites; individual old URLs are not.

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
5. Decide the 301 layer for the 12 legacy URLs (C3) and configure apex → `www`.
6. Publish and attach the custom domain; wait for the certificate.
7. Verify, from outside:
   - `/` returns 200, **no** `robots` meta, exactly one canonical on `https://www.happyjackrealestate.com`
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
