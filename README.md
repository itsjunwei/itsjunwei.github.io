# itsjunwei.github.io

Personal academic site. Hand-written HTML and CSS. No framework, no build
step, no npm, no Jekyll, no third-party requests.

## Files

| File | Purpose |
| --- | --- |
| `index.html` | The entire site. One page. |
| `style.css` | Screen (light + dark) and print styles. |
| `favicon.svg` | SVG favicon, adapts to light/dark. |
| `robots.txt` | Allows all crawlers, points to the sitemap. |
| `sitemap.xml` | Single-URL sitemap. Update `<lastmod>` when you edit the page. |
| `.nojekyll` | Tells GitHub Pages to serve the files as-is, with no Jekyll build. |
| `README.md` | This file. Not served as part of the site. |

Nothing is minified and nothing needs to be. Edit the files directly and
push; there is no pipeline between the source and what visitors see.

## Deploying to GitHub Pages

This is a **user site**, so the repository name must be exactly
`itsjunwei.github.io` and Pages serves from the root of the default branch.
No `/docs` folder, no `gh-pages` branch.

### 1. Create the repository

On GitHub, create a new **public** repository named exactly:

```
itsjunwei.github.io
```

Do not add a README, `.gitignore`, or licence from the creation screen —
this directory already has the files.

### 2. Push this directory to the default branch

From inside this folder:

```bash
git init -b main
```

```bash
git add .
```

```bash
git commit -m "Initial site"
```

```bash
git remote add origin https://github.com/itsjunwei/itsjunwei.github.io.git
```

```bash
git push -u origin main
```

### 3. Enable Pages

In the repository: **Settings → Pages**.

- **Source:** "Deploy from a branch"
- **Branch:** `main`, folder `/ (root)`
- Save.

The first build takes a minute or two. The site then lives at
<https://itsjunwei.github.io/>. Every later push to `main` republishes
automatically.

If you would rather not use the branch source, "GitHub Actions" with the
static-HTML starter workflow does the same thing. The branch source is
simpler and needs no workflow file.

### 4. Check it

- Visit <https://itsjunwei.github.io/> in a private window (avoids cache).
- Confirm <https://itsjunwei.github.io/robots.txt> and
  <https://itsjunwei.github.io/sitemap.xml> resolve.
- Submit the site in [Google Search Console](https://search.google.com/search-console)
  if you want indexing sooner.

## Adding a custom domain later

Say you buy `junweiyeow.com`.

1. **At your DNS provider**, for an apex domain (`junweiyeow.com`), create
   four `A` records pointing at GitHub's Pages IPs:

   ```
   185.199.108.153
   185.199.109.153
   185.199.110.153
   185.199.111.153
   ```

   Optionally add the matching `AAAA` records:

   ```
   2606:50c0:8000::153
   2606:50c0:8001::153
   2606:50c0:8002::153
   2606:50c0:8003::153
   ```

   For a `www` subdomain instead, create a single `CNAME` record for `www`
   pointing at `itsjunwei.github.io.` (note the trailing dot). A `CNAME`
   cannot be used at the apex.

2. **On GitHub**, in **Settings → Pages → Custom domain**, enter the domain
   and save. GitHub commits a `CNAME` file to the repo — leave it there.

3. Wait for the DNS check to pass, then tick **Enforce HTTPS**. The
   certificate is issued automatically; it can take up to 24 hours.

4. **Then update these three things in the repo**, because they still point
   at the `github.io` URL:

   - `index.html` — the `<link rel="canonical">`, `og:url`, and the `url`
     field in the JSON-LD block.
   - `robots.txt` — the `Sitemap:` line.
   - `sitemap.xml` — the `<loc>` element.

   Missing this step is the usual cause of duplicate-content warnings after
   a domain move.

## Running it locally

Any static server works. From this directory:

```bash
python -m http.server 8000
```

Then open <http://localhost:8000/>. Opening `index.html` straight from the
filesystem also works, but a server is closer to production and lets
Lighthouse run properly.
