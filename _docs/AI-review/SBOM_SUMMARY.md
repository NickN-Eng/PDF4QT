# SBOM Summary

## Repository
`https://github.com/JakubMelka/PDF4QT` (local path: `c:\Users\nniem\source\repos\PDF4QT`)

## Generation Details

| Field | Value |
|-------|-------|
| Generated date | 2026-05-11 |
| Tool(s) used | Manual manifest analysis (Syft/Trivy not installed) |
| Format(s) | N/A — manual analysis; no machine-readable SBOM generated |
| Output files | SBOM_SUMMARY.md, COMPONENT_INVENTORY.md |
| Project version | 1.5.3.1 |
| Ecosystems analysed | C++ / vcpkg, CMake, Flatpak, Docker |

> **Note:** Syft and Trivy were not available in the environment. This SBOM is derived entirely from
> manual analysis of `vcpkg.json`, `vcpkg-configuration.json`, overlay `vcpkg.json` files,
> `CMakeLists.txt`, `Flatpak/io.github.JakubMelka.Pdf4qt.json`, `Dockerfile`, and
> `3rdparty_licenses/`. Machine-generated SBOMs should be produced using Syft or Trivy for
> formal compliance use.

---

## Component Overview

| Metric | Count |
|--------|-------|
| Total components | 12 |
| Direct dependencies (vcpkg.json) | 9 |
| Transitive dependencies | 1 (asmjit, pulled by blend2d) |
| Framework (Qt6 modules) | 9 Qt6 modules grouped as 1 framework entry |
| Components with licence confirmed | 11 |
| Components without licence information | 0 |
| Components with version confirmed | 5 |
| Components with version unconfirmed/baseline-only | 7 |

> **Online research performed:** 2026-05-11. Latest versions and CVE data sourced from official project GitHub release pages and libpng.org.

---

## Ecosystem Breakdown

| Ecosystem | Components | Direct | Transitive | Notes |
|-----------|-----------|--------|------------|-------|
| C++ / vcpkg | 10 | 9 | 1 | openssl, lcms2, zlib, openjpeg, freetype, libjpeg-turbo, libpng, blend2d, tbb, asmjit |
| C++ / Qt6 framework | 9 modules | 1 (framework) | — | Core, Gui, Widgets, Svg, Xml, PrintSupport, TextToSpeech, Concurrent, Test |

---

## Licence Distribution

| Licence | Count | Category | Notes |
|---------|-------|----------|-------|
| MIT | 1 | Permissive | LittleCMS |
| Apache 2.0 | 2 | Permissive | OpenSSL, oneTBB |
| zlib/libpng | 1 | Permissive | zlib |
| Zlib | 2 | Permissive | blend2d, asmjit |
| BSD 2-Clause | 1 | Permissive | OpenJPEG |
| FreeType License (FTL) | 1 | Permissive (FTL or GPLv2) | FreeType |
| IJG (Independent JPEG Group) | 1 | Permissive (custom IJG) | libjpeg / libjpeg-turbo |
| PNG Reference Library Licence | 1 | Permissive | libpng |
| LGPL 3.0 | 1 (framework) | Weak copyleft | Qt6 — 9 modules |

---

## High-Risk Components (Updated with Online Research)

| Component | Latest Version (2026-05-11) | Used / Pinned Version | Severity | Reason | Action |
|-----------|-----------------------------|-----------------------|----------|--------|--------|
| **libpng** | **1.6.58** | unspecified (vcpkg baseline; Linux: system) | **CRITICAL** | Multiple HIGH-severity CVEs in 2025–2026: CVE-2026-34757 (medium, fixed in 1.6.57), CVE-2026-33416 & CVE-2026-33636 (HIGH, fixed 1.6.56 Mar 2026), CVE-2026-25646 (HIGH, fixed 1.6.55 Feb 2026), CVE-2025-66293 (HIGH, fixed 1.6.52), CVE-2025-64720 & CVE-2025-65018 (HIGH, fixed 1.6.51). All affect valid conformant PNG images. | **Pin immediately to ≥1.6.57** via vcpkg `overrides`. Verify Linux distro version. |
| **LittleCMS (Flatpak fork)** | 2.19.1 (upstream) | Fork commit c70dfeb (Dec 1 2024, ~lcms2.17 era) | **High** | Fork last synced Dec 2024; **missing upstream security fixes** from lcms2.18 (Jan 2026): heap buffer overflow in `convert_utf16_to_utf32()`, out-of-bounds read (issue #522), out-of-bounds in softproofing transforms. Upstream is now at 2.19.1 (May 2026). | Update Flatpak to `mm2/Little-CMS` tag `lcms2.19.1`. |
| **OpenSSL** | 4.0.0 / 3.6.2 (LTS) | unspecified (vcpkg baseline) | **High** | No version pinned. Recent CVEs in 3.6.2 (Apr 2026): CVE-2026-31789 (heap buffer overflow in hex conversion), CVE-2026-28387 (use-after-free in DANE), CVE-2026-28386 (OOB read in AES-CFB); HIGH-severity stack buffer overflow CVE-2025-15467 fixed in 3.6.1 (Jan 2026). | Pin to `3.6.2` via vcpkg `overrides`. |
| **FreeType** | **2.14.3** (Mar 2026) | unspecified (vcpkg baseline) | **High** | 2.14.3 (Mar 22 2026) "fixes potential security issues, all users should upgrade"; 2.14.2 (Mar 1 2026) also fixed security issues with 40% speed-up. | Pin to ≥2.14.3 via vcpkg `overrides`. |
| **zlib (Flatpak)** | **1.3.2** (Feb 2026) | 1.3.1 | **Medium** | 1.3.2 addresses findings of 7ASecurity audit (Feb 2026). Flatpak uses 1.3.1. | Update Flatpak manifest to 1.3.2. |
| **OpenJPEG (Flatpak)** | **2.5.4** (Sep 2025) | 2.5.2 | **Medium** | Two releases behind. 2.5.3 and 2.5.4 contain bug/security fixes. | Update Flatpak manifest to 2.5.4. |
| Qt6 | 6.9.1 | 6.9.1 | Low | LGPL 3.0 — weak copyleft. Dynamic linking required to avoid licence propagation. | Confirm Qt is dynamically linked. Consult `license-review` skill. |

---

## Components Without Licence Information

None identified. All components have a known licence from bundled `3rdparty_licenses/`, overlay
`vcpkg.json` files, or public documentation.

---

## Components Without Clear Provenance / Version Currency

| Component | Pinned Version | Latest Version (2026-05-11) | Gap / Risk | Notes |
|-----------|---------------|----------------------------|------------|-------|
| lcms2 (Windows/vcpkg) | unspecified (vcpkg baseline) | **2.19.1** | Version unknown; at minimum 2+ releases behind upstream | Resolved by vcpkg baseline at build time |
| lcms2 (Flatpak fork) | commit c70dfeb (Dec 2024) | **2.19.1** | **Missing security fixes from 2.18 (Jan 2026)** | `JakubMelka/Little-CMS` fork last commit Dec 1 2024 — needs update |
| libjpeg-turbo | unspecified (vcpkg baseline) | **3.1.4.1** | Version unknown; no recent security CVEs identified | Active development |
| freetype | unspecified (vcpkg baseline) | **2.14.3** | Missing security fixes from 2.14.2 and 2.14.3 (Mar 2026) | Should pin to ≥2.14.3 |
| libpng | unspecified (vcpkg, Windows) / system (Linux) | **1.6.58** | **HIGH — multiple recent HIGH-severity CVEs in 1.6.51–1.6.56** | Linux: verify distro version ≥1.6.57; Windows: pin via vcpkg |
| zlib (Flatpak) | 1.3.1 | **1.3.2** | 1.3.2 includes security audit fixes (Feb 2026) | Update Flatpak manifest |
| openjpeg (Flatpak) | 2.5.2 | **2.5.4** | 2 releases behind (2.5.3 Dec 2024, 2.5.4 Sep 2025) | Update Flatpak manifest |
| oneTBB (Flatpak) | commit d3b0a80 | **2023.0.0** | Older commit, several feature+fix releases behind | Flatpak Linux-only; update to v2023.0.0 |

---

## Known Gaps and Limitations

| Gap | Impact | Recommendation |
|-----|--------|----------------|
| No machine-generated SBOM | Low confidence on exact versions; transitive-of-transitive deps not captured | Install Syft (`choco install syft`) and run `syft scan dir:. -o cyclonedx-json=sbom.cdx.json` |
| vcpkg baseline not committed | Exact versions of most dependencies unknown without running a build | Commit a `vcpkg.json` with `builtin-baseline` or add explicit `overrides` entries |
| Flatpak lcms2 fork | `JakubMelka/Little-CMS` fork is used in Flatpak builds — may diverge from upstream | Review fork commits vs upstream `mm2solutions/Little-CMS` |
| Docker base image (ubuntu:22.04) adds OS-level packages | OS-level components not covered by this SBOM | Run Trivy against the Docker image for a complete container SBOM |
| Qt6 TextToSpeech / platform-specific speech API | On Windows, links to `ole32`, `sapi` (system COM) — no separate versioning | Document as OS-provided; no action required |
| Generated code (`generated_code_definition.xml`) | CodeGenerator output may not be reflected in SBOM tools | Manually reviewed — outputs are C++ source integrated into core library |
| JBIG2 decoding | CCITT/JBIG2 decoder (`pdfccittfaxdecoder`) appears hand-written, not a vendored library | Confirm no external JBIG2 lib is embedded in source |

---

## SBOM Quality Assessment

| Criterion | Status | Notes |
|-----------|--------|-------|
| All manifests represented | Partial | vcpkg.json, Flatpak manifest, Dockerfile analysed; no lockfile present |
| Lockfile versions accurate | No | No lockfile. vcpkg resolves versions at build time from baseline |
| Vendored code captured | Partial | blend2d and asmjit have custom overlay portfiles pinning specific commits |
| Container dependencies included | No | Docker/Ubuntu OS packages not inventoried |
| Build-time dependencies separated | Partial | Qt LinguistTools and Test components are build/test-only; noted in inventory |

---

## Recommended Next Steps

**Immediate (security):**
1. **Pin libpng to ≥1.6.57** via vcpkg `overrides` — HIGH severity CVEs (CVE-2026-33416, CVE-2026-25646 etc.) affect PDF rendering with palette/transparency PNGs.
2. **Update Flatpak LittleCMS fork** — switch Flatpak to upstream `mm2/Little-CMS` at tag `lcms2.19.1`; the fork is missing heap buffer overflow and out-of-bounds read fixes from 2.18 (Jan 2026).
3. **Pin OpenSSL to 3.6.2** in `vcpkg.json` `overrides` — multiple CVEs addressed in the Apr 2026 LTS release.
4. **Pin FreeType to ≥2.14.3** via vcpkg `overrides` — two security-fix releases in March 2026.
5. **Update Flatpak zlib to 1.3.2** — 1.3.1 is behind the current release which includes a 7ASecurity audit's findings.
6. **Update Flatpak openjpeg to 2.5.4** — two bug/security fix releases behind.

**Near-term (hygiene):**
7. **Generate a machine-readable SBOM** — install Syft (`choco install syft`) and run `syft scan dir:. -o cyclonedx-json=sbom.cdx.json`.
8. **Add `builtin-baseline`** to `vcpkg.json` so all dependency versions are reproducible and auditable.
9. **Run `license-review` skill** — LGPL 3.0 (Qt6) and FreeType License warrant a dedicated licence review for commercial distribution.
10. **Run `dependency-vulnerability-review` skill** — for a full CVE scan across all resolved dependency versions.
11. **Add Trivy to CI** — scan the Docker image to capture OS-level component inventory and flag unpatched system libpng.

---

## Assumptions

- vcpkg.json lists reflect the intended direct runtime dependencies for Windows/Linux builds.
- Qt6 is dynamically linked (standard Qt installer default); static linking would change the LGPL 3.0 obligation analysis.
- The Flatpak manifest versions (zlib v1.3.1, openjpeg v2.5.2, blend2d commit 6dbc2ce, asmjit commit c878602) are used as the best available version information for those components.
- The project's own code (PDF4QT libraries and applications) is licensed under MIT as of April 2025.
