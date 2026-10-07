# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Steve Boyer's personal resume site. Static HTML, no build step. Hosted on Netlify; deploys are triggered by pushing to `main` on `github.com:steveboyer/steveboyer.dev`.

Two pages:
- `index.html` — the resume page (everything: hero, experience, projects, apps, skills, contact).
- `blog/index.html` — the `/blog` route. Currently an empty-state placeholder; no post pipeline yet.

Plus the Sift Filter app pages, which App Store Connect links to (keep the URLs stable):
- `sift/support/index.html` and `sift/privacy/index.html`, served at `https://steveboyer.dev/sift/support/` and `https://steveboyer.dev/sift/privacy/` (Steve's choice; `sift.steveboyer.dev` has no DNS). Their shared styles, design tokens included, are in `sift/sift.css`, linked from both pages.
- `_redirects` still holds the old `sift.steveboyer.dev` host rules. They're left in place and do nothing today, but if the subdomain were ever revived, its `/*` → `/sift/:splat` rule would make the `/sift/…` paths 404 there (they'd map to `/sift/sift/…`).

No package manager, no bundler, no test suite, no CI config in-repo.

## Visual language is shared by copying, not by one site-wide stylesheet

The design tokens (`:root` custom properties) and the nav/footer styles from `index.html` are copied into three places:

- `index.html` (its `<style>` block)
- `blog/index.html` (its `<style>` block)
- `sift/sift.css` (linked from both Sift pages, which have no token copy of their own)

A design token change goes into all three. Any new page or stylesheet that copies the tokens joins this list. (Or extract them to a shared `styles.css` linked from every page.)

This is acceptable while the blog is one empty-state page; revisit if posts get added.

## Working in `index.html`

- CSS lives in a single `<style>` block in `<head>`. Sections are delimited by `/* ─── SECTION ─── */` comment dividers. Match that style when adding new sections.
- **Design tokens** are CSS custom properties defined in `:root`: `--bg`, `--bg-card`, `--bg-line`, `--fg`, `--fg-muted`, `--accent` (teal `#00c4cc`), `--font-display` (Sora), `--font-body` (DM Sans), `--max` (1080px content width), `--gutter`. Reuse these rather than hardcoding colors or fonts.
- **Section IDs are linked from the nav** (`#about`, `#experience`, `#projects`, `#apps`, `#skills`, `#contact`) plus `/blog`. If you rename or remove an `id`, update `nav-links` to match.
- **Responsive breakpoints** are at `760px` and `460px`. Test layout changes against both.
- **Preview locally** by opening `index.html` (or `blog/index.html`) directly in a browser. No server needed. The Sift pages are the exception: they link `/sift/sift.css` by a root-relative path, so opened from disk they render unstyled. Preview them through a local server run from the repo root (for example `python3 -m http.server`, then `http://localhost:8000/sift/support/`).

## Dynamic content: Currently Building

The "Currently Building" section is rendered client-side from `content/currently-building.json`. The script at the bottom of `index.html` fetches that file and replaces the contents of `[data-building-list]`.

The static markup inside `[data-building-list]` is a structurally identical fallback. It's what JS-disabled visitors and crawlers see, and what visitors see before the fetch resolves (so there's no layout shift).

**Source of truth is the JSON.** When updating Currently Building, edit `content/currently-building.json`. The static HTML fallback may drift; that's OK for this section (SEO not a concern for a personal status list). Sync the fallback to match when convenient.

To add another dynamic-content section later, follow the same pattern: render a static fallback, fetch JSON from `content/`, swap with `replaceChildren` using `textContent` (not `innerHTML`) so no escaping is needed.

## Resume PDF

`resume.pdf` at the repo root is served at `/resume.pdf`, linked from the Contact section.

**Source of truth is `~/git/resumes`** (separate repo, Typst). The workflow is:
1. Edit `main_servicenow.typ` in that repo.
2. `cp main_servicenow.typ main.typ` (the canonical compile target).
3. `typst compile main.typ SteveBoyer.pdf`.
4. `cp SteveBoyer.pdf ~/git/steveboyer.dev/resume.pdf` and commit here.

Don't try to compile typst from inside this repo or as a Netlify build step.

## Assets

- `icons/` — small per-app PNGs referenced by the Apps section.
- `*Icon.png` at the repo root and `og-image.png` are large source/social assets. Don't rename without updating `<meta property="og:image">` and any `<img src>` references.
- `favicon.svg` referenced from `<head>` on both pages.

## Content edits

When editing resume content in `index.html` (experience bullets, skills, projects), also update the matching summary numbers in the hero/about sections if they change (e.g., "10+ years", "60 apps", "2M+ daily transactions"). They're hand-maintained, not generated.

If a resume change should also flow to the PDF, edit the Typst source in `~/git/resumes` and follow the resume sync flow above.

## Deployment

`git push origin main` triggers a Netlify rebuild and publishes to https://steveboyer.dev. There is no staging environment; verify changes in a local browser before pushing.
