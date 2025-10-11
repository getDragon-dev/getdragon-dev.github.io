---
title: "Command Reference"
draft: false
---

# 🧰 Dragon CLI — Command Reference

This page lists all available commands, flags, and usage examples for the **Dragon CLI**.  
Use `dragon help [command]` at any time for quick inline help.

---

## 🐉 Overview

The Dragon CLI manages blueprints, registries, and Go project generation.  
It supports both **local** and **remote** registries and provides powerful templating tools with **Sprig** functions and interactive prompts.

```bash
dragon [command] [flags]
```

## ⚙️ Global Flags

|Flag|Description|Default|
|----|-----------|-------|
|--registry|Path or URL to a specific registry.json|https://getdragon.dev/registry.json|
|--help|Show help for any command|—|
|--version|Print the CLI version|—|

## 🏗️ Commands
### 🔹 dragon list

List blueprints available in your configured registry or across all registries.

```bash
dragon list
dragon list --all
dragon list --tag go
dragon list --json
```

#### Options

|Flag|Description|
|----|-----------|
|--all|Show blueprints from all registries|
|--tag `<tag>`|Filter results by tag|
|--json|Output raw JSON|

### 🔹 dragon search

Fuzzy search blueprints by name, description, or tags.

```bash
dragon search --query api
dragon search --query cli --all
```

#### Options

|Flag|Description|
|----|-----------|
|--query|Search term|
|--all|Search across all registries|

### 🔹 dragon gen

Generate a new Go project from a blueprint.
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

#### Options

|Flag|Description|
|----|-----------|
|-b, --blueprint|Blueprint name (required)|
|-o, --out|Output directory (default .)|
|--router|Router type (chi, gorilla, httprouter, servemux)|
|--db|Database driver (sqlite-gorm, postgres-native, etc.)|
|--version|Semantic version constraint|
|--remote|Fetch blueprint ZIP from release asset|
|--set|Set a variable manually (e.g. --set Module=github.com/me/app)|
|--vars|Load variables from a YAML/JSON file|
|--interactive|Prompt for missing variables|

### 🔹 dragon info

Display detailed metadata for a blueprint.
```bash
dragon info api-service
```

#### Options

|Flag|Description|
|----|-----------|
|--registry| Registry URL or path (optional)|

### 🔹 dragon publish

Publish a new or updated blueprint.
```bash
dragon publish
```

What it does

- Tags a new semantic version (vX.Y.Z)
- Generates or updates a changelog
- Uploads blueprint ZIPs as release assets
- Triggers registry update events

### 🔹 dragon release

Automated release management.
```bash
dragon release --auto
```

#### Options

|Flag|Description|
|----|-----------|
|--auto|    Automatically bump version and regenerate registry|
|--skip-tests|	Skip pre-release checks|

### 🔹 dragon get

Fetch and apply an existing blueprint.
```bash
dragon get api-service
```

#### Options

Flag	Description
--registry	Registry URL or path override

### 🔹 dragon registry

Manage multiple registries.

List registries
```bash
dragon registry list
```
Use a new registry (auto-add + set default)
```bash
dragon registry use https://getdragon.dev/registry.json
```
Add or remove registries manually
```bash
dragon registry add --name local --url ~/dev/dragon-registry/registry.json
dragon registry remove local
```
Change default
```bash
dragon registry set-default public
```
Control lookup order
```bash
dragon registry order set --names public,local,staging
```

#### 🔹 dragon completion

Generate shell completions (bash, zsh, fish, powershell).

# Bash
```bash
dragon completion bash > /etc/bash_completion.d/dragon
```
# Zsh
```bash
dragon completion zsh > "${fpath[1]}/_dragon" && autoload -U compinit && compinit
```
# Fish
```baash
dragon completion fish > ~/.config/fish/completions/dragon.fish
```
# PowerShell
```bash
dragon completion powershell | Out-String | Invoke-Expression
```

These completions include dynamic blueprint suggestions (from your active registry).

### 💡 Tips

Local dev:
Use `--registry ../dragon-registry/registry.json` to test unpublished blueprints.

CI/CD:
Combine `dragon publish` and `dragon release --auto` for a full automation pipeline.

Debug:
Run with `--verbose` for additional logs (if you enabled debug mode in config).

### 🧩 Environment Variables
|Variable|Purpose|
|---|---|
|DRAGON_CONFIG|	Override default config path|
|GITHUB_TOKEN|	Used for publishing releases and updating registries|
|XDG_CONFIG_HOME|	Base path for Dragon CLI config (~/.config/dragon/config.json fallback)|

### 🔗 Related

👉 [Getting Started Guide](https://getdragon.dev/guide)

👉 [Registry Browser](https://getdragon.dev/registry/)

👉 [Source on GitHub](https://github.com/getDragon-dev/dragon-cli)