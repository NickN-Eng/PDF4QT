# Risky Areas to Review

## Repository
`c:\Users\nniem\source\repos\PDF4QT`

> **Note:** These are observations for triage, not confirmed vulnerabilities. Each flagged area should be investigated by the appropriate specialist skill.

## Priority Areas for Security Review

| Priority | Area | Location | Reason | Recommended Skill |
|----------|------|----------|--------|-------------------|
| 1 | PDF parser — untrusted binary input | `Pdf4QtLibCore/sources/pdfparser.cpp`, `pdfdocumentreader.cpp`, `pdfxreftable.cpp` | Primary attack surface: all untrusted PDF data enters here. Parser bugs in PDF libraries historically yield RCE, heap corruption, OOB reads. | `security-code-review` |
| 2 | Stream decoders — compressed/binary data | `pdfstreamfilters.cpp`, `pdfjbig2decoder.cpp`, `pdfccittfaxdecoder.cpp`, `pdfimage.cpp` | Complex binary format parsing (JBIG2, CCITT Fax, JPEG2000) — historically CVE-prone (e.g. JBIG2 exploits in Acrobat). Wraps third-party libs. | `security-code-review` |
| 3 | Cryptographic operations | `pdfsecurityhandler.cpp`, `pdfsignaturehandler.cpp`, `pdfcertificatemanager.cpp` | RC4 (legacy, broken cipher) used for lower encryption revisions. Key derivation, IV handling, AES-CBC padding, PKCS#7 verification. | `security-code-review` |
| 4 | XFA engine | `pdfxfaengine.cpp` | XML parsing of data embedded in untrusted PDFs. XML parsers can be vulnerable to XXE, entity expansion (billion laughs), or malformed input. | `security-code-review` |
| 5 | JavaScript content extraction | `pdfjavascriptscanner.cpp`, `pdftoolinfojavascript.cpp` | Extracts JS strings from PDF for display. No execution observed, but verify no eval/script engine is invoked and that output is safely escaped in UI. | `security-code-review` |

## Security-Sensitive Code

### Authentication and Authorization

| Location | Observation | Risk Level |
|----------|-------------|-----------|
| `pdfsecurityhandler.cpp` | Implements Standard and Public Key security handlers. Password-based authentication with RC4 (40-bit and 128-bit, revisions 2–3) and AES (revision 4–6). RC4 is cryptographically broken. | High |
| `pdfsecurityhandler.cpp` | Owner password vs. user password distinction — verify that permission flags (printing, copying, editing) are correctly enforced after decryption. | Medium |
| `pdfcertificatemanager.cpp` | Manages PKCS#12 private key files; `privateKeyPasword` field (note: typo in source) stores key password in a struct. Verify memory handling of key material. | Medium |
| `pdfsignaturehandler.cpp`, `pdfsignaturehandler_impl.h` | Signature verification via OpenSSL PKCS#7. Verify certificate chain validation, revocation checking (OCSP/CRL), and handling of malformed signature byte ranges. | High |
| `SignaturePlugin/signdialog.*` | User-facing digital signing UI — private key password entry. Verify password is not logged or persisted. | Medium |

### Input Handling and Validation

| Location | Input Source | Observation | Risk Level |
|----------|-------------|-------------|-----------|
| `pdfparser.cpp` | PDF file bytes | Custom hand-written PDF tokenizer/parser. No fuzz testing observed. Integer arithmetic on offsets and lengths from untrusted file. | High |
| `pdfdocumentreader.cpp` | PDF file / buffer / QIODevice | Entry point for all PDF data. `readFromBuffer()` accepts arbitrary `QByteArray`. Permissive parsing mode (`permissive` constructor flag) may silently accept malformed PDFs. | High |
| `pdfxreftable.cpp` | PDF cross-reference table | XRef table parsing involves seeking to attacker-controlled offsets within the file. Off-by-one or integer overflow in offset arithmetic is a risk. | High |
| `pdfjbig2decoder.cpp` | JBIG2 stream in PDF | Complex binary compression format. JBIG2 has a history of critical vulnerabilities (CVE-2009-0658 class). Custom decoder implementation. | Critical |
| `pdfxfaengine.cpp` | XFA XML in PDF | Qt XML parsing of attacker-controlled XML. Check for XXE (external entity) protection — Qt's XML parser disables external entities by default, but verify. | High |
| `pdfimage.cpp` | Image streams in PDF | Coordinates JPEG, JPEG2000, PNG, and raw image decoding. Buffer sizes derived from PDF-embedded width/height values — verify bounds checking. | High |
| `pdfccittfaxdecoder.cpp` | CCITT Fax stream | Custom CCITT Group 3/4 decoder. Bit-level parsing of attacker-controlled data. | Medium |
| `pdfobjectutils.cpp`, `pdfobject.cpp` | PDF object values | String, name, array, dict parsing. Verify no unbounded recursion on deeply nested structures. | Medium |
| `pdfdocumentsanitizer.cpp` | PDF document | Sanitisation logic — verify it correctly removes all JavaScript, actions, and external references. | Medium |

### Cryptography and Secrets

| Location | Observation | Risk Level |
|----------|-------------|-----------|
| `pdfsecurityhandler.cpp` | RC4 encryption supported for legacy PDFs (revisions 2–3). RC4 is broken; verify no new PDFs are created with RC4 by default. | High |
| `pdfsecurityhandler.cpp` | AES-256 (AESV3) for revision 6. Verify correct use of AES-CBC vs. AES-CFB, IV generation, and padding. | Medium |
| `pdfcertificatemanager.h` | `privateKeyPasword` stored as `QString` in `NewCertificateInfo` struct. Verify it is cleared from memory after use and not serialised to settings. | Medium |
| `Pdf4QtLibCore/aatl/` | Bundled AATL (Adobe Approved Trust List). Verify the bundled list is current and the update mechanism (if any) is trustworthy. | Low |

### External Integrations

| Location | Service | Observation | Risk Level |
|----------|---------|-------------|-----------|
| `SignaturePlugin/`, `pdfsignaturehandler.cpp` | OpenSSL | PKCS#7/CMS signature verification — check for certificate pinning, full chain validation, handling of self-signed certs. | High |
| `pdfcms.cpp` | LittleCMS | ICC profile parsing from PDF. LittleCMS has had CVEs related to malformed ICC profiles (e.g. CVE-2018-16435). Verify version used via vcpkg. | Medium |
| `pdfsendmail.cpp` | System mailer | Verify no command injection when constructing mailer arguments with user-supplied filenames. | Medium |
| `Pdf4QtEditorPlugins/ScannerPlugin/` | System scanner | If shell commands are invoked for scanner access, verify no injection via device name. | Medium |

### Command Execution / Shell Invocation

| Location | Observation | Risk Level |
|----------|-------------|-----------|
| `pdfsendmail.cpp` | Sends email via system mailer. If it uses `QProcess` with a user-supplied filename in arguments, a maliciously named PDF file could lead to argument injection. | Medium |
| `Pdf4QtEditorPlugins/ScannerPlugin/` | Scanner interaction may invoke external processes. Verify input sanitisation. | Possible |

## Licence Review Areas

| Location | Concern | Priority |
|----------|---------|----------|
| `3rdparty_licenses/` | Bundled licence texts for: FreeType (FTL), libjpeg (IJG), LittleCMS (MIT), OpenJPEG (2-clause MIT), OpenSSL (Apache 2.0), zlib. Verify all bundled libraries are listed and attribution is complete. | Medium |
| Top-level `LICENSE` | Recently relicensed from LGPLv3 to MIT (April 2025). Verify all source files have updated headers (some may still carry LGPLv3 notices). | Medium |
| Qt framework | Used under LGPL; verify dynamic linking requirement is met for distribution (not statically linked). | Medium |
| Blend2D | zlib licence — verify compliance. | Low |
| Intel TBB | Apache 2.0 — included only on Linux/GCC. | Low |

## Dependency Review Areas

| Manifest/Lockfile | Concern | Priority |
|-------------------|---------|----------|
| `vcpkg.json` | OpenSSL — verify version is current (no known CVEs). High-value target. | High |
| `vcpkg.json` | FreeType — font parsing of untrusted embedded fonts; historical CVEs. Verify version. | Medium |
| `vcpkg.json` | OpenJPEG (JPEG2000) — complex format; CVEs in past versions. Verify version. | Medium |
| `vcpkg.json` | libjpeg-turbo — JPEG parsing; verify version. | Medium |
| `vcpkg.json` | LittleCMS — ICC profile parsing; historical CVEs. Verify version. | Medium |
| `vcpkg.json` | zlib — deflate; verify version (CVE-2022-37434 in older zlib). | Medium |
| `vcpkg.json` | blend2d — 2D rendering; relatively new library, less audit history. | Low |
| No lockfile found | vcpkg does not produce a lock file in the classic sense; `vcpkg-configuration.json` pins baselines. Verify the pinned baseline is current. | Low |

## CI/CD and Supply Chain

| Workflow/Config | Concern | Priority |
|-----------------|---------|----------|
| `.github/workflows/ci.yml` | `actions/checkout@v4`, `jurplel/install-qt-action@v4`, `actions/cache@v4` — all pinned to major version tags, not commit SHAs. A compromised action tag could inject malicious build steps. | Medium |
| `.github/workflows/ci.yml` | Clones vcpkg from GitHub via `git clone --depth=1` without a pinned commit SHA (uses `vcpkg-configuration.json` baseline instead). Verify baseline pins are enforced. | Medium |
| `Dockerfile` | Installs Qt via `pip install aqtinstall` + `aqt install-qt` — verify aqtinstall is pinned to a known version. | Low |
| `Dockerfile` | `git clone https://github.com/Microsoft/vcpkg.git` without a pinned ref in the Dockerfile. | Low |
| `.github/workflows/` | No `SECURITY.md` or vulnerability disclosure policy found in the repository. | Low |
| `.github/workflows/CreateReleaseDraft.yml` | Review whether release artifacts are signed or checksummed. | Low |

## Areas with Limited Visibility

| Area | Why Limited | What Would Help |
|------|-------------|-----------------|
| JBIG2 decoder implementation | Complex custom binary decoder (`pdfjbig2decoder.cpp`) not fully read | Full code review + fuzz testing |
| XFA engine | Large, complex XML processing (`pdfxfaengine.cpp`) | XML security review + fuzz testing |
| Password/key memory handling | C++ memory management of sensitive strings not traced | Memory safety audit; use of `SecureZeroMemory`/`OPENSSL_cleanse` patterns |
| Plugin trust model | Runtime plugin loading via `QPluginLoader` — no code-signing verification observed | Review plugin discovery/loading code in `pdfplugin.cpp` |
| Test coverage | Only 2 unit test files for 100+ source files | Code coverage analysis |

## Recommended Review Order

1. **PDF parser and XRef table** (`pdfparser.cpp`, `pdfxreftable.cpp`, `pdfdocumentreader.cpp`) — primary untrusted input ingestion; highest exploitation risk
2. **JBIG2 and stream decoders** (`pdfjbig2decoder.cpp`, `pdfstreamfilters.cpp`, `pdfccittfaxdecoder.cpp`) — complex binary parsing; historically critical CVE class
3. **Security handler and signature verification** (`pdfsecurityhandler.cpp`, `pdfsignaturehandler.cpp`) — cryptographic correctness and certificate validation logic
4. **XFA engine** (`pdfxfaengine.cpp`) — XML parsing of attacker-controlled data
5. **Dependencies** (via `dependency-vulnerability-review`) — check OpenSSL, FreeType, OpenJPEG, libjpeg-turbo, LittleCMS versions for known CVEs

## Notes

- These are **observations for triage**, not confirmed vulnerabilities.
- Each flagged area should be investigated by the `security-code-review` or `dependency-vulnerability-review` skill as indicated.
- Absence from this list does not mean an area is safe.
- The project's primary risk profile is **file parsing safety** — a sophisticated attacker could craft a malicious PDF that exploits parsing bugs when opened.

## Assumptions

- JavaScript embedded in PDFs is not executed (confirmed by code scan and README).
- No server-side or networked deployment is present — this is a desktop application; network exposure is minimal.
- The plugin system loads from a known directory set at compile time — but the security of that directory (write permissions) is a deployment concern.
