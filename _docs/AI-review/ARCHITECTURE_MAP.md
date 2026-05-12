# Architecture Map

## Repository
`c:\Users\nniem\source\repos\PDF4QT`

## High-Level Architecture

PDF4QT follows a layered library + application architecture. A core engine library handles all PDF parsing and processing; GUI layers sit on top; end-user applications and plugins consume those layers.

```
┌─────────────────────────────────────────────────────────────────┐
│  Applications                                                   │
│  Pdf4QtViewer | Pdf4QtEditor | Pdf4QtPageMaster | Pdf4QtDiff    │
│  PdfTool (CLI) | Pdf4QtLaunchPad | JBIG2_Viewer                 │
├─────────────────────────────────────────────────────────────────┤
│  Plugins                                                        │
│  Pdf4QtEditorPlugins (9 plugins: Signature, Redact, Scanner…)   │
├─────────────────────────────────────────────────────────────────┤
│  GUI Library                                                    │
│  Pdf4QtLibGui   (main windows, dialogs, settings, TTS, UndoRedo)│
├─────────────────────────────────────────────────────────────────┤
│  Widgets Library                                                │
│  Pdf4QtLibWidgets  (viewer widget, annotation tools, toolbar)   │
├─────────────────────────────────────────────────────────────────┤
│  Core PDF Engine                                                │
│  Pdf4QtLibCore  (parser, renderer, security, fonts, streams)    │
├─────────────────────────────────────────────────────────────────┤
│  Third-Party (via vcpkg / Qt)                                   │
│  OpenSSL | zlib | FreeType | OpenJPEG | libjpeg-turbo |         │
│  LittleCMS | Blend2D | libpng | TBB | Qt6                       │
└─────────────────────────────────────────────────────────────────┘
```

## Component Structure

| Component | Location | Purpose | Key Dependencies |
|-----------|----------|---------|-----------------|
| `Pdf4QtLibCore` | `Pdf4QtLibCore/sources/` | Core PDF engine: parsing, rendering, security, encryption, fonts, images, forms, annotations, signatures, XFA, diff | OpenSSL, zlib, FreeType, OpenJPEG, libjpeg, lcms2, Blend2D, Qt6::Core/Gui |
| `Pdf4QtLibWidgets` | `Pdf4QtLibWidgets/sources/` | Reusable PDF viewer and annotation widgets | Pdf4QtLibCore, Qt6::Widgets |
| `Pdf4QtLibGui` | `Pdf4QtLibGui/` | Shared GUI: main window framework, dialogs, viewer settings, TTS, undo/redo, recent files | Pdf4QtLibCore, Pdf4QtLibWidgets, Qt6::Widgets/PrintSupport/TextToSpeech |
| `Pdf4QtViewer` | `Pdf4QtViewer/` | Standalone viewer application (thin shell over LibGui) | Pdf4QtLibGui |
| `Pdf4QtEditor` | `Pdf4QtEditor/` | Full-featured editor application | Pdf4QtLibGui, plugins |
| `Pdf4QtPageMaster` | `Pdf4QtPageMaster/` | Page manipulation (split, merge, reorder) | Pdf4QtLibCore, Pdf4QtLibWidgets |
| `Pdf4QtDiff` | `Pdf4QtDiff/` | Side-by-side document comparison | Pdf4QtLibCore (`pdfdiff.cpp`) |
| `PdfTool` | `PdfTool/` | CLI tool with 30+ sub-commands | Pdf4QtLibCore, Qt6::Core |
| `Pdf4QtLaunchPad` | `Pdf4QtLaunchPad/` | Application launcher / hub | Qt6::Widgets |
| `Pdf4QtEditorPlugins` | `Pdf4QtEditorPlugins/` | 9 dynamically-loaded plugins extending editor | Pdf4QtLibCore, Pdf4QtLibWidgets |
| `CodeGenerator` | `CodeGenerator/` | Developer tool: generates C++ code from XML definitions | Qt6 |
| `JBIG2_Viewer` | `JBIG2_Viewer/` | Debug/developer viewer for JBIG2 images | Pdf4QtLibCore |
| `PdfExampleGenerator` | `PdfExampleGenerator/` | Tool for generating test/example PDFs | Pdf4QtLibCore |
| `UnitTests` | `UnitTests/` | Qt Test unit test suite | Pdf4QtLibCore, Qt6::Test |

## Layers

```
Layer 1: Applications & CLI
  → consume Pdf4QtLibGui or Pdf4QtLibCore directly (PdfTool, PageMaster)

Layer 2: Plugins
  → loaded dynamically at runtime by editor applications
  → depend on Pdf4QtLibCore + Pdf4QtLibWidgets

Layer 3: Pdf4QtLibGui (GUI framework)
  → PDFProgramController orchestrates application lifecycle
  → Manages viewer, editor main windows, settings, TTS, undo/redo

Layer 4: Pdf4QtLibWidgets (Widget layer)
  → PDF viewer widget, annotation tools, sidebar, search

Layer 5: Pdf4QtLibCore (Engine)
  → PDFDocumentReader → PDFParser → PDFObject graph
  → PDFPageContentProcessor → PDFRenderer / PDFTransparencyRenderer
  → PDFSecurityHandler → OpenSSL (RC4, AES, RSA)
  → PDFSignatureHandler → OpenSSL PKCS#7
  → Stream filters (Deflate/zlib, JPEG, JPEG2000, JBIG2, CCITT Fax, LZW)
  → Font subsystem (FreeType, built-in CMap tables)
  → Colour management (LittleCMS)
  → Blend2D painter backend
```

### Layer Descriptions

| Layer | Purpose | Key Files |
|-------|---------|-----------|
| Applications | Entry points, `main()`, app-specific UI | `Pdf4QtEditor/main.cpp`, `PdfTool/main.cpp`, `Pdf4QtViewer/`, `Pdf4QtPageMaster/`, `Pdf4QtDiff/mainwindow.cpp` |
| Plugins | Optional, dynamically loaded editor extensions | `Pdf4QtEditorPlugins/SignaturePlugin/`, `RedactPlugin/`, `ScannerPlugin/`, `AudioBookPlugin/`, etc. |
| LibGui | Shared GUI framework, program controller | `pdfprogramcontroller.cpp`, `pdfviewermainwindow.cpp`, `pdfeditormainwindow.cpp`, `pdfviewersettings.cpp` |
| LibWidgets | PDF-specific widgets | (sources in `Pdf4QtLibWidgets/sources/`) |
| LibCore | PDF parsing, rendering, security | `pdfparser.cpp`, `pdfdocumentreader.cpp`, `pdfrenderer.cpp`, `pdfsecurityhandler.cpp`, `pdfsignaturehandler.cpp`, `pdfstreamfilters.cpp`, `pdfxfaengine.cpp` |

## Module Relationships

- All applications link against `Pdf4QtLibGui` (except `PdfTool` and `Pdf4QtPageMaster` which link `Pdf4QtLibCore`/`Pdf4QtLibWidgets` directly).
- Plugins are shared libraries discovered and loaded at runtime; they use the `pdfplugin.h` interface from `Pdf4QtLibCore`.
- `Pdf4QtLibGui` contains `PDFProgramController`, which is the central orchestrator for document lifecycle, plugin loading, and action dispatch.
- `PDFDocumentReader` (LibCore) is the primary ingestion point: reads from file path, `QIODevice`, or `QByteArray`.

## External Integrations

| Service/System | Integration Point | Protocol | Files |
|----------------|-------------------|----------|-------|
| OpenSSL (crypto) | Encryption/decryption, PKCS#7 signatures, certificate management | Library calls | `pdfsecurityhandler.cpp`, `pdfsignaturehandler.cpp`, `pdfcertificatemanager.cpp`, `SignaturePlugin/` |
| Qt TextToSpeech | Audio book / TTS output | Qt API | `pdftexttospeech.cpp`, `AudioBookPlugin/` |
| Qt PrintSupport | PDF → printer output | Qt API | `Pdf4QtLibGui` |
| System fonts / FreeType | Font rasterisation | Library calls | `pdffont.cpp` |
| LittleCMS | ICC colour profile conversion | Library calls | `pdfcms.cpp` |
| Windows TaskBar | Progress display on Windows | WinAPI/Qt | `pdfwintaskbarprogress.cpp` |
| Email (`pdfsendmail.cpp`) | Sending PDF via email | Platform mail API | `pdfsendmail.cpp` |

## Data Storage

| Store | Type | Location in Code | Purpose |
|-------|------|-----------------|---------|
| PDF files (local filesystem) | Binary file | `pdfdocumentreader.cpp`, `pdfdocumentwriter.cpp` | Primary input/output |
| Certificate store | Local directory (PKCS#12 files) | `pdfcertificatemanager.cpp`, `pdfcertificatestore.cpp` | Digital signature keys/certs |
| Application settings | Qt settings (registry on Windows, INI on Linux) | `pdfviewersettings.cpp` | User preferences |
| AATL trust list | Bundled resource (`aatl/`) | `Pdf4QtLibCore/aatl/` | Adobe Approved Trust List for signature validation |
| CMap tables | Bundled binary resources (`cmaps/`) | `Pdf4QtLibCore/cmaps/` | PDF font encoding maps |
| Liberation fonts | Bundled TTF resources | `Pdf4QtLibCore/liberation-fonts-ttf/` | Fallback fonts |
| Recent files list | Qt settings | `pdfrecentfilemanager.cpp` | MRU list |

## Plugin/Extension Points

| Extension Point | Mechanism | Location |
|-----------------|-----------|----------|
| Editor plugins | Qt shared libraries (`.dll`/`.so`) discovered from `pdfplugins/` directory at runtime | `Pdf4QtEditorPlugins/`, `pdfplugin.h` |
| Code generator (dev) | XML-driven C++ code generation tool | `CodeGenerator/`, `generated_code_definition.xml` |

## Build and Deployment

### Build Process
CMake 3.16+; vcpkg resolves C++ dependencies. Qt must be installed separately (via aqtinstall in CI/Docker). AUTOMOC, AUTOUIC, AUTORCC are all enabled. Translation targets built with `qt_collect_translation_source_targets`.

### Deployment Model
- **Windows**: MSI installer via WixInstaller; or standalone binary zip
- **Linux**: Flatpak (Flathub), AppImage, or Arch AUR package
- **Docker**: `Dockerfile` builds full Ubuntu-based image with all dependencies

### Environments
- Windows (MSVC or MinGW)
- Linux (GCC; TBB required for parallelism)
- Linux Flatpak (special `PDF4QT_FLATPAK_BUILD` flag)
- Docker (Ubuntu 22.04)

## Confidence Assessment

| Aspect | Confidence | Notes |
|--------|-----------|-------|
| Component boundaries | Confirmed | Clearly delineated by CMake subdirectories |
| Layer structure | Confirmed | Verified via CMakeLists.txt `target_link_libraries` |
| External integrations | Confirmed | Cited in CMakeLists.txt and source headers |
| Data flow | Likely | Inferred from class names and headers; not traced end-to-end in source |

## Assumptions

- Plugin discovery path (`pdfplugins/` or `pdf4qt/` on Linux) is set at compile time via `config.h`.
- No network communication is present beyond email sending and Qt's own update mechanisms (not observed).
