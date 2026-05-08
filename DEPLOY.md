# Deploying to Vercel — UI dashboard, no CLI required

The `deploy/` folder is a fully self-contained static site. Drag-and-drop into Vercel and your manager gets a public URL with the landing page + every corridor viewer.

---

## Step-by-step (UI only, ~3 minutes)

### 1 · Generate the bundle

```bash
python scripts/build_deploy_bundle.py
```

This walks `docs/dry_runs_v3_a/` for every `<CORRIDOR>_html_viewer_<DATE>.html`, copies them into `deploy/` with cleaner names (`<CORRIDOR>_<DATE>.html`), extracts metadata from each, and writes:

- `deploy/index.html` — landing page with one card per corridor
- `deploy/<CORRIDOR>_<DATE>.html` — one per viewer
- `deploy/vercel.json` — clean URLs + cache headers

Re-run this script any time you build a new viewer. It's idempotent.

### 2 · Open Vercel

Go to <https://vercel.com>. Sign in with GitHub / GitLab / email — any account works.

### 3 · Add a new project

Click **Add New… → Project**.

You have two equally easy paths:

- **(a) Drag and drop**: on the import screen, look for the small **"Deploy without a Git repository"** option (Vercel sometimes labels it "Deploy a folder"). Drag the entire `deploy/` folder onto the upload area. That's it.
- **(b) Push to GitHub first** — create a public repo with just the `deploy/` contents as its root, push, then on Vercel pick "Import Git Repository" and select that repo. This path lets you re-deploy by pushing new commits.

### 4 · Configure (literally nothing to configure)

When Vercel asks about framework / build command / output directory:

- **Framework preset**: Other (or "Static / no framework")
- **Build command**: leave empty
- **Output directory**: leave empty (defaults to root)
- **Install command**: leave empty

Vercel detects it's pure HTML and serves it as-is.

### 5 · Click "Deploy"

In ~15 seconds you get a URL like `https://<project-name>.vercel.app`. Share that with your manager.

---

## What your manager will see

- `https://<project-name>.vercel.app/` → landing page with one card per corridor (DEL_AUROBINDO, KOL_B, …)
- Click a card → opens that corridor's interactive viewer
- Everything works offline once the page loads. Just a Google Maps tile fetch.

---

## Updating

Anything change in the source data?

```bash
python scripts/build_html_viewer.py --corridor <ID> --date <YYYY-MM-DD>
python scripts/build_deploy_bundle.py
```

Then either drag-and-drop again **or** if you used the Git path, commit and push — Vercel auto-redeploys.

---

## Custom domain (optional)

In the Vercel project's **Settings → Domains**, add your own domain. Vercel handles HTTPS automatically.

---

## Troubleshooting

**"Map doesn't load on the deployed URL"**
The Google Maps API key is hardcoded in each HTML. By default Google Cloud Console allows it from any referrer. If your manager sees an "RefererNotAllowedMapError", go to <https://console.cloud.google.com/apis/credentials>, edit the key, and add the Vercel URL to the allowed referrers.

**"index.html doesn't show all corridors"**
Re-run `python scripts/build_deploy_bundle.py` after you build new viewers. The landing page is regenerated each time.

**"deploy/ folder is huge"**
Each viewer is ~300–900 KB. With 5 corridors you're looking at ~3 MB total. Vercel's free tier handles this trivially.
