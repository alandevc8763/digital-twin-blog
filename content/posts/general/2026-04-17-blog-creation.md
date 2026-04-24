---
title: "2026 04 17 Blog Creation"
date: 2026-04-24
categories: ["General"]
tags: ["General", "Plans", "2026-04-17-Blog-Creation.Md"]
draft: false
---

# Digital Twin Blog Implementation Plan

> **For Hermes:** Use subagent-driven-development skill to implement this plan task-by-task.

**Goal:** Build a professional, structured knowledge base (MkDocs Material) that acts as a digital twin for Alan, hosted on GitHub Pages, featuring an "Elite" visual style and rewritten high-value content.

**Architecture:**
- Static Site Generator: MkDocs with Material theme.
- Visuals: Custom CSS for "Gold-on-Slate" aesthetic (Dark slate backgrounds, gold accents).
- Content Pipeline: Extraction from `sz9751210.github.io` $\rightarrow$ Rewrite into "Elite" format $\rightarrow$ Markdown files.
- Deployment: GitHub Pages via GitHub Actions.

**Tech Stack:**
- Python 3.11+
- mkdocs, mkdocs-material
- GitHub Actions
- Custom CSS

---

## Phase 1: Infrastructure & Base Setup

### Task 1: Initialize Project Directory
**Objective:** Create basic folder structure.
- Create: `~/digital-twin-blog/docs`
- Create: `~/digital-twin-blog/overrides/main.css`

### Task 2: Install Dependencies
**Objective:** Ensure all required packages are present.
Run: `pip install mkdocs mkdocs-material`
Expected: SUCCESS

### Task 3: Basic MkDocs Initialization
**Objective:** Generate default config.
Run: `mkdocs new .` in `~/digital-twin-blog`
Expected: `mkdocs.yml` and `docs/index.md` created.

### Task 4: Configure Material Theme
**Objective:** Enable Material theme and dark mode.
Modify: `~/digital-twin-blog/mkdocs.yml`
Add:
```yaml
theme:
  name: material
  palette:
    - scheme: slate
      primary: gold
      accent: gold
  features:
    - navigation.tabs
    - navigation.sections
    - content.code.copy
```

### Task 5: Implement "Gold-on-Slate" Visuals
**Objective:** Apply custom elite styling.
Modify: `~/digital-twin-blog/mkdocs.yml` to include:
```yaml
extra_css:
  - overrides/main.css
```
Modify: `~/digital-twin-blog/overrides/main.css`
Add:
```css
:root {
  --md-primary-fg-color: #D4AF37; /* Gold */
  --md-accent-fg-color: #D4AF37;
}
.md-header {
  background-color: #1a1a1a;
}
```

---

## Phase 2: Taxonomy & Structure Design

### Task 6: Define Navigation Taxonomy
**Objective:** Create a logical structure based on the user's professional focus.
Modify: `~/digital-twin-blog/mkdocs.yml`
Add `nav` section:
- Cloud Infrastructure (AWS, MSK, IAM)
- Database Engineering (MongoDB, Optimization)
- Automation & Tooling (Python, Telegram Bots)
- Digital Twin Journey (Reflection, Vibe Coding)

### Task 7: Create Folder Structure
**Objective:** Mirror navigation in the filesystem.
Create directories:
- `docs/cloud_infra/`
- `docs/db_eng/`
- `docs/automation/`
- `docs/journey/`

---

## Phase 3: Content Engineering (The Digital Twin Part)

### Task 8: Source Content Extraction
**Objective:** Extract posts from `sz9751210.github.io`.
Action: Use browser to read posts $\rightarrow$ save raw text to `docs/raw/`.

### Task 9: Rewrite Content to "Elite" Format
**Objective:** Transform raw posts into high-value knowledge assets.
Template:
- **Taxonomy**: (Domain/Signal)
- **Core Insight**: (The "So What?")
- **Implementation/Details**: (Technical walkthrough)
- **Synergy**: (Relation to other topics)
Files: `docs/[category]/[post_name].md`

### Task 10: Create Homepage (The Digital Twin Identity)
**Objective:** Define the "Digital Twin" persona.
Modify: `docs/index.md`
Content: Professional summary, current trajectory, and link to the a-priori brainstorming.

---

## Phase 4: Deployment Pipeline

### Task 11: Initialize GitHub Repository
**Objective:** Create the remote host.
Run: `git init`, `git add .`, `git commit -m "init: digital twin blog"`
Action: Create repo `digital-twin-blog` via GH CLI/API and push.

### Task 12: Setup GitHub Actions for Deployment
**Objective:** Automate the build and deploy process.
Create: `.github/workflows/deploy.yml`
Content: MkDocs-Material standard deployment workflow to `gh-pages` branch.

### Task 13: Final Verification
**Objective:** End-to-end check.
Run: `mkdocs serve` $\rightarrow$ Check visuals $\rightarrow$ Push to GH $\rightarrow$ Verify live URL.