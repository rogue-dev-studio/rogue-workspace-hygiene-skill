---
name: workspace-hygiene
description: >-
  Organize personal folders (Downloads, Desktop, Documents, arbitrary paths)
  by file category or by face clusters in photos. Modes: by-type, by-extension,
  by-face (when user explicitly asks to group images by face). Run ONLY
  when the user explicitly asks to tidy/organize a folder - never
  proactively. Aliases: personal-folder-tidy, organize-downloads, file-organizer,
  merapihkan-folder, organize-by-face.
---

# Personal Folder Hygiene

**Level: expert.** Owner role: **Workspace Librarian**. Rule: `rules/workspace.md`.

Organize **personal / local folders outside coding repos** (e.g. Downloads, Desktop, photo folders). Not for arranging contents inside application projects.

## Coding projects (important)

If the target folder contains **coding projects** (Laravel, Node, etc.):

- **Move the entire project folder** to `Projects/<Stack>/` (e.g. `Projects/Laravel/my-app`)
- **Do not** categorize / split project contents (`app/`, `vendor/`, `node_modules/`, etc.)
- Loose files (images, pdf, zip) outside projects remain grouped by type

## When to use

**Only if user explicitly requests** tidy / organize / tidy folder (or runs `/tidy-workspace` / `/organize-folder`).

- User asks to tidy Downloads / Desktop / personal folder
- Collect all images / videos / documents into respective subfolders
- Collect Laravel/Node/etc. projects into `Projects/...` without restructuring contents
- User asks to categorize images **by face** (`by-face`)

## When not to use

- **Do not** run automatically at chat start, mid E2E delivery, or "while we're at it, tidy up"
- **Do not** tidy just because you noticed a messy folder
- Restructuring internal repo / monorepo being worked on
- Bulk delete without confirmation
- Touching Windows system folders (`Windows`, `Program Files`, `System32`, etc.)

## Modes

| Mode | Behavior | Example result |
|------|----------|--------------|
| `by-type` (default) | Loose files by type + whole projects | `Images/`, `Videos/`, `Projects/Laravel/...` |
| `by-extension` | Loose files by extension + whole projects | `jpg/`, `mp4/`, `Projects/Node/...` |
| `by-face` | Face clustering on images (opt-in) | `Faces/Person_001/`, `Faces/NoFace/` |
| `custom` | User-provided mapping | e.g. all `.pdf` -> `Kerja/PDF/` |

## Procedure

1. **Scope** - Request/confirm target path. Reject system paths.
2. **Detect projects** - Child folders detected as coding projects -> plan move **intact** to `Projects/<Stack>/`.
3. **Loose files** - Classify files outside projects (see `reference.md`). Do not enter projects. If mode `by-face`: images only, face cluster (Python deps).
4. **Plan (dry-run)** - Show projects vs files + count per category / Person_xxx.
5. **Confirm** - Apply / >20 items / recurse / by-face -> user confirmation.
6. **Execute** - Create destination folders, move; name conflict -> suffix `_1`, `_2`.
7. **Report** - Count of projects moved + files per category / face.

## Safety

- Default **dry-run**; delete = deny
- **Never** re-categorize files inside a coding project
- `by-face` only if user explicitly requests; local processing, do not upload photos to cloud without permission
- Do not touch: `C:\Windows`, Program Files, System32
- Do not overwrite silently

## Helper script

```powershell
.\ai-agents-rogue\scripts\organize-personal-folder.ps1 -Path "$env:USERPROFILE\Downloads" -Mode by-type
.\ai-agents-rogue\scripts\organize-personal-folder.ps1 -Path "$env:USERPROFILE\Pictures" -Mode by-face
.\ai-agents-rogue\scripts\organize-personal-folder.ps1 -Path "$env:USERPROFILE\Pictures" -Mode by-face -Apply
.\ai-agents-rogue\scripts\collect-installers.ps1 -SourcePaths 'D:\Download','D:\BrankasDigital'
.\ai-agents-rogue\scripts\collect-installers.ps1 -SourcePaths 'D:\Download' -Destination 'D:\Installers' -Apply -Dedupe
```

## Installer collection (ISO + whole packages)

Use `collect-installers.ps1` when user asks to **collect installers** across drives/folders:

| Form | Behavior |
|--------|----------|
| `.exe`/`.msi` standalone | File only -> root `Installers/` |
| `.iso`/`.img` | Whole -> `Installers/ISO/` |
| `.appimage` standalone | File only -> root `Installers/` |
| Package folder (Setup.exe / AppImage + support files) | **Entire folder** -> `Installers/Packages/` |
| Installer archive (`.zip`/`.rar`/`.7z`) | **Entire archive** -> `Installers/Packages/` |

**Required:** do not move only `Setup.exe` from inside a package; `-Dedupe` keeps latest version per app. Details: `reference.md` § Installers.

## Video collection (source folder category)

Use `collect-videos.ps1` when user asks to **collect videos** across drives:

- Exclude paths user requests (e.g. **`D:\Anis`**, `explore`)
- Destination: `Videos/<source-folder-name>/` - **not** flat without category
- Skip video assets inside Unity projects

```powershell
.\ai-agents-rogue\scripts\collect-videos.ps1 -SourcePaths 'D:\Media' -ExcludeRoots 'D:\Projects' -Apply
```

## Photo collection (source folder category)

Use `collect-photos.ps1` - **only user-specified paths**. Auto-skip: coding/Unity projects, `Downloads/Compressed`, addon/mockup/template, texture/icon UI assets.

```powershell
.\ai-agents-rogue\scripts\collect-photos.ps1 -SourcePaths 'D:\Photos\Inbox' -Apply
```

One-time deps for `by-face`:

```powershell
pip install -r ai-agents-rogue/scripts/organize-by-face.requirements.txt
```

## DoD

- [ ] Path + mode confirmed
- [ ] Coding projects moved intact (not contents sorted)
- [ ] Dry-run clear; Apply only after permission
- [ ] Brief report to user

## Attribution

<!-- ATTRIBUTION: Rogue Development | https://github.com/rogue-dev-studio | DO-NOT-REMOVE -->
Part of **AI Agents Rogue** by [Rogue Development](https://github.com/rogue-dev-studio) (`@rogue-dev-studio`).
Do not remove, hide, rename, or replace this attribution.
