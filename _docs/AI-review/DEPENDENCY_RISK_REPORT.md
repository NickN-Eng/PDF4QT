# Dependency Vulnerability Risk Report

## Repository
`PDF4QT` — `c:\Users\nniem\source\repos\PDF4QT`

## Report Metadata

| Field | Value |
|-------|-------|
| Generated | 2026-05-11 |
| Project version | 1.5.3.1 |
| Tools used | Manual manifest analysis (Trivy not available; no native vcpkg audit tool exists) |
| Ecosystems | C++ / vcpkg, CMake, Docker |
| Data sources | OpenSSL vulnerability page, NVD, manual CVE research |

---

## Executive Summary

**Overall Risk: MEDIUM**

PDF4QT is a C++ project using vcpkg for dependency management. The repository does **not pin dependency versions** via a `builtin-baseline` or `overrides` in `vcpkg.json`, meaning the exact versions resolved depend on the vcpkg registry state at build time. This makes vulnerability assessment partially speculative — the actual risk depends on when dependencies were last resolved.

Key findings:
- **OpenSSL** (unpinned) has multiple recent CVEs including 1 High-severity stack buffer overflow (CVE-2025-15467) fixed in Jan 2026 and a Moderate RSA KEM issue (CVE-2026-31790) fixed in Apr 2026
- **zlib** (1.3.x) has no known critical CVEs in recent versions
- **FreeType**, **OpenJPEG**, **libjpeg-turbo**, **libpng** have historical CVEs but recent versions are generally safe
- **Docker base image** (`ubuntu:22.04`) introduces unaudited OS-level packages
- No lockfile exists, making reproducibility and audit difficult

---

## Findings Summary

| # | Package | Severity | CVE | Type | Fix Available | Status |
|---|---------|----------|-----|------|---------------|--------|
| 1 | OpenSSL | **High** | CVE-2025-15467 | Stack buffer overflow in CMS parsing | Yes (≥3.0.19, ≥3.3.6, ≥3.4.4, ≥3.5.5, ≥3.6.1) | Confirmed |
| 2 | OpenSSL | **Moderate** | CVE-2026-31790 | RSA KEM RSASVE incorrect failure handling (data leak) | Yes (≥3.0.20, ≥3.3.7, ≥3.4.5, ≥3.5.6, ≥3.6.2) | Confirmed |
| 3 | OpenSSL | **Moderate** | CVE-2025-11187 | PBMAC1 stack overflow in PKCS#12 | Yes (≥3.4.4, ≥3.5.5, ≥3.6.1) | Confirmed |
| 4 | OpenSSL | Low | CVE-2026-28387 | Use-after-free in DANE client code | Yes (≥3.0.20, ≥3.3.7, ≥3.4.5, ≥3.5.6, ≥3.6.2) | Confirmed |
| 5 | OpenSSL | Low | CVE-2026-28388 | NULL ptr deref in delta CRL processing | Yes (≥3.0.20, ≥3.3.7, ≥3.4.5, ≥3.5.6, ≥3.6.2) | Confirmed |
| 6 | OpenSSL | Low | CVE-2026-28389 | NULL ptr deref in CMS EnvelopedData | Yes (≥3.0.20, ≥3.3.7, ≥3.4.5, ≥3.5.6, ≥3.6.2) | Confirmed |
| 7 | OpenSSL | Low | CVE-2025-68160 | Heap OOB write in BIO_f_linebuffer | Yes (≥3.0.19, ≥3.3.6, ≥3.4.4, ≥3.5.5, ≥3.6.1) | Confirmed |
| 8 | OpenSSL | Low | CVE-2025-69419 | OOB write in PKCS12_get_friendlyname UTF-8 | Yes (≥3.0.19, ≥3.3.6, ≥3.4.4, ≥3.5.5, ≥3.6.1) | Confirmed |
| 9 | OpenSSL | Low | CVE-2025-9230 | OOB read/write in RFC 3211 KEK Unwrap | Yes (≥3.0.18, ≥3.3.5, ≥3.4.3, ≥3.5.4) | Confirmed |
| 10 | OpenSSL | Low | CVE-2026-2673 | TLS 1.3 unexpected key group negotiation | Yes (≥3.5.6, ≥3.6.2) | Confirmed |
| 11 | FreeType | Moderate | Historical | Multiple OOB read/write in font parsing (pre-2.13) | Yes (recent versions) | Likely |
| 12 | OpenJPEG | Low-Moderate | Historical | Heap overflow in JPEG2000 parsing (pre-2.5) | Yes (recent versions) | Likely |
| 13 | libjpeg-turbo | Low | Historical | Integer overflow in marker parsing (pre-3.0) | Yes (recent versions) | Possible |
| 14 | Docker: ubuntu:22.04 | Unknown | Multiple | Unaudited OS packages in container image | Upgrade to newer base or scan | Informational |

---

## Detailed Findings

### Finding 1: OpenSSL — CVE-2025-15467 (High)

| Field | Value |
|-------|-------|
| Package | openssl |
| Version | Unpinned (vcpkg baseline) |
| Advisory | [CVE-2025-15467](https://www.cve.org/CVERecord?id=CVE-2025-15467) |
| Severity | **High** |
| Confidence | Confirmed (CVE exists; affected range covers all OpenSSL 3.x before patched versions) |
| Dependency type | Direct (vcpkg.json) |
| Manifest | `vcpkg.json` |
| Detected by | Manual research (OpenSSL advisories) |

**Description:** Stack buffer overflow in CMS (Auth)EnvelopedData parsing. When processing crafted CMS messages using AEAD ciphers (e.g., AES-GCM), a stack-based write occurs prior to authentication. No valid key material required to trigger.

**Reachability in PDF4QT:** PDF4QT uses OpenSSL for PDF encryption/decryption operations. CMS is not directly used by PDF4QT's core functionality, but OpenSSL APIs may be called indirectly. **Reachability: Possible but unlikely** — PDF encryption uses symmetric ciphers and certificate verification, not CMS decryption directly.

**Remediation:** Pin OpenSSL to ≥3.4.4 (or latest) via vcpkg overrides.

---

### Finding 2: OpenSSL — CVE-2026-31790 (Moderate)

| Field | Value |
|-------|-------|
| Package | openssl |
| Advisory | [CVE-2026-31790](https://www.cve.org/CVERecord?id=CVE-2026-31790) |
| Severity | **Moderate** |
| Confidence | Confirmed |

**Description:** Incorrect failure handling in RSA KEM RSASVE Encapsulation. If RSA encryption fails, encapsulation can still return success, potentially leaking stale/uninitialized ciphertext buffer contents to an attacker.

**Reachability in PDF4QT:** RSASVE (KEM) is not used in PDF operations. **Reachability: Very unlikely.**

**Remediation:** Pin OpenSSL to ≥3.4.5 via vcpkg overrides.

---

### Finding 3: OpenSSL — CVE-2025-11187 (Moderate)

| Field | Value |
|-------|-------|
| Package | openssl |
| Advisory | [CVE-2025-11187](https://www.cve.org/CVERecord?id=CVE-2025-11187) |
| Severity | **Moderate** |

**Description:** PBMAC1 parameters in PKCS#12 files are missing validation, which can trigger a stack-based buffer overflow or NULL pointer dereference during MAC verification. Only affects OpenSSL 3.4+.

**Reachability in PDF4QT:** PDF4QT processes PDF files that may contain embedded certificates. If PKCS#12 files are parsed (e.g., for digital signature operations), this could be reachable. **Reachability: Low-Moderate** depending on signature plugin usage.

**Remediation:** Pin OpenSSL to ≥3.4.4 via vcpkg overrides.

---

### Finding 4–10: OpenSSL Low-Severity Issues

Multiple Low-severity CVEs exist in OpenSSL (2025–2026) affecting DANE, CRL processing, PKCS12/PKCS7 parsing, and BIO operations. These generally require specific configurations or attacker-controlled input to untrusted data parsing functions.

**Reachability in PDF4QT:** Most are not directly reachable through normal PDF processing. Risk is **low** but present if OpenSSL is not kept current.

---

### Finding 11: FreeType — Historical Vulnerabilities

FreeType has had multiple High and Critical CVEs related to font parsing (e.g., CVE-2020-15999, CVE-2022-27404/05/06). PDF files can embed fonts, making PDF4QT a potential attack vector.

**Reachability:** **High** — PDF4QT parses embedded fonts in PDF documents.

**Remediation:** Ensure vcpkg resolves FreeType ≥2.13.0. Pin version via overrides.

---

### Finding 12: OpenJPEG — Historical Vulnerabilities

OpenJPEG has had heap overflow and denial-of-service CVEs in JPEG2000 parsing. PDF files support JPEG2000 image streams.

**Reachability:** **High** — PDF4QT decodes JPEG2000 images embedded in PDF files.

**Remediation:** Ensure vcpkg resolves OpenJPEG ≥2.5.0. Pin version via overrides.

---

### Finding 13: libjpeg-turbo — Historical Vulnerabilities

libjpeg-turbo has had integer overflow and buffer overread issues in older versions. PDFs commonly embed JPEG images.

**Reachability:** **High** — PDF4QT decodes JPEG images in PDF documents.

**Remediation:** Ensure vcpkg resolves libjpeg-turbo ≥3.0. Pin version via overrides.

---

### Finding 14: Docker Base Image — ubuntu:22.04

The Dockerfile uses `ubuntu:22.04` which may contain unpatched system packages (libc, libcurl, etc.).

**Reachability:** Only affects containerized deployments.

**Remediation:** Run `trivy image` against built container. Consider upgrading to `ubuntu:24.04`.

---

## Risk Matrix

| Risk Factor | Assessment |
|-------------|-----------|
| Unpinned dependency versions | **High** — impossible to guarantee which versions are deployed |
| OpenSSL exposure | **Medium** — used for PDF encryption; most recent CVEs have low reachability |
| Image parsing libraries (FreeType, OpenJPEG, libjpeg-turbo) | **Medium-High** — directly processes untrusted PDF content containing fonts/images |
| No vulnerability scanning in CI | **High** — no automated detection of new CVEs |
| Transitive dependencies | **Low** — blend2d/asmjit are small libraries with no known CVEs |

---

## Recommendations (Priority Order)

1. **Pin all dependency versions** — Add `builtin-baseline` and/or `overrides` to `vcpkg.json` to ensure reproducible, auditable builds:
   ```json
   {
     "builtin-baseline": "<commit-hash>",
     "overrides": [
       { "name": "openssl", "version": "3.4.5" }
     ]
   }
   ```

2. **Ensure OpenSSL ≥3.4.5** — Addresses all High and Moderate CVEs listed above.

3. **Ensure FreeType ≥2.13.0, OpenJPEG ≥2.5.0, libjpeg-turbo ≥3.0** — Addresses historical parsing vulnerabilities directly reachable through PDF content.

4. **Install Trivy and add to CI** — Enable automated vulnerability scanning:
   ```bash
   trivy repo --scanners vuln .
   trivy image <docker-image-name>
   ```

5. **Upgrade Docker base image** — Move from `ubuntu:22.04` to `ubuntu:24.04` for newer system packages.

6. **Generate a vcpkg lockfile** — After pinning versions, commit the resolved versions for auditability.

---

## Limitations

| Limitation | Impact |
|-----------|--------|
| Trivy not available | Could not perform automated multi-ecosystem vulnerability scan |
| No vcpkg lockfile | Exact resolved versions unknown; findings are based on version ranges |
| No build performed | Could not verify actual linked library versions |
| Historical CVEs for FreeType/OpenJPEG/libjpeg-turbo | Listed as "Likely" — version confirmation requires build output |

---

## Confidence Summary

| Confidence Level | Count |
|-----------------|-------|
| Confirmed | 10 (OpenSSL CVEs with documented affected ranges) |
| Likely | 2 (FreeType, OpenJPEG — historical, version unconfirmed) |
| Possible | 1 (libjpeg-turbo) |
| Informational | 1 (Docker base image) |

---

## Methodology

1. Identified package managers: **vcpkg** (primary), Docker, Flatpak
2. No native audit tool exists for vcpkg; `dotnet list package --vulnerable` is not applicable to C++ projects
3. Attempted Trivy installation — not available on this system
4. Manually researched CVEs for all direct dependencies using official advisory pages
5. Assessed reachability based on PDF4QT's architecture (PDF parsing, rendering, encryption)
6. Prioritised findings by severity, reachability, and fix availability
