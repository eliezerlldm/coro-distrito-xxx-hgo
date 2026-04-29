# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This repository stores **attendance dashboards** (`.html`) generated from CSV/Excel attendance files for church districts in the state of **Hidalgo (HGO)**. Each dashboard is a single self-contained HTML file — no build step, no dependencies installed locally.

## Generating a Dashboard

When the user uploads a CSV or Excel attendance file, follow the skill defined in `README.md` exactly:

1. **Parse** the file (CSV or `.xlsx`) using Python — deduplicate by name, keeping the earliest timestamp.
2. **Separate** Staff records (those with `'no encontrada'` in clave and `'ACREDITACIONES'` in iglesia).
3. **Compute** metrics: total, iglesia count, average entry time, 10-min blocks, 30-min slots, grade distribution. Save to `/tmp/datos_dashboard.json`.
4. **Report duplicates** to the user before building the HTML.
5. **Build** the HTML using Python string concatenation (NOT f-strings — JS `{}` braces conflict). Use `function()` + string concatenation in all JS callbacks — never arrow functions or template literals (V8 parser bug with large DATA).
6. **Validate JS** with `node --check /tmp/validate.js` before saving the file.
7. **Save** to the corresponding zone folder with naming pattern `dashboard_asistencia_<zona>_<mes><año>_<dd>abr.html`.
8. **Update `index.html`** — add the new entry and increment the report count for that zone.

## HTML Dashboard Structure (required sections in order)

1. Header — eyebrow (zone), title with date, pills (asistentes + iglesias), meta (inicio/fin), print button
2. Metrics cards — Total · Iglesias · Promedio entrada · Bloque pico
3. Chart: 10-min blocks — **always mixed type** (bar + line); bars colored by time zone, golden moving-average line (window=3)
4. Chart: 30-min slots — bar only
5. Chart: Grado distribution — donut (Coro/Miembro/Staff)
6. Iglesias list — clickable rows that expand to show person table (Nombre, Hora, Grado)
7. Credencial de Staff — gold section at the bottom
8. Print section — `@media print` expands all iglesia tables

## Key Rules

- **Deduplication**: always deduplicate before any processing — one name = one record (earliest). Report removed duplicates to the user.
- **Time thresholds**: retardo ≥ `inicio + 60 min`, tarde ≥ `inicio + 120 min`. Ask the user for the start time; default to 15:00 if not provided.
- **Colors**: use the CSS variables defined in `README.md` — greens (`--gd`, `--gm`, `--gl`) and gold (`--go`, `--gol`). Gradient accent line: `linear-gradient(90deg, #1b5e35, #2e7d52, #c9a227)`.
- **`Hrs` label**: only on metric cards, header meta, chart legends/tooltips — never in person-row Hora column.
- **Sentence case** throughout the UI (Spanish natural writing), except column labels, siglas (PDF, STAFF), and proper names.
- **CDN**: only `cdnjs.cloudflare.com` for Chart.js 4.4.1 (`chart.umd.js`). File must work offline after first load.
- **No manipulation**: data is read-only — no `contenteditable`, no `localStorage`, no editable inputs.
- **JS style**: always `function()` + string concatenation — never arrow functions or template literals anywhere in `<script>`. V8 fails silently with template literals when DATA is large (>~10 KB).
- **Validate JS**: run `node --check` on the extracted JS before saving the final HTML.
- **Always update `index.html`**: add the new entry at the top of the zone's list and increment the report count.

## Folder Convention

Crear una carpeta por distrito. Nombrarla con el número o nombre del distrito:

```
distrito-XX/   → un folder por cada distrito de Hidalgo
```

File naming: `dashboard_asistencia_distrito<XX>_hgo_<mes><año>_<dd><mes>.html`

## Delivery

After generating, mention hosting options:
- **Netlify Drop** (`netlify.com/drop`) — fastest, no account needed (expires 24 h without account)
- **GitHub Pages** — recommended when the user will publish multiple dashboards across zones
