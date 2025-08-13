# West Workspace — T2 Topology Template

This repo is a **template** for a T2-style workspace: one root repo that keeps top-level folders visible while sourcing code from external repos.

## Layout

- `manifest-repo/` – west manifests  
- `apps/`          – application projects (ignored by default)  
- `modules/`       – extra modules (ignored by default)  
- `bootloader/`    – bootloader sources (ignored by default)  
- `tools/`         – tooling (ignored by default)  
- `.west/`         – west state/config (ignored by default)  
- `.venv/`         – Python environment (ignored by default)  
- `zephyr/`        – **submodule pointer** to the Zephyr repo (no files tracked here)

> **What’s a submodule pointer?** A **Git** link to an exact commit of another repo. The `zephyr/` folder shows here, but its files live in that repo. To update what this repo points at, bump the submodule pointer and commit.

---

## Getting started

```bash
# If you see `zephyr/` as a submodule after cloning:
git submodule update --init --recursive
```

Use `west` with the manifests under `manifest-repo/`.

---

## Keep folders visible, ignore their contents

Each root folder has a tiny `.gitignore` that keeps the folder while ignoring everything inside:

```gitignore
*
!.gitignore
```

Folders using this pattern: `apps/`, `bootloader/`, `modules/`, `tools/`, `.west/`, `.venv/`.

---

## Track only one app (e.g., `apps/blinky`) while keeping other apps ignored

Edit `apps/.gitignore` to allow `blinky`:

```gitignore
*
!.gitignore
!blinky/
!blinky/**
```

Then stage and commit:

```bash
git add apps/.gitignore
git add -A apps/blinky
git commit -m "Track apps/blinky; keep other apps ignored"
git push
```

---

## Updating the Zephyr submodule pointer (optional)

To move the `zephyr/` pointer to whatever commit you currently have checked out inside `zephyr/`:

```bash
git -C zephyr status           # ensure desired state
git add zephyr                 # stage the submodule pointer change
git commit -m "Bump zephyr submodule pointer"
git push
```

To discard local changes inside the submodule and sync to the recorded pointer:

```bash
git submodule update --init --recursive
```
