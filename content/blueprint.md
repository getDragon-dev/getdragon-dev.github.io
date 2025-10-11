---
title: "Blueprint Authoring Guide"
draft: false
---

# 🧱 Authoring Blueprints

This guide explains how to **create, structure, and publish** reusable blueprints for Dragon.  
A **blueprint** is a versioned project template — written in Go’s `text/template` syntax — that Dragon can generate locally or fetch remotely.

---

## 🪄 What is a Blueprint?

A blueprint defines:

- The **project structure** and files to generate  
- Template variables (e.g. `{{ .Name }}`, `{{ .Module }}`)  
- Optional metadata (`tags`, `description`, `version`)  
- A simple manifest (`manifest.yaml`) describing it  

Blueprints are stored in the **`dragon-blueprints`** repository and bundled during CI releases to update the public registry.

---

## 🧩 Blueprint Structure

A minimal blueprint lives under `blueprints/<name>/`:

```
blueprints/
└── api-service/
├── manifest.yaml
├── README.md
└── template/
├── go.mod.tmpl
├── cmd/{{.Name}}/main.go.tmpl
├── internal/server/handler.go.tmpl
├── internal/config/config.go.tmpl
├── Dockerfile.tmpl
└── atlas.hcl.tmpl
```


### 🧾 `manifest.yaml`
This file defines your blueprint’s metadata and tags.

```yaml
name: api-service
version: 1.0.0
description: A production-ready REST API service using Chi router and GORM.
tags: [go, api, chi, gorm]
```

#### Fields
| Field         | Description                          |
| ------------- | ------------------------------------ |
| `name`        | Unique identifier for your blueprint |
| `version`     | Semantic version (e.g. `1.0.0`)      |
| `description` | Short description                    |
| `tags`        | Keywords to help users find it       |

### ✨ Template Rendering

Blueprint templates use Go’s text/template engine + Sprig functions.
#### Common variables
| Variable  | Description                                               |
| --------- | --------------------------------------------------------- |
| `.Name`   | The project name (e.g. `mysvc`)                           |
| `.Module` | The Go module path (e.g. `github.com/acme/mysvc`)         |
| `.DB`     | Database backend (`postgres-gorm`, `sqlite-native`, etc.) |
| `.Router` | Router selection (`chi`, `httprouter`, etc.)              |
| `.Now`    | Current timestamp (auto-populated)                        |

You can set additional variables via CLI flags:
```bash
dragon gen -b api-service -o mysvc --set Author="Jane Doe"
```

Sprig examples

Sprig provides dozens of helpful functions:
```
{{- if hasPrefix .DB "postgres" }} // logic for postgres
{{ end }}

package {{ lower .Name }}

{{ default "localhost" .Host }}
```

### 🧠 Dynamic Paths

Dragon also templates file and folder names, so you can do things like:
```
cmd/{{.Name}}/main.go.tmpl → cmd/mysvc/main.go
internal/{{.Name}}_service → internal/mysvc_service
```

### ⚙️ Example Template Snippets

go.mod.tmpl
```
module {{ .Module }}

go 1.25

require (
    github.com/go-chi/chi/v5 v5.1.0
    gorm.io/gorm v1.26.0
)
```

cmd/{{.Name}}/main.go.tmpl
```
package main

import (
    "log"
    "{{ .Module }}/internal/server"
)

func main() {
    log.Println("Starting {{ .Name }} service...")
    server.Start()
}
```

### 🚀 Testing Your Blueprint Locally

You can generate from your working copy without publishing:
```bash
dragon gen -b api-service -o demo --registry ../dragon-registry/registry.json
```

Or test unpublished changes directly from ../dragon-blueprints:
```bash
dragon gen -b api-service -o demo
```

### 🧱 Publishing Your Blueprint

Once your blueprint works locally:

Commit your changes:
```bash
git add blueprints/api-service
git commit -m "update: api-service blueprint"
```

Tag a new version:
```bash
git tag v1.1.0
git push origin v1.1.0
```

GitHub Actions automatically:

- Packages each blueprint into .zip files
- Creates a release for the tag
- Updates the dragon-registry via repository_dispatch
- Deploys the updated registry.json to getdragon.dev

### 🪄 Advanced Features
Variables file

Define your defaults in a file and pass with --vars:

vars.yaml
```yaml
Module: github.com/acme/mysvc
Author: Jane Doe
```
```bash
dragon gen -b api-service --vars vars.yaml
```
Interactive mode

Let Dragon prompt for missing variables:
```bash
dragon gen -b api-service --interactive
```

Remote generation

Skip local templates and use released ZIPs:
```bash
dragon gen -b api-service --remote
```

### 🧩 Registry Integration

Every published blueprint updates the public registry:
```json
{
  "blueprints": [
    {
      "name": "api-service",
      "version": "1.1.0",
      "repo": "github.com/getDragon-dev/dragon-blueprints",
      "path": "blueprints/api-service",
      "download_url": "https://github.com/getDragon-dev/dragon-blueprints/releases/download/v1.1.0/api-service.zip",
      "description": "A production-ready REST API service using Chi router and GORM",
      "tags": ["go", "api", "chi", "gorm"]
    }
  ]
}
```

### 🧰 Useful Commands Recap
- Create from a local repo (dev mode)
```bash
dragon gen -b api-service -o demo
```
- Generate from the published registry
```bash
dragon gen -b api-service -o demo --remote
```
- Validate a blueprint before release
```bash
dragon validate --blueprint api-service
```
- Publish new version
```bash
dragon publish
```

### 🔗 Next Steps

👉 [Getting Started Guide](https://getdragon.dev/guide/)

👉 [Command Reference](https://getdragon.dev/command/)

👉 [Browse Public Registry](https://getdragon.dev/registry/)