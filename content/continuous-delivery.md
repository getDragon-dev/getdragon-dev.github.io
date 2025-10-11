---
title: "Continuous Delivery with GitHub Actions"
draft: false
---

# ⚙️ Continuous Delivery with GitHub Actions

The Dragon ecosystem is designed for **automated releases** — when you publish a new blueprint, your registry and website update themselves automatically.  

This guide explains how that works, and how you can adapt it for your own organization.

---

## 🧩 Dragon Repositories Overview

| Repository | Purpose | Deployment |
|-------------|----------|-------------|
| **[`dragon-blueprints`](https://github.com/getDragon-dev/dragon-blueprints)** | Contains all published blueprints and their manifests | Creates ZIPs and triggers registry updates |
| **[`dragon-registry`](https://github.com/getDragon-dev/dragon-registry)** | Stores `registry.json` with all blueprint metadata | Updated automatically from releases |
| **[`dragon-website`](https://github.com/getDragon-dev/dragon-website)** | Hugo static site (this site!) | Deployed to **GitHub Pages** at [getdragon.dev](https://getdragon.dev) |

---

## 🚀 Workflow Overview

When you **tag a new version** in `dragon-blueprints`, three things happen automatically:

1. A new GitHub Release is created  
2. Blueprint ZIPs are generated and attached  
3. The `dragon-registry` is notified to rebuild its `registry.json`  
4. The registry copy is also pushed to `dragon-website/static/registry.json`  
5. The website deploys and your new blueprint appears instantly at  
   [getdragon.dev/registry/](https://getdragon.dev/registry/)

---

## 🧱 1️⃣ Blueprints Release Workflow

In **`dragon-blueprints/.github/workflows/release.yml`**:

```yaml
name: Release Blueprints
on:
  push:
    tags:
      - 'v*.*.*'

permissions:
  contents: write

jobs:
  package:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Zip blueprints
        run: |
          mkdir -p dist
          for bp in blueprints/*; do
            name=$(basename "$bp")
            (cd "$bp" && zip -r "../../dist/${name}.zip" template manifest.yaml README.md)
          done
      - name: Create Release
        uses: softprops/action-gh-release@v2
        with:
          files: dist/*.zip
      - name: Notify registry
        uses: peter-evans/repository-dispatch@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          repository: getDragon-dev/dragon-registry
          event-type: dragon-blueprint-released
          client-payload: '{"tag": "${{ github.ref_name }}"}'
```

### 🔍 What it does

- Zips each blueprint folder
- Publishes those ZIPs as release assets
- Sends a webhook (repository_dispatch) to the registry repo to rebuild

## 🧮 2️⃣ Registry Update Workflow

In dragon-registry/.github/workflows/update.yml:
```yaml
name: Update Registry
on:
  repository_dispatch:
    types: [dragon-blueprint-released]

permissions:
  contents: write

jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: 1.25.2
      - name: Update registry.json
        env:
          TAG: ${{ github.event.client_payload.tag }}
          BLUEPRINTS_REPO: getDragon-dev/dragon-blueprints
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          go run ./update_registry.go
      - name: Commit and push
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add registry.json
          git commit -m "update registry for $TAG"
          git push
      - name: Sync to dragon-website
        uses: peter-evans/repository-dispatch@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          repository: getDragon-dev/dragon-website
          event-type: dragon-registry-updated
```

### 🔍 What it does

- Fetches release metadata via GitHub API
- Extracts each blueprint’s manifest (manifest.yaml)
- Updates registry.json with name, version, tags, description, download URL
- Commits and pushes it
- Triggers dragon-website rebuild

## 🌐 3️⃣ Website Deployment Workflow

In dragon-website/.github/workflows/deploy.yml:
```yaml
name: Deploy Website
on:
  push:
    branches: [main]
  repository_dispatch:
    types: [dragon-registry-updated]

permissions:
  contents: write
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Sync registry.json
        run: |
          curl -sSL https://raw.githubusercontent.com/getDragon-dev/dragon-registry/main/registry.json \
            -o static/registry.json
      - name: Build site
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: '0.134.2'
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

### 🔍 What it does

- Automatically downloads the updated registry file
- Rebuilds the static site using Hugo
- Publishes it to GitHub Pages at https://getdragon.dev

## 🔐 Permissions & Tokens

Each workflow uses the built-in GITHUB_TOKEN (no PAT needed)
Ensure the GitHub App or workflow has:
- contents: write (for commits, releases, deployments)
- repository_dispatch access across repos in the same org

Since all repos belong to the same org (getDragon-dev), the default app token works fine.
🪄 Optional Improvements
### 1️⃣ Draft & pre-release support

You can publish beta blueprints with:
```yaml
release:
  draft: true
  prerelease: true
```

### 2️⃣ Automatic changelog generation

Integrate goreleaser or git-chglog for richer changelogs.

### 3️⃣ Staging registries

Add a staging branch or alternate registry.json and wire it into dragon registry use for pre-production testing.

### 🧠 How Everything Fits Together
```
+---------------------------+
| dragon-blueprints         |
|   ↓ tag v1.2.0            |
|   → creates release .zip  |
|   → triggers registry     |
+------------┬--------------+
             ↓
+---------------------------+
| dragon-registry           |
|   → fetches manifest.yaml |
|   → rebuilds registry.json|
|   → triggers website      |
+------------┬--------------+
             ↓
+---------------------------+
| dragon-website            |
|   → syncs registry.json   |
|   → rebuilds site         |
|   → deploys to Pages      |
+---------------------------+
```
### 🧩 Local Testing

You can simulate this chain locally by setting environment variables:
```bash
export TAG=v1.0.0
export BLUEPRINTS_REPO=getDragon-dev/dragon-blueprints
export GITHUB_TOKEN=<your-token>
go run ./update_registry.go
```

Then manually copy registry.json to your website repo for preview:
```bash
cp registry.json ../dragon-website/static/registry.json
hugo server -D
```
## 🔗 Related Docs

👉 Blueprint Authoring Guide

👉 Getting Started Guide

👉 Command Reference
