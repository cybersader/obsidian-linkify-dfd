# Changelog

All notable changes to the DFD-Excalidraw-System project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.7.9] - 2026-01-12

### Added
- **Shape-based auto-classification** (`AUTO_CLASSIFY_BY_SHAPE` setting)
  - When enabled, classify elements by shape type without requiring markers:
    - Rectangle/Diamond → Asset (uses bound/group text as name)
    - Ellipse/Circle → Entity (uses bound/group text as name)
    - Arrow → Transfer
  - Explicit markers (`asset=`, `entity=`, `transfer`) still override when present
  - Requires `REQUIRE_EXPLICIT_MARKER = false` to take effect

### Changed
- Faster diagramming workflow: no need to type `asset=` or `entity=` prefixes
- Shape semantics: ellipses now become entities (not assets) when auto-classify is on

## [1.7.8] - 2026-01-12

### Added
- **Assets and entities now store `source_diagram_ids` (UUID) alongside `source_drawings`**
  - Consistent with transfers which already had `_source_diagram_ids`
  - New helper function: `addSourceDiagramToNode()`
  - All object types now have rename-proof diagram identity tracking

### Changed
- Replaced `pushArr()` calls with `addSourceDiagramToNode()` for assets/entities
- Assets/entities frontmatter now includes `source_diagram_ids` array

## [1.7.7] - 2026-01-12

### Fixed
- **UUID detection now checks for Excalidraw frontmatter instead of file extension**
  - Works with both `.excalidraw.md` AND `.md` files (2025 Excalidraw default)
  - Checks for `excalidraw-plugin` key in frontmatter (standard Excalidraw marker)
  - Fixes issue where accidental rename (dropping `.excalidraw`) broke UUID support

### Changed
- Log messages updated to reference frontmatter detection instead of file extension

### Documentation
- Added file format comparison table to README
- Documented Excalidraw plugin compatibility mode recommendation
- Added Test 10.23 for `.md` format UUID verification

## [1.7.6] - 2026-01-12

### Added
- **Stable diagram UUID for rename-proof transfer ownership**
  - Stores `dfd_diagram_id` in diagram frontmatter (requires Excalidraw markdown format)
  - Transfers store `_source_diagram_ids` array alongside existing `_source_diagrams`
  - Lookup priority: UUID match first, then wiki link fallback, then legacy field
- New helper functions: `generateUUID()`, `ensureDiagramUUID()`

### Changed
- When diagram is renamed, transfers are still found via UUID match
  - Previously, transfers became orphaned when diagram name changed
- Pure `.excalidraw` files (JSON, no frontmatter) fall back to wiki link matching

### Prerequisites
- Excalidraw markdown format (`.md` or `.excalidraw.md`) required for UUID support
- Settings → Saving → Excalidraw file format → Select a markdown option

## [1.7.5] - 2026-01-10

### Added
- `TRANSFER_FUZZY_MATCH` setting for cross-naming-scheme matching
  - `false` (default): Only match transfers using current naming scheme
  - `true`: Match any transfer with same endpoints, regardless of naming scheme
- Helps when transitioning between `TRANSFER_INCLUDE_DIAGRAM = true/false`
- Note: Explicit `transfer=reuse` marker always uses fuzzy matching (ignores this setting)

## [1.7.4] - 2026-01-09

### Changed
- **Refactored**: Transfer naming into orthogonal settings for cleaner configuration
  - `TRANSFER_INCLUDE_DIAGRAM` (true/false): Whether to include diagram name in filename
  - `TRANSFER_POSITION` ("suffix"/"prefix"): Position for diagram name and collision suffix
  - `TRANSFER_COLLISION_MODE` ("none"/"random"/"sequential"): How to handle filename collisions
- Default collision mode is now `"none"` (expect unique filenames from diagram name inclusion)

### Fixed
- **Orphan detection now works with diagram name suffix in filenames**
  - Bug: `endsWith("_forward")` didn't match `transfer_a_to_b_forward_diagram-name.md`
  - Fix: Changed to `includes("_forward")` throughout the codebase
  - Now correctly detects bidirectional→unidirectional conversion and orphans the reverse transfer
- Forward/reverse pair detection now uses frontmatter `paired_transfer` first (more reliable)

### Removed
- `TRANSFER_NAMING_MODE` setting (replaced by orthogonal settings above)
- `TRANSFER_NUMBERING_MODE` setting (replaced by `TRANSFER_COLLISION_MODE`)

## [1.7.3] - 2026-01-09

### Added
- `TRANSFER_NAMING_MODE` setting for diagram-aware transfer filenames
  - `"include_diagram_suffix"` (default): `transfer_asset-x_to_asset-y_my-diagram.md`
  - `"include_diagram_prefix"`: `transfer_my-diagram_asset-x_to_asset-y.md`
  - `"auto"`: Legacy behavior (random/numbered suffix only on collision)
- `CLEAN_MARKER_TEXT` setting to control whether marker text is cleaned after processing
  - `true` (default): Remove marker text after linking (cleaner diagrams)
  - `false`: Keep marker text visible (useful for testing/debugging)
- Stale link detection with helpful user notice
  - When arrow has link to deleted transfer but no marker, shows guidance: "Add 'transfer' as bound text (double-click arrow) to re-process"

### Fixed
- **CRITICAL**: Arrow link not being set when reusing existing transfer
  - This caused arrows to show "transfer" text but have no link (Ctrl+K showed nothing)
  - Root cause: Multiple code paths were returning without setting `arr.link`
  - All code paths now properly set the arrow link before returning
- Script no longer silently fails when arrow has link to deleted transfer file
  - Now continues checking for markers (bound text, group text) after detecting stale link
- Pattern matching for transfer lookup now handles all naming modes correctly

### Changed
- Logging now shows `TRANSFER_NAMING_MODE` setting value at startup
- Transfer filenames now clearly indicate which diagram created them, making orphan identification easier

## [1.7.2] - 2026-01-09

### Breaking Changes
- **Default transfer behavior changed to "per_diagram"**: Each diagram now gets its own transfer files. Same endpoints on different diagrams = different transfer files. This prevents accidental context coupling between diagrams (e.g., Marketing DFD vs Finance DFD using same systems).

### Added
- `TRANSFER_REUSE_MODE` setting: Controls cross-diagram transfer behavior
  - `"per_diagram"` (default): Each diagram gets its own transfers
  - `"reuse_existing"`: Old behavior - reuse if same endpoints exist
- `transfer=reuse` marker syntax for explicit reuse requests
  - `transfer=reuse` - Reuse existing transfer with same endpoints
  - `transfer=reuse:[[path]]` - Link to specific existing transfer
- Same-diagram exception: Multiple arrows on SAME diagram can share transfers (visual variants)
- Transfer marker text cleanup: "transfer" text is now cleaned from arrows after processing (like assets)
- Helper functions: `isTransferOwnedByCurrentDiagram()`, `findExistingTransferForDiagram()`, `findAnyTransferBetweenObjects()`

### Changed
- Logging now shows `TRANSFER_REUSE_MODE` setting value at startup

## [1.7.1] - 2026-01-09

### Added
- Multi-diagram tracking for transfers via `_source_diagrams` array
  - Transfers now track ALL diagrams that reference them, not just the first
- Backward compatibility migration from legacy `source_drawing` field
- `addSourceDiagram()` helper for consistent array management

### Changed
- Orphan detection is now multi-diagram aware
  - Only orphans transfers when NO diagrams reference them
  - Warns instead of auto-orphaning when multiple diagrams are tracked

### Fixed
- Re-linking orphaned reverse transfer now works correctly
  - Bug was in pattern matching (missing OBJECT_SEPARATOR in find/replace)
  - Now correctly derives reverse path from forward path

## [1.7.0] - 2026-01-09

### Added
- Orphan detection for bidirectional-to-unidirectional conversion
  - When a bidirectional arrow becomes unidirectional, the orphaned `_reverse` transfer is flagged
- `_status` field for transfers: `"active"` | `"orphaned"` | `"archived"`
- Orphan metadata fields: `_orphan_detected`, `_orphan_reason`, `_last_linked_diagram`
- Clears orphan flags when a transfer is re-linked

### Fixed
- Re-linking orphaned reverse when arrow changes back to bidirectional
- Properly finds and re-links orphaned `_reverse` transfers

## [1.6.3] - 2026-01-07

### Removed
- TR-XXXX label feature (redundant - transfers already uniquely named)

### Changed
- Arrow customData no longer includes edgeId

## [1.6.2] - 2026-01-06

### Fixed
- Multiline text in markers now normalized (newlines to spaces)
  - Prevents YAML parse errors when user types "asset=Paired<Enter>Source"

### Changed
- Log output escapes newlines for clarity

## [1.6.1] - 2025-12-15

### Fixed
- Bound text duplication bug - text no longer duplicates outside rectangles when script is run multiple times
- Improved error message when arrow not connected to shapes (now says "Arrow START/END not connected" instead of generic error)

### Added
- `safeCopyToEA()` helper prevents duplicate element copying
- `modifiedTextElements` tracking prevents duplicate text modifications
- Better logging for text modification flow

## [1.6.0] - 2025-12-18

### Added
- Stale transfer detection: when shape endpoints change, the script now renames the transfer file to match new endpoints (e.g., `transfer_a_to_a` → `transfer_a_to_b`) instead of keeping wrong name
- Arrow link automatically updates when transfer is renamed

### Fixed
- No longer need to manually delete transfer links after changing shape endpoints

## [1.5.6] - 2025-12-18

### Fixed
- Bidirectional conversion in dual_transfers mode now properly renames existing transfer to `_forward` and creates `_reverse`

## [1.5.5] - 2025-12-15

### Fixed
- Existing transfer properties now ALWAYS update to match diagram (object_a/b, from/to now overwrite existing values)
- Old dfd_in/dfd_out references cleaned up when endpoints change

### Changed
- Diagram is now strict source of truth for transfer relationships

## [1.5.4] - 2025-12-15

### Fixed
- Existing transfers now update object_a/b, from/to, source_drawing

### Added
- `source_drawings[]` array on assets/entities tracking which diagrams they appear in (populated for both new and existing nodes)

### Changed
- All node/transfer paths now consistently track diagram sources

## [1.5.3] - 2025-12-15

### Added
- Debug file logging (`DEBUG_LOG_TO_FILE` setting)
- Log clear modes: `"on_run"` (clear each run) or `"manual"` (append)
- Logs include timestamps and diagram path for each run

## [1.5.2] - 2025-12-15

### Fixed
- Arrows with existing transfer links now update dfd_in/dfd_out
- `classifyElement` returning "existing" for wikilinks now properly detected as transfers when pointing to Transfers/ folder

### Added
- Notice when existing transfer relationships are updated

## [1.5.1] - 2025-12-15

### Fixed
- Bound text cleanup now uses elementsDict workflow
- Uses `ea.getElement()` after `copyViewElementsToEAforEditing()`

### Added
- rawText/originalText properties updated for text elements
- `refreshTextElementSize()` call after text modification

## [1.5.0] - 2025-12-15

### Added
- Transfer numbering mode for duplicate transfer names
  - `"prefix"` → `transfer_2_dna_to_imm`
  - `"suffix"` → `transfer_dna_to_imm_2`
  - `"random"` → `transfer_dna_to_imm-a7b2` (legacy)
- Bound text marker support (markers typed INTO shapes)

### Fixed
- Only process elements with explicit markers/text
- Smart matching for `object=name` syntax (reuse existing pages)
- Skip auto-generation if legitimate page already linked

## [1.4.0] - 2025-11-24

### Added
- `CROSS_TYPE_MATCHING` setting
- Explicit marker requirement mode

### Fixed
- Cross-type matching bug (Test 2.3)

---

## Delete DFD Transfers Script

### [1.7] - 2026-01-09

#### Added
- Multi-diagram awareness with `WARN_MULTI_DIAGRAM` and `SKIP_MULTI_DIAGRAM` settings
- Helper to check transfer references: `transferRefersToCurrentDiagram()`
- Backward compatibility with legacy `source_drawing` field

#### Changed
- Defaults to skipping multi-diagram transfers (safer)

### [1.6] - 2026-01-09

#### Added
- `TARGET_ORPHANS_ONLY` mode to only process orphaned transfers
- `ARCHIVE_MODE` to move files instead of deleting them
- Support for `_status` field filtering

---

## Documentation

### [2026-01-09]
- Added CHANGELOG.md (this file)
- Updated DFD Testing Guide with v1.7.2 test scenarios (Tests 10.10-10.15)
- Added v2.0+ roadmap items to project tracker

---

## Roadmap

### v1.7.x (Remaining Script Improvements)

~~**Shape-Based Auto-Classification**~~ → ✅ **Implemented in v1.7.9**
- Setting: `AUTO_CLASSIFY_BY_SHAPE` (true/false)
- Requires `REQUIRE_EXPLICIT_MARKER = false`
- Rectangle/Diamond → Asset, Ellipse/Circle → Entity, Arrow → Transfer
- Uses bound/group text as name (no need for `asset=` prefix)
- Explicit markers still override when present

**Bare Wikilink Support** (optional, low priority):
- Currently: `[[Name]]` in Ctrl+K (non-existent file) falls through to bound text
- Enhancement: Optionally treat bare `[[Name]]` as object name with shape-based type
- Note: `asset=Name` in Ctrl+K already works and is the recommended approach

**Group Text Edge Cases** (low priority, limited use cases):
- Group text = text element grouped with a shape (vs bound text typed INTO shape)
- Current: Falls back to group text if no Ctrl+K link or bound text marker
- Need: More thorough testing of edge cases (multiple text elements in group, priority order)
- Defer until more users rely on grouped text workflow

**Diagram Format Normalization** (warning mode for v1.7.x):
- Warn when diagram extension doesn't match Excalidraw plugin setting
- Full auto-fix deferred to v2.0

### v2.0 (Future - Plugin Interactive Mode)
- Modal: "Transfer from X to Y exists. Reuse or create new?"
- **Fuzzy Auto-Match Modal**: When shape text could match multiple existing objects:
  - Show modal with potential matches ranked by similarity
  - User selects the correct match or confirms new creation
  - Learns from choices to improve future suggestions
  - Supports partial name matching, aliases, and common abbreviations
- Settings UI for default behavior
- Context field: `_context: "Marketing"` to distinguish same-endpoint transfers
- **Naming scheme migration**: Smart matching when changing settings
  - Detect `transfer_a_to_b` could match existing `transfer_a_to_b_diagram-name.md`
  - Prompt: "Found existing transfer with diagram suffix. Use it or create new?"
  - Helps users transition between `TRANSFER_INCLUDE_DIAGRAM` true/false
- ~~**Diagram rename handling**~~ → ✅ **Implemented in v1.7.6** via stable UUID
  - Diagrams now have `dfd_diagram_id` in frontmatter (persists through renames)
  - Transfers track `_source_diagram_ids` for rename-proof ownership matching
- **Diagram format normalization**
  - Script setting: `AUTO_FIX_DIAGRAM_FORMAT` with options:
    - `"off"` - No auto-fix, just process as-is
    - `"warn"` - Warn about format mismatches
    - `"fix"` - Auto-convert to match plugin setting
  - **Case 1: Markdown extension mismatch**
    - Plugin set to `.excalidraw.md` but file is `.md` (from accidental rename), or vice versa
    - Auto-rename to match configured extension
  - **Case 2: Legacy `.excalidraw` JSON file**
    - Downloaded/imported `.excalidraw` file needs conversion to markdown format
    - Convert to `.md` or `.excalidraw.md` (based on plugin setting) to enable frontmatter/UUID
    - Uses Excalidraw plugin's conversion API if available
  - Ensures all diagrams support frontmatter features (UUID, metadata)

### v2.1 (Future - Smart Context Detection)
- Detect if diagrams are in same folder = more likely same context
- Naming convention detection: "Marketing - *.excalidraw" = context = Marketing
- Transfer grouping by context

### v2.2 (Future - Transfer Inheritance)
- "This transfer is same context as [[other-transfer]]"
- Copy frontmatter properties from parent
- Linked context chains

### v2.x (Future - Advanced Features)

**Custom Shape Mapping in Object Configuration**:
- Add `DFD__SHAPE` property to custom object configs
- Example: `DFD__SHAPE: diamond` → diamonds become this type
- Combines with existing `DFD__MARKER` for full customization
- Removes need for two flags (`REQUIRE_EXPLICIT_MARKER` + `AUTO_CLASSIFY_BY_SHAPE`)

**Auto-Styling Elements by Object Type**:
- Once element is linked to an object type, apply visual styling
- Colors, stroke width, fill based on asset/entity/transfer type
- Custom styling per object config
- Excalidraw API: `element.strokeColor`, `element.backgroundColor`, etc.

**Batch Processing Selected Elements**:
- Script variant that processes only selected shapes/arrows
- Use case: Incrementally link parts of a diagram
- Complexity: High (edge cases with partial selections, arrows to unselected nodes)
- May need interactive prompts for ambiguous cases

**Transfer Direction Word by Mode**:
- Differentiate direction word in filenames based on bidirectional mode setting
- `dual_transfers` mode: Use `tnf` (to aNd from) → `transfer_a_tnf_b_forward.md`, `transfer_a_tnf_b_reverse.md`
- `single_bidirectional` mode: Use `fot` (from Or to) → `transfer_a_fot_b.md`
- Unidirectional: Keep `to` → `transfer_a_to_b.md`
- Clearer semantics: "tnf" = two transfers exist, "fot" = one transfer, bidirectional

**External Process/Role Object Type**:
- New object type for processes or roles outside your organizational scope
- Use case: Diagramming handoffs to external teams, vendors, or upstream/downstream processes
- Potential shape: Diamond (currently falls back to asset)
- Example marker: `external=Vendor Onboarding Process` or `process=External Review`
- Distinct from entities (people) - represents workflows/functions, not actors
- Could have different frontmatter schema and folder (`Processes/` or `External/`)

### Documentation Backlog

**DFD Schema Recovery Guide**:
- Guide for fixing messed up DFD object schemas
- Common issues: orphaned transfers, broken frontmatter, schema version mismatches
- Recovery procedures: bulk reset, migration scripts, validation queries
- Dataview queries for detecting schema problems
