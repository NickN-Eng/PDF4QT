# Licence Review Report

## Disclaimer

> **This is an engineering and compliance triage review, not legal advice. Licence interpretation and commercial-use decisions should be reviewed by qualified legal/commercial stakeholders.**

## Summary

| Field | Value |
|-------|-------|
| Repository | `c:\Users\nniem\source\repos\PDF4QT` (PDF4QT by Jakub Melka) |
| Review date | 2026-05-11 |
| Intended use | Open-source distributed product (MIT-licensed desktop PDF toolkit) |
| Repository licence | MIT |
| Tools used | Manual manifest analysis (`vcpkg.json`, `3rdparty_licenses/`, CMakeLists.txt), internet research (GitHub, official project sites) |
| Total dependencies scanned | 11 (9 vcpkg direct + 1 vcpkg transitive + Qt6 framework) |
| Licence categories found | Permissive (MIT, Apache-2.0, BSD-2-Clause, BSD-3-Clause, Zlib, IJG, FTL, libpng), Weak copyleft (LGPL-3.0) |

---

## Repository Licence

| Field | Value |
|-------|-------|
| Licence file | `LICENSE` (root) |
| SPDX identifier | MIT |
| Classification | Permissive |
| Confidence | Confirmed |
| Commercial use | Yes |
| Modification | Yes |
| Redistribution | Yes |
| Patent grant | No explicit patent grant |
| Notes | Copyright © 2018–2025 Jakub Melka and Contributors. Standard MIT with no additional clauses. All source files confirmed with matching MIT header comments. |

---

## Dependency Licence Summary

| Category | Count | Percentage | Risk Level |
|----------|-------|-----------|-----------|
| Permissive (MIT, Apache-2.0, BSD-*, Zlib, FTL, IJG, libpng) | 10 | 91 % | Low |
| Weak copyleft (LGPL-3.0) | 1 | 9 % | Medium |
| Strong copyleft (GPL) | 0 | 0 % | — |
| Network copyleft (AGPL) | 0 | 0 % | — |
| Source-available | 0 | 0 % | — |
| Unknown / Missing | 0 | 0 % | — |

---

## Items Requiring Legal Review

| ID | Package / File | Licence | Concern | Priority |
|----|---------------|---------|---------|----------|
| LIC-001 | Qt6 (LGPL community edition) | LGPL-3.0 | Binary redistribution obligations — users must be able to replace the Qt shared library. Verify dynamic linking or provide relinking mechanism. Mandatory for any binary release. | Medium |
| LIC-002 | freetype | FTL (dual FTL/GPL-2.0; FTL chosen) | FTL requires a credit clause: "This product uses the FreeType Project" (or equivalent) in documentation. Verify this notice exists in distributed packages. | Low |
| LIC-003 | libjpeg-turbo | IJG + BSD-3-Clause | IJG Licence requires that binary distributions include the statement "This software is based in part on the work of the Independent JPEG Group." Verify this is present in shipped documentation. | Low |
| LIC-004 | OpenSSL (≥ 3.0) | Apache-2.0 | Apache-2.0 requires NOTICE file to be preserved when distributing compiled binaries. Verify the OpenSSL NOTICE file is included in installer/package. | Low |
| LIC-005 | oneTBB | Apache-2.0 | Same Apache-2.0 NOTICE requirement as OpenSSL (see LIC-004). | Low |

---

## Licence Findings

### High-Risk Findings

None.

---

### Medium-Risk Findings

#### LIC-001 — Qt6 LGPL-3.0 (Weak Copyleft)

- **Package**: Qt6 — modules used: `Qt6::Core`, `Qt6::Gui`, `Qt6::Widgets`, `Qt6::Xml`, `Qt6::Svg`, `Qt6::PrintSupport`, `Qt6::TextToSpeech`
- **Licence**: LGPL-3.0 (community edition)
- **Source**: https://doc.qt.io/qt-6/licensing.html, https://doc.qt.io/qt-6/qttexttospeech-index.html (TextToSpeech confirmed LGPL-3.0 / GPL-2.0; LGPL-3.0 option applies)
- **Confidence**: Confirmed
- **Concern**: LGPL-3.0 requires that end-users of a binary distribution must be able to replace the LGPL-licensed component (Qt) with a modified version. For Qt this typically means:
  1. Distribute Qt as shared/dynamic libraries (DLLs on Windows, `.so` on Linux), **or**
  2. Provide application object files to allow relinking (required for statically linked Qt builds).
- **Current state**: The Windows installer (WixInstaller/) and Flatpak build both use Qt as an external shared library — this satisfies the LGPL-3.0 relinking requirement in those configurations.
- **Risk for adopters**: Any downstream user wishing to embed PDF4QT in a **closed-source commercial product** must obtain a **commercial Qt licence** or ensure their own LGPL compliance. This obligation does not fall on the PDF4QT project itself, but should be documented in the README for adopters.
- **Action**: Verify that no build configuration statically links Qt without providing user relinking capability.

---

### Low-Risk / Informational Findings

#### LIC-002 — FreeType FTL Credit Clause

- **Package**: `freetype` (vcpkg), licence file `3rdparty_licenses/freetype_FTL.TXT`
- **Licence**: FTL (FreeType Licence) — BSD-style permissive with credit clause
- **Source**: https://freetype.org/license.html; local file `3rdparty_licenses/freetype_FTL.TXT`
- **Confidence**: Confirmed (FTL chosen over GPL-2.0 option; FTL file present in repo)
- **Obligation**: Any distributed product using FreeType must acknowledge its use in documentation (e.g., "Portions of this software are copyright © The FreeType Project (www.freetype.org). All rights reserved."). This is noted in the FTL.
- **Action**: Confirm credit is present in installer/About dialog/documentation.

#### LIC-003 — libjpeg-turbo IJG Attribution

- **Package**: `libjpeg-turbo` (vcpkg), licence file `3rdparty_licenses/libjpeg_README.txt`
- **Licence**: IJG Licence + BSD-3-Clause (Modified BSD for TurboJPEG API)
- **Source**: https://github.com/libjpeg-turbo/libjpeg-turbo/blob/main/LICENSE.md
- **Confidence**: Confirmed
- **Obligation**: Binary distributions must include the statement: *"This software is based in part on the work of the Independent JPEG Group."* (IJG Licence, Clause 2). If the TurboJPEG API is used, the Modified BSD licence text must also be included.
- **Note**: The vcpkg dependency is `libjpeg-turbo` (not the original `libjpeg` IJG package), but the IJG licence obligations still apply as libjpeg-turbo inherits libjpeg code.
- **Action**: Confirm the IJG acknowledgement appears in release documentation.

#### LIC-004 — OpenSSL 3.x Apache-2.0 NOTICE

- **Package**: `openssl` (vcpkg)
- **Licence**: Apache-2.0 (applies to OpenSSL ≥ 3.0)
- **Source**: https://openssl-library.org/source/license; local file `3rdparty_licenses/OpenSSL_license.txt` (Apache-2.0 text confirmed)
- **Confidence**: Confirmed
- **Obligation**: Apache-2.0 requires the preservation of the NOTICE file when distributing compiled works. OpenSSL ships a NOTICE file; it should be included in binary releases.
- **Action**: Verify NOTICE file is included in installer/package.

#### LIC-005 — oneTBB Apache-2.0 NOTICE

- **Package**: `tbb` (oneTBB, vcpkg)
- **Licence**: Apache-2.0
- **Source**: https://github.com/uxlfoundation/oneTBB/blob/master/LICENSE.txt (confirmed Apache-2.0)
- **Confidence**: Confirmed
- **Obligation**: Same Apache-2.0 NOTICE requirement as LIC-004.
- **Action**: Verify NOTICE file is included in installer/package.

#### LIC-006 — Blend2D / AsmJIT (transitive) Zlib Licence

- **Package**: `blend2d` (vcpkg overlay), `asmjit` (transitive dependency via blend2d JIT feature, vcpkg overlay)
- **Licence**: Zlib
- **Source**: https://github.com/blend2d/blend2d/blob/master/LICENSE.md; https://github.com/asmjit/asmjit/blob/master/LICENSE.md
- **Confidence**: Confirmed
- **Obligation**: Zlib licence requires that altered source versions be plainly marked and that the original source is not misrepresented. No binary attribution required. Essentially no obligations for unmodified binary distribution.
- **Note**: The repo does not include a `blend2d` or `asmjit` file in `3rdparty_licenses/`. While no binary attribution is strictly required by the Zlib licence, adding entries for completeness is best practice.
- **Action** (informational): Consider adding licence notices for blend2d and asmjit in `3rdparty_licenses/` for completeness.

#### LIC-007 — Little CMS (lcms2) MIT

- **Package**: `lcms` (vcpkg), licence file `3rdparty_licenses/LittleCMS_COPYING.txt`
- **Licence**: MIT
- **Source**: Local file confirmed; https://www.littlecms.com ("Free under MIT License")
- **Confidence**: Confirmed
- **Obligation**: Attribution notice must be included in distributions. Already present in `3rdparty_licenses/LittleCMS_COPYING.txt`.

#### LIC-008 — zlib Zlib Licence

- **Package**: `zlib` (vcpkg), licence file `3rdparty_licenses/zlib_README.txt`
- **Licence**: Zlib (custom permissive, SPDX: `Zlib`)
- **Source**: Local file confirmed; https://zlib.net/zlib_license.html
- **Confidence**: Confirmed
- **Obligation**: Minimal — do not misrepresent origin; mark alterations. No binary attribution required.

#### LIC-009 — OpenJPEG BSD-2-Clause

- **Package**: `openjpeg` (vcpkg), licence file `3rdparty_licenses/OpenJPEG_LICENSE.txt`
- **Licence**: BSD-2-Clause
- **Source**: Local file confirmed; https://github.com/uclouvain/openjpeg/blob/master/LICENSE
- **Confidence**: Confirmed
- **Obligation**: Retain copyright notice in source; reproduce notice in binary documentation. Minor attribution obligation.

#### LIC-010 — libpng libpng/PNG Licence

- **Package**: `libpng` (vcpkg)
- **Licence**: libpng (PNG Reference Library License v2 — permissive, similar to MIT/Zlib)
- **Source**: https://libpng.org/pub/png/src/libpng-LICENSE.txt
- **Confidence**: Confirmed
- **Obligation**: Do not misrepresent origin. Acknowledgement appreciated but not required.
- **Note**: No `libpng` entry in `3rdparty_licenses/`. Consider adding for completeness.

---

## Commercial Use Assessment

### For Intended Use: Open-source distributed product (MIT, binary installers)

| Question | Assessment | Confidence |
|----------|-----------|-----------|
| Can we use this commercially? | Yes — all dependencies permit commercial use | Confirmed |
| Can we modify the code? | Yes — all dependencies permit modification | Confirmed |
| Can we redistribute? | Yes — with conditions (see attribution obligations LIC-001 to LIC-005) | Confirmed |
| Are there SaaS restrictions? | No — no AGPL or SSPL dependencies | Confirmed |
| Attribution requirements? | Yes — FTL credit (FreeType), IJG statement (libjpeg-turbo), Apache NOTICE files (OpenSSL, oneTBB), Qt LGPL notice | Confirmed |
| Patent implications? | Apache-2.0 includes patent grant (TBB, OpenSSL). No patent risks identified for the permissive licences. Qt commercial patent indemnification not available under LGPL. | Likely |
| Copyleft obligations? | Weak copyleft from Qt LGPL-3.0: dynamic linking maintained in current builds satisfies obligation. Downstream closed-source users must handle separately. | Confirmed |

---

## Licence Conflicts

No licence conflicts identified. All dependency licences are mutually compatible when combined under the MIT host project licence. The LGPL-3.0 (Qt) does not conflict with the MIT project licence, provided dynamic linking or relinking capability is preserved.

---

## Vendored / Copied Code

| Location | Source | Licence | Concern |
|----------|--------|---------|---------|
| `3rdparty_licenses/` | Various (FreeType, lcms, libjpeg-turbo, OpenJPEG, OpenSSL, zlib) | Mixed permissive | Licence files present ✔ |
| `vcpkg/overlays/general/blend2d/` | blend2d vcpkg port | Zlib | Port metadata only; licence file not vendored in `3rdparty_licenses/` — informational gap |
| `vcpkg/overlays/general/asmjit/` | asmjit vcpkg port | Zlib | Same as above |
| `xfa/` directory | Unknown (XFA form rendering) | Needs investigation | Not covered by this review; check for any third-party XFA implementation code with different licensing |

---

## Recommended Actions

| Priority | Action | Reason |
|----------|--------|--------|
| Medium | Verify Qt is always distributed as shared libraries (DLLs/.so) in all binary releases, or document relinking provision | LGPL-3.0 compliance (LIC-001) |
| Medium | Add notice to README/documentation informing downstream commercial-product adopters that a commercial Qt licence is required for closed-source use | Protects downstream users from unintentional LGPL-3.0 violation |
| Low | Confirm "Portions of this software are copyright © The FreeType Project" credit appears in About dialog or shipped documentation | FTL credit clause (LIC-002) |
| Low | Confirm "This software is based in part on the work of the Independent JPEG Group." appears in shipped documentation | IJG Licence clause 2 (LIC-003) |
| Low | Include OpenSSL NOTICE file in Windows installer and Flatpak package | Apache-2.0 obligation (LIC-004) |
| Low | Include oneTBB NOTICE file in Windows installer and Flatpak package | Apache-2.0 obligation (LIC-005) |
| Low | Add `blend2d` and `asmjit` licence files to `3rdparty_licenses/` | Completeness / attribution hygiene |
| Low | Add `libpng` licence file to `3rdparty_licenses/` | Completeness / attribution hygiene |
| Informational | Review `xfa/` directory for any third-party code with separate licensing | Unknown provenance |

---

## Assumptions

- The project uses **OpenSSL ≥ 3.0** (Apache-2.0); the `3rdparty_licenses/OpenSSL_license.txt` contains the Apache-2.0 text confirming this. If an older OpenSSL 1.x version is used in any build, the dual OpenSSL/SSLeay licence applies instead (also permissive but different conditions).
- Qt is used under the **LGPL community edition** (not a commercial Qt licence). The project is open-source (MIT), which is consistent with LGPL-3.0 use.
- Trivy was unavailable for automated scanning; all findings are based on manifest analysis and internet research of official sources.
- The `xfa/` subdirectory was not fully inspected in this review.

## Unknowns

- Whether NOTICE files for Apache-2.0 packages (OpenSSL, oneTBB) are currently included in the Windows installer (WixInstaller/) or Flatpak bundle.
- Full licence provenance of any third-party code in `xfa/`.
- Whether a commercial Qt licence is held for any proprietary downstream use of this codebase.
