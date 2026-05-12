# AMZ Prep Proposal Builder

A single-page web app that converts a presale doc + a few form inputs into a fully formatted AMZ Prep rate-card proposal (.xlsx). Built around the warehouse-specific template structure used by the proposal team.

[Live demo: enable GitHub Pages on this repo and visit your-username.github.io/this-repo](#enabling-github-pages)

## What it does

- **Upload a presale.** Drop a .txt / .pdf / .docx and the dashboard auto-extracts client name, warehouse, and volume hints (DTC orders, FBA cases, pallets, etc.) using regex heuristics.
- **Pick a warehouse.** The warehouse selection drives which rate-card template gets generated. Five templates are baked in: Dothan / Qualfon / Upstate (standard), K&A / Shipmate, Selery, UNIS, ES+.
- **Toggle services & quantities.** Each template surfaces only the services its warehouse offers. The dashboard previews which FR tier rate applies as you type quantities.
- **Override rates (optional).** Custom per-client pricing when Goku has approved a deviation.
- **Generate.** Click the button and the .xlsx downloads. Three tabs: Fulfillment Rates, Sample Invoice Calculator, Zone Mapping — all formatted to match the reference templates (blue 5B95F9 headers, alternating row fills, merged section bars, tier IF formulas in the calc).
- **Export JSON brief.** As an alternative output, you can export everything as a JSON brief for downstream tooling or for Claude to consume.

No backend. Everything runs in the browser. Hosted on GitHub Pages for free.

## Repo layout

```
.
├── index.html                          ← the whole app, single file
├── README.md                           ← this file
├── LICENSE                             ← MIT
├── .gitignore
└── .github/
    └── workflows/
        └── pages.yml                   ← (optional) auto-deploy to Pages
```

Future enhancements live in `assets/` (zone-map PNGs per warehouse, etc.) — see the [Roadmap](#roadmap) below.

## Quickstart — push this to your GitHub

1. **Create a new GitHub repo.** Name it whatever you like — `amzprep-proposal-builder` is a fine default. Don't initialize with a README; we already have one.

2. **From this folder, in a terminal:**

   ```bash
   git init
   git add .
   git commit -m "Initial commit: AMZ Prep proposal builder"
   git branch -M main
   git remote add origin https://github.com/YOUR-USERNAME/YOUR-REPO.git
   git push -u origin main
   ```

3. **Enable GitHub Pages.**
   - Go to **Settings → Pages** in the repo.
   - Under "Build and deployment", set **Source** to "Deploy from a branch".
   - Set **Branch** to `main` and folder to `/ (root)`. Save.
   - Wait ~30 seconds. GitHub will show the URL: `https://YOUR-USERNAME.github.io/YOUR-REPO/`

That's it. Open the URL and the dashboard is live.

### Enabling GitHub Pages

If you'd rather use the included GitHub Actions workflow (`.github/workflows/pages.yml`) instead of the branch-based deploy, you can also:

- Settings → Pages → Source: **GitHub Actions**.
- On every push to `main`, the workflow will deploy your site.

Either option works. The branch-based deploy is simpler if you don't plan to add a build step.

## How the templates work

The dashboard ships with five embedded warehouse templates, extracted from the canonical .xlsx files used by the proposal team:

| Template | Warehouses it covers | FR rows | Calc lines |
|---|---|---|---|
| Dothan / Qualfon / Upstate (Standard) | Dothan (AL), Qualfon LV/MI/PA, Upstate (SC) | 42 | 22 |
| K&A / Shipmate | Kitting & Assembly (CO), Shipmate (Philadelphia) | 28 | 16 |
| Selery | Selery California / Chicago / Dallas / Orlando / Utah | 42 | 22 |
| UNIS | UNIS | 48 | 0 (custom — calc skipped) |
| ES+ | Black Mountain (Las Vegas) | 28 | 20 |

Each template defines: the FR section structure (DTC / Storage / FBA / B2B / Inbound / Other), per-section rate rows with default prices, and the mapping between calc inputs (J column) and FR rows (C column) including tier IF chains. Look in `index.html` under `window.AMZ_TEMPLATES` if you want to inspect the raw data.

## Differences from the Apr 23 framework spec

The actual warehouse templates differ from the Apr 23 framework in a few important ways. This dashboard follows the **actual templates** as source of truth:

- **DTC rates are 4 explicit tier rows** (R9-R12), not a single R9 that gets overridden per client. Same for FBA and B2B case picking.
- **FNSKU is also 5 explicit tier rows** (R23-R27), not an inline IF formula in the calc.
- **Calc tab uses cols C-F** for the calculator and **I-J** for inputs (not F-K + M-N as the framework describes).
- **No Bubble-Wrap / Oversize 50-100 lbs** in the Dothan template — only `DTC Fulfillment (10-50 lbs)` at the standard rate.
- **No Inline FNSKU formula** — the calc uses a tiered IF that references the explicit FR tier rows.
- **UNIS** has its own bespoke rate structure (weight-based DTC, container-based inbound, accessorial schedule) — the dashboard renders the FR but skips the calc since there's no canonical mapping.

The blue header color (`5B95F9`), alternating row fills (`F3F3F3` / `D9D9D9`), and section logic still match.

## What's intentionally simplified vs. the openpyxl tooling

This is a v1, in-browser generator. A few things the Python tooling does that this doesn't yet:

1. **Zone map images.** The Zone Mapping tab currently only places the logo. To add per-warehouse map images, drop PNGs into `assets/maps/` and extend `buildZMSheet()` in `index.html` to fetch the right one based on the warehouse choice.
2. **Carrier rate card tabs.** The dashboard doesn't ingest UPS / USPS / SpeedX rate card files. Easiest path: generate the core proposal here, then drop in the carrier tabs manually in Excel (or extend the generator).
3. **QC border audit.** The Python framework prescribes a rigorous border audit pass after generation. The in-browser generator builds the borders directly so audit failures shouldn't happen, but if you spot one, file an issue with the .xlsx attached.

## Customizing

- **Update default prices.** Edit `window.AMZ_TEMPLATES` in `index.html`. Each template entry has a `frRows` array where you can change `price` values. Commit and push — the live dashboard picks them up.
- **Add a new template variant.** Add a new key under `window.AMZ_TEMPLATES`, copy the structure of an existing one (e.g. `dothan`), and update `frRows`, `calcInputs`, and `mappings`. The warehouse dropdown is auto-built from the union of all warehouses across all templates.
- **Tweak the auto-parse heuristics.** See `parseHintsFromText()` in `index.html`. Patterns are regex-based and conservative — extend them as you see common presale formats.

## Local development

You can just open `index.html` in a browser — there's no build step. For convenience:

```bash
# Python 3
python3 -m http.server 8000
# then open http://localhost:8000
```

Or use any static file server you like.

## Roadmap

- [ ] Per-warehouse zone map images via `assets/maps/`
- [ ] Carrier rate card tab import (drag-drop .xlsx → append sheets)
- [ ] UNIS calc support
- [ ] PDF export of the rendered proposal
- [ ] Save / reload drafts via local storage

## License

MIT — see `LICENSE`.

## Credits

Built from the Apr 23 2026 AMZ Prep Proposal Builder framework and the warehouse-specific xlsx templates.
