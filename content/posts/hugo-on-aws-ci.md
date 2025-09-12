---
title: "Building a Production-Ready Hugo Pipeline on AWS with GitHub Actions"
date: 2025-09-12
draft: false
summary: "How I built a secure, low-cost, zero-servers pipeline: Hugo → GitHub Actions → S3 → CloudFront with Porkbun DNS, ACM TLS, least-privilege IAM, and edge functions."
tags: ["hugo","aws","cloudfront","s3","github-actions","dns","devops","portfolio"]
---

## Overview & Rationale
I built this portfolio/blog as a **production-ready static platform from day one—** Hugo for content, **GitHub Actions for CI/CD**, a **private S3 origin**, and **CloudFront** for global delivery and TLS via **ACM**. The pipeline emphasizes low operational overhead, OIDC-based deploys (no long-lived keys), and a **locked-down origin via OAC**, so it’s suitable for real work, not just demos.

**Highlights**
- Zero servers to patch; everything is static and cached at the edge
- Automatic deploys on `git push`
- TLS for `glsngr.xyz` and `www.glsngr.xyz`
- Edge logic for pretty URLs and canonical redirects
- Tight IAM + private S3; CloudFront is the only public surface

---

## Architecture at a glance

```
Hugo (source in git)
   │  (push)
   ▼
GitHub Actions (build on runner)
   │  hugo --minify
   │  aws s3 sync public/ s3://<bucket> --delete
   │  aws cloudfront create-invalidation /* 
   ▼
Amazon S3 (private bucket, origin)
   ▲     ▲
   │     └─ CloudFront Origin Access Control (OAC)
   │
Amazon CloudFront (CDN + TLS via ACM)
   │
Porkbun DNS (ALIAS apex → CloudFront, CNAME www → apex)
```

---

## Goals and constraints
- **Simplicity:** one repo, one workflow, one clickless deploy path
- **Security:** no public S3, no access keys in repo, least-privilege IAM
- **Performance:** global CDN, Brotli/Gzip compression, HTTP/2+HTTP/3
- **Cost awareness:** static hosting with CDN; DNS kept at Porkbun

---

## The final stack
- **Hugo** (extended) for content and theming
- **GitHub Actions** for CI/CD (OIDC to assume AWS role—no stored secrets)
- **S3** as a **private** origin (no website endpoint)
- **CloudFront** in front of S3 (TLS, caching, compression, HTTP→HTTPS)
- **ACM** certificate in `us-east-1` for `glsngr.xyz` and `*.glsngr.xyz`
- **Porkbun DNS**: ALIAS (apex) → CloudFront, CNAME `www` → apex
- **CloudFront Function** (viewer request) for:
  - `www` → apex canonical 301
  - pretty URLs (`/path` → `/path/index.html`)

---

## Implementation notes (the short version)

### 1) TLS
- Request ACM cert **in us-east-1** for `glsngr.xyz` and `*.glsngr.xyz` (DNS validation).
- Attach the cert to the CloudFront distribution and list both hostnames under **Alternate domain names**.

### 2) DNS (Porkbun)
- **ALIAS** (blank host) → `dxxxxx.cloudfront.net`
- **CNAME** `www` → `glsngr.xyz`
- Keep ACM validation **CNAME(s)** for auto-renewal.

### 3) CloudFront behavior (key toggles)
- **Viewer protocol policy:** Redirect HTTP → HTTPS  
- **Compress objects automatically:** On  
- **Default root object:** `index.html`  
- **HTTP versions:** enable `HTTP/2 and HTTP/3`

### 4) Security posture
- **S3 bucket:** private; block public access on; no public bucket policy
- **OAC** from CloudFront to S3 (bucket policy allows `cloudfront.amazonaws.com` with the distribution ARN condition)
- **Deploy role via GitHub OIDC** scoped to my repo/branch; permissions limited to:
  - `s3:ListBucket` on the bucket
  - `s3:PutObject/DeleteObject` on `bucket/*`
  - `cloudfront:CreateInvalidation` on the distribution

---

## CI/CD workflow (GitHub Actions)
I deploy on every push to `main`. The runner installs Hugo (extended), builds the site, mirrors `public/` to S3, then invalidates CloudFront.

```yaml
name: Deploy Hugo to S3 + CloudFront

on:
  push:
    branches: [ main ]
  workflow_dispatch:

permissions:
  id-token: write
  contents: read

env:
  AWS_REGION: us-east-1
  S3_BUCKET: <YOUR_BUCKET_NAME>
  CF_DISTRIBUTION_ID: <YOUR_DISTRIBUTION_ID>
  ROLE_ARN: arn:aws:iam::<ACCOUNT_ID>:role/GitHubActionsDeployRole

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    concurrency:
      group: hugo-deploy
      cancel-in-progress: true

    steps:
      - name: Checkout (with submodules)
        uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Setup Hugo (extended)
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.150.0'
          extended: true

      - name: Build site
        run: hugo --minify

      - name: Configure AWS credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-region: ${{ env.AWS_REGION }}
          role-to-assume: ${{ env.ROLE_ARN }}

      - name: Sync to S3 (mirror)
        run: aws s3 sync public/ "s3://${{ env.S3_BUCKET }}/" --delete

      - name: Invalidate CloudFront
        run: aws cloudfront create-invalidation --distribution-id "${{ env.CF_DISTRIBUTION_ID }}" --paths "/*"
```

**Why this is safe:** the `ROLE_ARN` is not a secret; the OIDC **trust policy** only allows tokens from **my** repo/branch to assume the role. No long-lived AWS keys anywhere.

---

## Edge logic (CloudFront Function)
I merged canonical redirect + pretty URLs into one **viewer request** function:

```js
function handler(event) {
  var req = event.request;
  var host = req.headers.host ? req.headers.host.value : '';

  // 1) Canonical: www → apex
  if (host === 'www.glsngr.xyz') {
    return {
      statusCode: 301,
      statusDescription: 'Moved Permanently',
      headers: { location: { value: 'https://glsngr.xyz' + req.uri } }
    };
  }

  // 2) Pretty URLs (/path → /path/index.html)
  var u = req.uri;
  if (u.endsWith('/')) req.uri = u + 'index.html';
  else if (!u.includes('.')) req.uri = u + '/index.html';

  return req;
}
```

Attach it to the **Default behavior → Viewer request**.

---

## Local authoring flow
- Create a post:
  ```bash
  hugo new posts/my-new-post.md
  ```
- Draft locally:
  ```bash
  hugo server -D
  # open http://localhost:1313
  ```
- Publish: set `draft: false`, then:
  ```bash
  git add -A && git commit -m "post: my-new-post" && git push
  ```

**Images:** place under `static/img/` and reference as `/img/…`.

---

## Troubleshooting and Resolutions
- **Deprecated config**: `paginate` → `[pagination].pagerSize`
- **Homepage as leaf**: rename `content/index.md` → `content/_index.md`
- **403 in prod**: OAC/bucket policy mismatch or files not synced
- **`www` NXDOMAIN**: add `CNAME www → apex` at Porkbun
- **“No cert found”**: ACM cert must be in `us-east-1` and cover the apex (wildcard alone doesn’t)

---

## Security hardening checklist
- Private S3; enable **Block Public Access**
- OAC-only read path from CloudFront to S3
- Least-privilege IAM on the deploy role; OIDC trust scoped to `repo:OWNER/REPO:ref:refs/heads/main` (and optionally `repository_id`)
- CloudFront **Response headers policy** with:
  - `Strict-Transport-Security: max-age=15552000; includeSubDomains`
  - `X-Content-Type-Options: nosniff`
  - `Referrer-Policy: strict-origin-when-cross-origin`
  - `Content-Security-Policy` tuned to the theme

---

## Why this approach
Compared to a dynamic app stack, this is **simpler, faster, and cheaper** for a portfolio/blog. Versus all-in-one hosts, I keep full control (IAM, headers, edge logic) while still enjoying CDN-level performance and a one-command authoring loop.

**Next steps**
- Add a 404 page and map it in CloudFront Error pages
- PR preview environments (upload to `/preview/PR-###/`)
- Light analytics (e.g., Plausible) and a “Projects” section with cards

---

*Repo notes for readers:* Hugo config lives in `hugo.toml`; content under `content/`; static files under `static/`. CI workflow is `.github/workflows/deploy.yml`. The domain is `glsngr.xyz`, with Porkbun handling DNS to a CloudFront distribution fronting a private S3 origin.
