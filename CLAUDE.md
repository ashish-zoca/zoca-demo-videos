# zoca-demo-videos

Static, single-page hosting for the Zoca platform demo: an email-gated gallery of 13 feature walkthrough videos. No build step, no framework, no package manager — just HTML, inline CSS/JS, and media assets served as-is.

## What's in the repo

- `index.html` — the entire demo site. Hero, three sections (Network / Discovery / AI Agents), `<video>` players with poster fallbacks, and an email-gate overlay. All CSS and JS are inline; no external bundles.
- `viewer-log.html` — admin-only page. Reads `localStorage.zoca_viewers` and renders a table with CSV export. Local-only data; not the source of truth.
- `videos/*.mp4` — the 13 walkthrough videos referenced from `index.html`. Tracked via Git LFS for GitHub Pages deploy.
- `posters/*.jpg` — poster frames shown before each video plays. Filenames match the corresponding `videos/*.mp4`.
- `vercel.json` — `{"public": true}`. Vercel deploy as a public static site.
- `.netlify/netlify.toml` — Netlify config. Note: `publish` currently points to another contributor's absolute path; only relevant if you redeploy on Netlify.
- `.github/workflows/deploy.yml` — GitHub Pages deploy on push to `main` (checks out with LFS, uploads the repo root as the artifact).

## Email gate & viewer logging

`index.html` writes the entered email to `localStorage` (`zoca_demo_email`, 7-day TTL) and POSTs `{ email, timestamp, userAgent, referrer }` to a Google Apps Script webhook (`SHEET_WEBHOOK` constant near the bottom of `index.html`) using `mode: 'no-cors'`. The gate is **not** a security boundary — it's lead capture. Anyone can bypass it via devtools or by hitting the video URLs directly. Don't put anything truly private in `videos/`.

## Practices

**Edit `index.html` directly.** No templating, no components. When you add a video, copy an existing `.video-item` block and change the `<source>`, `poster`, `<h3>`, `.dur`, and `.video-desc`. Resist refactoring this into a framework — the whole point is zero-build.

**Keep CSS and JS inline.** A single self-contained HTML file deploys cleanly to GitHub Pages, Vercel, and Netlify with no config drift. If a snippet feels worth extracting, ask whether it's actually reused before splitting it out.

**Filename convention for media.** Videos and posters share the same stem: `feature_<slug>.mp4` ↔ `posters/feature_<slug>.jpg` (or the numbered `featureN_<slug>` variants). When adding a new feature, add both files in the same commit and reuse the stem — the HTML relies on this 1:1 pairing.

**Reuse the existing section pattern.** Three sections, each with a `.section-header` (number + title + description) and either a `.video-featured` (full width) or `.video-grid` / `.video-grid.cols-3` block. New features should slot into one of the existing sections rather than introduce a fourth — if you genuinely need a new section, copy the `<!-- SECTION N -->` block verbatim and bump the number.

**Durations in `.dur` are hand-entered.** Update them when re-rendering a video; nothing reads the actual file metadata.

**Git LFS for `videos/`.** MP4s are tracked via LFS so GitHub Pages can serve them. When cloning, `git lfs install && git lfs pull`. When adding a new video, confirm `.gitattributes` covers `*.mp4` before committing.

**Don't commit `.DS_Store` or `.netlify/state.json` changes** unless you're intentionally rewiring the Netlify site.

**The video generation pipeline lives in another repo.** Recent commits (`Fix audio/video sync with per-segment audio generation`, `Add animated cursor overlay`) refer to that pipeline; the rendered MP4s land here. Don't try to regenerate videos from this repo.

**No secrets in `index.html`.** The Google Apps Script webhook URL is public by design (the page ships to the browser). Treat anything in this repo as world-readable.

## Run locally

No build, no install. Serve the directory over HTTP (videos won't play reliably from `file://`):

```bash
# from the repo root
python3 -m http.server 8000
# then open http://localhost:8000/
```

Any static server works (`npx serve .`, `php -S localhost:8000`, etc.). To inspect logged viewers from the current browser, open `http://localhost:8000/viewer-log.html`.

## Deploy

Push to `main` — `.github/workflows/deploy.yml` publishes to GitHub Pages with LFS. Vercel/Netlify also auto-deploy if their integrations are still wired up.
