# Personal folder taxonomy (reference)

For the `workspace-hygiene` skill — personal folders (Downloads, Desktop, etc.).

## Coding projects — move intact (do not split contents)

Detect child folders as projects -> move to `Projects/<Stack>/<folder-name>/`.

| Stack | Detection signal |
|-------|----------------|
| Laravel | `artisan` + `composer.json` |
| PHP | `composer.json` (without `artisan`) |
| Node | `package.json` |
| NextJS | `package.json` + `next.config.*` |
| Vue | `package.json` + vue/nuxt config |
| Angular | `angular.json` |
| Python | `pyproject.toml` or `requirements.txt` + app entry |
| Django | `manage.py` |
| DotNet | `*.sln` / `*.csproj` |
| Go | `go.mod` |
| Rust | `Cargo.toml` |
| JVM | `pom.xml` / `build.gradle` |
| Rails | `Gemfile` + `config/application.rb` |
| GitProject | `.git` + (`src`/`lib`/`app`) fallback |

**Required:** do not sort files inside a project (`vendor/`, `node_modules/`, `app/`, ...).

## Mode `by-type` — loose files only

| Category | Folder | Extensions (examples) |
|----------|--------|---------------------------|
| Images | `Images` | `.png` `.jpg` `.jpeg` `.webp` `.gif` `.bmp` `.tif` `.tiff` `.heic` `.svg` `.ico` |
| Videos | `Videos` | `.mp4` `.mkv` `.mov` `.webm` `.avi` `.wmv` `.m4v` |

## Videos — collect by source folder category

When collecting videos across drives (script `collect-videos.ps1`):

| Rule | Behavior |
|--------|----------|
| Explicit exclude | Do not touch paths the user excludes (e.g. `D:\Anis`, `explore`, `UnityProjects`) |
| Category | Move to `Videos/<parent-folder-name>/` — use **the folder where the file lives**, not a flat dump |
| Unity project | Skip videos inside projects (`Assets/` + `ProjectSettings/`) |
| Already flat | File-name heuristics (Telegram, Screen Recordings, Camera/VID_, etc.) when source folder is unknown |

```powershell
.\ai-agents-rogue\scripts\collect-videos.ps1 -SourcePaths 'D:\BrankasDigital','D:\Download' -ExcludeRoots 'D:\ZonaKreatif\explore','D:\Anis' -Apply
```

## Photos — collect by source folder category

Only user-specified paths (e.g. one subfolder or several media folders):

| Rule | Behavior |
|--------|----------|
| Scope | Only source folders the user names — do not recurse a full drive without permission |
| Category | `Photos/<parent-folder-name>/` (folder where the file lives) |
| Exclude | `Anis`, `explore`, `Installers`/`Packages`/`ISO`, `UnityProjects`, Unity projects (`Assets`+`ProjectSettings`), coding projects (`package.json`, `composer.json`, etc.) |
| Skip assets | Unity `Assets/`, `Downloads/Compressed`, addon/extension/mockup/template, `icons`/`demo`/`textures`/`logo` folders in download packages, files with `.meta`, texture maps (diffuse/normal/albedo), mockup previews |
| `.jpg.rigj` | Valid (header+footer JPEG) -> strip `.rigj`. Corrupt -> `Photos/<category>/_corrupt/` |

```powershell
.\ai-agents-rogue\scripts\collect-photos.ps1 -SourcePaths 'D:\BrankasDigital','D:\Foto','D:\Download' -ExcludeRoots 'D:\Anis','D:\ZonaKreatif\explore','D:\Installers' -Apply
.\ai-agents-rogue\scripts\restore-project-photos.ps1 -PhotosRoot 'D:\Photos' -Apply
```
| Audio | `Audio` | `.mp3` `.wav` `.flac` `.aac` `.m4a` `.ogg` `.wma` |
| Documents | `Documents` | `.pdf` `.doc` `.docx` `.xls` `.xlsx` `.ppt` `.pptx` `.txt` `.rtf` `.odt` `.csv` `.md` |
| Archives | `Archives` | `.zip` `.rar` `.7z` `.tar` `.gz` `.bz2` |
| Installers | `Installers` | `.exe` `.msi` `.msix` `.dmg` `.apk` `.iso` `.img` `.appimage` |

## Installers — whole-package rules

When collecting installers (skill `workspace-hygiene`, script `collect-installers.ps1`):

| Form | Behavior | Example destination |
|--------|----------|---------------|
| Standalone file | Move file only | `Installers/CursorUserSetup-x64-3.9.16.exe` |
| ISO / boot image | Move whole file | `Installers/ISO/ubuntu-24.10-desktop-amd64.iso` |
| Package folder | Move **entire folder** — not only `Setup.exe` or `.AppImage` | `Installers/Packages/Navicat Premium 16.1.2 Linux64/` |
| AppImage bundle | `.AppImage` + support files/folders in the same folder -> move whole parent folder | `Installers/Packages/.../` |
| Installer archive | Move **entire** `.zip`/`.rar`/`.7z` clearly meant as installer distribution | `Installers/Packages/flutter_windows_3.32.5-stable.zip` |

**Required:**

- Package detection: `Setup.exe` / `Install.exe` **or** `.AppImage` + support files/folders -> move parent package folder.
- Installer bundle folders (WinRAR, RUFUS, Adobe, IObit, etc.): `ReadMe (How to Install).txt`, `Crack` subfolder, `BlockHost*.cmd`, or large `.rar` -> move **entire folder** even if `Setup.exe` was moved separately before.
- Version dedupe: keep **only the latest version** per app family (compare version numbers + modification date).
- Do not run installers; only `Move-Item`.
- Do not split package contents (keygen/crack stay with folder — user reviews).

**Helper:**

```powershell
.\ai-agents-rogue\scripts\collect-installers.ps1 -SourcePaths 'D:\Download','D:\BrankasDigital'
.\ai-agents-rogue\scripts\collect-installers.ps1 -SourcePaths 'D:\Download' -Apply -Dedupe
```
| Code | `Code` | **loose** code files (not inside a project) |
| Other | `Other` | remainder |

## Mode `by-extension`

For **loose files** only. Projects still go to `Projects/<Stack>/`.

## Mode `by-face` (only if requested)

Cluster images by face (local, DeepFace/Facenet):

| Result | Folder |
|-------|--------|
| Person detected (cluster) | `Faces/Person_001/`, `Person_002/`, ... |
| No face | `Faces/NoFace/` |

- Deps: `pip install -r ai-agents-rogue/scripts/organize-by-face.requirements.txt`
- Script: `organize-by-face.py` via `-Mode by-face`
- Multi-face photos: use largest face as primary label
- Accuracy is not perfect; renaming `Person_xxx` folders to a person's name may be done manually after review

## Depth

| Option | Behavior |
|------|----------|
| Flat (default) | Files at root + project folders at root |
| Recurse | Deeper files, **still skip** coding project contents |

## Forbidden paths

`C:\Windows`, Program Files, raw drive root (`C:\`).
