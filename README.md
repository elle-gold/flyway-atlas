# Following Summer

A three-part visualisation of bird migration for fun. Three self-contained pages, no build step,
no framework, no binary assets.

| Page | What it shows |
|------|---------------|
| `index.html` | Landing page tying the three volumes together |
| `tree.html` | **Vol. I — Canopy of Migration.** The bird family tree as a rotating 3D canopy, coloured by migratory intensity, with each branch as thick as its clade is species-rich |
| `atlas.html` | **Vol. II — Flyway Atlas.** Six species on a world map, moving along their flyways as you scrub the year |
| `globe.html` | **Vol. III — Chasing the Sun.** The same routes as great circles on a WebGL globe, with a day/night terminator that moves with the date |

## Preview locally

```bash
python3 -m http.server 8000
```

Then open <http://localhost:8000>. Any static file server works; opening the files
directly with `file://` also works, since nothing is fetched at runtime.

## Deploy

Everything is static and every internal path is **relative**, so the folder works at a
domain root *or* in a subdirectory (`example.com/migration/`) with no changes.

**Netlify** — drag the folder onto <https://app.netlify.com/drop>, or:
```bash
npx netlify-cli deploy --prod --dir .
```

**Vercel**
```bash
npx vercel --prod
```

**Cloudflare Pages**
```bash
npx wrangler pages deploy . --project-name following-summer
```

**GitHub Pages** — commit the folder to a repo, then Settings → Pages → deploy from
branch. `.nojekyll` is included so nothing gets filtered.

**Any web host / S3 / nginx** — upload the folder as-is. No server-side anything, no
redirects required. `netlify.toml` and `vercel.json` only add cache and security headers;
they're safe to delete on other hosts.

## Before you go live

**1. Domain — already set.** Canonical and `og:url` tags, `sitemap.xml` and `robots.txt`
all point at <https://eauwith.github.io/flyway-atlas/>. If you move the site to
another host or a custom domain, update it everywhere at once:

```bash
grep -rl 'eauwith.github.io/flyway-atlas' . | xargs sed -i '' 's|eauwith.github.io/flyway-atlas|yourdomain.com|g'
```

(Drop the `''` after `-i` on Linux.)

**2. Add social preview images.** The `og:image` tags expect
`assets/og-index.png`, `og-tree.png`, `og-atlas.png`, `og-globe.png` at 1200×630.
Screenshots of each page work well. Until those exist the block stays commented out —
a broken `og:image` is worse than none.

**3. Optional: self-host the fonts.** Google Fonts is the only third-party request the
site makes at runtime. If you need to avoid it (GDPR, offline, air-gapped), download the
families with something like [google-webfonts-helper](https://gwfh.mranftl.com/fonts),
drop the `.woff2` files into `assets/fonts/`, and swap the `<link>` in each page's head
for a local `@font-face` block. Every page already declares a real fallback stack, so the
site stays legible if the fonts never load.

## Editing the content

Each page keeps its data in one array near the top of its `<script>`, so adding or
changing a species is a local edit — no other code needs touching.

`atlas.html` and `globe.html` share the same shape:

```js
{ id:"tern", name:"Arctic Tern", latin:"Sterna paradisaea", color:"#ff6a4d",
  breeding:{ lon:-20, lat:74, name:"Greenland & the High Arctic" },
  wintering:{ lon:-15, lat:-68, name:"Antarctic pack ice" },
  nb:[3.0, 4.7],    // northbound window, as decimal months (0 = Jan)
  sb:[7.5, 9.6],    // southbound window
  distance:"~90,000 km / year",
  note:"..." }
```

Positions are interpolated between the two anchor points, so a new entry needs nothing
but coordinates and dates. `tree.html` uses a similar `nodes` array keyed by `parent`.

Coastlines live in the `CONTINENTS` array as `[lon, lat]` outlines; the globe paints its
earth texture from them at runtime rather than loading an image.

## A note on the data

**These are modelled illustrations built from published satellite-tracking and range
studies — not a live radar or telemetry feed.** Breeding and wintering locations, route
shapes and migration windows follow the literature for each species; positions between
those anchors are interpolated, not observed. Daylight hours and the terminator are
computed from real solar declination for the selected date, so those are genuine
astronomy. Please keep the on-page disclaimers if you republish this.

For live nocturnal migration radar over North America see [BirdCast](https://birdcast.info);
for observation records see [eBird](https://ebird.org).

## Browser support

Modern evergreen browsers. Vol. III needs WebGL and shows a plain-text fallback without
it. All three respect `prefers-reduced-motion`. Vol. I and II are 2D canvas and run
essentially anywhere.

## Credits and licensing

- [Three.js](https://threejs.org) r128 — MIT (vendored at `assets/vendor/three.min.js`)
- [GSAP](https://gsap.com) 3.12.5 — free standard licence; check
  <https://gsap.com/licensing> if your use is commercial
- Fraunces, Newsreader, IBM Plex Mono — SIL Open Font License, served by Google Fonts

The libraries are vendored rather than hot-linked, so the site has no runtime dependency
on a third-party CDN. To update one, drop in a new file and keep the same path.
