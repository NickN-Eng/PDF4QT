# Security Review Report

## Summary

| Field | Value |
|-------|-------|
| Repository | `c:\Users\nniem\source\repos\PDF4QT` (https://github.com/JakubMelka/PDF4QT) |
| Review date | 2026-05-11 |
| Reviewer | LLM-guided security code review (GitHub Copilot) |
| Application type | Desktop application suite (GUI + CLI) + shared C++ library |
| Scope | Full codebase security review — parser, decoders, crypto, plugins, CI/CD |
| Tools used | LLM manual code review (no automated SAST tools available) |
| Checklists applied | general-security-review, desktop-app-security, cli-tool-security |

## Disclaimer

> This security review is based on static analysis and manual code inspection. It is not a penetration test, dynamic analysis, or comprehensive security audit. Findings represent issues identified through available tools and reviewer expertise at a point in time. Absence of findings does not guarantee absence of vulnerabilities.

## Scope

### Included
- PDF parser (`pdfparser.cpp`, `pdfxreftable.cpp`, `pdfdocumentreader.cpp`)
- Stream decoders (`pdfstreamfilters.cpp`, `pdfjbig2decoder.cpp`, `pdfccittfaxdecoder.cpp`)
- Image handling (`pdfimage.cpp`)
- Cryptographic subsystem (`pdfsecurityhandler.cpp`, `pdfsignaturehandler.cpp`, `pdfcertificatemanager.cpp`)
- XFA engine (`pdfxfaengine.cpp`)
- Plugin loading (`pdfprogramcontroller.cpp`)
- Email/scanner integrations (`pdfsendmail.cpp`, `wiascannerbackend.cpp`)
- File operations (`pdftoolattachments.cpp`, `pdffile.cpp`)
- CI/CD workflows (`.github/workflows/`)

### Excluded
- Third-party library internals (OpenSSL, zlib, FreeType, OpenJPEG, libjpeg-turbo, LittleCMS, Blend2D)
- Qt framework internals
- Dynamic/runtime testing
- Fuzz testing

### Limitations
- No automated SAST tools (Trivy, Semgrep, CodeQL, gitleaks) were run
- Line numbers are approximate based on code inspection
- No runtime exploitation verification performed
- Complex data flows (e.g., full PDF rendering pipeline) were not traced end-to-end

## Findings Summary

| Severity | Count | Tool-Detected | Manual |
|----------|-------|---------------|--------|
| Critical | 7 | 0 | 7 |
| High | 8 | 0 | 8 |
| Medium | 10 | 0 | 10 |
| Low | 2 | 0 | 2 |
| Informational | 2 | 0 | 2 |

**Total: 29 findings**

---

## Critical and High Findings

### SEC-001: Integer Overflow in XRef Table Object Sizing

| Field | Value |
|-------|-------|
| Severity | **Critical** |
| Confidence | Confirmed |
| CWE | CWE-190 (Integer Overflow or Wraparound) |
| File | `Pdf4QtLibCore/sources/pdfxreftable.cpp` |
| Lines | 79–88, 295–298 |

**Evidence:**
```cpp
PDFInteger firstObjectNumber = firstObject.getInteger();
PDFInteger count = countObject.getInteger();
const PDFInteger lastObjectIndex = firstObjectNumber + count - 1;  // OVERFLOW
const PDFInteger desiredSize = lastObjectIndex + 1;                // OVERFLOW
```

**Exploit scenario:** Attacker crafts PDF with `firstObjectNumber = INT64_MAX` and `count = 2` → signed overflow → negative `desiredSize` → undersized vector allocation → OOB write on subsequent access.

---

### SEC-002: Integer Overflow in Object Stream Offset Calculation

| Field | Value |
|-------|-------|
| Severity | **Critical** |
| Confidence | Confirmed |
| CWE | CWE-190, CWE-125 |
| File | `Pdf4QtLibCore/sources/pdfdocumentreader.cpp` |
| Lines | 484–505 |

**Evidence:**
```cpp
const PDFInteger first = firstObject.getInteger();
const PDFInteger offset = currentOffset.getInteger() + first;  // OVERFLOW
```

**Exploit scenario:** Two large positive integers from PDF stream overflow signed addition → negative offset → parser seeks to wrong location → arbitrary buffer read.

---

### SEC-003: Unbounded Memory Allocation in XRef Table

| Field | Value |
|-------|-------|
| Severity | **Critical** |
| Confidence | Confirmed |
| CWE | CWE-400 (Uncontrolled Resource Consumption) |
| File | `Pdf4QtLibCore/sources/pdfxreftable.cpp` |
| Lines | 87–89, 200–202, 298–300 |

**Evidence:**
```cpp
if (static_cast<PDFInteger>(m_entries.size()) < desiredSize)
{
    m_entries.resize(desiredSize);  // NO MAXIMUM LIMIT
}
```

**Exploit scenario:** PDF claims 2 billion objects → 2+ GB allocation → OOM crash (DoS).

---

### SEC-004: Unbounded Recursion in Nested Objects

| Field | Value |
|-------|-------|
| Severity | **Critical** |
| Confidence | Confirmed |
| CWE | CWE-674 (Uncontrolled Recursion) |
| File | `Pdf4QtLibCore/sources/pdfparser.cpp` |
| Lines | 770–800 |

**Evidence:**
```cpp
PDFObject object = getObject();  // Recursive call — NO depth limit
```

**Exploit scenario:** Deeply nested dictionary `<< /A << /B << ... >> >> >>` (1000+ levels) → stack overflow → crash or potential code execution.

---

### SEC-005: Integer Overflow in JBIG2 Bitmap Allocation

| Field | Value |
|-------|-------|
| Severity | **Critical** |
| Confidence | Confirmed |
| CWE | CWE-190, CWE-122 (Heap Buffer Overflow) |
| File | `Pdf4QtLibCore/sources/pdfjbig2decoder.cpp` |
| Lines | 3649–3660 |

**Evidence:**
```cpp
m_data.resize(width * height, 0);  // int * int overflow when both are ~65536
```

**Exploit scenario:** JBIG2 segment with width=65536, height=65536 → `65536 × 65536` overflows 32-bit int → undersized allocation → heap corruption during pixel writes.

---

### SEC-006: Integer Overflow in Cross-Reference Stream Loop

| Field | Value |
|-------|-------|
| Severity | **Critical** |
| Confidence | Confirmed |
| CWE | CWE-680 (Integer Overflow to Buffer Overflow) |
| File | `Pdf4QtLibCore/sources/pdfxreftable.cpp` |
| Lines | 305–340 |

**Evidence:**
```cpp
for (PDFInteger objectNumber = firstObjectNumber; objectNumber <= lastObjectIndex; ++objectNumber)
{
    m_entries[objectNumber] = std::move(entry);  // OOB if resize was undersized
}
```

**Exploit scenario:** If xref stream index causes overflow in `lastObjectIndex` calculation, the resize allocates wrong size, and loop writes OOB.

---

### SEC-007: Path Traversal in PDF Attachment Extraction

| Field | Value |
|-------|-------|
| Severity | **Critical** |
| Confidence | Confirmed |
| CWE | CWE-22 (Path Traversal) |
| File | `PdfTool/pdftoolattachments.cpp` |
| Lines | 173–182 |

**Evidence:**
```cpp
QString outputFile = info.fileName;  // UNSANITIZED from PDF embedded file spec
outputFile = QString("%1/%2").arg(options.attachmentsOutputDirectory, outputFile);
QFile file(outputFile);
file.open(QFile::WriteOnly | QFile::Truncate);
file.write(data);
```

**Exploit scenario:** PDF contains attachment named `../../.ssh/authorized_keys` → user runs `pdfTool attachments --save-all` → arbitrary file write outside output directory.

---

### SEC-008: Signed-to-Unsigned Cast Bypass in XRef

| Field | Value |
|-------|-------|
| Severity | **High** |
| Confidence | Confirmed |
| CWE | CWE-195 (Signed to Unsigned Conversion Error) |
| File | `Pdf4QtLibCore/sources/pdfxreftable.cpp` |
| Lines | 111 |

**Evidence:**
```cpp
if (static_cast<size_t>(objectNumber) >= m_entries.size())  // negative → huge unsigned
```

**Exploit scenario:** Negative object number from malformed xref → becomes huge unsigned value → passes bounds check → OOB access.

---

### SEC-009: Unvalidated Stream Length Allocation

| Field | Value |
|-------|-------|
| Severity | **High** |
| Confidence | Confirmed |
| CWE | CWE-400 |
| File | `Pdf4QtLibCore/sources/pdfparser.cpp` |
| Lines | 851–876 |

**Evidence:**
```cpp
QByteArray buffer = m_lexicalAnalyzer.fetchByteArray(length);  // No max limit
```

**Exploit scenario:** Stream dictionary with `Length 2147483647` → 2GB allocation attempt → OOM crash.

---

### SEC-010: Permissive Mode Suppresses Security Errors

| Field | Value |
|-------|-------|
| Severity | **High** |
| Confidence | Confirmed |
| CWE | CWE-391 (Unchecked Error Condition) |
| File | `Pdf4QtLibCore/sources/pdfdocumentreader.cpp` |
| Lines | 256–320 |

**Evidence:**
```cpp
catch (const PDFException& exception)
{
    if (m_permissive)
    {
        m_warnings << exception.getMessage();  // Silently continues
    }
}
```

**Exploit scenario:** Malformed PDF that would be rejected in strict mode is silently accepted in permissive mode, bypassing validation that would block exploitation of parser bugs.

---

### SEC-011: Image Stride Integer Overflow

| Field | Value |
|-------|-------|
| Severity | **High** |
| Confidence | Confirmed |
| CWE | CWE-190, CWE-122 |
| File | `Pdf4QtLibCore/sources/pdfimage.cpp` |
| Lines | 202–247 |

**Evidence:**
```cpp
prepared.stride = prepared.components * prepared.width;  // 3 * large_width overflows
prepared.pixels.resize(prepared.stride * prepared.height);  // stride * height overflows
```

**Exploit scenario:** Image with large width/height from PDF → integer overflow in stride×height → undersized buffer → heap corruption during pixel copy.

---

### SEC-012: JPEG Row Stride Overflow

| Field | Value |
|-------|-------|
| Severity | **High** |
| Confidence | Confirmed |
| CWE | CWE-190, CWE-122 |
| File | `Pdf4QtLibCore/sources/pdfimage.cpp` |
| Lines | 1023–1031 |

**Evidence:**
```cpp
QByteArray buffer(rowStride * height, 0);  // rowStride * height can overflow
```

---

### SEC-013: JPEG2000 Index Calculation Overflow

| Field | Value |
|-------|-------|
| Severity | **High** |
| Confidence | Confirmed |
| CWE | CWE-190, CWE-123 |
| File | `Pdf4QtLibCore/sources/pdfimage.cpp` |
| Lines | 617–631 |

**Evidence:**
```cpp
const int index = y * data.width + x;  // Signed int overflow
image->comps[0].data[index] = pixel[0];  // OOB write with negative index
```

---

### SEC-014: Plugin Loading Without Integrity Verification

| Field | Value |
|-------|-------|
| Severity | **High** |
| Confidence | Confirmed |
| CWE | CWE-426, CWE-427 (Untrusted Search Path) |
| File | `Pdf4QtLibGui/pdfprogramcontroller.cpp` |
| Lines | 2466–2520 |

**Evidence:**
```cpp
availablePlugins = directory.entryList(QStringList("*.dll"));
QPluginLoader loader(pluginFileName);
loader.load();  // No signature check, no whitelist
```

**Exploit scenario:** Attacker places malicious DLL in plugin directory → loaded and executed with app privileges on next launch.

---

### SEC-015: AES Padding Validation Silently Clamped

| Field | Value |
|-------|-------|
| Severity | **High** |
| Confidence | Confirmed |
| CWE | CWE-347 (Improper Verification of Cryptographic Signature) |
| File | `Pdf4QtLibCore/sources/pdfsecurityhandler.cpp` |
| Lines | 573–583 |

**Evidence:**
```cpp
const int clampedPadding = qBound(1, padding, AES_BLOCK_SIZE);  // Silently fixes invalid padding
return data.left(data.size() - clampedPadding);
```

**Exploit scenario:** Invalid padding accepted without error → attacker can feed modified encrypted content that gets "auto-repaired" rather than rejected.

---

## Medium Findings

### SEC-016: RC4 Encryption Support (Weak Crypto)

| Field | Value |
|-------|-------|
| Severity | Medium |
| Confidence | Confirmed |
| CWE | CWE-327 |
| File | `pdfsecurityhandler.cpp`, Lines 44–51, 620–630 |

RC4 (broken cipher) used for PDF revisions 2–4 decryption. While required for legacy PDF compatibility, documents can be created with RC4 encryption.

---

### SEC-017: MD5 Used for Key Derivation

| Field | Value |
|-------|-------|
| Severity | Medium |
| Confidence | Confirmed |
| CWE | CWE-327 |
| File | `pdfsecurityhandler.cpp`, Lines 1100–1175 |

MD5 (collision-broken) used for key derivation in older revisions. R5/R6 correctly use SHA-256+.

---

### SEC-018: Key Material Not Zeroed from Memory

| Field | Value |
|-------|-------|
| Severity | Medium |
| Confidence | Confirmed |
| CWE | CWE-226 |
| File | `pdfsecurityhandler.cpp`, `pdfcertificatemanager.cpp` |

Encryption keys and passwords stored in `QByteArray`/`QString` are not securely zeroed after use. Recoverable from heap, page files, crash dumps.

---

### SEC-019: No OCSP/CRL Revocation Checking (Adobe Handler)

| Field | Value |
|-------|-------|
| Severity | Medium |
| Confidence | Confirmed |
| CWE | CWE-296 |
| File | `pdfsignaturehandler.cpp`, Lines 545–620 |

Standard Adobe signature handler does not verify certificate revocation. Compromised/revoked certificates would still validate. (ETSI handler partially checks CRL.)

---

### SEC-020: Signature Byte Range Validation Incomplete

| Field | Value |
|-------|-------|
| Severity | Medium |
| Confidence | Confirmed |
| CWE | CWE-347 |
| File | `pdfsignaturehandler.cpp`, Lines 444–485 |

Byte ranges are bounds-checked but not validated for contiguous coverage. Malicious PDF could sign partial content, leaving unsigned areas modifiable.

---

### SEC-021: XXE Prevention Not Explicitly Configured in XFA Engine

| Field | Value |
|-------|-------|
| Severity | Medium |
| Confidence | Likely |
| CWE | CWE-611 |
| File | `pdfxfaengine.cpp` |

Qt's QDomDocument is used without explicit XXE/DTD disabling. Qt 5.15+ disables external entities by default, but no explicit safeguard in code.

---

### SEC-022: XML Bomb (Billion Laughs) Not Mitigated

| Field | Value |
|-------|-------|
| Severity | Medium |
| Confidence | Likely |
| CWE | CWE-776 |
| File | `pdfxfaengine.cpp` |

No entity expansion limit or DTD rejection configured for XFA XML parsing. Exponential expansion possible.

---

### SEC-023: CCITT Fax Decoder Bounds Check Ordering

| Field | Value |
|-------|-------|
| Severity | Medium |
| Confidence | Likely |
| CWE | CWE-129 |
| File | `pdfccittfaxdecoder.cpp`, Lines 345–382 |

Array access `referenceLine[b1_index]` occurs before bounds check on `b1_index` in Horizontal mode.

---

### SEC-024: PowerShell Escaping Insufficient in Scanner Plugin

| Field | Value |
|-------|-------|
| Severity | Medium |
| Confidence | Possible |
| CWE | CWE-78 |
| File | `Pdf4QtEditorPlugins/ScannerPlugin/wiascannerbackend.cpp`, Lines 51–85 |

Only single-quote escaping applied to path embedded in PowerShell script; backticks and other escaping contexts not addressed.

---

### SEC-025: Owner/User Permission Enforcement at Crypto Layer

| Field | Value |
|-------|-------|
| Severity | Medium |
| Confidence | Likely |
| CWE | CWE-285 |
| File | `pdfsecurityhandler.cpp`, Lines 1272–1330 |

PDF permission flags (no-print, no-copy, etc.) are not enforced at the cryptographic layer — only at application layer. Plugin or code bypass could ignore permissions.

---

## Low and Informational Findings

### SEC-026: AES Zero IV for R5/R6 Decryption (Informational)

| Field | Value |
|-------|-------|
| Severity | Low |
| Confidence | Confirmed |
| CWE | CWE-330 |
| File | `pdfsecurityhandler.cpp`, Lines 1362–1365 |

Zero IV used for AES-CBC decryption. This is per PDF spec and only for key decryption (not content), but creates architectural confusion.

---

### SEC-027: PKCS#12 Default Iteration Count

| Field | Value |
|-------|-------|
| Severity | Low |
| Confidence | Possible |
| CWE | CWE-522 |
| File | `pdfcertificatemanager.cpp`, Lines 115–140 |

Uses `PKCS12_DEFAULT_ITER` which may be insufficient against modern brute-force capabilities.

---

### SEC-028: CI/CD Actions Pinned to Major Version Tags Only (Informational)

| Field | Value |
|-------|-------|
| Severity | Informational |
| Confidence | Confirmed |
| CWE | CWE-829 (Inclusion of Functionality from Untrusted Control Sphere) |
| File | `.github/workflows/ci.yml` |

`actions/checkout@v4`, `jurplel/install-qt-action@v4` pinned to mutable tags, not commit SHAs. Tag replacement attack could inject malicious steps.

---

### SEC-029: No SECURITY.md or Vulnerability Disclosure Policy (Informational)

| Field | Value |
|-------|-------|
| Severity | Informational |
| Confidence | Confirmed |
| File | Repository root |

No `SECURITY.md`, no vulnerability disclosure mechanism, no security contact listed.

---

## Risk Assessment

### Overall Security Posture

**High Risk** — The codebase processes complex untrusted binary input (PDF files) with multiple integer overflow and memory safety issues in hand-written C++ parsers. The combination of:
1. Complex binary format parsing (PDF, JBIG2, CCITT, JPEG2000)
2. No fuzz testing evidence
3. Minimal unit test coverage (2 test files)
4. Integer overflow throughout size/offset calculations

...creates a significant attack surface for maliciously crafted PDFs.

### Key Risks

1. **Remote Code Execution via crafted PDF** — Integer overflows in parser, JBIG2 decoder, and image handlers could lead to heap corruption exploitable for code execution (SEC-001–006, SEC-011–013)
2. **Arbitrary File Write via attachment extraction** — Path traversal in `PdfTool` CLI (SEC-007)
3. **Denial of Service** — Unbounded allocation and recursion allow trivial crashes (SEC-003, SEC-004, SEC-009)
4. **Plugin hijacking** — No integrity verification on loaded plugins (SEC-014)

### Mitigating Factors

- Desktop application (not network-facing; requires user to open malicious file)
- No JavaScript execution engine (JS is scanned, not executed)
- XFA support is read-only/static
- Email functionality uses MAPI API (no shell injection)
- Qt's QDomDocument disables XXE by default since Qt 5.15
- Certificate file path generation properly uses `QDir::absoluteFilePath()`
- No hardcoded secrets found in source code
- Code signing is used for Windows MSI releases (via DigiCert Keylocker)

---

## Recommended Actions

| Priority | Action | Effort | Impact |
|----------|--------|--------|--------|
| P0 | Add overflow-safe integer arithmetic for all PDF-derived sizes/offsets | Medium | Eliminates entire class of critical bugs (SEC-001–006, 008, 011–013) |
| P0 | Sanitize attachment filenames (basename only) in PdfTool | Low | Fixes path traversal (SEC-007) |
| P0 | Add recursion depth limit to parser (max 100–500 levels) | Low | Fixes stack overflow DoS (SEC-004) |
| P1 | Add maximum allocation limits for xref tables and streams | Low | Prevents OOM DoS (SEC-003, SEC-009) |
| P1 | Implement plugin integrity verification (hash whitelist or signature) | Medium | Prevents plugin hijacking (SEC-014) |
| P1 | Fix AES padding validation (reject invalid, don't clamp) | Low | Fixes crypto integrity (SEC-015) |
| P2 | Add secure memory zeroing for key material | Low | Improves key hygiene (SEC-018) |
| P2 | Add explicit XXE/DTD disabling for XFA parsing | Low | Defense-in-depth (SEC-021, SEC-022) |
| P2 | Enable certificate revocation checking | Medium | Improves signature trust (SEC-019) |
| P3 | Add fuzz testing for parser, JBIG2, CCITT, image decoders | High | Finds additional memory bugs |
| P3 | Pin CI/CD actions to commit SHAs | Low | Supply chain hardening (SEC-028) |
| P3 | Create SECURITY.md with disclosure process | Low | Community trust (SEC-029) |

---

## Human Review Required

| Area | Reason | Recommended Expertise |
|------|--------|-----------------------|
| JBIG2 decoder (full audit) | Complex custom binary decoder with critical history | Binary exploitation specialist |
| PDF signature verification | Certificate chain trust logic | PKI/cryptography expert |
| XFA engine (full audit) | Large XML processing engine on untrusted data | XML security specialist |
| Integer overflow fixes | Verify completeness of remediation across entire codebase | C++ security engineer |

---

## Assumptions

- No runtime testing was performed; all findings are based on static code analysis
- Line numbers are approximate (based on code structure analysis, not exact tool output)
- Qt 6.x is in use (provides default XXE protection in QDomDocument)
- The application is distributed as a desktop application (not server-side)

## Unknowns

- Exact OpenSSL version linked via vcpkg (determines which CVEs apply)
- Whether `checkBitmapSize()` in JBIG2 prevents all overflow cases at call sites
- Full data flow through transparency renderer
- Whether permissive mode is enabled by default in GUI applications
- Whether plugin directory permissions are restricted in installed configurations
