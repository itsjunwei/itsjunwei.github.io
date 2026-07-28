# itsjunwei.github.io

Personal academic site. Hand-written HTML and CSS. No framework, no build
step, no npm, no Jekyll, no third-party requests.

## Files

| File | Purpose |
| --- | --- |
| `index.html` | The entire site. One page. |
| `style.css` | Screen (light + dark) and print styles. |
| `fonts/` | IBM Plex Sans, self-hosted. Two `.woff2` files. |
| `cv.pdf` | Linked as "CV (PDF)". Currently a placeholder — replace it. |
| `og-image.png` | 1200×630 social card, referenced by `og:image`. |
| `og-image.svg` | Editable source for the card. Not referenced by the page. |
| `favicon.svg` | SVG favicon, adapts to light/dark. |
| `robots.txt` | Allows all crawlers, points to the sitemap. |
| `sitemap.xml` | Single-URL sitemap. Update `<lastmod>` when you edit the page. |
| `.nojekyll` | Tells GitHub Pages to serve the files as-is, with no Jekyll build. |
| `README.md` | This file. Not served as part of the site. |

Nothing is minified and nothing needs to be. Edit the files directly and
push; there is no pipeline between the source and what visitors see.

Page weight is about 96 KB: 27 KB of markup, styles and favicon, plus 68 KB
of fonts. Images are excluded from that figure.

## Typeface

The site is set in **IBM Plex Sans**, under the
[SIL Open Font License 1.1](https://openfontlicense.org/). It is served from
`fonts/`, not from a font CDN, so the page still makes zero third-party
requests and does not break if an external service goes away.

Two files, both variable across the 100–700 weight axis and subset to Latin,
so one file covers every weight the page uses:

| File | Size |
| --- | --- |
| `fonts/ibm-plex-sans-latin.woff2` | 44.6 KB |
| `fonts/ibm-plex-sans-latin-italic.woff2` | 23.8 KB |

The roman file is preloaded in `<head>`, which needs the `crossorigin`
attribute even though the file is same-origin — fonts are always fetched in
CORS mode, and without it the browser downloads the file twice.

Two things to know before changing this:

- **`ch` is not "characters".** `--measure` is in `ch`, which is the width of
  a `0`, and in IBM Plex Sans the average glyph is only 0.74 of that. The
  measure is set to `54ch` because that lands at roughly 73 characters per
  line. If you swap the typeface, re-measure rather than keeping the number.
- **Two font files is the budget.** Adding a second family, or a separate
  bold file, pushes the page past 100 KB.

To replace the typeface entirely: drop new `.woff2` files into `fonts/`,
update the two `@font-face` blocks and the `--font-body` / `--font-ui`
custom properties at the top of `style.css`, update the preload `<link>` in
`index.html`, and re-check `--measure`.

## Regenerating the social card

`og-image.svg` is the source; `og-image.png` is what the page actually
references. After editing the SVG, serve the directory and screenshot it:

```bash
chrome --headless=new --window-size=1200,630 --force-device-scale-factor=1 --screenshot=og-image.png http://localhost:8000/og-image.svg
```

It has to be served over HTTP. Opened as a `file://` URL, the card's
`@font-face` is blocked by CORS and the PNG comes out in a system font
instead of IBM Plex Sans, with no error.

Social scrapers cache aggressively, so after changing the card re-run the
live URL through the [Facebook Sharing Debugger](https://developers.facebook.com/tools/debug/)
to force a refetch.

## Deploying to GitHub Pages

This is a **user site**, so the repository name must be exactly
`itsjunwei.github.io` and Pages serves from the root of the default branch.
No `/docs` folder, no `gh-pages` branch.

**The local repository is already set up.** `git init`, the first commit, and
the `origin` remote are done. Only steps 1 and 2 below remain, and both need
your GitHub account.

### 1. Create the repository on GitHub

Go to <https://github.com/new> and create a **public** repository named
exactly:

```
itsjunwei.github.io
```

Leave "Add a README", `.gitignore`, and licence **unticked** — this
directory already has the files, and an extra initial commit on GitHub's
side would force you to merge before you can push.

### 2. Push

From inside this folder:

```bash
git push -u origin main
```

Git will prompt for your GitHub credentials the first time. Use a **personal
access token**, not your account password — GitHub stopped accepting
passwords for Git operations in 2021. Create one at
<https://github.com/settings/tokens> with the `repo` scope, then paste it
when prompted for the password. Windows will store it in Credential Manager
so you are asked only once.

If the remote ever needs changing:

```bash
git remote set-url origin https://github.com/itsjunwei/itsjunwei.github.io.git
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

Python is not currently installed on this machine. If you don't want to
install it, any of these work too — or just double-click `index.html`.

---

# Updating the site

Everything lives in `index.html`. You will not need to touch `style.css`
unless you want to change how something *looks*.

The publish loop is always the same three commands:

```bash
git add .
```

```bash
git commit -m "Add IEEE SPL paper"
```

```bash
git push
```

GitHub Pages rebuilds within about a minute. Hard-refresh with
**Ctrl+Shift+R** if you still see the old version.

## Adding a publication

Find the right `<ol class="pubs">` — there are three, one per section
(`Journal and conference papers`, `Preprints and technical reports`,
`In preparation`). Copy an existing `<li>` and edit it. Entries run
reverse chronologically, newest first, and the `[n]` numbers are generated
by CSS — you never type them and they renumber themselves.

The full shape of an entry:

```html
<li>
  <p class="title"><a href="https://doi.org/10.xxxx/yyyy">Paper Title Here</a></p>
  <p class="authors"><strong>Jun-Wei Yeow</strong>, Ee-Leng Tan, Woon-Seng Gan</p>
  <p class="venue">Journal Name, vol. 1, no. 2, pp. 3–4, 2026.</p>
  <p class="links"><a href="https://arxiv.org/abs/XXXX.XXXXX">arXiv:XXXX.XXXXX</a> · <a href="https://github.com/itsjunwei/REPO">code</a></p>
</li>
```

Rules that keep the list honest:

- `<strong>` goes around **your name only**. That is what bolds it.
- Link the title to the DOI **only once the paper is actually published**.
  No DOI yet? Leave the title as plain text inside `<p class="title">`.
- Anything not yet published needs a visible status label, placed inside
  the `<p class="title">` right after the title text:

  ```html
  <span class="status status-pending">Under review</span>
  ```

  Use `status-pending` for under review / accepted / in preparation, and
  `status-report` for preprints and technical reports. The only difference
  is emphasis.
- Drop the `<p class="links">` line entirely if there is nothing to link.
- Use `–` (en dash) for page and year ranges, not a hyphen.

## When a paper gets published

Four edits, in this order:

1. **Delete the `<span class="status">…</span>`** from the title line.
2. **Wrap the title in the DOI link**, as in the template above.
3. **Update `<p class="venue">`** with the real volume, issue, pages, year.
4. **Add a citation block in `<head>`** so Google Scholar indexes it.

Step 4 is the one that is easy to forget. Copy an existing block and edit.
For a journal paper:

```html
<!-- Journal Name 2026 -->
<meta name="citation_title" content="Paper Title Here">
<meta name="citation_author" content="Yeow, Jun-Wei">
<meta name="citation_author" content="Tan, Ee-Leng">
<meta name="citation_journal_title" content="Journal Name">
<meta name="citation_volume" content="1">
<meta name="citation_issue" content="2">
<meta name="citation_firstpage" content="3">
<meta name="citation_lastpage" content="4">
<meta name="citation_publication_date" content="2026">
<meta name="citation_doi" content="10.xxxx/yyyy">
```

For a conference paper, swap `citation_journal_title` for
`citation_conference_title` and drop volume/issue.

**Only add these blocks for genuinely published papers.** Authors go
`Family, Given`, one tag each, in author order. If you add a block for a
preprint or an accepted-but-unpublished paper, Scholar will index it as
published — which is exactly the confusion the status labels exist to
prevent.

## Other common edits

| To change | Where in `index.html` |
| --- | --- |
| Contact links | `<ul class="contact">` near the top |
| Research summary or thesis title | `<section id="research">` |
| Teaching, education, experience | the matching `<section>`, in `<ul class="dated">` |
| Page title / search snippet | `<title>` and `<meta name="description">` in `<head>` |

A `<ul class="dated">` row looks like this — `detail` is optional:

```html
<li>
  <span class="what">Role or qualification</span>
  <span class="when">Aug 2023 – present</span>
  <span class="detail">Anything extra.</span>
</li>
```

## Hidden sections

Three blocks are commented out in `index.html` rather than deleted:

- Preprints and technical reports
- In preparation
- Other experience (ICASSP 2024 demo, industry internships)

Each is wrapped in a clearly labelled `<!-- HIDDEN … -->` comment that tells
you how to restore it. Restoring is two steps: delete the comment opener and
its matching closer, then re-add the nav link in `<ul>` inside `<nav>` if the
section should appear in the navigation.

Two consequences worth knowing. Commented-out markup still ships to the
browser — it is invisible and unindexed, but anyone reading View Source can
see it, so do not hide anything there that is actually confidential. And the
CSS for those sections (`.status-report`) is still in `style.css`,
deliberately, so the sections render correctly when restored.

A "Selected results" section existed briefly between Research and
Publications but was removed outright (not hidden) at the author's request,
along with its `.results` CSS. It is not recoverable from a comment the way
the sections above are — restoring it means re-adding the markup from git
history (`git log -- index.html`) if it's ever wanted back.

## Still to fill in

- **`github.com/itsjunwei/MAGENTA` returns 404.** There is a `BROKEN LINK`
  comment on that entry. Create the repo or delete the `code` link before
  you launch — it is the only known broken link on the site.
- **MAGENTA has no DOI yet.** It is marked `[TO UPDATE]` in the file. When
  the DOI is issued: delete the status label, link the title, update the
  venue line, and add a `citation_*` block in `<head>`.

No `UNCONFIRMED` markers remain.

## Before you push — a 60-second check

1. Open <http://localhost:8000/> and read the section you changed.
2. **Ctrl+P** and look at the print preview. It should be black on white
   with no navigation bar. This is your CV; it is easy to break and easy to
   forget to look at.
3. Paste the file into <https://validator.w3.org/nu/> (the "Upload" tab).
   It should say zero errors. An unclosed tag can silently swallow the rest
   of the page.
4. Update `<lastmod>` in `sitemap.xml` to today's date.

If you add a new page later, add a matching `<url>` block to `sitemap.xml`.
A single-page site needs nothing else.
