# Hugo + AWS + GitHub Actions

Personal site built with Hugo (Stack theme), deployed to AWS S3 + CloudFront via GitHub Actions (OIDC, no long‑lived keys).

- Static Hugo site with the Stack theme
- Compact homepage layout with left‑hand thumbnails
- Private S3 origin behind CloudFront (OAC), TLS via ACM
- One‑click deploys on push to `main` using OIDC

Live site: https://glsngr.xyz/

## Quick Start

Prerequisites
- Hugo Extended (>= 0.150)
- Git

Clone with submodules (theme):

```
git clone --recurse-submodules https://github.com/<you>/hugo-aws-gh.git
cd hugo-aws-gh
```

Run locally:

```
hugo server -D
# open http://localhost:1313
```

## Content & Images

Create a new post as a page bundle so it can carry its own image:

```
hugo new posts/my-post/index.md
```

Add a preview image next to it and reference in front matter:

```toml
# content/posts/my-post/index.md
---
title: "My Post"
date: 2025-01-01
image: "hero.jpg"   # file lives next to index.md
---
```

Place the image at:
- `content/posts/my-post/hero.jpg`

Homepage thumbnails come from that `image` field (or any Stack‑supported cover).

Sidebar avatar (circle on the left):
- Put a square image at `assets/img/avatar.png` (or .jpg)
- Configure in `hugo.toml` under `[params.sidebar.avatar]`:
  - `local = true`
  - `src = "img/avatar.png"`

## Theme

This repo uses the Stack theme as a submodule with a few site overrides:
- `themes/hugo-theme-stack` (submodule)
- `layouts/index.html` — use compact list on the homepage
- `layouts/partials/article-list/compact.html` — add summary + image first
- `assets/scss/custom.scss` — bump thumbnail size

Update the theme:

```
git submodule update --remote --merge themes/hugo-theme-stack
```

## Deploy (GitHub Actions → S3 + CloudFront)

Workflow: `.github/workflows/deploy.yml`
- Builds the site with Hugo
- Assumes an AWS role via OIDC (no secrets)
- Mirrors `public/` to S3
- Creates a CloudFront invalidation

Environment in workflow (edit as needed):
- `AWS_REGION`
- `S3_BUCKET`
- `CF_DISTRIBUTION_ID`
- `ROLE_ARN`

### AWS setup (summary)
1) ACM certificate in `us-east-1` for your domain(s)
2) CloudFront distribution using that cert
3) S3 bucket (private), set as CloudFront origin
4) Origin Access Control (OAC) on the distribution; bucket policy allows CloudFront via OAC
5) IAM role for GitHub OIDC:
   - Trust policy allows tokens from your repo/branch
   - Permissions:
     - `s3:ListBucket` on the bucket
     - `s3:PutObject`/`s3:DeleteObject` on `bucket/*`
     - `cloudfront:CreateInvalidation` on the distribution
6) DNS A/ALIAS apex → CloudFront, CNAME `www` → apex

Deploy:
- Push to `main` (or trigger `workflow_dispatch`) and the site is built and published.

## Repository Layout

- `content/` — Markdown content (posts, about, etc.)
- `assets/` — SCSS, images processed by Hugo Pipes (e.g., `img/avatar.png`)
- `static/` — Files served as‑is under `/` (optional)
- `layouts/` — Site‑specific template overrides
- `themes/hugo-theme-stack/` — Theme (git submodule)
- `.github/workflows/deploy.yml` — CI/CD pipeline

## Notes

- Build outputs (`public/`) and generated resources (`resources/_gen/`) are git‑ignored.
- If previews don’t update locally, run: `hugo server --disableFastRender` and clear `resources/_gen`.

---

Questions or ideas for improvements are welcome.
