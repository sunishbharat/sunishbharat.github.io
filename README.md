# Sunish Bharathan - Portfolio Website

Personal portfolio and blog of Sunish Bharathan, Technical Program Manager specializing in Engineering Delivery and AI Systems.

**Live site:** [sunishbharat.github.io](https://sunishbharat.github.io)

## Stack

- [MkDocs Material](https://squidfunk.github.io/mkdocs-material/) - static site framework
- GitHub Actions - automated deployment to GitHub Pages on every push to `main`
- Custom CSS (`docs/stylesheets/`) - white/charcoal/blue palette matching the original design

## Local Development

### Option 1: Docker (recommended for Windows)

```bash
chmod +x start_server.sh
./start_server.sh
```

### Option 2: Local Python

```bash
pip install "mkdocs-material[imaging]"

# macOS - install system deps
brew install cairo freetype libffi libjpeg libpng zlib pngquant

mkdocs serve
```

Visit `http://localhost:8000`.

## Deployment

Pushing to `main` triggers the GitHub Actions workflow (`.github/workflows/deploy.yml`) which builds the site and deploys it to the `gh-pages` branch automatically.

## Content Structure

```
docs/
├── index.md              # Homepage with hero, skills, project previews
├── portfolio/            # Project case studies
│   └── projects/
├── blog/                 # Blog posts (MkDocs Material blog plugin)
│   └── posts/
└── assets/               # Images, favicon
```
