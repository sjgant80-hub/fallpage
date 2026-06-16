# ◊ FallPage · sovereign multi-page layout

> The InDesign wedge. Multi-page documents · brochures · reports · invoices · product sheets → printable PDF. Single HTML, no server, no subscription.
>
> v1 · prime **457** · MIT · ◊·κ=1

**Live:** [sjgant80-hub.github.io/fallpage](https://sjgant80-hub.github.io/fallpage/)
**Source:** [github.com/sjgant80-hub/fallpage](https://github.com/sjgant80-hub/fallpage)

Phase-2 tool of the **FallStudio** plan — the sovereign Adobe Creative Cloud alternative being shipped one HTML file at a time. FallPDF (449) writes single-file PDFs from markdown; FallPage (457) lays out multi-page documents with columns, master styles, and page numbers, then hands off to the browser print engine for the PDF.

---

## What it does

Three-pane workspace: page thumbnails on the left, the live page in the middle (scaled, paper-shadow on dark workspace), block list + page setup + master styles on the right. Build the document block by block — heading, paragraph, image, pull quote, column break, spacer, divider, table-of-contents. Switch pages with one click. **↓ pdf** opens the print dialog with a stylesheet that hides the editor chrome and emits each page at its exact mm size.

### Block types

| Type | What it does |
|---|---|
| **heading** | H1 / H2 / H3, brass by default — feeds the auto-TOC |
| **paragraph** | Rich text from markdown (`**bold**`, `*italic*`, `` `code` ``, links, lists, tables) |
| **image** | File upload, embedded as base64. Width in mm (or auto) |
| **pull quote** | Large brass-bordered quote |
| **columns split** | Force-break to the next column on multi-column pages |
| **spacer** | mm of vertical space |
| **divider** | Horizontal rule |
| **table of contents** | Auto-generated from every heading in the doc |

### Page setup

- Size: **A4 · A5 · Letter · Legal · business card · custom (mm)**
- Orientation: portrait / landscape
- Columns: 1 / 2 / 3 (CSS multi-column, 6mm gutter)
- Margins: top / bottom / left / right (mm)

### Master styles

- Body font: Helvetica · Georgia · Times · system serif · system sans
- Body size (pt) and line-height
- Heading colour (default brass `#b8974a`)
- Page numbers: none · centre · outer (alternates left/right by parity)
- Running header / footer with `{{page}}`, `{{title}}`, `{{total}}` placeholders

### Templates (Ω palette · T0)

**brochure** (3 pages · 2-col A4) · **one-pager** (single A4) · **report** (cover + TOC + body + appendix) · **invoice** (single page · markdown table) · **product sheet** (image left + spec right · 2-col).

### Ω autopilot (Ctrl+K)

- *"brochure"* / *"invoice"* / *"report"* / *"one-pager"* → loads the template
- *"add page"* / *"duplicate"* / *"export pdf"* → action
- T3 (BYOK Anthropic / Gemini *free* / OpenAI / OpenRouter): *"draft a 2-page company overview for an SME audience"* → Ω returns a JSON page graph, FallPage builds the doc

### Save / load / interchange

- **Auto-save** to IndexedDB on every change (debounced 800ms)
- **⌂ save** / Ctrl+S — force save now
- **⇪ load** — modal lists every saved doc with title · pages · size · timestamp
- **↓ json** / **⇪ json** — full doc as `.fallpage.json` for backup or moving between machines

### Export

- **↓ pdf** (Ctrl+E) — injects `@page { size: WxH mm; margin: 0 }` for the current doc, builds the print root, calls `window.print()`. The browser's PDF engine produces the file. Editor chrome is hidden by `@media print`.

---

## Use it

1. Open `index.html` from `file://` or visit the live URL
2. Pick a template (**Ctrl+K** → *brochure*) or start blank
3. Add blocks on the right, switch pages on the left
4. Adjust page setup + master styles
5. **↓ pdf** — choose *Save as PDF* in the print dialog

Auto-saves on every change. A friendly starter page appears on first launch.

## Keyboard

- **Ctrl+K / ⌘K** — Ω autopilot
- **Ctrl+S / ⌘S** — save
- **Ctrl+E / ⌘E** — export PDF (print)
- **Esc** — close palette / modal
- **↑ / ↓ / Enter** in the palette

## Sovereignty stack

| Layer | Status |
|---|---|
| **UI** | runs from `file://` |
| **Compute** | T0 lays out + prints in-browser · T3 your direct API call (Ω drafting only) |
| **Storage** | IndexedDB for docs + settings · PDF downloads via your browser's print engine |
| **Mesh** | `BroadcastChannel('fall-signal')` · prime 457 · responds to ping · `postMessage` API: `ping`, `addPage`, `getDocJson`, `loadTemplate`, `exportPdf` |

---

## What this v1 does NOT do — and what's planned

**v1 scope:** layout *editor* + browser print export. Real PDF tooling (page imposition, bleed marks, CMYK) is v2.

| Feature | Reason | Plan |
|---|---|---|
| **Bleed / crop marks** | print engine has no direct hook | v2 — render to canvas then to PDF via pdf-lib |
| **CMYK colour** | browsers are sRGB-only | v2 — pair with FallPDF's writer |
| **Master pages (true)** | v1 ships master *styles* (header/footer/numbers); true master *layouts* require a second editor surface | v1.1 |
| **Per-block text formatting (inline colour, font swap mid-paragraph)** | markdown is the wire format | v1.1 — extend the markdown |
| **Linked text frames across pages** | flow engine is non-trivial | v2 |
| **Image cropping / masking** | needs a crop UI | v1.1 |

The 80% that people use InDesign for is *page setup → blocks → multi-page → print*. That's v1.

---

## For developers

```
index.html      one file · ~30KB · vanilla JS · no deps · zero CDN
README.md       this
LICENSE         MIT
.nojekyll       Pages legacy deploy
```

### Document model

```js
{
  id, title, created, modified,
  pageSize: 'A4'|'A5'|'Letter'|'Legal'|'card'|'custom',
  customW, customH,                  // mm, when pageSize==='custom'
  orientation: 'P'|'L',
  columns: 1|2|3,
  margins: { t, b, l, r },           // mm
  masterStyles: {
    font, size, lineHeight, headingColor,
    pageNumbers: 'none'|'center'|'outer',
    header, footer                   // {{page}} {{total}} {{title}}
  },
  pages: [{ id, blocks: [{ id, type, ...payload }] }]
}
```

Block payloads: `heading {level,text}` · `paragraph {text}` · `image {src,width}` · `pullquote {text}` · `columns {}` · `spacer {height}` · `divider {}` · `toc {}`.

### Architecture

- **Render:** `renderPageContent(page,i,doc)` returns the inner HTML for one `.page-content` box (absolute-positioned inside margin rect). The preview wraps that box in a scaled `.page` (CSS transform). The print path reuses the same function inside `.print-page` boxes at 1:1 mm.
- **Print:** `@media print` hides everything except `.print-root`. `exportPDF()` injects a `@page { size: WmmxHmm; margin:0 }` style tag for the active doc dims, builds the print root, then `window.print()`.
- **Persistence:** two object stores — `docs` (keyPath `id`) and `kv` (settings + `last` doc pointer). Auto-save debounced; `beforeunload` flushes dirty state.
- **Cascade:** Ω drafts via Anthropic / Gemini / OpenAI / OpenRouter; same pattern as FallOffice, FallMage, FallPDF, ACG Mapper, FallMap.

### Adding a template

Add a key to `TEMPLATES` — the Ω router matches on the key name, the name field, or the description. Each template provides `title`, page setup, master styles, and `pages: [{blocks:[...]}]`. The loader assigns fresh ids on import.

### Adding a block type

Three touch points: `renderBlock()` for HTML, `renderBlockList()` for the right-panel preview line, and `renderBlockEdit()` for the editor form. Then add a case to the `addBlock()` factory.

### Adding a page size

Extend `PAGE_SIZES` (`[wMm, hMm]`) and add an `<option>` to `#pgSize`. Orientation flips at render time.

## Credit

- Same cascade pattern as [FallOffice](https://github.com/sjgant80-hub/falloffice), [FallMage](https://github.com/sjgant80-hub/fallmage), [FallPDF](https://github.com/sjgant80-hub/fallpdf), [ACG Mapper](https://github.com/sjgant80-hub/acg-mapper), [FallMap](https://github.com/sjgant80-hub/fallmap)
- The layout engine is CSS — `position:absolute` margin rect + native `column-count` for flow. No measurement library, no JS shim. Print is browser-native.
- Part of the FallStudio plan — Adobe Creative Cloud, sovereign, one HTML at a time

⚒ Part of the [fall* estate](https://github.com/sjgant80-hub) · prime 457 · ◊·κ=1
