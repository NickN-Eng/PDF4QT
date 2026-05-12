# Component Inventory

## Repository
`https://github.com/JakubMelka/PDF4QT` (local path: `c:\Users\nniem\source\repos\PDF4QT`)

## Generated
2026-05-11 (manual analysis — Syft/Trivy not available; versions verified via online research)

---

## Full Inventory

> Versions verified via online research (2026-05-11). "Pinned" = value in manifest files; "Latest" = current upstream release.

| # | Component | Pinned Version | Latest (2026-05-11) | Ecosystem | Type | Licence | Supplier | Direct/Trans | Risk | Notes |
|---|-----------|---------------|---------------------|-----------|------|---------|----------|-------------|------|-------|
| 1 | OpenSSL | unspecified | **3.6.2 / 4.0.0** | vcpkg | Library | Apache 2.0 | OpenSSL Software Foundation | D | High | No version pinned. CVE-2025-15467 (HIGH stack overflow), CVE-2026-31789 (heap overflow) in recent releases. Pin to 3.6.2. |
| 2 | LittleCMS (lcms2) | unspecified (vcpkg) / fork c70dfeb (Flatpak) | **2.19.1** | vcpkg / Flatpak | Library | MIT | Marti Maria Saguer | D | High | Flatpak uses fork last updated Dec 2024 (~lcms2.17 era). Missing 2.18 security fixes: heap buffer overflow in `convert_utf16_to_utf32()`, OOB read (issue #522). |
| 3 | zlib | 1.3.1 (Flatpak) / unspecified (vcpkg) | **1.3.2** | vcpkg / Flatpak | Library | zlib/libpng | Jean-loup Gailly & Mark Adler | D | Medium | 1.3.2 (Feb 2026) addresses 7ASecurity audit findings. Update Flatpak. |
| 4 | OpenJPEG | 2.5.2 (Flatpak) / unspecified (vcpkg) | **2.5.4** | vcpkg / Flatpak | Library | BSD 2-Clause | UCLouvain | D | Medium | Flatpak 2 releases behind. Update to 2.5.4. |
| 5 | FreeType | unspecified | **2.14.3** | vcpkg | Library | FreeType License (FTL) | The FreeType Project | D | High | 2.14.3 (Mar 22 2026) and 2.14.2 (Mar 1 2026) both fix potential security issues; "all users should upgrade". Pin to ≥2.14.3. |
| 6 | libjpeg-turbo | unspecified | **3.1.4.1** | vcpkg | Library | IJG / BSD | Independent JPEG Group / libjpeg-turbo Project | D | Low | No recent security CVEs. Active development, latest 3.1.4.1 (Mar 2026). |
| 7 | libpng | unspecified (vcpkg, Win) / system (Linux) | **1.6.58** | vcpkg / system | Library | PNG Reference Library Licence | Glenn Randers-Pehrson et al. | D | **CRITICAL** | Multiple HIGH CVEs: CVE-2026-33416 & CVE-2026-33636 (fixed 1.6.56 Mar 2026), CVE-2026-25646 (fixed 1.6.55 Feb 2026), CVE-2025-64720 & CVE-2025-65018 (fixed 1.6.51 Nov 2025). Pin to ≥1.6.57. |
| 8 | blend2d | commit 6dbc2ce (2025-11-29) | commit 6dbc2ce | vcpkg (overlay) | Library | Zlib | blend2d Project | D | Low | Custom overlay portfile pinned to commit. No CVEs identified. |
| 9 | oneTBB | commit d3b0a80 (Flatpak) / unspecified (vcpkg) | **2023.0.0** | vcpkg / Flatpak | Library | Apache 2.0 | Intel / UXL Foundation | D | Low | Windows: vcpkg; Linux/GCC only. Flatpak commit is from 2022.x era. Latest 2023.0.0. |
| 10 | asmjit | commit c878602 (2025-12-13) | commit c878602 | vcpkg (overlay) | Library | Zlib | asmjit Project | T | Low | Transitive via blend2d `jit`. Pinned to commit. |
| 11 | Qt6 | 6.9.1 | 6.9.1 | Qt installer | Framework | LGPL 3.0 | The Qt Company | D | Low | Modules: Core, Gui, Widgets, Svg, Xml, PrintSupport, TextToSpeech, Concurrent, LinguistTools (build), Test (dev). Dynamically linked. |
| 12 | PDF4QT (this project) | 1.5.3.1 | — | — | Application | MIT | Jakub Melka | — | — | Core library + applications. Relicensed MIT on 2025-04-27. |

---

## Components by Type

### Runtime Application Dependencies

| Component | Version | Ecosystem | Licence | Direct |
|-----------|---------|-----------|---------|--------|
| OpenSSL | unspecified | vcpkg | Apache 2.0 | Yes |
| LittleCMS (lcms2) | unspecified | vcpkg | MIT | Yes |
| zlib | 1.3.1 | vcpkg | zlib/libpng | Yes |
| OpenJPEG | 2.5.2 | vcpkg | BSD 2-Clause | Yes |
| FreeType | unspecified | vcpkg | FreeType License (FTL) | Yes |
| libjpeg-turbo | unspecified | vcpkg | IJG | Yes |
| libpng | unspecified | vcpkg / system | PNG Reference Library | Yes |
| blend2d | commit 6dbc2ce | vcpkg (overlay) | Zlib | Yes |
| asmjit | commit c878602 | vcpkg (overlay) | Zlib | No (via blend2d) |
| Qt6 (Core, Gui, Widgets, Svg, Xml, PrintSupport, TextToSpeech, Concurrent) | 6.9.1 | Qt installer | LGPL 3.0 | Yes |
| oneTBB | unspecified | vcpkg | Apache 2.0 | Yes (Linux/GCC only) |

### Development / Test Dependencies

| Component | Version | Ecosystem | Licence | Purpose |
|-----------|---------|-----------|---------|---------|
| Qt6::Test | 6.9.1 | Qt installer | LGPL 3.0 | Unit testing framework (UnitTests target) |
| Qt6::LinguistTools | 6.9.1 | Qt installer | LGPL 3.0 | Translation file generation at build time |

### Build / CI Dependencies (not shipped)

| Component | Version | Ecosystem | Licence | Purpose |
|-----------|---------|-----------|---------|---------|
| CMake | ≥3.16 | System / installer | BSD 3-Clause | Build system |
| vcpkg | (from git) | git clone | MIT | C++ package manager |
| aqtinstall | (pip) | PyPI | MIT | Qt installation helper (Dockerfile) |
| Ninja | — | apt / installer | Apache 2.0 | Build executor |

### Container / OS Dependencies (Dockerfile — not inventoried in detail)

| Component | Version | Source | Licence |
|-----------|---------|--------|---------|
| ubuntu:22.04 base image | 22.04 | Docker Hub | Various (GPL/MIT/etc.) |
| libxcb-* / libGL / libfontconfig / libpulse | OS-provided | Ubuntu apt | Various |

### Flatpak Platform Dependencies (Linux Flatpak distribution)

| Component | Version | Source | Licence |
|-----------|---------|--------|---------|
| org.kde.Platform | 6.9 (stable) | Flatpak runtime | LGPL / GPL |
| org.kde.Sdk | 6.9 (stable) | Flatpak SDK | LGPL / GPL |

---

## Vendored / Embedded Components

| Component | Location | Version | Licence | Source |
|-----------|----------|---------|---------|--------|
| blend2d (custom overlay) | `vcpkg/overlays/general/blend2d/` | commit 6dbc2ce (2025-11-29) | Zlib | https://github.com/blend2d/blend2d |
| asmjit (custom overlay) | `vcpkg/overlays/general/asmjit/` | commit c878602 (2025-12-13) | Zlib | https://github.com/asmjit/asmjit |
| libpng stub overlay (Linux) | `vcpkg/overlays/linux/libpng/` | stub (system library) | N/A | System-provided |
| xfa-spec.xml | `xfa/xfa-spec.xml` | — | Unknown | XFA specification data |

---

## Components Requiring Manual Review

| Component | Reason | Priority |
|-----------|--------|----------|
| **libpng** | CRITICAL — HIGH-severity CVEs in versions below 1.6.57. Exact vcpkg baseline version unknown; must pin to ≥1.6.57. | **Critical** |
| **LittleCMS (Flatpak fork)** | Fork (`JakubMelka/Little-CMS`) last commit Dec 2024 (~lcms2.17 era). Missing lcms2.18 (Jan 2026) security fixes (heap overflow, OOB read). Upstream now at 2.19.1. | **High** |
| **OpenSSL** | No version pinned; recent HIGH-severity CVEs in the 3.x line. Pin to 3.6.2. | **High** |
| **FreeType** | No version pinned; 2.14.2 and 2.14.3 (Mar 2026) both fix potential security issues — "all users should upgrade". | **High** |
| Qt6 (all modules) | LGPL 3.0 — dynamic vs. static linking matters for licence obligations. Commercial distribution may require Qt commercial licence if static linking is used. | Medium |
| zlib (Flatpak) | 1.3.1 used; 1.3.2 includes security audit fixes (Feb 2026). | Medium |
| OpenJPEG (Flatpak) | 2.5.2 used; 2.5.3/2.5.4 released since (Sep 2025). | Medium |
| xfa/xfa-spec.xml | Licence/provenance of embedded XFA specification XML is unclear. | Low |
| libpng (Linux) | System-provided via stub overlay — version may vary by distro; must be ≥1.6.57. | High |

---

## Statistics

| Metric | Value |
|--------|-------|
| Total runtime components | 11 |
| Unique licences | 8 (Apache 2.0, MIT, zlib/libpng, Zlib, BSD 2-Clause, FTL, IJG, PNG Reference, LGPL 3.0) |
| Components with confirmed pinned version | 5 (zlib 1.3.1 Flatpak, openjpeg 2.5.2 Flatpak, blend2d commit 6dbc2ce, asmjit commit c878602, Qt6 6.9.1) |
| Components with unconfirmed version (vcpkg baseline) | 6 (openssl, lcms2, freetype, libjpeg-turbo, libpng, oneTBB) |
| Components with known current CVEs | 3 (libpng CRITICAL, OpenSSL High, FreeType High) |
| Components with copyleft licence | 1 (Qt6 — LGPL 3.0) |
| Vendored/overlay components | 3 (blend2d, asmjit, libpng stub) |
| Components needing immediate action | 4 (libpng, LittleCMS fork, OpenSSL, FreeType) |
