# DocNavigator

A static web preview that helps a field tech (or anyone non-technical) find the relevant detail on a reference drawing without reading the whole drawing set.

The user picks a system, picks one of ten common questions, and the answer is shown as a **cropped image taken straight from the reference drawing** — never as paraphrased text. If the question doesn't apply to that system, a short note explains why. If a question isn't in the preview yet, a placeholder points to working examples.

## Design principle

The tool does not interpret the drawing. It points to the part of the drawing that contains the answer. The reader's eyes do the rest. This keeps the original document — not the tool — as the source of truth.

## What's in this preview

- **9 example answers** (cropped from the reference drawing PDFs)
- **1 universal answer** for "Can we use these drawings for construction?"
- **12 "doesn't apply" notes** (e.g., MRI systems don't have an X-ray-in-use light)
- The remaining slots show a *"not in preview"* placeholder with one-click links to the examples that do work

## Example scenarios

| # | System              | Question                                           | Sheet  |
|---|---------------------|----------------------------------------------------|--------|
| 1 | SOMATOM GO.TOP      | What is the floor levelness requirement?           | S-101  |
| 2 | NAEOTOM Alpha       | What size breaker is needed?                       | E-102  |
| 3 | LUMINOS Agile       | What is the ceiling height requirement?            | A-101  |
| 4 | (any)               | Can we use these drawings for construction?        | —      |
| 5 | MAGNETOM Sola       | How much does the equipment weigh?                 | A-101  |
| 6 | YSIO XPree Gen2     | What size wiring do I need?                        | E-102  |
| 7 | ARTIS ICONO Biplane | How does an X-ray-in-use light get connected?      | E-501  |
| 8 | LUMINOS Agile       | What are the environmental requirements?           | A-101  |
| 9 | ARTIS Zee MP        | What is the levelness requirement of the unistrut? | S-102  |
| 10| MAGNETOM Sola       | How do the EPOs connect?                           | E-102  |

## File layout

```
deploy/
├── index.html              # single-page app, data inline
├── vercel.json             # security headers + image caching (Vercel only)
├── README.md
└── data/
    └── crops/
        ├── artis_zee_mp/unistrut_levelness.png
        ├── icono_biplane/xray_light.png
        ├── luminos_agile/ceiling_height.png
        ├── luminos_agile/environmental.png
        ├── magnetom_sola/epo_connection.png
        ├── magnetom_sola/equipment_weight.png
        ├── naeotom_alpha/breaker_size.png
        ├── somatom_gotop/floor_levelness.png
        └── ysio_xpree/wiring_size.png
```

No build step. No `package.json`. No server. No API key.

## Deploy to Vercel

```bash
cd deploy
git init
git add .
git commit -m "Initial preview"
# create the repo on github.com, then:
git remote add origin git@github.com:<you>/docnavigator.git
git push -u origin main
```

Then on vercel.com:
1. **New Project** → import the repo
2. **Framework preset**: Other
3. **Root directory**: `./`
4. **Build / Output / Install**: leave blank
5. **Deploy**

No environment variables. First deploy is ~10 seconds.

## Deploy to GitHub Pages

Same `git push` as above, then in the repo on github.com:
1. **Settings** → **Pages**
2. **Source**: Deploy from a branch
3. **Branch**: `main`, folder `/ (root)`
4. **Save**

Live at `https://<you>.github.io/docnavigator/` within about a minute.

## Adding more questions

For each new (system, question) pair you want to wire up:

1. Identify the relevant box on the source drawing (PDF).
2. Crop it to `data/crops/<system_id>/<question_id>.png`.
3. Add one line to the `SNIPPETS` object in `index.html`:
   ```js
   "<system_id>|<question_id>": { image: "data/crops/<system_id>/<question_id>.png", sheet: "X-XXX", page: N },
   ```
4. Commit, push. Vercel / GitHub Pages redeploys automatically.

No re-indexing, no rebuild. The lookup is constant-time and entirely client-side.

## Browser support

Built with system fonts and standard DOM — no framework, no external font load. Works on any modern browser (last ~3 years of Chrome / Safari / Edge / Firefox). Mobile-friendly layout falls back to a single column under 900px wide.

## Limits & honesty

- This is a preview. It covers 10 example questions, not all 80 possible combinations.
- The example images are cropped from the published reference drawings — those drawings carry their own titleblocks and notes, which appear in the crops as part of the source. The tool does not add or remove anything from the drawing artwork.
- The tool never speaks for the drawing. If the cropped area looks wrong or incomplete, that is a signal that the cropping needs adjusting, not that the answer is being paraphrased.
