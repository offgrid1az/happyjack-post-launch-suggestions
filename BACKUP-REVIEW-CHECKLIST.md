# What a Wix backup must contain, and how it will be reviewed

**Audience: whoever is exporting the existing site and its DNS settings out of Wix.**
Written 25 Nov 2026 by Bot 1. Read this **before** uploading, not after.

A backup that cannot be checked is not a backup — it is a promise. The point of the list below is that
every item has a way to prove it is complete, so the review either passes quickly or names one specific
gap instead of arguing about whether the export "looks right".

---

## 1. Where to put it

One branch, one commit, nothing else mixed in:

```
backup/wix-2026-11-25/          # pages, assets, manifests (this repo if public-safe)
```

Rules:

- **Do not mix it with launch changes.** The backup branch must be readable as "the Wix site, as it was",
  so that later nobody has to work out which files were Wix's and which were ours.
- **Commit the manifests, not 2 GB of images.** Asset *inventory* with sizes and SHA-256 is what gets
  reviewed; the binaries themselves can stay in a zip on Wix's export or object storage. If binaries do
  come in, put them under `assets/` and say so in the manifest; do not commit either one without the other.
- **One commit per item class** (pages, assets, DNS, Wix-app data) so each can be checked separately.
- State the **source and the timestamp** at the top of every manifest: which Wix account, which site,
  which moment. An export with no timestamp cannot later be distinguished from a later one.

---

## 2. The six items, with the check that proves each one

| # | Item | What to send | How it gets verified |
|---|---|---|---|
| 1 | **Page inventory** | Every page: URL path, page title, whether published / hidden / draft, template, and the HTTP status it returns today | Count is compared to the 62 pages already verified in the mirror. **Every difference must be explained by name** — a hidden page, a deleted page, a redirect. Unexplained difference = fail |
| 2 | **Rendered HTML** | The rendered page for each entry in item 1, captured from the **live Wix site**, saved as `.html` | Inventory and reconcile, **not** byte-compare: every Wix URL maps to a replacement URL, and every content block is accounted for. Byte-matching old Wix markup against a rebuilt page would flag the entire site as a gap, since the replacement is a rebuild — not a copy |
| 3 | **Asset manifest** | Every image, font, video and PDF: original URL, size, SHA-256, and the local path it was saved to | **Count of URLs still pointing at `wixstatic.com` / `parastorage.com` must be zero.** A manifest that lists assets but leaves them hotlinked is a mirror of a site that breaks the day Wix is switched off |
| 4 | **DNS zone export** | Every record: type, host, value, TTL, plus the four nameservers | Compared record-by-record to the baseline in §4. TXT and MX records matter more than A records — they are the ones nobody remembers |
| 5 | **Wix apps and integrations** | Each installed app by name (forms, chat, bookings, store, blog comments), its settings, and whether it stores data outside Wix's own export | Named apps that are still in use must have a replacement or an explicit "retired" decision. ~~Open Map has no backup~~ — **corrected 25 Nov:** Open Parcel is a separate Replit/Android project, not an installed Wix app, and it holds a verified 227-file backup at `open-parcel-android:backup/replit-full-20261125-154950`. Nothing is missing there |
| 6 | **Data held inside Wix** — note that the Wix Editor provides no externally restorable site archive; the in-account duplicate also excludes contacts, orders, campaigns and third-party secrets, so this item cannot be closed by the site backup | Form submissions, contacts/members, orders, invoices, reviews, blog comments — counts plus an export where the dashboard allows it | Counts are what matter. If a count cannot be produced, say so explicitly rather than omitting the row |

**Measured on the current mirror, for comparison:** 62 `.html` files, 18 local image/font files, 4.6 MB,
**zero** references to `wixstatic.com` or `parastorage.com`, and nine absolute `src=` URLs that are all
third-party widgets (RateMyAgent's carousel and four IDX Broker embeds). So the mirror is already free of
Wix dependencies; if the export contains a hotlinked asset, that is new information rather than something
inherited.

---

## 3. What a backup does NOT cover — do not let this be read as a guarantee

State these in the upload, so nobody assumes a green review means everything is safe:

- **There is no restorable Wix site export.** Wix's own documentation says a site cannot be stored outside Wix as a restorable archive; the in-account duplicate is the only copy, and it does not include contacts, orders, campaigns or app secrets. That is a limit of the platform, not a gap in the export.
- **Wix form submissions and Contacts live in Wix.** An HTML export does not include them. If any enquiry
  is still only in Wix, it is lost the moment the account is closed — and nothing about the DNS cutover
  affects it, so this is a separate task with its own deadline.
- **Analytics history** (Wix Analytics) does not export as data. If it matters, capture screenshots or
  CSVs of the periods that matter before the account closes.
- **Image variants.** Wix serves resized derivatives from its CDN. Saving the "original" URL does not
  always give the original; verify dimensions on a sample and say so.
- **Redirects configured inside Wix** are not in the HTML export. They must be added to
  `hosting/redirects.json`, which is the single source of truth for the 301 map.
- **Anything a third-party script loads at runtime** is not in the backup by definition.

---

## 3b. DNSSEC — the one item that can take the whole domain down

`dnssecEnabled: true` in the Wix zone, with `keyTag 21451` and a digest. Verified independently on
25 Nov against two public resolvers (Google and Cloudflare): **the zone serves DNSKEY and RRSIG, but the
parent publishes no DS record** (`DS` → NONE, `AD` false on both). So the zone is *signed but not
chained*: nothing validates it today, and the cutover cannot break resolution through DNSSEC.

Two things follow, and they are not obvious:

- **Do not enable DNSSEC at the registrar** while moving nameservers. A DS published for Wix's key, with
  nameservers at Replit that do not hold that key, makes the entire domain fail to resolve — website,
  mail, everything — and the failure looks like "the site is down", not like a DNSSEC problem.
- **After the move, Wix's zone and its keys are gone.** If the registrar ever shows a DS record for this
  domain, remove it *before* changing the nameservers. Confirm the registrar's DNSSEC state explicitly
  rather than inferring it from Wix's toggle.

---

## 4. DNS baseline to check the export against

Confirmed from public DNS on 25 Nov 2026:

| Type | Host | Value | Note |
|---|---|---|---|
| NS | `@` | `ns8.wixdns.net`, `ns9.wixdns.net` | The zone is hosted by Wix, registrar is GoDaddy |
| A | `@` | `185.230.63.107` | Wix |
| A | `@` | `185.230.63.171` | Wix |
| A | `@` | `185.230.63.186` | Wix |
| CNAME | `www` | `cdn3.wixdns.net` | Wix |
| MX | — | **none** | No mail exchanger; Wix also reports `hasMailbox: false`. Strong evidence, not proof, that no mail depends on this zone |
| TXT | — | **none** | No TXT records at all. This does **not** prove Search Console is unverified — other verification methods exist |
| AAAA | — | **none** | No IPv6 |
| Subdomains | `mail`, `ftp`, `autodiscover`, `calendar`, `webmail`, `smtp`, `imap`, `portal`, `sites`, `dev`, `staging`, `shop`, `blog`, `api`, `_domainconnect` | **none exist** | Checked one by one |

**If the export contains a record that is not in this table, that record is the most important thing in
the export.** It is something we do not know about, and it is the thing most likely to break at cutover.
Do not summarise it — list it and flag it.

---

## 5. What the review will conclude

One of three outcomes, per item:

- **PASS** — complete and self-proving. Nothing more needed.
- **PASS WITH NOTE** — complete, but contains something we must act on (an app still in use, a count that
  could not be produced, a record we did not know about). The note names the action and who owns it.
- **GAP** — an item is missing or cannot be verified. The gap is named specifically, with what would
  close it. A gap does not block the cutover unless it is item 1, 2 or 4.

**What does block the cutover:** an unexplained difference in the page inventory (item 1), a page whose
content differs beyond the two injected tags (item 2), or a DNS record nobody can account for (item 4).

**What does not block the cutover:** Wix form submissions, analytics history, image variants. Those are
real losses with their own deadline, and they should be handled on their own track rather than holding the
domain move hostage.

---

## 6. What only the owner can do

These cannot be delegated to a bot or to Replit, because they need the Wix account holder's authorisation:

1. **Export Wix form submissions and Contacts** from the Wix dashboard, or approve an export that a
   delegate performs. If there are no submissions, say so — that is a valid and useful answer.
2. **Decide the fate of Wix-only apps** that are still in use (Open Map is the open one). Keep, replace, or
   retire — the backup cannot decide this.
3. **Approve the GoDaddy fallback** if Wix stops allowing DNS edits after the site is detached.
4. **Make the five DNS changes** at cutover, or authorise someone to make them.
5. **Decide when Wix gets switched off** — recommended: not before seven clean days.
