# Migration Plan: HTML Portfolio to MkDocs Material

Migrating from hand-coded HTML/CSS (archived in `Archive_Html/`) to MkDocs Material. The template is now the repo root.

---

## Overview

| Item | Detail |
|---|---|
| Source | `Archive_Html/` (old HTML/CSS files, reference only) |
| Target | `sunishbharat/` root (MkDocs Material - already in place) |
| Estimated Total Effort | ~2 hours active work |
| Deployment Target | GitHub Pages + custom domain |

---

## Milestone 1 - Setup and Configuration

**Goal:** Get the template running locally with your identity.

**Tasks:**
- [x] Template is already at repo root - no copy needed
- [x] Install dependencies via Docker (`start_server.sh`) or `pip install mkdocs-material[imaging]`
- [x] Verify local dev server runs at `localhost:8000`
- [x] Update `mkdocs.yml`:
  - `site_name`: Sunish Bharathan
  - `site_description`: Technical Program Manager - Engineering Delivery and AI Systems
  - `site_url`: your domain (e.g., `atlasmind.de` or GitHub Pages URL)
  - Social links: GitHub, LinkedIn
  - Remove YouTube / company links not applicable
- [x] Update `docs/CNAME` with your custom domain
- [ ] Replace `docs/assets/profile.jpg` with your own profile photo

**Done when:** Site loads locally showing your name and links.

---

## Milestone 2 - Design and Theming

**Goal:** Match the visual identity of the current site (white, charcoal, blue accent).

**Reference from current `style.css`:**
- Background: `#ffffff`
- Primary text: `#111827`
- Secondary text: `#4b5563`
- Accent: `#3b82f6`
- Borders: `#e5e7eb`

**Tasks:**
- [x] Update `docs/stylesheets/extra.css`:
  - Replace primary color (`#000111`) with `#111827`
  - Replace light accent (`#2856f7`) with `#3b82f6`
  - Remove dark mode color overrides if you want a light-only theme (or keep and adjust)
  - Update footer background to match
- [x] Update `docs/stylesheets/hero.css`:
  - Adjust profile image dimensions to match your photo aspect ratio
  - Keep hover effects (they translate well)
- [x] Update `mkdocs.yml` font: change Roboto to Inter (or keep Roboto - both work)
- [ ] Test light and dark mode toggle visually
- [ ] Test on mobile (resize browser to 375px width)

**Done when:** Color palette and typography match the current site.

---

## Milestone 3 - Homepage Migration

**Goal:** Convert `index.html` content into `docs/index.md`.

**Current sections to migrate:**
- Hero: name, title, short bio, CTA (contact button)
- About / Skills section
- Featured projects preview
- Featured blog articles preview
- Contact section

**Tasks:**
- [x] Write `docs/index.md` hero section using the template's grid HTML block (supported in MkDocs Material via `md_in_html` extension - already enabled)
- [x] Add profile image reference: `![Profile](assets/your-photo.jpg)` (using GitHub avatar as placeholder)
- [x] Port skills list as a Markdown grid or definition list
- [x] Add featured project cards linking to `portfolio/`
- [x] Add featured article cards linking to `blog/`
- [x] Add contact section with email and LinkedIn link
- [x] Add JSON-LD schema markup (Person) in a `<script>` block - MkDocs allows raw HTML
- [x] Update page frontmatter: `title`, `description`, `hide: [toc]`

**Done when:** Homepage renders with hero, skills, project previews, and contact.

---

## Milestone 4 - Projects Migration

**Goal:** Convert `projects.html` into the `docs/portfolio/` section.

**Tasks:**
- [x] Update `docs/portfolio/index.md` with your own project cards
- [x] Create `docs/portfolio/projects/atlasmind.md` (Challenge, approach, results, tech stack, GIF demo, GitHub + live site links)
- [x] Create additional project pages for each project in `projects.html`:
  - `docs/portfolio/projects/gpt2.md`
  - `docs/portfolio/projects/aspice-pipeline.md`
  - `docs/portfolio/projects/can-simulator.md`
  - `docs/portfolio/projects/sband-telemetry.md` (includes M.Sc. research thesis section)
- [x] Update `mkdocs.yml` nav to list all portfolio pages
- [x] Verify project pages link correctly from the portfolio index

**Done when:** All projects from `projects.html` are accessible under `/portfolio/`.

---

## Milestone 5 - Blog Migration

**Goal:** Convert 3 HTML blog posts to Markdown.

**Posts to migrate:**
1. `blog/atlasmind.html` - Why I Built My Own AI PM Assistant
2. `blog/docling-pgvector.html` - RAG Pipeline with Docling and pgvector
3. `blog/gpt2.html` - Implementing GPT-2 from Scratch

**Tasks:**
- [x] Update `docs/blog/.authors.yml` with your name, avatar, and LinkedIn URL
- [x] Create `docs/blog/posts/atlasmind.md` with frontmatter (date: 2026-05-22, categories: [AI, Product Management])
- [x] Create `docs/blog/posts/docling-pgvector.md` (date: 2026-06-05, categories: [AI, Engineering])
- [x] Create `docs/blog/posts/gpt2.md` (date: 2026-03-01, categories: [AI, Deep Learning])
- [ ] Move any blog images into `docs/blog/posts/images/` (GIF served from `assets/` via URL - works fine)
- [x] Verify blog listing page (`/blog/`) auto-populates all three posts
- [x] Check category pages generate correctly

**Done when:** All 3 posts visible at `/blog/` with correct dates, categories, and author.

---

## Milestone 6 - SEO and Metadata

**Goal:** Preserve or improve SEO from the current site.

**Tasks:**
- [x] Verify `docs/robots.txt` is configured (updated sitemap URL, removed asset blocking)
- [x] Confirm social card plugin is enabled in `mkdocs.yml` and generating previews (background color updated to `#111827`)
- [ ] Add `canonical_url` frontmatter to key pages if using a custom domain
- [x] Add Open Graph `description` to blog post frontmatter (handled via `description:` field + social plugin)
- [ ] Set up Google Analytics if desired (commented-out block already in `mkdocs.yml`)
- [x] Check auto-generated `sitemap.xml` after first build (`mkdocs build --strict` passes clean)

**Done when:** Build output includes `sitemap.xml` and social card images for key pages.

---

## Milestone 7 - Deployment

**Goal:** Live site on GitHub Pages with custom domain.

**Tasks:**
- [ ] Create new GitHub repository (e.g., `sunish-portfolio` or reuse current repo)
- [x] Add `.gitignore` entries: `site/`, `.python-version`, `.claude/`
- [x] Create `.github/workflows/deploy.yml` (includes system imaging deps for social cards plugin)
- [ ] Push to `main` and verify GitHub Action runs successfully
- [ ] Enable GitHub Pages in repo settings (source: `gh-pages` branch)
- [ ] Configure custom domain in repo settings and update DNS `CNAME` record
- [ ] Verify HTTPS is active

**Done when:** Site is live at your custom domain with automatic deploys on push.

---

## Milestone 8 - QA and Cleanup

**Goal:** Verify quality and archive the old site.

**Tasks:**
- [x] Check all internal links (MkDocs warns on broken links during build - clean)
- [ ] Test on mobile (iOS Safari, Android Chrome)
- [ ] Test on desktop (Chrome, Firefox, Edge)
- [x] Verify blog post dates and ordering are correct (docling: Jun 5, atlasmind: May 22, gpt2: Mar 1)
- [ ] Check dark mode does not break any custom sections
- [x] Run `mkdocs build --strict` - passes with zero warnings
- [ ] Delete `Archive_Html/` once satisfied with the new site
- [ ] Update LinkedIn and CV with new portfolio URL if domain changed

**Done when:** Zero build warnings, site verified on mobile and desktop, old repo archived.

---

## Summary Timeline

| Milestone | Description | Effort |
|---|---|---|
| 1 | Setup and Configuration | 15 min |
| 2 | Design and Theming | 20 min |
| 3 | Homepage Migration | 25 min |
| 4 | Projects Migration | 20 min |
| 5 | Blog Migration | 30 min |
| 6 | SEO and Metadata | 10 min |
| 7 | Deployment | 15 min |
| 8 | QA and Cleanup | 20 min |
| **Total** | | **~2.5 hours** |

---

## Key Files Reference

| Current site | MkDocs equivalent |
|---|---|
| `index.html` | `docs/index.md` |
| `projects.html` | `docs/portfolio/index.md` + per-project `.md` files |
| `blog/atlasmind.html` | `docs/blog/posts/atlasmind.md` |
| `blog/docling-pgvector.html` | `docs/blog/posts/docling-pgvector.md` |
| `blog/gpt2.html` | `docs/blog/posts/gpt2.md` |
| `style.css` (colors) | `docs/stylesheets/extra.css` |
| `style.css` (hero layout) | `docs/stylesheets/hero.css` |
| Navigation (in every HTML file) | `mkdocs.yml` nav section (one place) |
