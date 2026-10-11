# CLAUDE.md

## What this repo is

Steve Boyer's personal resume site. Static HTML, no build step. Hosted on Netlify; deploys are triggered by pushing to `main` on `github.com:steveboyer/steveboyer.dev`.

Main pages:
- `index.html` — the resume page (everything: hero, experience, projects, apps, skills, contact).
- `blog/index.html` — the `/blog` route. Currently an empty-state placeholder; no post pipeline yet.

Plus the app pages, which App Store Connect links to (keep the URLs stable):
- Sift Filter: `sift/support/index.html` and `sift/privacy/index.html`, served at `https://steveboyer.dev/sift/support/` and `https://steveboyer.dev/sift/privacy/` (Steve's choice, D094; `sift.steveboyer.dev` has no DNS). App Store Connect's support and privacy URLs must point there.
- Bolt: `bolt/support/index.html` and `bolt/privacy/index.html`, served at `https://steveboyer.dev/bolt/support/` and `https://steveboyer.dev/bolt/privacy/`.
- All four app pages share one stylesheet, design tokens included: `app-pages.css` at the repo root, linked as `/app-pages.css`.
- `_redirects` still holds the old `sift.steveboyer.dev` host rules. They're left in place and do nothing today, but if the subdomain were ever revived, its `/*` → `/sift/:splat` rule would make the `/sift/…` paths 404 there (they'd map to `/sift/sift/…`).

No package manager, no bundler, no test suite, no CI config in-repo.

No issues.md backlog; commit subjects carry no issue ID.

## Visual language is shared by copying, not by one site-wide stylesheet

The design tokens (`:root` custom properties) and the nav/footer styles from `index.html` are copied into three places:

- `index.html` (its `<style>` block)
- `blog/index.html` (its `<style>` block)
- `app-pages.css` (linked from the Sift and Bolt pages, which have no token copy of their own)

A design token change goes into all three. Any new page or stylesheet that copies the tokens joins this list. (Or extract them to a shared `styles.css` linked from every page.)

This is acceptable while the blog is one empty-state page; revisit if posts get added.

## Working in `index.html`

- CSS lives in a single `<style>` block in `<head>`. Sections are delimited by `/* ─── SECTION ─── */` comment dividers. Match that style when adding new sections.
- **Design tokens** are CSS custom properties defined in `:root`: `--bg`, `--bg-card`, `--bg-line`, `--fg`, `--fg-muted`, `--accent` (teal `#00c4cc`), `--font-display` (Sora), `--font-body` (DM Sans), `--max` (1080px content width), `--gutter`. Reuse these rather than hardcoding colors or fonts.
- **Section IDs are linked from the nav** (`#about`, `#experience`, `#projects`, `#apps`, `#skills`, `#contact`) plus `/blog`. If you rename or remove an `id`, update `nav-links` to match.
- **Responsive breakpoints** are at `760px` and `460px`. Test layout changes against both.
- **Preview locally** by opening `index.html` (or `blog/index.html`) directly in a browser. No server needed. The Sift and Bolt pages are the exception: they link `/app-pages.css` by a root-relative path, so opened from disk they render unstyled. Preview them through a local server run from the repo root (for example `python3 -m http.server`, then `http://localhost:8000/sift/support/`).

## Dynamic content: Currently Building

The "Currently Building" section is rendered client-side from `content/currently-building.json`. The script at the bottom of `index.html` fetches that file and replaces the contents of `[data-building-list]`.

The static markup inside `[data-building-list]` is a structurally identical fallback. It's what JS-disabled visitors and crawlers see, and what visitors see before the fetch resolves (so there's no layout shift).

**Source of truth is the JSON.** When updating Currently Building, edit `content/currently-building.json`. The static HTML fallback may drift; that's OK for this section (SEO not a concern for a personal status list). Sync the fallback to match when convenient.

To add another dynamic-content section later, follow the same pattern: render a static fallback, fetch JSON from `content/`, swap with `replaceChildren` using `textContent` (not `innerHTML`) so no escaping is needed.

## Resume PDF

`resume.pdf` at the repo root is served at `/resume.pdf`, linked from the Contact section.

`resume.pdf` is replaced by hand. Its Typst source is not in a known repo; don't go looking for it or try to rebuild the PDF unless Steve asks.

## Assets

- `icons/` — small per-app PNGs referenced by the Apps section.
- `og-image.png` at the repo root is the large social image. Don't rename without updating `<meta property="og:image">` and any `<img src>` references.
- `favicon.svg` referenced from `<head>` on all six pages.

## Content edits

When editing resume content in `index.html` (experience bullets, skills, projects), also update the matching summary numbers in the hero/about sections if they change (e.g., "10+ years", "60 apps", "2M+ daily transactions"). They're hand-maintained, not generated.

## Deployment

`git push origin main` triggers a Netlify rebuild and publishes to https://steveboyer.dev. There is no staging environment. Ask Steve before every push to main (other branches are free). Review and commit first; after the deploy, check the changed pages with curl.
