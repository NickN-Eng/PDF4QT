# Entry Points and Data Flows

## Repository
`c:\Users\nniem\source\repos\PDF4QT`

## Entry Points

### CLI Commands

`PdfTool` exposes 30+ sub-commands via `PDFToolApplicationStorage`. Each sub-command accepts a PDF file path as primary input.

| Command | Handler File | Purpose | Accepts External Input |
|---------|-------------|---------|----------------------|
| `render` | `pdftoolrender.cpp` | Render pages to image files | Yes (PDF path, output path) |
| `fetch-text` | `pdftoolfetchtext.cpp` | Extract text from PDF | Yes (PDF path) |
| `fetch-images` | `pdftoolfetchimages.cpp` | Extract embedded images | Yes (PDF path) |
| `encrypt` | `pdftoolencrypt.cpp` | Encrypt PDF with password/public key | Yes (PDF path, password/cert) |
| `decrypt` | `pdftooldecrypt.cpp` | Decrypt PDF | Yes (PDF path, password) |
| `sign` | (via `pdftoolverifysignatures.cpp`) | Verify/create digital signatures | Yes (PDF path, key/cert) |
| `diff` | `pdftooldiff.cpp` | Compare two PDF documents | Yes (two PDF paths) |
| `optimize` | `pdftooloptimize.cpp` | Compress/optimise PDF | Yes (PDF path) |
| `redact` | `pdftoolredact.cpp` | Redact content | Yes (PDF path, redaction spec) |
| `separate` | `pdftoolseparate.cpp` | Split PDF into pages | Yes (PDF path) |
| `unite` | `pdftoolunite.cpp` | Merge PDFs | Yes (multiple PDF paths) |
| `info-javascript` | `pdftoolinfojavascript.cpp` | Extract JavaScript from PDF | Yes (PDF path) |
| `xml` | `pdftoolxml.cpp` | Export PDF structure as XML | Yes (PDF path) |
| `audio-book` | `pdftoolaudiobook.cpp` | Convert PDF to audio book | Yes (PDF path) |
| `attachments` | `pdftoolattachments.cpp` | Manage embedded file attachments | Yes (PDF path) |
| `ink-coverage` | `pdftoolinkcoverage.cpp` | Calculate ink coverage | Yes (PDF path) |
| `cert-store` | `pdftoolcertstore.cpp` | Manage certificate store | Yes (cert files) |
| `remove-external-links` | `pdftoolremoveexternallinks.cpp` | Strip external hyperlinks | Yes (PDF path) |
| `sanitize` | (via `pdfdocumentsanitizer.cpp`) | Sanitize PDF document | Yes (PDF path) |
| `statistics` | `pdftoolstatistics.cpp` | Print document statistics | Yes (PDF path) |
| `info-*` | `pdftoolinfo*.cpp` | Various info extraction commands | Yes (PDF path) |

### GUI Application Entry Points

| Application | Entry File | Primary User Action |
|-------------|-----------|-------------------|
| Pdf4QtViewer | `Pdf4QtViewer/main.cpp` | File open dialog / drag-drop / command-line path argument |
| Pdf4QtEditor | `Pdf4QtEditor/main.cpp` | File open dialog / drag-drop / command-line path argument |
| Pdf4QtPageMaster | `Pdf4QtPageMaster/` | File open dialog |
| Pdf4QtDiff | `Pdf4QtDiff/main.cpp` | Two file open dialogs |
| Pdf4QtLaunchPad | `Pdf4QtLaunchPad/main.cpp` | Launch hub for other applications |

### Plugin Entry Points (loaded by Editor at runtime)

| Plugin | Location | Trigger |
|--------|----------|---------|
| `SignaturePlugin` | `Pdf4QtEditorPlugins/SignaturePlugin/` | User initiates digital signing via menu |
| `RedactPlugin` | `Pdf4QtEditorPlugins/RedactPlugin/` | User applies redaction |
| `ScannerPlugin` | `Pdf4QtEditorPlugins/ScannerPlugin/` | Scan document / OCR |
| `AudioBookPlugin` | `Pdf4QtEditorPlugins/AudioBookPlugin/` | TTS / audio book generation |
| `DimensionsPlugin` | `Pdf4QtEditorPlugins/DimensionsPlugin/` | Measurement annotations |
| `EditorPlugin` | `Pdf4QtEditorPlugins/EditorPlugin/` | Page content editing |
| `ObjectInspectorPlugin` | `Pdf4QtEditorPlugins/ObjectInspectorPlugin/` | Low-level PDF object inspection |
| `OutputPreviewPlugin` | `Pdf4QtEditorPlugins/OutputPreviewPlugin/` | Print/colour output preview |
| `SoftProofingPlugin` | `Pdf4QtEditorPlugins/SoftProofingPlugin/` | Colour soft-proofing |

### Other Entry Points

| Type | Location | Purpose | Notes |
|------|----------|---------|-------|
| File open (drag-drop) | GUI main windows | Open PDF by dropping onto window | Common secondary entry point |
| Command-line path arg | All GUI apps | Open PDF passed as argv[1] | Checked at startup |
| Email send | `pdfsendmail.cpp` | Export/send PDF via system mailer | Output path |
| Certificate import | `pdfcertificatemanager.cpp` | Import PKCS#12 cert from file | Reads untrusted `.p12`/`.pfx` files |

## Data Flows

### Inbound Data

| Source | Entry Point | Data Type | Validation | Storage |
|--------|-------------|-----------|-----------|---------|
| Local filesystem (PDF file) | `PDFDocumentReader::readFromFile()` | Binary PDF stream | PDF structure validation in parser; password prompt if encrypted | In-memory `PDFDocument` object graph |
| `QIODevice` / `QByteArray` | `PDFDocumentReader::readFromDevice()` / `readFromBuffer()` | Binary PDF stream | Same as above | In-memory |
| Command-line arguments | `QCommandLineParser` in `main.cpp` | Strings (file paths, passwords) | Minimal — paths passed to file open | Transient |
| Certificate files (.p12/.pfx) | `PDFCertificateManager` | Binary PKCS#12 | OpenSSL parsing | Local cert directory |
| ICC colour profiles | `pdfcms.cpp` | Binary ICC profile | LittleCMS parsing | In-memory |
| Embedded attachments (in PDF) | `pdfattachments.cpp` | Arbitrary binary data | None beyond stream decode | Re-extracted to filesystem by user |
| JavaScript (in PDF) | `pdfjavascriptscanner.cpp` | JS text strings | Scan/extract only; no execution observed | In-memory scan results |
| XFA forms (in PDF) | `pdfxfaengine.cpp` | XML | Qt XML parser | In-memory XFA document |

### Outbound Data

| Destination | Exit Point | Data Type | Sensitivity | Notes |
|-------------|-----------|-----------|-------------|-------|
| Filesystem (modified PDF) | `PDFDocumentWriter` | Binary PDF | May contain user data, signatures | Primary output |
| Filesystem (rendered images) | `pdftoolrender.cpp` | PNG/JPEG/etc. | Page content | CLI render command |
| Filesystem (extracted text) | `pdftoolfetchtext.cpp` | Plain text | Document content | CLI extract |
| Filesystem (extracted images) | `pdftoolfetchimages.cpp` | Image files | Embedded image data | CLI extract |
| Printer (system) | Qt PrintSupport | Rendered pages | Document content | Via print dialog |
| System mailer | `pdfsendmail.cpp` | PDF file attachment | Full document | Email send feature |
| Audio output (TTS) | `pdftexttospeech.cpp` | Audio stream | Document text | Qt TextToSpeech |
| XML output | `pdftoolxml.cpp` | XML text | PDF structure/content | CLI export |

### Internal Data Flows

| From | To | Data | Mechanism | Notes |
|------|-----|------|-----------|-------|
| `PDFDocumentReader` | `PDFParser` | Raw byte buffer | Direct call | Parses PDF token stream |
| `PDFParser` | `PDFDocument` | `PDFObject` graph | In-memory | Builds document object tree |
| `PDFDocument` | `PDFPageContentProcessor` | Page content streams | Per-page rendering | Processes PDF operators |
| `PDFPageContentProcessor` | `PDFRenderer` / `PDFTransparencyRenderer` | Draw calls | Qt painter / Blend2D | Renders to `QImage` |
| `PDFDocument` | `PDFSecurityHandler` | Encrypted byte data | Per-object decryption | Decrypts strings and streams |
| `PDFSecurityHandler` | OpenSSL | Key material | OpenSSL API | AES/RC4/RSA operations |
| `PDFPageContentProcessor` | `PDFStreamFilterStorage` | Compressed stream data | Decode chain | Decompresses zlib/JPEG/JBIG2/etc. |
| `PDFDocument` | `PDFSignatureHandler` | Signature byte ranges | OpenSSL PKCS#7 verify | Validates signatures |
| `PDFDocument` | `PDFXFAEngine` | XFA XML data | Qt XML | Renders static XFA forms |

## Sensitive Data Paths

| Data Type | Where Enters | Where Processed | Where Stored | Where Exits |
|-----------|-------------|-----------------|--------------|-------------|
| PDF password | User input dialog / CLI arg | `PDFSecurityHandler` | Transient (used to derive encryption key) | Never persisted |
| Private key (PKCS#12) | Filesystem import | `PDFCertificateManager` + OpenSSL | Local cert directory | Used for signing; key in memory during op |
| Document content (PII) | PDF file open | `PDFDocument` in-memory | RAM only | Written to output PDF / printed / emailed |
| Digital signatures | PDF file | `PDFSignatureHandler` | In-memory verification result | Shown in UI; not re-exported separately |
| Embedded file attachments | PDF stream | `PDFFile` | In-memory; user extracts to filesystem | User-chosen location on filesystem |
| JavaScript strings | PDF stream | `PDFJavaScriptScanner` | In-memory scan results | Displayed in UI (not executed) |

## Trust Boundaries

| Boundary | Components Inside | Components Outside | Crossing Points |
|----------|------------------|-------------------|-----------------|
| Trusted application | Pdf4QtLibCore, LibGui, LibWidgets, applications | PDF files (untrusted, adversarial input) | `PDFDocumentReader::readFromFile/Buffer/Device` |
| Trusted application | Pdf4QtLibCore | Certificate/PKCS#12 files | `PDFCertificateManager` |
| Trusted application | PDFXFAEngine | XFA XML embedded in PDF | `QXmlStreamReader` or Qt XML parser on PDF-embedded XML |
| Trusted application | Plugin loader | Plugin shared libraries on disk | Runtime `QPluginLoader` / dlopen |
| Trusted application | `pdfsendmail.cpp` | System mailer process | Process launch |

## Confidence Assessment

| Aspect | Confidence | Notes |
|--------|-----------|-------|
| Entry points completeness | Likely | CLI commands enumerated from directory listing; GUI entry points confirmed |
| Data flow accuracy | Likely | Traced from class names and headers; not fully verified in implementation |
| Trust boundaries | Confirmed | PDF input as untrusted is architecturally clear |
| Plugin trust boundary | Possible | Plugin validation (code signing etc.) not confirmed from available headers |

## Assumptions

- JavaScript embedded in PDFs is **scanned/extracted only** and not executed by the library itself (confirmed: `pdfjavascriptscanner.cpp` scans; no JS engine found).
- XFA support is read-only / static rendering (confirmed by README: "static XFA support (readonly, simple XFA only)").
- Network access is not performed by the core library (no network code found in LibCore headers).
