---
title: Project Layout Guideline
date: 2026-11-18T18:45:00-05:00
genre: sheet
category: comp
tags:
  - se
lang: en
---

Guidelines for organizing project folders, symlinks, and gitignore rules.

<!-- more -->

## Folder Organization

- The repository root `<repo>/` SHOULD be kept minimal and only include configurations and project metadata (like `pyproject.toml`, `.gitignore`, `AGENTS.md`, etc.).
- We SHOULD use folders to separate source/ops, device-local workflow/docs, and artifacts.
- Follow each language’s preferred layout for code, such as `<repo>/src/`.
- Use `<repo>/scripts/` for ops scripts, such as launching jobs or preparing backups.
- Use `<repo>/instructions/` for ops documentation, such as compute resources or style guidelines. `AGENTS.md` MAY link here.
- Use `<repo>/data/` to store stable and important artifacts, while `<repo>/var/` is for fast-changing iteration.
- Some `var/` subfolder examples are shown below. Also applicable to `data/`.
  - `var/figures/` and `var/tables/` for display items.
  - `var/logs/` for logs.
  - `var/plans/` for design plans and reviews, `var/scribble/` for one-off experimental scripts and artifacts. Keep their children’s names timestamped with `YYYYMMDD-` prefix.
- For complex projects, use `<project>/<repo>` structure.
  - `<project>` and `<repo>` MAY share the same name.
  - Put `data/`, `var/` and other non-repo contents (such as reference projects or academic writings) under `<project>`, NOT `<repo>`.
  - Symlink `data/` and `var/` relatively from `<project>/<repo>` (e.g. `var -> ../var/`), and symlink other non-repo contents from `var/` (e.g. `var/references -> ../references`, or `var/references -> ../../references` if `var/` is a real directory under `<repo>`).
  - `<project>/data/` and `<project>/var/` MAY be symlinks to the device’s designated location for large file storage.

## Version Control

- `<repo>` MUST be Git-tracked.
- `<repo>/var/` MUST be gitignored. If `<repo>/data/` is not gitignored, it MUST use Git LFS.
- Files with `.local.` infix (such as `AGENTS.local.md`) MUST be gitignored.
- Device-specific configurations, scripts, and secrets (such as `.env`) MUST be gitignored. A sanitized `.example` infix or suffix (such as `.env.example`) MAY be used as a tracked template.
- Editors, tools, and language toolchain specific rules MAY use `github/gitignore` GitHub repo as gitignore reference.
- Use absolute path pattern such as `/data` and `/var/scribble` to prevent accidentally ignoring `src/` subfolders.
