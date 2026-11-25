# Moving happyjackrealestate.com from Wix to the Replit site

**Written 25 Nov 2026 by Bot 1. Written to be worked straight through, top to bottom.**
Every command can be pasted as-is. Every "check" has an expected answer, so you know you got it right.

---

## The whole job in one minute

The new site already exists, complete, at `happyjack-site.replit.app`. It has been checked page by
page against that address: 62 of 62 pages identical, plus its own internal test suite passing
(`ALL CHECKS PASSED`).

So this is a **house move where the furniture is already in the new house**. Two things are left:

1. **Stop the new site from saying "do not index me".** Right now it is a staging copy wearing a
   "no entry" sign on every page. That sign is deliberate — it stops Google indexing the test
   address. We remove the sign as part of the move.
2. **Change the signpost outside.** The name `happyjackrealestate.com` points at Wix. We point it
   at Replit instead. That is five small DNS records — nothing more.

**Nothing gets deleted.** The Wix site stays online until you deliberately switch it off, so any
mistake is undone by typing the old numbers back in (Part 7).

**Words used below, in plain English**

- **DNS records** — the address book of the internet. Five lines that say "this name goes to that
  computer". You edit them at Wix because Wix currently holds the address book.
- **A record** — "this name goes to this numeric address."
- **CNAME** — "this name is an alias of that name."
- **TXT record** — a note stuck to the name. Google and Replit use one note each to prove the
  domain is really yours. **TXT notes must never be deleted, even years later.** Deleting Replit's
  one breaks the security certificate renewal and the site eventually stops loading.
- **Propagation** — how long the rest of the internet takes to notice a change. Minutes, usually.

---

## Part 0 — Before you touch anything (5 minutes)

### 0.1 Copy today's settings onto paper

This is your undo button. These are the real, checked values as of 25 Nov 2026:

| Record | Host | Value today | Where it is edited |
|---|---|---|---|
| A | `@` (bare domain) | `185.230.63.107` | Wix → Domains → DNS |
| A | `@` | `185.230.63.171` | Wix → Domains → DNS |
| A | `@` | `185.230.63.186` | Wix → Domains → DNS |
| CNAME | `www` | `cdn3.wixdns.net` | Wix → Domains → DNS |

Also true, and worth knowing before you start:

- The domain is **registered at GoDaddy** (paid through May 2027) but its **address book is held by
  Wix** (`ns8.wixdns.net`, `ns9.wixdns.net`).
- There is **no email on this domain** — no MX record, no TXT records at all, and none of the usual
  subdomains (`mail.`, `ftp.`, `autodiscover.`, `webmail.`, `smtp.`, `imap.`) exist. Checked today.
  So there is nothing to break except the website itself. That is good news.
- Bare domain → `www` is today handled by Wix. Replit does **not** do that for you (see Part 4).

### 0.2 Have these logins ready

1. **Replit** — the project that publishes `happyjack-site.replit.app`.
2. **Wix** — the account that owns the domain, so you can reach Domains → DNS.
3. **GoDaddy** — only needed for the fallback in Part 4.4. Have it anyway.

### 0.3 Two facts you must not get wrong

- **The site to release is the backup branch, not `main`.** Branch
  `backup/current-20261125-154950` (tip `9f378709`) is what is live today and is 67 commits ahead of
  `main`, which also carries an old 16 Nov "Revert". Deploy the backup branch.
- **Do not change the address to the bare domain.** The bare domain must keep redirecting to `www`.
  Every page's canonical tag and every structured-data ID already says
  `https://www.happyjackrealestate.com`. Flipping to bare would invalidate 62 pages' worth of them.

---

## Part 1 — Make the new site indexable (in Replit's code)

**Why:** the deployment today serves `build/staging-noindex`, a copy where every page carries
`<meta name="robots" content="noindex, nofollow, noarchive">` and `robots.txt` says `Disallow: /`.
That is why the site is invisible to Google. The ready-to-serve copy is `build/happyjack-mirror-hosted`
— 62 pages, all with a `www` canonical and `index, follow`.

**Watch out:** changing only the `.replit` build line does nothing. The web server (`server.py`) has
the staging folder and the "staging" safety profile hard-coded, and it is the server that chooses the
folder. **Both files must change, or the deployed site stays invisible.**

### 1.1 Three exact string changes

In **`.replit`**, in the `[deployment]` build line, replace both occurrences of
`build/staging-noindex` with `build/happyjack-mirror-hosted`. The line then reads:

```
build = ["sh", "-c", "test -s build/happyjack-mirror-hosted/index.html && python3 public_root.py build/happyjack-mirror-hosted && python3 -m unittest test_chat.py"]
```

In **`server.py`**, line ~57:

```python
public = Path(directory or ROOT / "build/happyjack-mirror-hosted").resolve()
```

In **`server.py`**, line ~58, the safety profile must match the tree:

```python
if violations(str(public), "hosted"):
```

### 1.2 Check it, before you redeploy

Run these in the Replit shell from the project root:

```bash
python3 public_root.py build/happyjack-mirror-hosted      # expect: PASS ... 8 allowed entries / OK
python3 -m unittest discover -p 'test_*.py'
```

The first must pass. The second will report **4 failures** — and they are *expected*, because four
tests were written to protect the staging copy and now assert the very thing we removed. They are:

```
test_chat.ServerTests.test_public_paths_only_and_preserved_headers   (path='/')
test_delivery.DeliveryTests.test_gzip_negotiation_and_head_consistency
test_delivery.DeliveryTests.test_real_error_statuses_and_private_files
test_property_search_pages.PropertySearchPageTests.test_served_new_routes_and_staging_protection
```

Change those four assertions to expect `index, follow` on a served page. Do not "fix" them by putting
the staging folder back.

### 1.3 Three old Wix addresses will 404 unless you add the 301 map

Two patches ship together: `patches/happyjack-moqui-drive-deletion.patch` (removes the leftover rule and
makes the test enforce the deletion) and `patches/happyjack-cutover-server.patch` (the server switch).

Checked live today on `happyjack-site.replit.app`. Nine old addresses need to keep working:

| Old address | Today | Must be |
|---|---|---|
| `/home` | 301 → `/` | 301 ✅ already correct |
| `/post/owning-vs-renting-the-cabin-lifestyle-in-happy-jack-arizona` | 301 → the rebuilt article | ✅ |
| `/post/escape-to-happy-jack-arizona-…` | 301 → the rebuilt article | ✅ |
| `/post/*` (any other Wix blog link) | 301 → `/blog` | ✅ |
| `/blog-feed.xml` | 301 → `/blog` | ✅ |
| `/pages-sitemap.xml` | **404** | 301 → `/sitemap.xml` ❌ |
| `/blog-posts-sitemap.xml` | **404** | 301 → `/sitemap.xml` ❌ |
| `/blog-categories-sitemap.xml` | **404** | 301 → `/sitemap.xml` ❌ |
| `/moqui-drive` | **404** | **404 — leave it alone** |

The three Wix sitemap children are addresses Google has already seen; a 404 there costs a little
traffic and a Search Console error, a 301 costs nothing.

**`/moqui-drive` is deleted, and stays deleted.** The owner removed that page on 29 Oct 2026 — it was
one sold listing, and listings now come from the IDX feed. It returns 404 today and must still return
404 afterwards. Do not "helpfully" add a redirect for it.

It is worth knowing why it kept appearing in plans: `hosting/redirects.json` was written before a host
was chosen and still carried a 301 to `/` for it, which `make_redirects.py` propagated into
`_redirects`, `netlify.toml`, `vercel.json` and `.htaccess`, and which `test_redirects.py` enforced by
treating the dead URL as live. All of that is removed in `patches/happyjack-moqui-drive-deletion.patch`,
and the test now asserts a 404 instead — so the deletion is enforced, not merely described.

For the other eight, all the rules already exist in `hosting/redirects.json` and are used to generate
`_redirects`; they are simply not wired into the Python server. A drop-in patch is attached:
**`patches/happyjack-cutover-server.patch`**. It

- serves the hosted tree,
- answers the eight old URLs with real 301s read from `hosting/redirects.json` (query strings kept,
  security headers kept), and holds `/moqui-drive` at 404,
- sends the **bare domain to `www`** with a 301 — which is the piece Replit does not provide.

Verified locally today: all eight 301s answering correctly, `/moqui-drive` and `/moqui-drive/` both
404 with the designed error page, bare-domain redirect correct.

### 1.4 Tag what you actually shipped — do not call the backup "production"

The backup commit `9f378709` is a **pre-cutover baseline**, not the release. It still contains the
staging-serving setup, the dead old URLs and the `noindex` sign. Once the Part 1 changes are committed,
tag *that* commit as the approved production revision:

```bash
git status --short                       # expect: the changes you just made, nothing else
git commit -am "Launch: serve indexable tree, wire legacy 301s, apex to www"
git tag -a production-2026-11-25 -m "Approved production revision for the Wix cutover"
git push origin HEAD && git push origin production-2026-11-25
```

Leave `backup/current-20261125-154950` exactly where it is, unmodified — it is the evidence that what
was deployed before the change was what was verified. **Never deploy the unchanged backup and call that
the production switch.** Build and deploy from the tagged commit, from a branch off `9f378709` — do not
merge `main`, which is 67 commits behind and carries an old revert.

### 1.5 Optional: turn off Replit's own extras

Replit injects two `<script>` tags into every page (an analytics/feedback loader) and can show a
"Made with Replit" badge. Both are deployment settings, not repository settings. Look under the
project's **Publish / deployment settings** for the feedback widget and badge switches. I could not
confirm the exact switch names from outside the account, so if you cannot find them, ask Replit to
turn them off for the deployment. They do not block the move.

**Then publish the deployment from the backup branch, and prove Part 1 worked:**

```bash
curl -s https://happyjack-site.replit.app/robots.txt | head -4        # expect: User-agent: * / Allow: /
curl -s https://happyjack-site.replit.app/sitemap.xml | grep -c '<loc>'   # expect: 58
curl -s https://happyjack-site.replit.app/ | grep -o '<meta name="robots"[^>]*>'  # expect: index, follow, ...
curl -s -o /dev/null -w '%{http_code}\n' https://happyjack-site.replit.app/llms.txt   # expect: 200
curl -s -o /dev/null -w '%{http_code} %{redirect_url}\n' https://happyjack-site.replit.app/moqui-drive   # expect: 301 ...
```

Until all five answer as shown, stop. There is no point moving a name to a site that is still
telling Google to go away.

---

## Part 2 — Add the two permanent notes at Wix DNS (do this first)

**Neither of these is a prerequisite for moving the domain.** The move works without them. Do them
first only because there is exactly one convenient moment — while Wix still holds the address book —
and they become fiddly afterwards. If you are impatient, skip to Part 3 and come back.

These survive the move and must stay forever.

1. **Google Search Console note.** In Search Console → Add property → **Domain** → it gives you a
   TXT value starting `google-site-verification=`. Add it at Wix DNS as a TXT record on `@`.
   *Why:* the domain currently has no TXT records and no verification file, so Search Console is
   either unverified or verified in a way that dies the moment Wix stops serving the site. A DNS
   record keeps working after the move.
2. **Replit's note.** You get this in Part 3. Add it at the same time. It must never be deleted.

Check both are visible before moving on:

```bash
curl -s 'https://dns.google/resolve?name=happyjackrealestate.com&type=TXT' \
  | python3 -c 'import json,sys;[print(r["data"]) for r in json.load(sys.stdin).get("Answer",[])]'
```

---

## Part 3 — Add the domain inside Replit, and copy the numbers it gives you

In the Replit project: **Tools → Domains → Use a domain you already own**.

1. Enter `happyjackrealestate.com` → Next. It shows you **one A record value and one TXT value**.
   Write both down.
2. Repeat for `www.happyjackrealestate.com`. **`www` is a separate entry with its own A and its own
   TXT value.** Replit does not create it for you when you link the bare domain.
3. Replit only supports **A records** for this purpose — there is no CNAME to paste. That matters,
   because the existing `www` record is a CNAME and must be deleted, not edited.

Replit then checks your DNS every 1–2 minutes for about **6 hours**. If the records appear inside
that window, it verifies and issues the security certificate by itself. If the window closes with the
records missing, the domain shows **Failed** and Replit stops looking — the fix is to delete the
domain entry and add it again.

So: add the records *immediately* after this step.

---

## Part 4 — The switch itself (Wix → Domains → DNS)

Edit only these five things. Leave every TXT record alone.

| Do this | Record type | Host | Value |
|---|---|---|---|
| **Delete** | A | `@` | `185.230.63.107` |
| **Delete** | A | `@` | `185.230.63.171` |
| **Delete** | A | `@` | `185.230.63.186` |
| **Delete** | CNAME | `www` | `cdn3.wixdns.net` |
| **Add** | A | `@` | the IP Replit showed for `happyjackrealestate.com` |
| **Add** | TXT | `@` | the `replit-verify=…` value Replit showed (permanent) |
| **Add** | A | `www` | the IP Replit showed for `www.happyjackrealestate.com` |
| **Add** | TXT | `www` | that entry's own `replit-verify=…` value (permanent) |

Rules for this step:

- **Delete the CNAME before adding the `www` A record.** Two records for the same name confuse both
  browsers and Replit's certificate check.
- **Do not leave a second A record pointing at Wix.** Replit cannot issue a certificate while a
  name resolves to two different servers.
- **Do not add an AAAA (IPv6) record.** Replit supports A only.
- **Do not put the domain behind Cloudflare's orange proxy.** Replit cannot renew certificates for a
  proxied name. (Not applicable today — DNS is at Wix — but it is the trap if you ever move DNS.)

### 4.4 If Wix will not let you edit records

This can happen once the domain is no longer treated as a Wix-hosted site. In that case:

1. At **GoDaddy**, switch the domain's nameservers from Wix's to GoDaddy's own DNS.
2. Before you switch, re-create the four records above (the three A records + the `www` CNAME) in
   GoDaddy's DNS so nothing goes dark during the change.
3. Then do the five edits above in GoDaddy instead of Wix.

There is no ALIAS/ANAME requirement and no email to preserve, so plain GoDaddy DNS is enough.

---

## Part 5 — Wait, then confirm Replit is happy

Usually minutes; the outer limit is 48 hours.

- **Tools → Domains** must show **Verified** for both entries.
- `https://www.happyjackrealestate.com` must load the new site with a padlock.
- If it says **Failed**, confirm the A and TXT values with the check commands in Part 6, then remove
  and re-add the domain entry.

---

## Part 6 — Prove it worked (paste these, compare the answers)

```bash
# 1. www serves the new site, live
curl -s -o /dev/null -w '%{http_code}\n' https://www.happyjackrealestate.com/            # expect 200

# 2. indexable again
curl -s https://www.happyjackrealestate.com/robots.txt | grep -E '^(User-agent|Allow|Disallow|Sitemap)' | head -5
#    expect: User-agent: * / Allow: / / Disallow: /thank-you / Sitemap: https://www.happyjackrealestate.com/sitemap.xml
curl -s -o /dev/null -w '%{http_code}\n' https://www.happyjackrealestate.com/sitemap.xml  # expect 200
curl -s https://www.happyjackrealestate.com/sitemap.xml | grep -c '<loc>'                 # expect 58
curl -s https://www.happyjackrealestate.com/ | grep -o '<meta name="robots"[^>]*>'       # expect index, follow — no noindex
curl -s https://www.happyjackrealestate.com/ | grep -o '<link rel="canonical"[^>]*>'     # expect one, https://www...

# 3. the bare domain redirects to www
curl -s -o /dev/null -w '%{http_code} %{redirect_url}\n' https://happyjackrealestate.com/  # expect 301 https://www.happyjackrealestate.com/

# 4. old Wix addresses still resolve
for u in /home /pages-sitemap.xml /blog-posts-sitemap.xml /blog-categories-sitemap.xml /blog-feed.xml; do
  printf '%-30s ' "$u"; curl -s -o /dev/null -w '%{http_code} %{redirect_url}\n' "https://www.happyjackrealestate.com$u"; done
#    expect 301 for every one
curl -s -o /dev/null -w 'moqui-drive %{http_code}\n' https://www.happyjackrealestate.com/moqui-drive
#    expect 404 — the sold listing was deleted 2026-10-29 and stays deleted (see 1.3)

# 5. DNS is on Replit, not Wix
curl -s 'https://dns.google/resolve?name=www.happyjackrealestate.com&type=A' \
  | python3 -c 'import json,sys;print([r["data"] for r in json.load(sys.stdin)["Answer"]])'
#    expect exactly one address, the Replit one — no wixdns, no 185.230.63.x
```

Then open five pages in a browser and click through: home, a listing page, the blog, contact, and the
map search. A visual pass catches what curl cannot.

---

## Part 7 — After it is confirmed

1. **Search Console → Sitemaps** → submit `https://www.happyjackrealestate.com/sitemap.xml`.
2. **Search Console → URL Inspection** → homepage → **Request indexing**. Without this, Google may
   wait weeks to notice the new site.
3. **Leave Wix online for at least 7 days.** It costs nothing and it is your rollback. Only after a
   week of clean service should you unpublish the Wix site or disconnect the domain there.
4. **Never delete the two `replit-verify` TXT records or the GSC TXT record.** The certificate renews
   itself from them. Deleting them means a dead site months from now, with nothing obviously
   connected to the deletion.

### The undo card

If anything is wrong, you can be back on Wix in five minutes: at Wix DNS, delete the four records you
added and put back **exactly** the four values in Part 0.1. Browsers will follow within the record's
TTL.

---

## What is not blocking, but you should know

- **`/financing` is `noindex` and has no internal links.** It is a real page, deployed, but marked
  "don't index" and unreachable. Either link it and index it, or remove it. Someone should decide.
- **Rebuilding the site re-dates all 58 sitemap entries** to the build date. Harmless, but Search
  Console will report 58 "changed" URLs the next day. Do not be alarmed.
- **The `replit.app` address keeps working** after the move, alongside `www`. Its pages carry a `www`
  canonical, so Google will consolidate to `www` on its own.
- **The CSP still permits `https://i.replit.com`.** It can be tightened later, once Replit's injected
  scripts are switched off (Part 1.5).

## What I verified, and what I did not

**Verified today, by running or fetching it:**

- The three-string change serves the indexable tree — I applied it to a clone, booted the server and
  measured: `/robots.txt` `Allow: /`, `/sitemap.xml` 200 with 58 URLs, `/llms.txt` 200, homepage
  canonical + `index, follow`.
- The 301 map plus bare-domain redirect work — the eight legacy URLs answer 301 with query strings
  preserved, `/moqui-drive` and `/moqui-drive/` both answer 404 with the designed error page, and
  `http://happyjackrealestate.com/blog` answers `301 https://www.happyjackrealestate.com/blog`.
- The held page stays held: `/financing` answers `noindex, nofollow` and appears zero times in the
  58-URL sitemap.
- Current DNS: three Wix A records at the apex, `cdn3.wixdns.net` CNAME at `www`, no TXT, no MX, no
  common subdomains. Nothing else lives on this domain.
- Live behaviour of `happyjack-site.replit.app`, including which four old URLs 404 today.
- Replit's documented custom-domain rules (A records only, separate entry for `www`, permanent
  `replit-verify` TXT, 6-hour verification window).

**Not verified — needs someone inside the accounts:**

- The exact names and locations of the Replit badge / feedback-widget switches.
- Whether Wix still lets you edit DNS after the site is disconnected (Part 4.4 is the fallback).
- That Wix's GoDaddy registration is still paid through May 2027 — that comes from a 24 Nov record,
  not from a check today.
