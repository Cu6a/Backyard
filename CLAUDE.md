# Backyard Ultra Namibia, Team Namibia

Single-page site for Namibia's national team at the Backyard Ultra World Team
Championship, 17 October 2026 at Avis Dam, Windhoek. Organised by the Wednesday
Social Running Crew.

- Live at https://backyard-ultra.trailrunna.com

## Hosting: NOT GitHub Pages any more

**Pushing to `main` does not publish anything.** The repo is the source of
truth; publishing is a separate manual upload. This caught us out once, so it
is the first thing in this file.

The site is served from the account's own cPanel hosting:

- Document root `/home/traibyaa/backyard-ultra.trailrunna.com`
- DNS: an A record for `backyard-ultra` to `199.188.205.217`, TTL 300, in the
  cPanel Zone Editor for `trailrunna.com` (nameservers are
  `dns1`/`dns2.namecheaphosting.com`, so that zone is authoritative; Namecheap's
  Advanced DNS panel edits the same zone)
- A dormant WordPress install is parked in `_wp-old` inside the document root.
  It was moved there, not deleted, and should be cleaned up.
- `CNAME` is left in the repo but no longer does anything

**To publish a change:** zip the site (everything except `.git`, `CLAUDE.md`,
`README.md`, `CNAME`, `.gitignore`), upload to the document root via cPanel
File Manager, extract, overwrite. Nothing during an event needs this, because
the live board reads from the Google Sheet.

### Why it moved, so nobody moves it back

MTC, Namibia's largest mobile network, **refuses connections to GitHub Pages'
IP range** (`185.199.108-111.153`), returning `ERR_CONNECTION_REFUSED`.
`github.com` itself is fine, so it is that range specifically. The site was
unreachable for MTC mobile visitors, which is a large share of the Namibian
audience, while loading normally on WiFi and from abroad.

Diagnosis that settled it: on MTC with no VPN, `pages.github.com` fails and
`github.com` loads. If GitHub Pages is ever reconsidered, re-run that test
first.

## Shape of the repo

Everything is one self-contained `index.html`: Tailwind from its CDN, fonts from
Google Fonts, all CSS and JS inline. There is no build step. Edit the file,
commit, push.

```
index.html              the whole site
images/                 roster photos, team badge, OG image
images/sponsors/        sponsor logos
icons/                  favicon set
favicon.ico             16/32/48 multi-resolution
robots.txt sitemap.xml site.webmanifest
```

## House style

- **No em dashes anywhere.** Use a colon, a comma or `&middot;`. One `&mdash;`
  survives as a decorative placeholder in the live yards panel.
- Sponsor role lines are terse and lowercase after the first word: "Venue",
  "Media partner", "ICT partner &amp; WiFi provider".
- Brand colours are Tailwind-extended: `sand #F0EDE4`, `nearblack #151412`,
  `ember #D96A3E`, `ash #C4B8A6`, `rule #3A352E`, `panel #201D19`.

## Sponsor logos, the part that bites

Tiles sit on `bg-panel` (#201D19, near-black) and render at `grayscale(1)` and
`opacity 0.65` until hovered. Only luminance matters at rest, so colour is
irrelevant until hover. Dark artwork disappears.

Prepare every logo as: trim the transparent margin, resize to **700px wide**,
save as PNG, then `quantize(colors=255, method=FASTOCTREE)` to keep the file
small. Skip the quantize step only for photographic artwork with gradients,
where it bands (see `images/ivan-heigan.png`, left as full RGBA for that
reason).

Then pick a treatment by what the artwork is:

| Artwork | Treatment |
|---|---|
| Already light or white on transparency | Nothing. Plain tile. |
| Mid-tone, reads at rest | Nothing, or `is-bright` if it still looks dim |
| Dark but **single colour** | Repaint the ink `#F0EDE4`, keeping alpha. No brand colour is lost on a monochrome mark. See `windhoek-hire.png`, `pcrf.png`. |
| Dark and **multi-colour** | Raise HSL lightness to a floor (0.55 to 0.80 depending on how dark), keeping hue and saturation, then add `is-bright`. See `ocean-conservation-namibia.png`, `kickstart.png`, `alpha-sports-nutrition.png`. |

`is-bright` raises resting opacity to 0.88 and brightens the image. It cannot
rescue pure black, because brightening black leaves it black. It is declared
**above** the `:hover` rule on purpose: both are specificity 0-2-0, so source
order decides and hover must win. Do not reorder that block.

The brightness sits on the inner `img`, not the tile, because the tile carries
the panel background and border and would otherwise be lifted too.

**Always render a preview before committing a logo.** Composite it onto
`(32,29,25)`, apply greyscale and the resting opacity, and look at it next to
its neighbours.

## Sponsor grid

`.sponsor-grid` is flex-wrap with `justify-content: center`, not a CSS grid, so
a part-full final row centres instead of hanging off the left edge. Widths
mirror a 2-up / 4-up grid with a 1.5rem gap. It self-corrects for any sponsor
count, so adding or removing a sponsor needs no layout change.

## Roster

Ranks use **standard competition ranking**: ties share a rank and the next rank
skips the tied places, so five athletes tied at 9 are followed by 14. Sort by
personal best descending, then recompute every rank. The staggered vertical
offsets cycle `0, 6, 3, 8, 2` every five cards, so reassign them by position
when the order changes.

Roster photos are hex-cropped transparent PNGs at 700x875. New ones must match
that hexagon geometry; measure an existing file's alpha profile to check.

## Live yards

Driven by a Google Sheet so it can be updated from a phone during the event
with no deploy. Sheet ID is in the script at the bottom of `index.html`.

One tab holds both blocks, fetched in a single request over `A1:M500`:

- **A to E**, the yard log: `yard, time, runners_left, namibians_left, note`
- **G2** status pill text, **H2** the intro line
- **J to M**, the team board: `runner, yards, status, note`, one row per athlete

While G2 reads "Not yet started" the yard log keeps its placeholder and no log
rows render. The team board shows whenever columns J to M hold any names, and
hides when they do not, so it degrades to nothing rather than to an empty chart.

Read via the gviz CSV endpoint, which requires the sheet shared as
anyone-with-the-link can view. A failed fetch leaves the current screen
untouched rather than blanking the section.

## Team board avatars

`images/avatars/<slug>.png` are 160px circular head crops cut from the roster
photos, matched to sheet rows **by runner name** at runtime by reading the
gallery already in the page, so changing the roster needs no second edit.

The crop is a square taken from the roster photo as `(side, centre_x, top_y)`
expressed as fractions of the frame. Default `(0.55, 0.50, 0.04)` suits a
runner facing the camera near the top of frame. Four needed their own values,
found by looking at the photo and reading off where the head sits:

| Runner | side, centre x, top y | Why |
|---|---|---|
| `danielle-zimny` | 0.38, 0.52, 0.14 | holding a phone up across her face |
| `tobias-verwey` | 0.60, 0.50, 0.00 | cap brim and sunglasses, sits high |
| `jana-van-dyk` | 0.32, 0.42, 0.29 | small in frame, left of centre |
| `buks-gous` | 0.30, 0.45, 0.24 | distant, head a third of the way down |

Automatic subject detection was tried and does not work on these photos: the
hexagon edge and the horizon line dominate any contrast or background-distance
measure. Read the coordinates off the image by eye instead.

## SEO

The `h1` is "One loop. Every hour. Together.", which carries no searchable term,
so a `sr-only` span names the event ahead of it. Keep that span if the hero
changes. There is schema.org `SportsEvent` and `WebSite` JSON-LD in the head,
plus canonical, Open Graph and Twitter tags, `robots.txt` and `sitemap.xml`.
GA4 is `G-NHHLNH4CP9`, loaded immediately after `<head>`.

## Working on this repo

- **Verify against the live site, not a local browser.** The sandbox proxy
  blocks `cdn.tailwindcss.com`, so a local headless render has no Tailwind at
  all and will look broken in ways the real page is not. A local test once
  reported 700px tiles stacked one per row; the page was fine.
- Validate before committing: tag balance in the section you touched, that
  every `src` resolves, and the tile or card count.
- Remove a block by exact literal string, not a regex. A pattern that stops at
  the first `</div>` cuts the inner one and orphans the caption.
- After uploading, confirm the change on the live URL. Pushing to `main` is
  not publishing; see the hosting section at the top.
- Verifying from outside is worth the effort: it has caught a stale deploy, a
  subtitle reading "deepest yard 1" before anyone had run a yard, and an
  avatar overlapping its label.
