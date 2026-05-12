# Repository Overview

## Summary

| Field | Value |
|-------|-------|
| Repository | `c:\Users\nniem\source\repos\PDF4QT` |
| Review date | 2026-05-11 |
| Primary language(s) | C++ (C++20) |
| Framework(s) | Qt 6 (Core, Gui, Widgets, Svg, Xml, PrintSupport, TextToSpeech, Concurrent, Test) |
| Runtime model | Desktop application suite (GUI apps + CLI tool + shared libraries) |
| Package manager(s) | vcpkg (CMake-integrated) |
| Licence (top-level) | MIT (relicensed from LGPLv3 on 2025-04-27) |
| Last commit date | 2026-01-22 (v1.5.3.1 release) |
| Contributors (approx.) | 1 primary author (Jakub Melka) + community contributors |
| Lines of code (approx.) | Large (100+ source files in core library alone) |

## What This Repository Does

PDF4QT is a comprehensive PDF processing suite implementing the PDF 2.0 specification. It provides:

- A C++ shared library (`Pdf4QtLibCore`) for parsing, rendering, and manipulating PDF documents
- A GUI library (`Pdf4QtLibGui`) with viewer/editor main window and dialogs
- A widgets library (`Pdf4QtLibWidgets`) with reusable PDF-specific Qt widgets
- Four end-user GUI applications: advanced document viewer, editor, page organiser (PageMaster), and document diff tool
- A CLI tool (`PdfTool`) for scripted PDF processing
- A plugin system (`Pdf4QtEditorPlugins`) for extending editor functionality
- Developer utilities: `CodeGenerator`, `JBIG2_Viewer`, `PdfExampleGenerator`

## Technology Stack

### Languages

- **C++20** — all production code
- **CMake** — build system
- **QML/UI files** — Qt Designer `.ui` files for dialogs (generated C++ via `uic`)
- **Translations** — Qt `.ts`/`.qm` files (en, de, cs, es, ko, zh_CN, zh_TW, fr, tr, ru)

### Frameworks and Libraries

| Library | Purpose | Licence |
|---------|---------|---------|
| Qt 6 | UI framework, file I/O, concurrency, TTS | LGPL |
| OpenSSL | PDF encryption/decryption, digital signatures, certificate handling | Apache 2.0 |
| zlib | Deflate stream filter | zlib |
| FreeType | Font rasterisation | FTL (FreeType Licence) |
| OpenJPEG | JPEG 2000 image decoding | 2-clause MIT |
| libjpeg-turbo | JPEG image decoding | IJG / BSD-style |
| LittleCMS (lcms2) | ICC colour management | MIT |
| Blend2D | 2D vector rendering backend | zlib |
| libpng | PNG image support | PNG licence |
| Intel TBB | Parallelism on Linux/GCC | Apache 2.0 |

### Build System

CMake 3.16+, with vcpkg for dependency resolution. Supports MSVC (Windows), GCC (Linux), MinGW. AUTOMOC/AUTOUIC/AUTORCC enabled. Wix installer support on Windows.

### Test Framework

Qt Test (`UnitTests/` — `tst_imageoptimizertest.cpp`, `tst_lexicalanalyzertest.cpp`). Coverage is **minimal** (only 2 test files for a large codebase).

### Infrastructure

- Docker (`Dockerfile`) — Ubuntu-based build environment using vcpkg + aqtinstall
- Flatpak (`Flatpak/`, `io.github.JakubMelka.Pdf4qt.json`) — Linux Flatpak packaging
- Wix installer (`WixInstaller/`) — Windows MSI packaging
- AppImage — available via GitHub Releases

## Project Status

| Indicator | Assessment |
|-----------|-----------|
| Active development | Yes — last release 2026-01-22, active issue tracker |
| Recent releases | Yes — v1.5.3.1 (Jan 2026) |
| CI/CD configured | Yes — GitHub Actions (Ubuntu build, Windows install, Linux Flatpak, release draft) |
| Documentation quality | Minimal (README covers features; no API docs found) |
| Test presence | Minimal (2 unit test files) |

## Key Files and Directories

| Path | Purpose |
|------|---------|
| `CMakeLists.txt` | Root build configuration; orchestrates all sub-projects |
| `vcpkg.json` | vcpkg dependency manifest |
| `Pdf4QtLibCore/sources/` | Core PDF engine — parser, renderer, security, fonts, streams |
| `Pdf4QtLibGui/` | Shared GUI layer — main windows, dialogs, settings, TTS |
| `Pdf4QtLibWidgets/` | Reusable PDF widgets (viewer widget, annotation tools) |
| `Pdf4QtEditorPlugins/` | 9 editor plugins (Signature, Redact, Scanner, AudioBook, etc.) |
| `PdfTool/` | CLI application with 30+ sub-commands |
| `Pdf4QtViewer/` | Standalone PDF viewer application |
| `Pdf4QtEditor/` | Full-featured PDF editor application |
| `Pdf4QtPageMaster/` | Page manipulation application |
| `Pdf4QtDiff/` | Document comparison application |
| `.github/workflows/` | CI/CD pipelines |
| `UnitTests/` | Minimal unit test suite |
| `xfa/` | XFA (XML Forms Architecture) static support resources |
| `translations/` | Localisation `.ts` files |

## Dependencies Summary

| Ecosystem | Manifest File | Approx. Count |
|-----------|--------------|---------------|
| vcpkg (C++) | `vcpkg.json` | 9 direct dependencies |
| Qt (system/aqt) | `CMakeLists.txt` | ~10 Qt modules |

## Initial Observations

- The project is a mature, single-author open-source PDF suite with active maintenance.
- Relicensed to MIT in April 2025 — clean permissive licence.
- Heavy use of OpenSSL for cryptographic operations (encryption, signatures, certificates) — this is a significant security-relevant surface area.
- The PDF parser and stream filter subsystem processes untrusted, potentially adversarially crafted binary input files — a classic high-risk attack surface.
- JavaScript scanning capability exists (`pdfjavascriptscanner.cpp`) — PDF JavaScript is a known attack vector.
- Test coverage is extremely sparse relative to codebase size.
- No `SECURITY.md` or vulnerability disclosure process found.

## Areas Requiring Further Review

- [x] Security review needed: PDF parser (`pdfparser.cpp`), stream filters (`pdfstreamfilters.cpp`), JBIG2 decoder (`pdfjbig2decoder.cpp`), XFA engine (`pdfxfaengine.cpp`), JavaScript scanner (`pdfjavascriptscanner.cpp`), security handler (`pdfsecurityhandler.cpp`), certificate manager (`pdfcertificatemanager.cpp`)
- [x] Licence review needed: Third-party libraries bundled in `3rdparty_licenses/` — verify licence compatibility and attribution completeness; Qt LGPL compliance
- [x] Dependency review needed: vcpkg-managed C++ dependencies — check for known CVEs in OpenSSL, zlib, FreeType, OpenJPEG, libjpeg-turbo

## Assumptions

- Repository is the primary source; no separate deployment repository observed.
- Version in `CMakeLists.txt` (1.5.3.1) is the current latest.

## Unknowns

- No `docs/` directory found; no API documentation site identified.
- Contributor count beyond the primary author is unknown without `git shortlog`.
- Total lines of code not computed (no `cloc` run).
