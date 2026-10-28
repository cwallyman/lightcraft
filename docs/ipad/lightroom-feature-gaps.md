# Lightroom features missing or incomplete in LightCraft

Reviewed 2026-10-16. LightCraft source snapshot: `f34c7a8a89e0d1bade0a6889185f40acecc84a9f`
(main, version 0.8.0). The wrapper initially bundles the separately pinned official v0.8.0 web
release; source main can be ahead of that release. This is a feature inventory from source,
upstream tracking documents, and Adobe documentation, not a hands-on fidelity certification.

Lightroom desktop/mobile/web and **Lightroom Classic** have different capabilities. A Classic-only
feature below is not a claim that Lightroom on iPad has it. Availability also depends on Adobe's
platform, hardware, plan and supported image type. “Missing” means absent from this LightCraft
snapshot. “Partial” means an implementation exists with a specific limitation. “Outside scope”
means the LightCraft team intentionally excludes it; it remains absent from the user's perspective.

## Missing editing and organization features

| Feature Lightroom offers | LightCraft status | Source evidence / practical difference |
|---|---|---|
| AI Lens Blur with inferred depth and bokeh | Missing | `LR-EDIT-LENSBLUR`: a settings field is reserved but no rendered tool. [Adobe Lens Blur](https://helpx.adobe.com/ie/lightroom-classic/desktop/process-and-develop-photos/lens-blur.html) |
| Raw Details / enhanced demosaicing | Missing | `LR-EDIT-DETAIL-RAWDETAILS`, `LR-ENH-*`; no equivalent Enhance workflow. [Adobe Enhance](https://helpx-origin.aws116.adobeitc.com/ie/lightroom-classic/help/enhance-details.html) |
| Super Resolution | Missing | `LR-EDIT-DETAIL-SUPERRES`; ordinary export resizing is present and is not this feature. [Adobe Enhance](https://helpx-origin.aws116.adobeitc.com/ie/lightroom-classic/help/enhance-details.html) |
| AI Sharpen and Generative Upscale | Outside scope / no equivalent found | AI Sharpen explicitly excluded; no generative upscaler found in the source feature inventory. These newer Adobe capabilities are platform/plan dependent. [Adobe AI enhancement](https://helpx.adobe.com/lightroom-cc/using/enhance-images-with-generative-ai.html) |
| Generative Remove and generative people removal | Outside scope | `LR-REM-GEN`, `LR-REM-PEOPLE`; Heal and Clone do exist. [Adobe Generative Remove](https://blog.adobe.com/en/publish/2024/05/21/lightroom-introduces-power-adobe-firefly-with-generative-remove) |
| Generative Expand | Outside scope | `LR-CROP-GENEXPAND`; not the same as the implemented panorama Fill Edges. [Adobe AI capabilities by platform](https://helpx.adobe.com/lightroom/web/edit-photos/manage-ai-edits.html) |
| Reflection removal | Missing | `LR-REM-REFLECT`. [Adobe AI capabilities by platform](https://helpx.adobe.com/lightroom/web/edit-photos/manage-ai-edits.html) |
| Object detection specifically for the Remove tool | Missing | `LR-REM-DETECT`; separate from subject selection/masking. |
| People-part masks (skin, eyes, hair, etc.) | Missing | `LR-MASK-PEOPLE`; face detection elsewhere does not provide these masks. |
| Landscape component masks and depth-range masks | Missing | `LR-MASK-LANDSCAPE`, `LR-MASK-DEPTHRANGE`. |
| Adaptive profiles and adaptive presets | Missing | `LR-PROF-ADAPTIVE`, `LR-MASK-ADAPTIVE`, `LR-PRE-ADAPTIVE`; ordinary presets exist. |
| Recompute AI masks / AI-edit update tracking | Missing | `LR-MASK-UPDATE`, `LR-AI-UPDATE-INDICATOR`. |
| Natural-language photo search | Missing | `LR-FILT-SEARCH-AI`; field/keyword search is implemented. |
| Video import/editing workflow: playback, trimming, adjustments, cover frame and export | Missing | `LR-VID-*`; exporting a slideshow video is a separate, partially implemented feature. [Adobe Classic feature guide](https://helpx.adobe.com/dk/lightroom-classic/desktop.html) |

Adobe documents [people/landscape/depth masks](https://helpx.adobe.com/lightroom-classic/desktop/process-and-develop-photos/masking.html), [adaptive profiles](https://blog.adobe.com/en/publish/2024/10/14/the-adobe-adaptive-profile), and [natural-language search](https://helpx.adobe.com/lightroom/mobile/search-and-filter-photos/search-for-photos.html). AI/masking availability by platform is documented in its [AI Edit Status guide](https://helpx.adobe.com/lightroom/web/edit-photos/manage-ai-edits.html).
The specific LightCraft rows are in the [upstream checklist](../parity-checklist.md).

## Missing cloud, sharing and Classic workflows

| Feature | Lightroom edition | LightCraft status / evidence |
|---|---|---|
| Library/edit synchronization across devices | Lightroom ecosystem; Classic has a collection-sync workflow | Outside scope: `LR-CLOUD-SYNC`. Local backups and manual transfer are not continuous sync. [Adobe mobile sync](https://helpx.adobe.com/lightroom/mobile/get-started/manage-cloud-storage-and-sync.html) |
| Hosted shared albums, web links, invitations, collaboration, comments and likes | Lightroom ecosystem | Absent / outside scope: `LR-LIB-SHARED-ALBUM`, `LR-LIB-ACTIVITY`, `LR-SHARE-*`. [Adobe sharing](https://helpx.adobe.com/lightroom-cc/using/save-share-photos.html) |
| Community edits and recommended community presets | Lightroom ecosystem | Outside scope: `LR-SHARE-COMMUNITY`, `LR-PRE-RECOMMENDED`. |
| Tethered camera capture | Classic | Missing: `LRC-LIB-TETHER`. [Adobe tethering](https://helpx.adobe.com/lightroom-classic/desktop/import-photos/import-photos-tethered-camera.html) |
| Publish Services with managed publishing queues | Classic | Missing: `LRC-LIB-PUBLISH`. [Adobe Publish Services](https://helpx.adobe.com/lightroom-classic/desktop/export-photos/export-hard-drive-using-publish.html) |
| Interactive Map module, drag-to-geotag and saved map locations | Classic | Missing: `LRC-MAP-VIEW/GEOTAG/LOCATIONS/FILTER`. GPS metadata, GPX tracklog tagging and offline reverse geocoding already exist. [Adobe Classic guide](https://helpx.adobe.com/dk/lightroom-classic/desktop.html) |
| Book design and book export | Classic | Missing: `LRC-BOOK-*`. [Adobe Book module](https://helpx-origin.aws116.adobeitc.com/ie/lightroom-classic/help/create-photo-book.html) |
| Slideshow music | Classic | Missing: `LRC-SS-MUSIC`. Slideshows themselves are implemented in part; do not call the whole module absent. |
| HTML web-gallery design/output/upload | Classic | Outside scope: `LRC-WEB-*`. |
| Lightroom-style third-party export/publish/metadata plug-in ecosystem | Classic | No equivalent SDK/ecosystem; upstream gap 19. LightCraft's MCP automation is a different capability. |

## Present, but materially less complete

| Capability | What LightCraft has | Remaining difference |
|---|---|---|
| RAW camera color and camera-matching profiles | Camera-JPEG fitting, bundled profiles and spectral matrices | No broadly measured camera-profile library comparable to Adobe's; matching camera looks remain absent/incomplete. Do not equate a supported decoder with accurate calibrated color. |
| Lens profiles | Manual optics, DNG opcodes and selected in-file camera corrections | No general lens-profile database. [Adobe lens-profile workflow](https://helpx.adobe.com/lightroom-classic/desktop/process-and-develop-photos/retouch-photos.html) |
| RAW support | Many RAW formats including many CR3/NEF/RAF variants | Compressed ORF, Nikon HE/HE-star, compressed Samsung SRW and some rare variants remain preview-only or unsupported. Model coverage varies. |
| AI Subject/Sky/Background masks | Heuristic selections; optional object segmentation infrastructure | Not Lightroom-quality semantic selection; SAM model distribution/configuration is unfinished. |
| AI denoise | Opt-in Bayer RAW implementation | No shipped ready-to-use weight set; X-Trans and merged linear-DNG denoise are incomplete. It is not accurate to say all noise reduction is missing. |
| Content-aware removal | Heal, Clone, automatic-source removal and a partial fill implementation | Less complete fill/selection workflow; no generative model. |
| HDR editing/output | HDR data path and several HDR export formats | HDR display output absent with the current 8-bit egui surface. [Adobe HDR display/editing](https://helpx.adobe.com/il_en/lightroom-classic/desktop/process-and-develop-photos/hdr-output.html) |
| HDR merge | Alignment, deghosting and float DNG | No AI denoise on merged linear DNG; JPEG-bracket response handling and memory use remain gaps. |
| Lightroom catalog migration | Imports a substantial set of records and settings | Adobe profiles/AI settings and rendered appearance do not transfer exactly. |
| Print workflows | Layouts, cell editing, templates, color-managed PDF | Direct printer integration, several page overlays and template actions remain incomplete. |
| Slideshow workflows | Composition, playback, overlays and some exports | Music absent; template/layout/overlay/title/playback/export rows still partial. The prose gaps overview lags the more detailed checklist here. |
| Face recognition | Optional local detector/recognition models | Setup, shipped models and real-library recognition validation remain incomplete. |

LightCraft evidence: [checklist](../parity-checklist.md), [ranked gaps](../gaps.md),
[RAW coverage](../raw-parity.md), [hardware](../hardware-parity.md), [file formats](../file-format-parity.md).
Some tracker rows and prose notes disagree. This report prefers the current row and cited implementation;
none of the labels proves equal output quality or iPad usability.

## Additional limitations of our iPad wrapper

These are web/iPad limitations, separate from the Lightroom comparison:

- The same upstream egui UI and Rust/WASM engine run locally in WKWebView. Photo rendering uses
  CPU Web Workers; WebGL2 draws the UI. This does not inherit the desktop Metal photo pipeline.
- The library uses WebKit origin storage (OPFS or IndexedDB). Keep originals elsewhere and use
  File → Back Up Library; installing an app wrapper does not turn it into an iCloud library.
- Preset JSON file pickers are not wired upstream; desktop TCP control/MCP is unavailable.
- JPEG XL export is native-only; AVIF web export is untested upstream.
- Exports/auto adjustments can block the main thread, large backups have limitations, and
  decoder panics can terminate the page. Finger/Pencil behavior and large RAW memory use require
  physical-device testing.
- Our native export/download bridge and Files import need separate workflow verification.
- HEIC support depends on the bundled web build's feature flags; it must be tested on this exact
  release. The older Unveil build's missing HEIC feature says nothing about this wrapper bundle.

Evidence: [upstream web behavior and limitations](../web.md),
[wrapper provenance](../../AppleWrapper/web-release.json).

## Suggested priorities for this fork

1. Reliable Files import, export/downloads, library backup/restore and relaunch persistence.
2. HEIC and the user's actual RAW formats, including clear errors and memory limits.
3. Touch/Pencil, safe areas, panel sizes, and large-library responsiveness.
4. Camera/lens corrections and useful local masks/denoise as upstream makes progress.
5. Cloud/generative features only if explicitly wanted; they add separate service/model work.

See [the generated inventory](lightroom-checklist-inventory.md) for every tracked missing,
partial or intentionally excluded row, including individual menu/shortcut items. No list is
silently truncated. That appendix preserves upstream claims and can contain stale notes.
