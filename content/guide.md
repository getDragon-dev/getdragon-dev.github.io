---
title: "Getting Started Guide"
draft: false
---

# 🧭 DragonDev — Getting Started

Welcome to the **DragonDev** guide!  
This document walks you through installing, configuring, and using the **Dragon CLI** to create, publish, and manage Go-based project blueprints.

---

## 🚀 1️⃣ Install the CLI

### From source
```bash
go install github.com/getDragon-dev/dragon-cli/cmd/dragon@latest
```

Or clone for local development
```bash
git clone https://github.com/getDragon-dev/dragon-cli.git
cd dragon-cli
go build ./cmd/dragon
./dragon
```

## 🧩 2️⃣ Core Commands
### 🔧 Registry Management

Dragon supports multiple registries, local or remote.
You can list, add, remove, or set defaults — or just use a URL directly.

```bash
# list all configured registries
dragon registry list

# use a remote registry (auto-adds and sets as default)
dragon registry use https://getdragon.dev/registry.json

# add a local registry
dragon registry add --name local --url ~/dev/dragon-registry/registry.json

# control lookup order
dragon registry order set --names public,local
```

## 🔍 Discover Blueprints

Search or list available blueprints across all registries.
```bash
# list blueprints from default registry
dragon list

# list all registries and show where each blueprint comes from
dragon list --all

# filter by tag
dragon list --tag go

# fuzzy search (name, description, tags)
dragon search --query cli --all
```

## 🛠️ 3️⃣ Create Your First Project

Generate a new Go project from a published blueprint.

```bash
dragon gen \
  -b api-service \
  -o mysvc \
  --router chi \
  --db postgres-gorm \
  --remote \
  --version ">=1.0.0" \
  --set Module=github.com/acme/mysvc \
  --interactive
```

### What happens:

- Pulls the api-service blueprint (from your active registry or the public one at getdragon.dev)
- Downloads and extracts the template (if --remote is set)
- Prompts for missing variables interactively
- Renders all files using Go’s text/template + Sprig functions
- Resulting structure:
```
mysvc/
 ├── cmd/mysvc/main.go
 ├── internal/
 │   ├── config/config.go
 │   ├── db/open.go
 │   └── server/handler.go
 ├── migrations/
 ├── go.mod
 ├── Dockerfile
 └── atlas.hcl
```

## 📦 4️⃣ Publish a Blueprint

When your blueprint is ready to share, run:
```bash
dragon publish
```

This will:

- Tag a new semantic version (e.g. v1.0.0)
- Generate or update a changelog
- Package the blueprint and update the registry
- Push everything to your configured GitHub repository

Your CI/CD can also handle this via the built-in release flow.

## ⚙️ 5️⃣ Consume a Blueprint

Apply a published blueprint directly from the registry:
```bash
dragon get api-service
```

By default, this fetches from https://getdragon.dev/registry.json .

## 💻 6️⃣ Automation & CI

Integrate dragon into your CI pipelines (GitHub Actions, Drone, etc.):
```bash
dragon release --auto
```

This command bumps versions, regenerates the changelog, and updates registry.json automatically.

## 🧠 Pro Tips

- Interactive mode: --interactive prompts you for template variables.
- Dynamic completions: dragon gen -b <TAB> will suggest available blueprints.
- Local development: run with --registry ../dragon-registry/registry.json to test unpublished blueprints.
- Multi-registry search: dragon list --all aggregates across all configured registries.

## 🔗 Next Steps

👉 [Browse the Registry](https://getdragon.dev/registry/)

👉 [Download CLI Source](https://github.com/getDragon-dev/dragon-cli)

👉 [Contribute Blueprints](https://github.com/getDragon-dev/dragon-blueprints)