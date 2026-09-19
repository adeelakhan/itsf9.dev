# itsf9.dev

Minimal Astro + Markdown blog for Adeel Khan. Static output, GitHub Pages hosting.

## Local development

Use Node 24 or a compatible newer release.

```sh
npm ci
npm run dev
npm run build
npm run preview
```

## Writing

Create a Markdown file in `src/pages/writing/` with this frontmatter:

```yaml
---
layout: ../../layouts/Post.astro
title: Your article title
description: A short, factual description.
date: '2026-09-20'
draft: false
---
```

Astro excludes Markdown pages marked `draft: true` from production. The writing index also excludes drafts. Keep confidential source material outside this repository, even when an article is a draft. Only add material intended for eventual public release.

## Deployment

The workflow deploys pushes to `main`. Create the GitHub repository, push this directory, then select **Settings → Pages → Source → GitHub Actions**. Set **Custom domain** to `itsf9.dev` in the same settings. The Astro `site` setting and `public/CNAME` already use that domain; no repository base path is needed when serving through the custom domain.

Configure the domain provider's DNS using the records in GitHub's current custom-domain guide. Enable **Enforce HTTPS** after GitHub provisions the certificate. Do not remove unrelated DNS records.

- [Astro deployment guide](https://docs.astro.build/en/guides/deploy/github/)
- [GitHub custom-domain guide](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site)

The full handover and unpublished case-study research are intentionally stored outside this publishable repository.
