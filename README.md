# Z VAULT

A single-page archive for visual artifacts, filed by concept.

No build step, no framework, no upload form. One `index.html` holds the styling, the router, and the index itself. Work is placed on shelves by hand, the way a vault should be.

---

## Contents

- [Running it](#running-it)
- [File layout](#file-layout)
- [How it works](#how-it-works)
- [Adding a shelf](#adding-a-shelf)
- [Adding an artifact](#adding-an-artifact)
- [Field reference](#field-reference)
- [Source types](#source-types)
- [Shelf patterns](#shelf-patterns)
- [Routes](#routes)
- [Shortcuts](#shortcuts)
- [Deploying](#deploying)
- [Notes and gotchas](#notes-and-gotchas)
- [Colophon](#colophon)

---

## Running it

Open `index.html` in a browser and it works. But if you're using `source.type: "file"` or `"image"`, serve the folder instead — browsers block or misbehave with local iframes and images over `file://`.

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

---

## File layout

```
z-vault/
├── index.html          # the whole site — styles, data, router
├── README.md           # this file
└── artifacts/          # your plates
    ├── silicon-001.html
    ├── protocols-001.png
    └── …
```

`artifacts/` is a convention, not a requirement. Paths in the data are resolved relative to `index.html`.

---

## How it works

Two arrays near the top of the `<script>` block drive everything:

| Array | What it holds |
|---|---|
| `CATEGORIES` | The shelves. Order here is the order on the index page. |
| `ARTIFACTS` | The holdings. Each entry belongs to exactly one shelf. |

Everything below the `Below this line is the machine` comment is rendering, routing, and preview scaling. You shouldn't need to touch it.

Accession numbers (`ZV-01 · 003`) are computed at render time from the shelf code plus the artifact's position within its shelf. They are not stored.

---

## Adding a shelf

Add an object to `CATEGORIES`:

```js
{
  id:      "optics",
  code:    "ZV-09",
  name:    "Optics & Sensing",
  pattern: 2,
  blurb:   "Lenses, capture, and what the machine actually sees."
}
```

| Field | Required | Notes |
|---|---|---|
| `id` | yes | Lowercase slug. Becomes the URL (`#/c/optics`) and the value artifacts point at. Must be unique. |
| `code` | yes | Accession prefix, displayed on the plate and in the viewer. Convention is `ZV-NN`. |
| `name` | yes | Display name. |
| `pattern` | yes | `1`–`8`. Picks the moiré face. See [Shelf patterns](#shelf-patterns). |
| `blurb` | yes | One line. Shows on the plate and at the top of the shelf page. Keep it under ~90 characters so plates stay even. |

Existing shelves:

| Code | id | Name |
|---|---|---|
| ZV-01 | `silicon` | Silicon & Compute |
| ZV-02 | `protocols` | Protocols & Networks |
| ZV-03 | `systems` | Systems & Architecture |
| ZV-04 | `ai` | AI Concepts |
| ZV-05 | `interface` | Interface & Type |
| ZV-06 | `management` | Business & IT Management |
| ZV-07 | `psychology` | Psychology & Health |
| ZV-08 | `field` | Field Notes |

`field` is the unfiled shelf — put work there when it hasn't earned a home yet.

---

## Adding an artifact

1. Drop the file into `artifacts/`.
2. Add an entry to `ARTIFACTS`.

```js
const ARTIFACTS = [
  {
    id:       "silicon-001",
    category: "silicon",
    title:    "TSMC Node Roadmap, N3 → A16",
    date:     "2026-08-22",
    summary:  "Process nodes against announced volume dates and customer commitments.",
    tags:     ["tsmc","fabs","roadmap"],
    source:   { type: "file", path: "artifacts/silicon-001.html" }
  }
];
```

An empty shelf prints a ready-to-paste stub with the correct `category` and today's date already filled in — open the shelf and copy from there.

---

## Field reference

| Field | Required | Type | Notes |
|---|---|---|---|
| `id` | yes | string | Unique across the whole vault. Becomes the permalink: `#/a/silicon-001`. Changing it breaks any link you've shared. |
| `category` | yes | string | Must match a `CATEGORIES.id` exactly. A mismatch renders the artifact nowhere. |
| `title` | yes | string | Shown on the thumbnail and as the viewer heading. |
| `date` | no | string | `YYYY-MM-DD`. Anything else falls back to the raw string; omitted reads as `undated`. |
| `summary` | no | string | One line, shown in the viewer sidebar. Falls back to `—`. |
| `tags` | no | string[] | Searchable from the index finder. Falls back to `—`. |
| `source` | no | object | See below. Missing or unrecognized renders `No source attached`. |

All values are HTML-escaped on the way out, so apostrophes and angle brackets in titles are safe.

---

## Source types

**`file`** — a local HTML page, rendered live in an iframe.

```js
source: { type: "file", path: "artifacts/silicon-001.html" }
```

**`image`** — PNG, JPG, or SVG. `object-fit: cover` on the thumbnail, `contain` in the viewer.

```js
source: { type: "image", path: "artifacts/protocols-001.png" }
```

**`inline`** — markup written directly into the entry, no separate file. Good for a single SVG.

```js
source: { type: "inline", html: '<svg viewBox="0 0 800 500">…</svg>' }
```

Use single quotes around `inline.html` so the double quotes in your markup survive. The string is escaped into an iframe `srcdoc`, so it renders sandboxed from the page.

---

## Shelf patterns

`pattern` selects one of eight generated moiré faces. Nothing is loaded — they're layered CSS gradients drawn in pure ink on pure paper, and they tilt on hover.

| # | Face |
|---|---|
| 1 | Fine diagonal hatch, 38° |
| 2 | Offset concentric rings |
| 3 | Square grid |
| 4 | Conic radial burst |
| 5 | Crossed diagonals, uneven weight |
| 6 | Heavy vertical bars |
| 7 | Rings over horizontal rules |
| 8 | Tight diagonal hatch, 74° |

Reusing a pattern across shelves is fine — the tilt direction alternates with the number, so odd and even patterns lean opposite ways.

---

## Routes

Hash-based, so it works on any static host with no server config.

| Route | View |
|---|---|
| `#/` | Index — all shelves, with the finder |
| `#/c/{category-id}` | Shelf — thumbnails of everything filed there |
| `#/a/{artifact-id}` | Viewer — full-size stage plus metadata |

Unknown ids land on a *Not found* page rather than a blank screen.

---

## Shortcuts

| Key | Action |
|---|---|
| `/` | Jump to the finder (or return to the index if you're not on it) |
| `Esc` | Clear the finder |
| `Invert` | Flips paper and ink. Persists in `localStorage` under `zvault-mode`. |

The finder matches shelf names, codes, and blurbs, and also matches artifact titles and tags — so a shelf surfaces if something inside it matches, even when its own name doesn't.

---

## Deploying

Static files, so anywhere works. For GitHub Pages:

```bash
git add index.html README.md artifacts/
git commit -m "vault: add artifacts"
git push
```

Then Settings → Pages → deploy from branch. Because routing is hash-based, no `404.html` rewrite is needed.

---

## Notes and gotchas

- **Thumbnails render at 1280×800.** Live HTML previews are scaled down from that base into a 16:10 frame. Artifacts designed much wider or narrower will crop or letterbox in the thumbnail — they still open correctly in the viewer.
- **Accession numbers follow array order.** Reordering `ARTIFACTS` renumbers the shelf. The `id` is the stable handle; the accession number is presentation.
- **Two values only.** Every color in the sheet resolves to paper or ink. If you add an artifact with its own palette it will read as a colored object sitting on a monochrome shelf — that's the intent, but it's worth knowing before it surprises you.
- **Iframes are same-origin.** A local artifact page can reach the parent document. Keep third-party embeds out of `type: "file"` unless you trust them.
- **No search index.** Filtering is a linear scan over both arrays on every keystroke. Fine into the hundreds; revisit past that.

---

## Colophon

Set in **Cormorant Garamond** (display) and **Share Tech Mono** (labels, codes, metadata). Paper `#ffffff`, ink `#0b0b0b`, inverted to `#0b0b0b` and `#f6f6f6`. Rules at 1px, gutters fluid from 20px to 64px.

Motion respects `prefers-reduced-motion`.
