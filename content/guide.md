---
title: "Getting Started Guide"
draft: false
---

# 🧭 DragonDev Guide

Welcome to the **DragonDev** guide!  
This document walks you through creating, publishing, and managing Go-based project templates.

---

## 1️⃣ Install the CLI

```bash
go install github.com/getDragon-dev/cli@latest
```

Or clone directly for local development:

```bash
git clone https://github.com/getDragon-dev/cli.git
cd cli
go run .
```

## 2️⃣ Create Your First Blueprint
```bash
dragon new my-app
```

This generates a new blueprint folder with Go templates and metadata.

Blueprint structure example:

```
my-app/
 ├── blueprint.yaml
 ├── cmd/
 ├── internal/
 └── README.md
```
## 3️⃣ Publish Your Blueprint

When you’re ready:
```bash
dragon publish
```

This will:

Tag a new semantic version (vX.Y.Z)

Generate a changelog

Update registry.json automatically

Push everything to the configured GitHub repository

## 4️⃣ Consume a Blueprint

Use the registry to pull and apply a blueprint:
```bash
dragon get my-app
```

By default, this fetches from getdragon.dev/registry.json.

## 5️⃣ Customize or Contribute

Add templates under blueprints/

Submit a pull request to getDragon-dev

Update the registry through your CI release workflow

## 💡 Pro Tip

You can automate releases with your CI (GitHub Actions, Drone, etc.) using:
```bash
dragon release --auto
```

This command bumps versions, updates the changelog, and regenerates the registry.

## Next:
👉 [Browse the Registry](https://getdragon.dev/registry.json)

👉 [Visit GitHub](https://github.com/get-dragondev)