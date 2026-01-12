---
aliases: []
tags: []
date created: 2025-11-22T00:00:00-05:00
date modified: 2025-11-22T00:00:00-05:00
---
/*
```js*/
/*****************************************************************
 * Linkify DFD — v1.7.6  (2026-01-12)
 * ---------------------------------------------------------------
 * COMPATIBILITY:
 *   - Tested with: Excalidraw 2.19.0
 *   - Minimum: Excalidraw 2.18.0
 *   - See .claude/research/excalidraw-dependency.md for API changes
 * ---------------------------------------------------------------
 * CHANGELOG:
 *
 * v1.7.6 (2026-01-12)
 *   - Added: Stable diagram UUID for rename-proof transfer ownership
 *           Stores dfd_diagram_id in diagram frontmatter (requires .excalidraw.md format)
 *           Transfers store _source_diagram_ids array alongside _source_diagrams
 *   - Benefit: When diagram is renamed, transfers still found via UUID match
 *           (Previously, transfers became orphaned when diagram name changed)
 *   - Lookup priority: UUID match first, then wiki link fallback, then legacy field
 *   - Note: Pure .excalidraw files (no frontmatter) fall back to wiki link matching
 *   - Prereq: Excalidraw plugin setting "Excalidraw file format" = "Excalidraw Markdown"
 *
 * v1.7.5 (2026-01-10)
 *   - Added: TRANSFER_FUZZY_MATCH setting for cross-naming-scheme matching
 *           false (default): Only match transfers using current naming scheme
 *           true: Match any transfer with same endpoints, regardless of naming scheme
 *   - Benefit: When transitioning between TRANSFER_INCLUDE_DIAGRAM true/false,
 *           can choose whether to find existing transfers from old naming scheme
 *   - Note: Explicit "transfer=reuse" marker always uses fuzzy matching
 *
 * v1.7.4 (2026-01-09)
 *   - Refactored: Transfer naming into orthogonal settings (cleaner config)
 *           TRANSFER_INCLUDE_DIAGRAM (true/false) → Include diagram name in filename
 *           TRANSFER_POSITION ("suffix"/"prefix") → Position for diagram & collision suffix
 *           TRANSFER_COLLISION_MODE ("none"/"random"/"sequential") → How to handle collisions
 *   - Changed: Default collision mode is now "none" (expect unique filenames from diagram name)
 *   - Removed: Old TRANSFER_NAMING_MODE and TRANSFER_NUMBERING_MODE settings (replaced)
 *   - Fixed: Orphan detection now works with diagram name suffix in filenames
 *           Changed endsWith("_forward") to includes("_forward") throughout
 *           Now correctly detects bidirectional→unidirectional conversion
 *   - Fixed: Forward/reverse pair detection uses frontmatter paired_transfer first
 *   - Improved: Cleaner settings organization in USER SETTINGS section
 *
 * v1.7.3 (2026-01-09)
 *   - Added: TRANSFER_NAMING_MODE setting for diagram-aware filenames
 *           "include_diagram_suffix" (default): transfer_asset-x_to_asset-y_my-diagram.md
 *           "include_diagram_prefix": transfer_my-diagram_asset-x_to_asset-y.md
 *           "auto": Legacy behavior (random/numbered suffix only on collision)
 *   - Benefit: Transfer filenames now clearly show which diagram created them
 *           Makes it easy to identify orphaned transfers and their origins
 *   - Fixed: CRITICAL - Arrow link not being set when reusing existing transfer!
 *           This caused arrows to show "transfer" text but have no link (Ctrl+K empty)
 *           All code paths now properly set arr.link before returning
 *   - Fixed: Stale link detection - when arrow has link to deleted transfer file,
 *           script now continues checking for markers instead of silently failing
 *   - Added: Helpful user notice when arrow has stale link but no marker found
 *           Explains to use bound text (double-click arrow) to re-process
 *   - Improved: Pattern matching for transfer lookup handles all naming modes
 *   - Note: Existing transfers are not automatically renamed
 *
 * v1.7.2 (2026-01-09)
 *   - BREAKING: Default transfer behavior changed to "per_diagram"
 *           Each diagram now gets its own transfers (same endpoints = different transfers)
 *           This prevents accidental coupling between contexts (Marketing vs Finance DFDs)
 *   - Added: TRANSFER_REUSE_MODE setting ("per_diagram" | "reuse_existing")
 *           "per_diagram" (default): Each diagram gets its own transfers
 *           "reuse_existing": Old behavior - reuse if same endpoints exist
 *   - Added: transfer=reuse marker syntax for explicit reuse requests
 *           "transfer=reuse" → Reuse existing transfer with same endpoints
 *           "transfer=reuse:[[path]]" → Link to specific existing transfer
 *   - Added: Same-diagram exception - multiple arrows on SAME diagram can share transfers
 *   - Added: "transfer" marker text cleanup from arrows after processing
 *           (Similar to how asset markers are cleaned from shapes)
 *   - Added: isTransferOwnedByCurrentDiagram() helper for ownership checks
 *   - Added: findExistingTransferForDiagram() and findAnyTransferBetweenObjects() helpers
 *   - See: .claude/plans/cheeky-hatching-shell.md for design rationale
 *
 * v1.7.1 (2026-01-09)
 *   - Added: Multi-diagram tracking for transfers via _source_diagrams array
 *           Transfers now track ALL diagrams that reference them, not just the first
 *   - Added: Backward compatibility migration from legacy source_drawing field
 *   - Changed: Orphan detection is now multi-diagram aware
 *           Only orphans transfers when NO diagrams reference them
 *           Warns instead of auto-orphaning when multiple diagrams tracked
 *   - Added: addSourceDiagram() helper for consistent array management
 *   - Fixed: Re-linking orphaned reverse transfer now works correctly
 *           Bug was in pattern matching (missing OBJECT_SEPARATOR in find/replace)
 *           Now correctly derives reverse path from forward path
 *   - See: .claude/plans/cheeky-hatching-shell.md for design rationale
 *
 * v1.7.0 (2026-01-09)
 *   - Added: Orphan detection for bidirectional→unidirectional conversion
 *           When a bidirectional arrow becomes unidirectional, the orphaned
 *           _reverse transfer is flagged with _status: "orphaned"
 *   - Added: _status field for transfers ("active" | "orphaned" | "archived")
 *   - Added: _orphan_detected, _orphan_reason, _last_linked_diagram fields
 *   - Added: Clears orphan flags when a transfer is re-linked
 *   - Fixed: Re-linking orphaned reverse when arrow changes back to bidirectional
 *           Now properly finds and re-links orphaned _reverse transfers
 *   - See: .claude/development/true-orphan-detection.md for design details
 *
 * v1.6.3 (2026-01-07)
 *   - Removed: TR-XXXX label feature (redundant - transfers already uniquely named)
 *   - Simplified: Arrow customData no longer includes edgeId
 *
 * v1.6.2 (2026-01-06)
 *   - Fixed: Multiline text in markers now normalized (newlines → spaces)
 *           Prevents YAML parse errors when user types "asset=Paired<Enter>Source"
 *   - Improved: Log output escapes newlines for clarity
 *
 * v1.6.1 (2025-01-05)
 *   - Fixed: Bound text duplication bug - text no longer duplicates outside
 *           rectangles when script is run multiple times
 *   - Fixed: Improved error message when arrow not connected to shapes
 *           (now says "Arrow START/END not connected" instead of generic error)
 *   - Added: safeCopyToEA() helper prevents duplicate element copying
 *   - Added: modifiedTextElements tracking prevents duplicate text modifications
 *   - Improved: Better logging for text modification flow
 *
 * v1.6.0 (2025-12-18)
 *   - Added: Stale transfer detection - when shape endpoints change, the script
 *           now renames the transfer file to match new endpoints (e.g.,
 *           transfer_a_to_a → transfer_a_to_b) instead of keeping wrong name
 *   - Added: Arrow link automatically updates when transfer is renamed
 *   - Fixed: No longer need to manually delete transfer links after changing
 *           shape endpoints
 *   - Note: Orphaned paired transfers now handled in v1.7.0
 *
 * v1.5.6 (2025-12-18)
 *   - Fixed: Bidirectional conversion in dual_transfers mode now properly
 *           renames existing transfer to _forward and creates _reverse
 *
 * v1.5.5 (2025-12-15)
 *   - Fixed: Existing transfer properties now ALWAYS update to match diagram
 *           (object_a/b, from/to now overwrite existing values)
 *   - Fixed: Old dfd_in/dfd_out references cleaned up when endpoints change
 *   - Added: Diagram is now strict source of truth for transfer relationships
 *
 * v1.5.4 (2025-12-15)
 *   - Fixed: Existing transfers now update object_a/b, from/to, source_drawing
 *   - Added: source_drawings[] array on assets/entities tracking which diagrams
 *           they appear in (populated for both new and existing nodes)
 *   - Improved: All node/transfer paths now consistently track diagram sources
 *
 * v1.5.3 (2025-12-15)
 *   - Added: Debug file logging (DEBUG_LOG_TO_FILE setting)
 *   - Added: Log clear modes: "on_run" (clear each run) or "manual" (append)
 *   - Added: Logs include timestamps and diagram path for each run
 *
 * v1.5.2 (2025-12-15)
 *   - Fixed: Arrows with existing transfer links now update dfd_in/dfd_out
 *   - Fixed: classifyElement returning "existing" for wikilinks now properly
 *           detected as transfers when pointing to Transfers/ folder
 *   - Added: Notice when existing transfer relationships are updated
 *
 * v1.5.1 (2025-12-15)
 *   - Fixed: Bound text cleanup now uses elementsDict workflow
 *   - Fixed: Uses ea.getElement() after copyViewElementsToEAforEditing()
 *   - Added: rawText/originalText properties updated for text elements
 *   - Added: refreshTextElementSize() call after text modification
 *
 * v1.5.0 (2025-12-15)
 *   - Added: Transfer numbering mode for duplicate transfer names
 *           "prefix" → transfer_2_dna_to_imm
 *           "suffix" → transfer_dna_to_imm_2
 *           "random" → transfer_dna_to_imm-a7b2 (legacy)
 *   - Added: Bound text marker support (markers typed INTO shapes)
 *   - Fixed: Only process elements with explicit markers/text
 *   - Fixed: Smart matching for object=name syntax (reuse existing pages)
 *   - Fixed: Skip auto-generation if legitimate page already linked
 *
 * v1.4.0 (2025-11-24)
 *   - Added: CROSS_TYPE_MATCHING setting
 *   - Added: Explicit marker requirement mode
 *   - Fixed: Cross-type matching bug (Test 2.3)
 ******************************************************************/

/************* USER SETTINGS ************************************/
const DEBUG = true;
const REQUIRE_EXPLICIT_MARKER = true;  // Only link elements with explicit markers

// SMART MATCHING
const SMART_CUSTOM_NAME_MATCHING = true;   // If true, "asset=Customer DB" tries to find existing "Customer DB.md" first
const SEARCH_ALL_SUBFOLDERS = true;        // When smart matching, search vault-wide for same-type matches
const CROSS_TYPE_MATCHING = false;         // If true, "entity=X" can match existing "Assets/x.md" (usually false - types are semantically different)

// Storage options
const DB_PLACEMENT = "db_folder";
const DB_FOLDER_NAME = "DFD Objects Database";
const DB_DB_PARENT_PATH = "";  // Set this to parent path if needed, e.g., "Folder/Subfolder/"

// Config folder
const CFG_DIR = "./DFD Object Configuration";

// Optional inline fields
const WRITE_INLINE_FIELDS = false;

// Filename behavior for custom names
const CUSTOM_NAME_MODE = "replace";     // "replace" | "inject"

// ARROW DIRECTION DETERMINATION
const DIRECTION_DETERMINATION = "arrowhead_priority";
const BIDIRECTIONAL_MODE = "dual_transfers";
const TRANSFER_DIRECTION_WORD = "to";    // "to", "tnf", "toandfrom"
const OBJECT_SEPARATOR = "_";

// TRANSFER PROPERTY BEHAVIOR
const INCLUDE_FROM_TO_PROPERTIES = true;
const BIDIRECTIONAL_FROM_TO_BOTH = true;

// v1.7.2: TRANSFER REUSE MODE
// Controls whether to reuse existing transfers across diagrams
// "per_diagram" (default) → Each diagram gets its own transfers (same endpoints = different transfers)
// "reuse_existing" → Reuse if same endpoints exist (old behavior, for backward compatibility)
// Note: Explicit "transfer=reuse" marker always allows reuse regardless of this setting
const TRANSFER_REUSE_MODE = "reuse_existing";

// v1.7.4: ORTHOGONAL TRANSFER NAMING SETTINGS
// These three settings work together to control transfer filename format:

// 1. Include diagram name in filename?
// true (default) → transfer_asset-x_to_asset-y_my-diagram.md (clear ownership)
// false → transfer_asset-x_to_asset-y.md (legacy behavior)
const TRANSFER_INCLUDE_DIAGRAM = false;

// 2. Position for diagram name and collision suffix
// "suffix" (default) → _my-diagram or _2 at end of filename
// "prefix" → diagram name or number right after "transfer_"
const TRANSFER_POSITION = "suffix";

// 3. How to handle name collisions (file already exists)
// "none" (default) → Expect no collision; if one occurs, use random fallback
// "random" → Add random 4-char suffix: -a7b2
// "sequential" → Add numbered suffix: _2, _3, etc. (position controlled by TRANSFER_POSITION)
const TRANSFER_COLLISION_MODE = "none";

// v1.7.5: CROSS-NAMING-SCHEME MATCHING
// When looking for existing transfers, also match transfers with different naming schemes
// Helps when transitioning between TRANSFER_INCLUDE_DIAGRAM = true/false
// false (default) → Only match exact naming scheme
// true → Also match: transfer_a_to_b matches transfer_a_to_b_diagram-name (and vice versa)
const TRANSFER_FUZZY_MATCH = false;

// v1.7.3: MARKER TEXT CLEANUP
// Controls whether marker text (e.g., "transfer", "asset=Name") is cleaned from elements after processing
// true (default) → Remove marker text after linking (cleaner diagrams)
// false → Keep marker text visible (useful for testing/debugging)
const CLEAN_MARKER_TEXT = true;

// DEBUG FILE LOGGING
// Writes debug output to a file for easier review (especially for AI assistants)
const DEBUG_LOG_TO_FILE = true;           // Enable/disable file logging
const DEBUG_LOG_FILE = "debug.log";        // Log file name (relative to vault root)
const DEBUG_LOG_CLEAR_MODE = "on_run";     // "on_run" = clear at script start, "manual" = keep appending
/****************************************************************/

/* ---------- helpers ---------- */
const exists = p => !!app.vault.getAbstractFileByPath(p);
const create = (p,c) => app.vault.create(p,c);
const read = (p) => app.vault.read(app.vault.getAbstractFileByPath(p));
const write = (p,c) => app.vault.modify(app.vault.getAbstractFileByPath(p), c);
const bn = p => p.split("/").pop().replace(/\.md$/i,"");
const slug = s => s.replace(/[\\/#^|?%*:<>"]/g," ").trim().replace(/\s+/g,"-").toLowerCase() || "unnamed";
const rnd4 = () => Math.random().toString(36).slice(2,6);
const nowISO = () => new Date().toISOString();
const note = m => DEBUG && new Notice(m, 4000);

/* ---------- debug file logging ---------- */
let debugLogBuffer = [];

function clog(m) {
  if (!DEBUG) return;
  console.log("🔧 DFD:", m);
  if (DEBUG_LOG_TO_FILE) {
    debugLogBuffer.push(`[${new Date().toISOString()}] ${m}`);
  }
}

async function initDebugLog() {
  if (!DEBUG_LOG_TO_FILE) return;

  const logFile = app.vault.getAbstractFileByPath(DEBUG_LOG_FILE);

  if (DEBUG_LOG_CLEAR_MODE === "on_run") {
    // Clear/create the log file at script start
    const header = `=== Linkify DFD Debug Log ===\nStarted: ${new Date().toISOString()}\nDiagram: ${ea.targetView?.file?.path || "unknown"}\n${"=".repeat(50)}\n\n`;
    if (logFile) {
      await app.vault.modify(logFile, header);
    } else {
      await app.vault.create(DEBUG_LOG_FILE, header);
    }
  } else {
    // Manual mode - append with separator
    const separator = `\n\n${"=".repeat(50)}\n=== New Run: ${new Date().toISOString()} ===\nDiagram: ${ea.targetView?.file?.path || "unknown"}\n${"=".repeat(50)}\n\n`;
    if (logFile) {
      const existing = await app.vault.read(logFile);
      await app.vault.modify(logFile, existing + separator);
    } else {
      const header = `=== Linkify DFD Debug Log ===\nStarted: ${new Date().toISOString()}\nDiagram: ${ea.targetView?.file?.path || "unknown"}\n${"=".repeat(50)}\n\n`;
      await app.vault.create(DEBUG_LOG_FILE, header);
    }
  }
}

async function flushDebugLog() {
  if (!DEBUG_LOG_TO_FILE || debugLogBuffer.length === 0) return;

  const logFile = app.vault.getAbstractFileByPath(DEBUG_LOG_FILE);
  if (logFile) {
    const existing = await app.vault.read(logFile);
    const newContent = existing + debugLogBuffer.join("\n") + "\n";
    await app.vault.modify(logFile, newContent);
    debugLogBuffer = [];
  }
}

// Initialize debug log at script start
await initDebugLog();

// Normalize vault paths
function normalizePath(path) {
  if (!path) return "";
  return path.replace(/^\/+|\/+$/g, "");
}

// v1.7.4: Find the next available number for sequential collision handling
function findNextTransferNumber(folder, objA, direction, objB) {
  clog(`🔢 Finding next transfer number for: ${objA} ${direction} ${objB}`);

  // Only used when TRANSFER_COLLISION_MODE is "sequential"
  if (TRANSFER_COLLISION_MODE !== "sequential") {
    clog(`  Collision mode is "${TRANSFER_COLLISION_MODE}", not sequential - returning null`);
    return null;
  }

  const transferFolder = app.vault.getAbstractFileByPath(folder);
  if (!transferFolder || !transferFolder.children) {
    clog(`  Folder not found or empty: ${folder}`);
    return 1;
  }

  // Build regex patterns based on TRANSFER_POSITION setting
  let pattern;
  if (TRANSFER_POSITION === "prefix") {
    // Match: transfer_N_objA_direction_objB.md where N is optional (first one has no number)
    // First file: transfer_objA_direction_objB.md
    // Subsequent: transfer_2_objA_direction_objB.md, transfer_3_objA_direction_objB.md
    pattern = new RegExp(
      `^transfer(?:${OBJECT_SEPARATOR}(\\d+))?${OBJECT_SEPARATOR}${escapeRegex(objA)}${OBJECT_SEPARATOR}${escapeRegex(direction)}${OBJECT_SEPARATOR}${escapeRegex(objB)}\\.md$`,
      'i'
    );
  } else {
    // "suffix" (default) - Match: transfer_objA_direction_objB_N.md where N is optional
    // First file: transfer_objA_direction_objB.md
    // Subsequent: transfer_objA_direction_objB_2.md, transfer_objA_direction_objB_3.md
    pattern = new RegExp(
      `^transfer${OBJECT_SEPARATOR}${escapeRegex(objA)}${OBJECT_SEPARATOR}${escapeRegex(direction)}${OBJECT_SEPARATOR}${escapeRegex(objB)}(?:${OBJECT_SEPARATOR}(\\d+))?\\.md$`,
      'i'
    );
  }

  let maxNumber = 0;
  let hasBaseFile = false;

  for (const file of transferFolder.children) {
    if (file.extension !== "md") continue;

    const match = file.name.match(pattern);
    if (match) {
      if (match[1]) {
        // Has a number
        const num = parseInt(match[1], 10);
        if (num > maxNumber) maxNumber = num;
      } else {
        // Base file without number
        hasBaseFile = true;
      }
    }
  }

  // If no files exist, return 1 (will create base file without number)
  // If base file exists but no numbered files, return 2
  // If numbered files exist, return max + 1
  let nextNumber;
  if (!hasBaseFile && maxNumber === 0) {
    nextNumber = 1;  // First file - no number needed
  } else if (hasBaseFile && maxNumber === 0) {
    nextNumber = 2;  // Base exists, start numbering at 2
  } else {
    nextNumber = maxNumber + 1;
  }

  clog(`  Found: hasBaseFile=${hasBaseFile}, maxNumber=${maxNumber}, nextNumber=${nextNumber}`);
  return nextNumber;
}

// Helper to escape regex special characters
function escapeRegex(str) {
  return str.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
}

// Smart search for existing pages by name
function findExistingPageByName(targetName, kind) {
  clog(`🔍 Smart searching for existing page: "${targetName}" (${kind})`);

  const searchName = slug(targetName);
  const exactFileName = `${searchName}.md`;

  // Strategy 1: Look in the target folder first
  const targetFolder = getTargetFolder(kind);
  const primaryPath = `${targetFolder}/${exactFileName}`;

  clog(`  Checking primary path: ${primaryPath}`);
  if (exists(primaryPath)) {
    clog(`  ✓ Found exact match: ${primaryPath}`);
    return primaryPath;
  }

  if (!SEARCH_ALL_SUBFOLDERS) {
    clog(`  ✗ Not found, search limited to primary folder`);
    return null;
  }

  // Strategy 2: Search relevant subfolders (respects CROSS_TYPE_MATCHING setting)
  const searchFolders = [];
  if (kind === "asset") {
    searchFolders.push("Assets");
    if (CROSS_TYPE_MATCHING) searchFolders.push("Entities");
  } else if (kind === "entity") {
    searchFolders.push("Entities");
    if (CROSS_TYPE_MATCHING) searchFolders.push("Assets");
  }

  for (const folder of searchFolders) {
    const searchPath = `${ROOT}/${folder}/${exactFileName}`;
    clog(`  Checking ${folder}: ${searchPath}`);
    if (exists(searchPath)) {
      clog(`  ✓ Found in ${folder}: ${searchPath}`);
      return searchPath;
    }
  }

  // Strategy 3: Search by basename across entire vault (last resort)
  // Respects CROSS_TYPE_MATCHING - only matches same type unless cross-matching enabled
  clog(`  Searching vault-wide for basename: ${searchName}`);
  const allFiles = app.vault.getMarkdownFiles();

  // Build list of allowed folder patterns based on kind and CROSS_TYPE_MATCHING
  const allowedFolders = [];
  if (kind === "asset") {
    allowedFolders.push("/Assets/", "/assets/");
    if (CROSS_TYPE_MATCHING) allowedFolders.push("/Entities/", "/entities/");
  } else if (kind === "entity") {
    allowedFolders.push("/Entities/", "/entities/");
    if (CROSS_TYPE_MATCHING) allowedFolders.push("/Assets/", "/assets/");
  }

  const found = allFiles.find(f => {
    if (f.basename.toLowerCase() !== searchName.toLowerCase()) return false;

    // If we have allowed folders defined, check if file is in one of them
    if (allowedFolders.length > 0) {
      const pathLower = "/" + f.path.toLowerCase();
      const inAllowedFolder = allowedFolders.some(folder => pathLower.includes(folder.toLowerCase()));
      if (!inAllowedFolder) {
        clog(`    Skipping ${f.path} - not in allowed folders for ${kind}`);
        return false;
      }
    }
    return true;
  });

  if (found) {
    clog(`  ✓ Found vault-wide: ${found.path}`);
    return found.path;
  }

  clog(`  ✗ No existing page found for "${targetName}"`);
  return null;
}

// Detect if arrow is bidirectional
function isBidirectional(arrow) {
  const hasStart = arrow.startArrowhead && arrow.startArrowhead !== null;
  const hasEnd = arrow.endArrowhead && arrow.endArrowhead !== null;
  const result = hasStart && hasEnd;
  clog(`Arrow ${arrow.id}: startArrowhead=${arrow.startArrowhead}, endArrowhead=${arrow.endArrowhead} → bidirectional: ${result}`);
  return result;
}

// Determine arrow direction
function determineArrowDirection(arrow) {
  clog(`\n--- Determining direction for arrow ${arrow.id} ---`);
  clog(`  Direction method: ${DIRECTION_DETERMINATION}`);

  // DEBUG: Dump full binding structure to diagnose 2.18.x changes
  clog(`  DEBUG arrow.startBinding: ${JSON.stringify(arrow.startBinding)}`);
  clog(`  DEBUG arrow.endBinding: ${JSON.stringify(arrow.endBinding)}`);

  const hasStartArrowhead = arrow.startArrowhead && arrow.startArrowhead !== null;
  const hasEndArrowhead = arrow.endArrowhead && arrow.endArrowhead !== null;
  const hasStartBinding = arrow.startBinding?.elementId;
  const hasEndBinding = arrow.endBinding?.elementId;

  clog(`  Arrowheads: start=${arrow.startArrowhead}, end=${arrow.endArrowhead}`);
  clog(`  Bindings: start=${hasStartBinding}, end=${hasEndBinding}`);

  let fromElementId = null;
  let toElementId = null;
  let directionSource = "unknown";

  switch (DIRECTION_DETERMINATION) {
    case "binding_only":
      fromElementId = hasStartBinding;
      toElementId = hasEndBinding;
      directionSource = "binding";
      clog(`  Using binding_only: ${fromElementId} → ${toElementId}`);
      break;

    case "arrowhead_priority":
      if (hasEndArrowhead && !hasStartArrowhead) {
        fromElementId = hasStartBinding;
        toElementId = hasEndBinding;
        directionSource = "end_arrowhead";
        clog(`  End arrowhead detected: ${fromElementId} → ${toElementId}`);
      } else if (hasStartArrowhead && !hasEndArrowhead) {
        fromElementId = hasEndBinding;
        toElementId = hasStartBinding;
        directionSource = "start_arrowhead";
        clog(`  Start arrowhead detected (reversed): ${fromElementId} → ${toElementId}`);
      } else if (hasEndArrowhead && hasStartArrowhead) {
        fromElementId = hasStartBinding;
        toElementId = hasEndBinding;
        directionSource = "bidirectional";
        clog(`  Bidirectional arrows: ${fromElementId} ⟷ ${toElementId}`);
      } else {
        fromElementId = hasStartBinding;
        toElementId = hasEndBinding;
        directionSource = "binding_fallback";
        clog(`  No arrowheads, using binding: ${fromElementId} → ${toElementId}`);
      }
      break;

    case "arrowhead_fallback_binding":
      if (hasEndArrowhead && !hasStartArrowhead) {
        fromElementId = hasStartBinding;
        toElementId = hasEndBinding;
        directionSource = "end_arrowhead";
        clog(`  End arrowhead: ${fromElementId} → ${toElementId}`);
      } else if (hasStartArrowhead && !hasEndArrowhead) {
        fromElementId = hasEndBinding;
        toElementId = hasStartBinding;
        directionSource = "start_arrowhead";
        clog(`  Start arrowhead (reversed): ${fromElementId} → ${toElementId}`);
      } else {
        fromElementId = hasStartBinding;
        toElementId = hasEndBinding;
        directionSource = hasStartArrowhead && hasEndArrowhead ? "bidirectional" : "binding_fallback";
        clog(`  Fallback to binding: ${fromElementId} → ${toElementId}`);
      }
      break;

    default:
      fromElementId = hasStartBinding;
      toElementId = hasEndBinding;
      directionSource = "binding_default";
      clog(`  Default binding: ${fromElementId} → ${toElementId}`);
  }

  const result = {
    fromElementId,
    toElementId,
    directionSource,
    isBidirectional: hasStartArrowhead && hasEndArrowhead
  };

  clog(`  Final direction: ${result.fromElementId} → ${result.toElementId} (source: ${result.directionSource})`);
  return result;
}

// Resolve wikilink to actual file path
function resolveLink(linkText, fromPath) {
  clog(`Resolving link: "${linkText}" from ${fromPath}`);

  let cleanLink = linkText;
  if (linkText.startsWith("[[") && linkText.endsWith("]]")) {
    cleanLink = linkText.slice(2, -2);
  }

  if (cleanLink.includes("|")) {
    cleanLink = cleanLink.split("|")[0];
  }

  let resolved = null;

  // Strategy 1: Use Obsidian's built-in resolver
  resolved = app.metadataCache.getFirstLinkpathDest(cleanLink, fromPath);

  // Strategy 2: Direct path lookup
  if (!resolved) {
    if (exists(cleanLink)) {
      resolved = app.vault.getAbstractFileByPath(cleanLink);
    } else if (exists(`${cleanLink}.md`)) {
      resolved = app.vault.getAbstractFileByPath(`${cleanLink}.md`);
    }
  }

  // Strategy 3: Search by filename
  if (!resolved) {
    const filename = cleanLink.split("/").pop();
    const allFiles = app.vault.getMarkdownFiles();
    resolved = allFiles.find(f => f.basename === filename || f.name === filename);
  }

  const result = resolved ? resolved.path : null;
  clog(`  → Resolved to: ${result}`);
  return result;
}

/* shortest wiki link */
function shortWiki(path, fromPath) {
  const file = app.vault.getAbstractFileByPath(path);
  if (!file) {
    clog(`⚠️  File not found for shortWiki: ${path}`);
    // v1.7.2: Fall back to basename if file not found (race condition with newly created files)
    const baseName = path.split("/").pop().replace(/\.md$/i, "");
    return `[[${baseName}]]`;
  }

  const linkText = app.metadataCache.fileToLinktext(file, fromPath);

  // v1.7.2: If fileToLinktext returns a path with slashes (full path), fall back to basename
  // This happens when metadata cache hasn't indexed the newly created file yet
  if (linkText.includes("/")) {
    const baseName = path.split("/").pop().replace(/\.md$/i, "");
    clog(`shortWiki: ${path} → [[${baseName}]] (fallback - fileToLinktext returned path: ${linkText})`);
    return `[[${baseName}]]`;
  }

  const result = `[[${linkText}]]`;
  clog(`shortWiki: ${path} → ${result} (from: ${fromPath})`);
  return result;
}

/* ensure folder chain */
async function ensureFolder(path) {
  if (!path) return;
  const normalizedPath = normalizePath(path);
  clog(`Ensuring folder: ${normalizedPath}`);

  const parts = normalizedPath.split("/").filter(Boolean);
  let current = "";

  for (const part of parts) {
    current = current ? `${current}/${part}` : part;
    if (!exists(current)) {
      try {
        await app.vault.createFolder(current);
        clog(`  Created folder: ${current}`);
      } catch(e) {
        clog(`  Failed to create folder ${current}: ${e.message}`);
      }
    }
  }
}

/* frontmatter helpers */
async function setFM(fp, updater) {
  const f = app.vault.getAbstractFileByPath(fp);
  if (f) await app.fileManager.processFrontMatter(f, updater);
}

async function pushArr(fp, key, value) {
  await setFM(fp, fm => {
    const arr = Array.isArray(fm[key]) ? fm[key] : (fm[key] ? [fm[key]] : []);
    if (!arr.includes(value)) arr.push(value);
    fm[key] = arr;
  });
}

async function removeFromArr(fp, key, value) {
  if (!exists(fp)) return;
  await setFM(fp, fm => {
    const arr = Array.isArray(fm[key]) ? fm[key] : (fm[key] ? [fm[key]] : []);
    const idx = arr.indexOf(value);
    if (idx > -1) {
      arr.splice(idx, 1);
      clog(`  🗑️ Removed "${value}" from ${key} in ${fp}`);
    }
    fm[key] = arr;
  });
}

// v1.7.6: Generate UUID v4 for stable diagram identity
function generateUUID() {
  return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, c => {
    const r = Math.random() * 16 | 0;
    return (c === 'x' ? r : (r & 0x3 | 0x8)).toString(16);
  });
}

// v1.7.6: Get or create stable diagram UUID from frontmatter
// Returns null for .excalidraw files (no frontmatter support)
async function ensureDiagramUUID(diagramFile) {
  // Check if file supports frontmatter (.excalidraw.md format)
  if (!diagramFile.path.endsWith('.excalidraw.md')) {
    clog(`  ⚠️ Diagram is .excalidraw format - no UUID support. Convert to .excalidraw.md for stable identity.`);
    return null;
  }

  const cache = app.metadataCache.getFileCache(diagramFile);
  const fm = cache?.frontmatter;

  if (fm?.dfd_diagram_id) {
    clog(`  🔑 Found existing diagram UUID: ${fm.dfd_diagram_id}`);
    return fm.dfd_diagram_id;
  }

  // Generate and store new UUID
  const newUUID = generateUUID();
  clog(`  🔑 Generating new diagram UUID: ${newUUID}`);

  await app.fileManager.processFrontMatter(diagramFile, fm => {
    fm.dfd_diagram_id = newUUID;
  });

  return newUUID;
}

// v1.7.1: Helper to add diagram to _source_diagrams array with migration from legacy source_drawing
// v1.7.6: Also stores diagram UUID in _source_diagram_ids array
async function addSourceDiagram(fp, sourceWiki, sourceUUID = null) {
  await setFM(fp, fm => {
    // Migrate from legacy source_drawing (singular) to _source_diagrams (array)
    if (fm.source_drawing && !fm._source_diagrams) {
      const legacyValue = fm.source_drawing;
      fm._source_diagrams = Array.isArray(legacyValue) ? legacyValue : [legacyValue];
      clog(`  📍 Migrated source_drawing → _source_diagrams: ${JSON.stringify(fm._source_diagrams)}`);
      delete fm.source_drawing;  // Remove legacy field
    }

    // Ensure wiki link array exists
    if (!fm._source_diagrams) {
      fm._source_diagrams = [];
    } else if (!Array.isArray(fm._source_diagrams)) {
      fm._source_diagrams = [fm._source_diagrams];
    }

    // Add current diagram wiki link if not already present
    if (!fm._source_diagrams.includes(sourceWiki)) {
      fm._source_diagrams.push(sourceWiki);
      clog(`  📍 Added to _source_diagrams: ${sourceWiki} (now ${fm._source_diagrams.length} diagrams)`);
    } else {
      clog(`  📍 Already in _source_diagrams: ${sourceWiki}`);
    }

    // v1.7.6: Also store diagram UUID for rename-proof matching
    if (sourceUUID) {
      if (!fm._source_diagram_ids) {
        fm._source_diagram_ids = [];
      } else if (!Array.isArray(fm._source_diagram_ids)) {
        fm._source_diagram_ids = [fm._source_diagram_ids];
      }

      if (!fm._source_diagram_ids.includes(sourceUUID)) {
        fm._source_diagram_ids.push(sourceUUID);
        clog(`  🔑 Added to _source_diagram_ids: ${sourceUUID}`);
      }
    }
  });
}

// v1.7.1: Get count of source diagrams for a transfer (for orphan detection)
function getSourceDiagramCount(fm) {
  if (fm._source_diagrams) {
    return Array.isArray(fm._source_diagrams) ? fm._source_diagrams.length : 1;
  }
  // Legacy field
  if (fm.source_drawing) {
    return Array.isArray(fm.source_drawing) ? fm.source_drawing.length : 1;
  }
  return 0;
}

// v1.7.2: Check if a transfer is owned by (was created from) the current diagram
// v1.7.6: Now checks UUID first (survives diagram renames), then falls back to wiki link
function isTransferOwnedByCurrentDiagram(fm, currentDiagramWiki, currentDiagramUUID = null) {
  // Priority 1: Check UUID match (stable, survives renames)
  if (currentDiagramUUID && fm._source_diagram_ids) {
    const ids = Array.isArray(fm._source_diagram_ids) ? fm._source_diagram_ids : [fm._source_diagram_ids];
    if (ids.includes(currentDiagramUUID)) {
      clog(`    🔑 UUID match: ${currentDiagramUUID}`);
      return true;
    }
  }

  // Priority 2: Check wiki link match (fallback for older transfers or .excalidraw files)
  if (fm._source_diagrams) {
    const diagrams = Array.isArray(fm._source_diagrams) ? fm._source_diagrams : [fm._source_diagrams];
    if (diagrams.includes(currentDiagramWiki)) {
      return true;
    }
  }

  // Legacy field fallback
  if (fm.source_drawing) {
    const legacyValue = fm.source_drawing.toString();
    return legacyValue.includes(currentDiagramWiki) || legacyValue.includes(bn(currentDiagramWiki));
  }

  return false;
}

// v1.7.5: Check if a filename matches the current naming scheme
function matchesCurrentNamingScheme(fileName, corePattern, diagramSlug) {
  if (!TRANSFER_FUZZY_MATCH) {
    // Strict mode: must match current TRANSFER_INCLUDE_DIAGRAM setting
    if (TRANSFER_INCLUDE_DIAGRAM) {
      // Expect diagram name in filename
      return fileName.includes(diagramSlug);
    } else {
      // Expect NO diagram name - filename should be exactly transfer_{core} or transfer_{core}_forward/reverse
      // Check that after the core pattern, there's only _forward, _reverse, or nothing (plus optional collision suffix)
      const afterCore = fileName.split(corePattern)[1] || "";
      const hasUnexpectedSuffix = afterCore.length > 0 &&
        !afterCore.match(/^(_forward|_reverse)?(-[a-z0-9]{4})?$/);
      return !hasUnexpectedSuffix;
    }
  }
  // Fuzzy mode: any filename with core pattern is acceptable
  return true;
}

// v1.7.2: Find existing transfer between two objects created by the current diagram
// v1.7.6: Now accepts UUID for rename-proof matching
// Returns path if found, null otherwise
async function findExistingTransferForDiagram(folder, objAName, objBName, currentDiagramWiki, currentDiagramUUID = null) {
  const transferFolder = app.vault.getAbstractFileByPath(folder);
  if (!transferFolder || !transferFolder.children) {
    return null;
  }

  // Build the expected base pattern for transfer names
  const direction = TRANSFER_DIRECTION_WORD;
  const objASlug = slug(objAName);
  const objBSlug = slug(objBName);
  const diagramSlug = slug(currentDiagramWiki.replace(/^\[\[|\]\]$/g, ""));

  // v1.7.3: Core pattern is the object-direction-object part
  const corePatternAtoB = `${objASlug}${OBJECT_SEPARATOR}${direction}${OBJECT_SEPARATOR}${objBSlug}`;
  const corePatternBtoA = `${objBSlug}${OBJECT_SEPARATOR}${direction}${OBJECT_SEPARATOR}${objASlug}`;

  for (const file of transferFolder.children) {
    if (!file.name?.endsWith(".md")) continue;
    const fileName = file.name.replace(/\.md$/, "");

    // v1.7.3: Check if filename contains the core pattern (handles all naming modes)
    // This matches: transfer_a_to_b, transfer_a_to_b_diagram, transfer_diagram_a_to_b, etc.
    const hasCore = fileName.startsWith("transfer") &&
      (fileName.includes(corePatternAtoB) || fileName.includes(corePatternBtoA));

    if (!hasCore) continue;

    // v1.7.5: Check if filename matches current naming scheme (unless fuzzy matching enabled)
    const coreUsed = fileName.includes(corePatternAtoB) ? corePatternAtoB : corePatternBtoA;
    const matchesScheme = matchesCurrentNamingScheme(fileName, coreUsed, diagramSlug);

    if (!matchesScheme) {
      clog(`  v1.7.5: Skipping ${fileName} - doesn't match current naming scheme (fuzzy=${TRANSFER_FUZZY_MATCH})`);
      continue;
    }

    const cache = app.metadataCache.getFileCache(file);
    const fm = cache?.frontmatter;

    // v1.7.6: Pass UUID for rename-proof matching
    if (fm && isTransferOwnedByCurrentDiagram(fm, currentDiagramWiki, currentDiagramUUID)) {
      clog(`  v1.7.6: Found existing transfer owned by this diagram: ${file.path}`);
      return file.path;
    }
  }

  return null;
}

// v1.7.2: Find ANY existing transfer between two objects (regardless of ownership)
// Used for explicit "transfer=reuse" marker
// Note: This function always uses fuzzy matching (ignores TRANSFER_FUZZY_MATCH setting)
// because explicit reuse requests should find any matching transfer
async function findAnyTransferBetweenObjects(folder, objAName, objBName) {
  const transferFolder = app.vault.getAbstractFileByPath(folder);
  if (!transferFolder || !transferFolder.children) {
    return null;
  }

  // Build the expected base pattern for transfer names
  const direction = TRANSFER_DIRECTION_WORD;
  const objASlug = slug(objAName);
  const objBSlug = slug(objBName);

  // v1.7.3: Core pattern is the object-direction-object part
  const corePatternAtoB = `${objASlug}${OBJECT_SEPARATOR}${direction}${OBJECT_SEPARATOR}${objBSlug}`;
  const corePatternBtoA = `${objBSlug}${OBJECT_SEPARATOR}${direction}${OBJECT_SEPARATOR}${objASlug}`;

  for (const file of transferFolder.children) {
    if (!file.name?.endsWith(".md")) continue;
    const fileName = file.name.replace(/\.md$/, "");

    // v1.7.3: Check if filename contains the core pattern (handles all naming modes)
    // v1.7.5: This function always uses fuzzy matching for explicit reuse requests
    const matchesPattern = fileName.startsWith("transfer") &&
      (fileName.includes(corePatternAtoB) || fileName.includes(corePatternBtoA));

    if (matchesPattern) {
      clog(`  v1.7.3: Found existing transfer: ${file.path}`);
      return file.path;
    }
  }

  return null;
}

async function dvInline(fp, field, val) {
  if (!WRITE_INLINE_FIELDS) return;
  const content = await read(fp);
  const line = `${field}:: ${val}`;
  if (!content.includes(line)) {
    await write(fp, content.endsWith("\n") ? content + line + "\n" : content + "\n" + line + "\n");
  }
}

/* ---------- environment ---------- */
ea.reset();
ea.setView("active");
const view = ea.targetView;
if (!view?.file) {
  new Notice("Open an Excalidraw file first");
  return;
}

const DRAW_DIR = view.file.parent?.path || "";
clog(`Drawing directory: ${DRAW_DIR}`);

// Resolve CFG_DIR relative to drawing
const RESOLVED_CFG_DIR = CFG_DIR.startsWith("./") ?
  `${DRAW_DIR}/${CFG_DIR.slice(2)}` :
  CFG_DIR;
clog(`Config directory: ${RESOLVED_CFG_DIR}`);

// Database root path resolution
const ROOT = (() => {
  switch(DB_PLACEMENT) {
    case "flat":
      return DRAW_DIR;
    case "diagram_named":
      return `${DRAW_DIR}/${bn(view.file.path)}`;
    case "db_folder":
      if (DB_DB_PARENT_PATH) {
        const normalizedParent = normalizePath(DB_DB_PARENT_PATH);
        const result = `${normalizedParent}/${DB_FOLDER_NAME}`;
        clog(`Database path: ${normalizedParent} + ${DB_FOLDER_NAME} = ${result}`);
        return result;
      } else {
        return `${DRAW_DIR}/${DB_FOLDER_NAME}`;
      }
    default:
      return DRAW_DIR;
  }
})();
clog(`Storage root: ${ROOT}`);

/* ---------- load config notes ---------- */
function allMarkdown(dir) {
  clog(`Looking for markdown files in: ${dir}`);
  const root = app.vault.getAbstractFileByPath(dir);
  if (!root) {
    clog(`  Directory not found: ${dir}`);
    return [];
  }
  const out = [];
  const walk = f => {
    if (f.children) f.children.forEach(walk);
    else if (f.extension === "md") out.push(f);
  };
  walk(root);
  clog(`  Found ${out.length} markdown files`);
  return out;
}

const CFG = new Map();
const DEFAULT_SUBFOLDERS = { asset: "Assets", entity: "Entities", transfer: "Transfers" };

// Load config notes
for (const f of allMarkdown(RESOLVED_CFG_DIR)) {
  clog(`Processing config file: ${f.path}`);
  const fc = app.metadataCache.getFileCache(f);
  const fm = fc?.frontmatter || {};
  const kind = (fm["DFD__KIND"] || "").toLowerCase();

  if (!["asset", "entity", "transfer"].includes(kind)) {
    clog(`  Skipping - invalid kind: ${kind}`);
    continue;
  }

  const markers = fm["DFD__MARKER"];
  const markerList = Array.isArray(markers) ? markers : [markers || kind];
  const subfolder = fm["DFD__SUBFOLDER"] || DEFAULT_SUBFOLDERS[kind];
  const defaults = Object.fromEntries(
    Object.entries(fm).filter(([k]) => !k.startsWith("DFD__"))
  );

  const pos = fc?.frontmatterPosition;
  const body = pos ? (await app.vault.read(f)).slice(pos.end.offset).trim() : "";

  const cfg = { kind, defaults, subfolder, body };
  for (const marker of markerList) {
    const key = marker.toString().toLowerCase();
    CFG.set(key, cfg);
    clog(`  Registered marker "${key}" → ${kind}`);
  }
}

function getConfig(key) {
  const config = CFG.get(key.toLowerCase()) || {
    kind: key,
    defaults: { schema: `dfd-${key}-v1`, type: key },
    subfolder: DEFAULT_SUBFOLDERS[key] || key,
    body: ""
  };
  clog(`getConfig("${key}") → kind: ${config.kind}, subfolder: ${config.subfolder}`);
  return config;
}

function getTargetFolder(kind) {
  const config = getConfig(kind);
  const result = `${ROOT}/${config.subfolder}`;
  clog(`getTargetFolder(${kind}) → ${result}`);
  return result;
}

await ensureFolder(getTargetFolder("asset"));
await ensureFolder(getTargetFolder("entity"));
await ensureFolder(getTargetFolder("transfer"));

/* ---------- scene parsing ---------- */
const MARK = /^(?:\[\[)?(?:tpl:)?\s*(asset|entity|transfer)\s*(?:=\s*([^\]]+))?(?:\]\])?$/i;
const els = ea.getViewElements ? ea.getViewElements() : ea.getElements();
const byId = Object.fromEntries(els.map(e => [e.id, e]));

// v1.6.1: Track which elements have been copied to elementsDict to prevent duplicates
const copiedToEA = new Set();
const modifiedTextElements = new Set();  // Track text elements we've already modified

// v1.6.1: Safe copy helper - only copies elements that haven't been copied yet
function safeCopyToEA(elements) {
  const toCopy = elements.filter(el => !copiedToEA.has(el.id));
  if (toCopy.length > 0) {
    ea.copyViewElementsToEAforEditing(toCopy);
    toCopy.forEach(el => copiedToEA.add(el.id));
    clog(`  📋 Copied ${toCopy.length} elements to EA (skipped ${elements.length - toCopy.length} already copied)`);
  } else {
    clog(`  📋 All ${elements.length} elements already copied to EA, skipping`);
  }
}

clog(`Found ${els.length} elements on canvas`);

function getGroupEls(el) {
  const gid = el.groupIds?.at(-1);
  const group = gid ? els.filter(x => x.groupIds?.includes(gid)) : [el];
  clog(`Element ${el.id} (${el.type}) has group of ${group.length} elements`);
  return group;
}

function firstText(group) {
  const textEl = group.find(e => e.type === "text" && (e.text || "").trim());
  const result = textEl?.text.trim() || "";
  if (result) clog(`  Found text in group: "${result}"`);
  return result;
}

// NEW: Find bound text for a shape (text typed directly into rectangle/ellipse/etc)
function getBoundText(el) {
  // Check if this element has boundElements with type "text"
  if (!el.boundElements || !Array.isArray(el.boundElements)) return null;

  const boundTextRef = el.boundElements.find(b => b.type === "text");
  if (!boundTextRef) return null;

  const boundTextEl = byId[boundTextRef.id];
  if (boundTextEl && boundTextEl.type === "text" && boundTextEl.text) {
    clog(`  Found bound text for ${el.id}: "${boundTextEl.text.trim()}"`);
    return boundTextEl;
  }
  return null;
}

function parseMarker(s) {
  if (!s) return null;
  const m = s.trim().match(MARK);
  if (!m) return null;

  // v1.6.2: Normalize whitespace in customName (handle multiline text from bound text)
  let customName = (m[2] || "").trim() || null;
  if (customName) {
    customName = customName.replace(/[\r\n]+/g, ' ').replace(/\s+/g, ' ').trim();
  }

  const result = { kind: m[1].toLowerCase(), customName };

  // v1.7.2: Parse transfer=reuse syntax
  // "transfer=reuse" → reuse existing transfer with same endpoints
  // "transfer=reuse:[[path]]" → link to specific existing transfer
  if (result.kind === "transfer" && customName) {
    if (customName.toLowerCase() === "reuse") {
      result.reuseMode = "auto";  // Reuse existing with same endpoints
      result.customName = null;   // Clear customName since it's the reuse directive
      clog(`  v1.7.2: Detected transfer=reuse (auto mode)`);
    } else if (customName.toLowerCase().startsWith("reuse:")) {
      const target = customName.slice(6).trim();  // Remove "reuse:" prefix
      result.reuseMode = "specific";
      result.reuseTarget = target;  // Could be "[[Transfer Name]]" or "Transfer Name"
      result.customName = null;     // Clear customName
      clog(`  v1.7.2: Detected transfer=reuse:${target} (specific mode)`);
    }
    // Otherwise customName is a regular custom name for a new transfer
  }

  clog(`  Parsed marker "${s.replace(/\n/g, '\\n')}" → kind: ${result.kind}, customName: ${result.customName}${result.reuseMode ? `, reuseMode: ${result.reuseMode}` : ""}`);
  return result;
}

// Strict classification - only processes explicit markers
function classifyElement(el) {
  clog(`\n--- Classifying element ${el.id} (${el.type}) ---`);
  const group = getGroupEls(el);
  let hasStaleLink = false;  // v1.7.3: Track if element has link to deleted file

  // Check customData first
  const cd = el.customData?.dfd || el.customData?.DFD;
  if (cd?.kind && ["asset", "entity", "transfer"].includes(cd.kind)) {
    clog(`  ✓ Found in customData: ${cd.kind}`);
    return { kind: cd.kind, customName: null };
  }

  // Check element.link (what "Add link" sets) - MUST have marker text
  if (typeof el.link === "string" && el.link.trim()) {
    clog(`  Checking element.link: "${el.link}"`);
    const linkMarker = parseMarker(el.link);
    if (linkMarker) {
      clog(`  ✓ Found valid marker in element.link: ${linkMarker.kind}`);
      return linkMarker;
    }

    // Check if element.link is already a legitimate wikilink to existing file
    if (el.link.includes("[[") && el.link.includes("]]")) {
      const resolvedPath = resolveLink(el.link, view.file.path);
      if (resolvedPath && exists(resolvedPath)) {
        clog(`  ✓ Element already links to legitimate file: ${resolvedPath}`);
        return { kind: "existing", customName: null, existingPath: resolvedPath };
      } else {
        // v1.7.3: Link points to non-existent file (deleted?) - continue checking for markers
        clog(`  ⚠️ Element has stale link to non-existent file: ${el.link}`);
        clog(`  ℹ️ Will check for new markers (bound text, group text) to re-process`);
        hasStaleLink = true;
      }
    }
  }

  // NEW: Check bound text (text typed directly into shape) - MUST have marker text
  const boundTextEl = getBoundText(el);
  if (boundTextEl) {
    clog(`  Checking bound text: "${boundTextEl.text.trim()}"`);
    const boundMarker = parseMarker(boundTextEl.text.trim());
    if (boundMarker) {
      clog(`  ✓ Found valid marker in bound text: ${boundMarker.kind}`);
      // Store reference to bound text element for cleanup later
      boundMarker.boundTextElementId = boundTextEl.id;
      return boundMarker;
    }
  }

  // Check grouped text - MUST have marker text
  const groupText = firstText(group);
  if (groupText) {
    clog(`  Checking group text: "${groupText}"`);
    const textMarker = parseMarker(groupText);
    if (textMarker) {
      clog(`  ✓ Found valid marker in group text: ${textMarker.kind}`);
      return textMarker;
    }
  }

  // Fallback only if not requiring explicit markers
  if (!REQUIRE_EXPLICIT_MARKER) {
    if (el.type === "arrow") {
      clog(`  ✓ Fallback: arrow → transfer`);
      return { kind: "transfer", customName: null };
    }
    if (["rectangle", "ellipse", "diamond", "image", "frame"].includes(el.type)) {
      clog(`  ✓ Fallback: ${el.type} → asset`);
      return { kind: "asset", customName: null };
    }
  }

  // v1.7.3: Helpful warning when element has stale link but no marker to re-process
  if (hasStaleLink && el.type === "arrow") {
    clog(`  ⚠️ Arrow has stale link to deleted transfer but no "transfer" marker found`);
    clog(`  ℹ️ To re-process: Double-click arrow and type "transfer" (bound text), then re-run`);
    note(`↯ Arrow has stale link. Add "transfer" as bound text (double-click arrow) to re-process.`);
  }

  clog(`  ✗ No valid classification found`);
  return null;
}

/* ---------- create/ensure nodes ---------- */
async function createNodeNote(kind, customName, shapeType) {
  clog(`\n--- Creating ${kind} note ---`);
  const config = getConfig(kind);
  const folder = getTargetFolder(kind);
  await ensureFolder(folder);

  let fileName;
  if (customName && CUSTOM_NAME_MODE === "replace") {
    fileName = slug(customName);
    clog(`  Using custom name (replace): ${fileName}`);
  } else if (customName && CUSTOM_NAME_MODE === "inject") {
    fileName = `${kind}-${shapeType}-${slug(customName)}-${rnd4()}`;
    clog(`  Using custom name (inject): ${fileName}`);
  } else {
    fileName = `${kind}-${shapeType}-${rnd4()}`;
    clog(`  Using generated name: ${fileName}`);
  }

  let path = `${folder}/${fileName}.md`;
  let counter = 2;
  while (exists(path)) {
    path = `${folder}/${fileName}-${counter}.md`;
    counter++;
  }

  clog(`  Final path: ${path}`);

  const fm = Object.assign({}, config.defaults, {
    name: customName || config.defaults.name || kind,
    created: nowISO()
  });

  const fmLines = ["---"];
  for (const [key, value] of Object.entries(fm)) {
    fmLines.push(`${key}: ${typeof value === "string" ? `"${value.replace(/"/g, '\\"')}"` : JSON.stringify(value)}`);
  }
  fmLines.push("---");

  const content = config.body ?
    fmLines.join("\n") + "\n\n" + config.body :
    fmLines.join("\n") + "\n\n";

  await create(path, content);
  clog(`  ✓ Created note: ${path}`);
  return path;
}

// Better existing link detection and smart custom name matching
async function ensureNodeLinked(el, kind, customName, existingPath = null, boundTextElementId = null) {
  if (!["asset", "entity"].includes(kind) && kind !== "existing") return null;

  clog(`\n--- Ensuring ${kind} node linked ---`);
  const group = getGroupEls(el);

  // If classified as "existing", normalize the link and update source_drawings
  if (kind === "existing" && existingPath) {
    clog(`  ✓ Element already has legitimate link, normalizing: ${existingPath}`);
    const wikiLink = shortWiki(existingPath, view.file.path);
    const largest = group.reduce((a, b) =>
      (a.width * a.height >= b.width * b.height ? a : b), group[0]
    );
    largest.link = wikiLink;
    safeCopyToEA([largest]);

    // Track which diagrams this node appears in
    const sourceWiki = shortWiki(view.file.path, view.file.path);
    await pushArr(existingPath, "source_drawings", sourceWiki);
    clog(`  📍 Added source_drawing: ${sourceWiki}`);

    return existingPath;
  }

  // Check for existing legitimate wikilinks
  const existingLinkEl = group.find(e =>
    typeof e.link === "string" &&
    e.link.includes("[[") &&
    e.link.includes("]]") &&
    !parseMarker(e.link) // Exclude marker links like "asset=name"
  );

  if (existingLinkEl) {
    clog(`  Found existing wikilink: ${existingLinkEl.link}`);
    const actualPath = resolveLink(existingLinkEl.link, view.file.path);
    if (actualPath && exists(actualPath)) {
      clog(`  ✓ Existing file found, reusing: ${actualPath}`);
      const wikiLink = shortWiki(actualPath, view.file.path);
      const largest = group.reduce((a, b) =>
        (a.width * a.height >= b.width * b.height ? a : b), group[0]
      );
      largest.link = wikiLink;
      safeCopyToEA([largest]);

      // Track which diagrams this node appears in
      const sourceWiki = shortWiki(view.file.path, view.file.path);
      await pushArr(actualPath, "source_drawings", sourceWiki);
      clog(`  📍 Added source_drawing: ${sourceWiki}`);

      return actualPath;
    } else {
      clog(`  ✗ Existing link points to non-existent file: ${actualPath}`);
    }
  }

  // Smart custom name matching - check if page with exact name already exists
  if (customName && SMART_CUSTOM_NAME_MATCHING) {
    const existingByName = findExistingPageByName(customName, kind);
    if (existingByName) {
      clog(`  ✓ Found existing page for custom name "${customName}": ${existingByName}`);
      const wikiLink = shortWiki(existingByName, view.file.path);

      // Set link on the main shape element
      el.link = wikiLink;

      // Track elements to update
      const elementsToUpdate = [el];

      // Clean up bound text if marker was in bound text
      if (boundTextElementId) {
        clog(`  🔍 Looking for bound text element: ${boundTextElementId}`);
        const boundTextEl = byId[boundTextElementId];
        if (boundTextEl) {
          clog(`  🔍 Found bound text element, type: ${boundTextEl.type}, text: "${boundTextEl.text}"`);
          if (boundTextEl.text) {
            const oldText = boundTextEl.text;
            // Keep only the custom name part (after the =)
            boundTextEl.text = customName;
            boundTextEl.rawText = customName;
            boundTextEl.originalText = customName;
            clog(`  ✂️ Cleaned bound text: "${oldText}" → "${boundTextEl.text}"`);
            elementsToUpdate.push(boundTextEl);
            clog(`  📝 Added bound text to elementsToUpdate, total: ${elementsToUpdate.length}`);
          }
        } else {
          clog(`  ⚠️ Bound text element NOT found in byId map!`);
        }
      } else {
        clog(`  ℹ️ No boundTextElementId provided`);
      }

      // Also clean up grouped text elements
      // v1.6.1: Skip text elements that are already modified
      group.forEach(e => {
        if (!elementsToUpdate.includes(e)) {
          elementsToUpdate.push(e);
        }
        // Remove marker text to prevent duplicates - but only if not already modified
        if (typeof e.text === "string" && e.text.match(MARK) && !modifiedTextElements.has(e.id)) {
          const oldText = e.text;
          const newText = e.text.replace(MARK, "").trim();
          e.text = newText;
          e.rawText = newText;
          e.originalText = newText;
          modifiedTextElements.add(e.id);  // v1.6.1: Mark as modified
          clog(`  Cleaned marker text: "${oldText}" → "${e.text}"`);
        }
      });

      // First copy elements to EA's elementsDict
      safeCopyToEA(elementsToUpdate);

      // Now modify the text via elementsDict (this is the correct way!)
      // v1.6.1: Only modify if not already modified
      if (boundTextElementId && !modifiedTextElements.has(boundTextElementId)) {
        try {
          const eaTextEl = ea.getElement(boundTextElementId);
          if (eaTextEl) {
            clog(`  📝 Modifying text in elementsDict: "${eaTextEl.text}" → "${customName}"`);
            eaTextEl.text = customName;
            eaTextEl.rawText = customName;
            eaTextEl.originalText = customName;
            modifiedTextElements.add(boundTextElementId);  // v1.6.1: Mark as modified

            // Refresh text element size after modification
            if (ea.refreshTextElementSize) {
              ea.refreshTextElementSize(boundTextElementId);
              clog(`  🔄 Refreshed text element size for ${boundTextElementId}`);
            }
          }
        } catch (e) {
          clog(`  ⚠️ Could not modify text in elementsDict: ${e.message}`);
        }
      } else if (boundTextElementId) {
        clog(`  ⏭️ Skipping text modification - already modified: ${boundTextElementId}`);
      }

      // Track which diagrams this node appears in
      const sourceWiki = shortWiki(view.file.path, view.file.path);
      await pushArr(existingByName, "source_drawings", sourceWiki);
      clog(`  📍 Added source_drawing: ${sourceWiki}`);

      note(`✓ ${kind} (reused) → ${wikiLink}`);
      clog(`  ✓ Linked to existing ${kind} → ${wikiLink}`);

      return existingByName;
    }
  }

  // Create new note
  const path = await createNodeNote(kind, customName, el.type);
  const wikiLink = shortWiki(path, view.file.path);

  // Set link on the main shape element
  el.link = wikiLink;

  // Track elements to update
  const elementsToUpdate = [el];

  // Clean up bound text if marker was in bound text
  // v1.6.1: Only modify if not already modified
  if (boundTextElementId && !modifiedTextElements.has(boundTextElementId)) {
    clog(`  🔍 Looking for bound text element: ${boundTextElementId}`);
    const boundTextEl = byId[boundTextElementId];
    if (boundTextEl) {
      clog(`  🔍 Found bound text element, type: ${boundTextEl.type}, text: "${boundTextEl.text}"`);
      if (boundTextEl.text) {
        const oldText = boundTextEl.text;
        // Keep only the custom name part (after the =), or empty if no custom name
        const newText = customName || "";
        boundTextEl.text = newText;
        boundTextEl.rawText = newText;  // Also update rawText if it exists
        boundTextEl.originalText = newText;  // And originalText
        clog(`  ✂️ Cleaned bound text: "${oldText}" → "${boundTextEl.text}"`);
        elementsToUpdate.push(boundTextEl);
        modifiedTextElements.add(boundTextElementId);  // v1.6.1: Mark as modified
        clog(`  📝 Added bound text to elementsToUpdate, total: ${elementsToUpdate.length}`);
      }
    } else {
      clog(`  ⚠️ Bound text element NOT found in byId map!`);
    }
  } else if (boundTextElementId) {
    clog(`  ⏭️ Skipping bound text - already modified: ${boundTextElementId}`);
  } else {
    clog(`  ℹ️ No boundTextElementId provided`);
  }

  // Also clean up grouped text elements
  // v1.6.1: Skip text elements that are already modified
  group.forEach(e => {
    if (!elementsToUpdate.includes(e)) {
      elementsToUpdate.push(e);
    }
    // Remove marker text to prevent duplicates - but only if not already modified
    if (typeof e.text === "string" && e.text.match(MARK) && !modifiedTextElements.has(e.id)) {
      const oldText = e.text;
      const newText = e.text.replace(MARK, "").trim();
      e.text = newText;
      e.rawText = newText;
      e.originalText = newText;
      modifiedTextElements.add(e.id);  // v1.6.1: Mark as modified
      clog(`  Cleaned marker text: "${oldText}" → "${e.text}"`);
    }
  });

  // First copy elements to EA's elementsDict
  safeCopyToEA(elementsToUpdate);

  // Now modify the text via elementsDict (this is the correct way!)
  // v1.6.1: Only if not already modified
  if (boundTextElementId && !modifiedTextElements.has(boundTextElementId)) {
    try {
      const eaTextEl = ea.getElement(boundTextElementId);
      if (eaTextEl) {
        const newText = customName || "";
        clog(`  📝 Modifying text in elementsDict: "${eaTextEl.text}" → "${newText}"`);
        eaTextEl.text = newText;
        eaTextEl.rawText = newText;
        eaTextEl.originalText = newText;
        modifiedTextElements.add(boundTextElementId);  // v1.6.1: Mark as modified

        // Refresh text element size after modification
        if (ea.refreshTextElementSize) {
          ea.refreshTextElementSize(boundTextElementId);
          clog(`  🔄 Refreshed text element size for ${boundTextElementId}`);
        }
      }
    } catch (e) {
      clog(`  ⚠️ Could not modify text in elementsDict: ${e.message}`);
    }
  } else if (boundTextElementId) {
    clog(`  ⏭️ Skipping elementsDict modification - already done: ${boundTextElementId}`);
  }

  // Track which diagrams this node appears in (for new notes)
  const sourceWiki = shortWiki(view.file.path, view.file.path);
  await pushArr(path, "source_drawings", sourceWiki);
  clog(`  📍 Added source_drawing: ${sourceWiki}`);

  note(`✓ ${kind} (new) → ${wikiLink}`);
  clog(`  ✓ Linked ${kind} → ${wikiLink}`);

  return path;
}

/* ---------- Transfer naming helper ---------- */
function generateTransferBaseName(objectAPath, objectBPath, isBidirectional = false) {
  // Generate the base transfer name (without suffix like _forward/_reverse or numbering)
  const direction = isBidirectional ? TRANSFER_DIRECTION_WORD : "to";
  const objA = slug(bn(objectAPath));
  const objB = slug(bn(objectBPath));
  return `transfer${OBJECT_SEPARATOR}${objA}${OBJECT_SEPARATOR}${direction}${OBJECT_SEPARATOR}${objB}`;
}

/* ---------- Transfer creation helper - v1.7.4 with orthogonal naming settings ---------- */
async function createTransferNote(objectAPath, objectBPath, classification, isBidirectional = false, suffix = "", diagramName = null) {
  const config = getConfig("transfer");
  const folder = getTargetFolder("transfer");

  const direction = isBidirectional ? TRANSFER_DIRECTION_WORD : "to";
  const objA = slug(bn(objectAPath));
  const objB = slug(bn(objectBPath));

  // v1.7.4: Get diagram name for filename if TRANSFER_INCLUDE_DIAGRAM is true
  const diagramSlug = diagramName ? slug(diagramName) : slug(bn(view.file.path));

  let fileName;
  let path;

  // v1.7.4: Build filename using orthogonal settings
  clog(`  Building filename: INCLUDE_DIAGRAM=${TRANSFER_INCLUDE_DIAGRAM}, POSITION=${TRANSFER_POSITION}, COLLISION_MODE=${TRANSFER_COLLISION_MODE}`);

  // Step 1: Build base filename (without diagram or collision handling)
  const baseName = `transfer${OBJECT_SEPARATOR}${objA}${OBJECT_SEPARATOR}${direction}${OBJECT_SEPARATOR}${objB}${suffix}`;

  // Step 2: Add diagram name if enabled
  if (TRANSFER_INCLUDE_DIAGRAM) {
    if (TRANSFER_POSITION === "prefix") {
      // transfer_my-diagram_asset-x_to_asset-y.md
      fileName = `transfer${OBJECT_SEPARATOR}${diagramSlug}${OBJECT_SEPARATOR}${objA}${OBJECT_SEPARATOR}${direction}${OBJECT_SEPARATOR}${objB}${suffix}`;
    } else {
      // transfer_asset-x_to_asset-y_my-diagram.md (suffix, default)
      fileName = `${baseName}${OBJECT_SEPARATOR}${diagramSlug}`;
    }
    clog(`  Including diagram name (${TRANSFER_POSITION}): ${fileName}`);
  } else {
    // No diagram name
    fileName = baseName;
    clog(`  No diagram name: ${fileName}`);
  }

  path = `${folder}/${fileName}.md`;

  // Step 3: Handle collision if file exists
  if (exists(path)) {
    clog(`  File exists: ${path} - applying collision handling (${TRANSFER_COLLISION_MODE})`);

    if (TRANSFER_COLLISION_MODE === "sequential") {
      // Find next available number
      const nextNum = findNextTransferNumber(folder, objA, direction, objB);
      if (TRANSFER_POSITION === "prefix") {
        // transfer_2_asset-x_to_asset-y.md (or transfer_my-diagram_2_... if diagram included)
        if (TRANSFER_INCLUDE_DIAGRAM) {
          fileName = `transfer${OBJECT_SEPARATOR}${diagramSlug}${OBJECT_SEPARATOR}${nextNum}${OBJECT_SEPARATOR}${objA}${OBJECT_SEPARATOR}${direction}${OBJECT_SEPARATOR}${objB}${suffix}`;
        } else {
          fileName = `transfer${OBJECT_SEPARATOR}${nextNum}${OBJECT_SEPARATOR}${objA}${OBJECT_SEPARATOR}${direction}${OBJECT_SEPARATOR}${objB}${suffix}`;
        }
      } else {
        // transfer_asset-x_to_asset-y_2.md or transfer_asset-x_to_asset-y_my-diagram_2.md
        if (TRANSFER_INCLUDE_DIAGRAM) {
          fileName = `${baseName}${OBJECT_SEPARATOR}${diagramSlug}${OBJECT_SEPARATOR}${nextNum}`;
        } else {
          fileName = `${baseName}${OBJECT_SEPARATOR}${nextNum}`;
        }
      }
      path = `${folder}/${fileName}.md`;
      clog(`  Using sequential number #${nextNum}: ${path}`);
    } else if (TRANSFER_COLLISION_MODE === "random") {
      // Add random suffix
      path = `${folder}/${fileName}-${rnd4()}.md`;
      clog(`  Using random suffix: ${path}`);
    } else {
      // TRANSFER_COLLISION_MODE === "none" - fallback to random (shouldn't happen often)
      path = `${folder}/${fileName}-${rnd4()}.md`;
      clog(`  Collision mode 'none' but collision occurred - fallback to random: ${path}`);
    }
  }

  const fm = Object.assign({}, config.defaults, {
    name: classification.customName || config.defaults.name || "transfer",
    created: nowISO(),
    _status: "active"  // v1.7.0: Default status for orphan tracking
  });

  if (isBidirectional) {
    fm.direction = "bidirectional";
    fm.note = "This transfer represents bidirectional data flow";
  }

  const fmLines = ["---"];
  for (const [key, value] of Object.entries(fm)) {
    fmLines.push(`${key}: ${typeof value === "string" ? `"${value.replace(/"/g, '\\"')}"` : JSON.stringify(value)}`);
  }
  fmLines.push("---");

  const content = config.body ?
    fmLines.join("\n") + "\n\n" + config.body :
    fmLines.join("\n") + "\n\n";

  await create(path, content);
  clog(`  ✓ Created transfer note: ${path}`);

  return path;
}

/* ---------- v1.7.3: Helper to clean transfer marker text from arrow ---------- */
function cleanTransferMarkerText(arr) {
  // v1.7.3: Check if cleanup is enabled
  if (!CLEAN_MARKER_TEXT) {
    clog(`  ℹ️ Marker text cleanup disabled (CLEAN_MARKER_TEXT = false)`);
    return;
  }

  // Clean "transfer" marker text from arrow bound text
  const boundTextEl = getBoundText(arr);
  if (boundTextEl && parseMarker(boundTextEl.text?.trim()) && !modifiedTextElements.has(boundTextEl.id)) {
    clog(`  ✂️ Cleaning transfer marker from arrow bound text: "${boundTextEl.text}"`);
    boundTextEl.text = "";
    boundTextEl.rawText = "";
    boundTextEl.originalText = "";
    modifiedTextElements.add(boundTextEl.id);
    safeCopyToEA([boundTextEl]);
  }

  // Also clean grouped text
  const groupEls = getGroupEls(arr);
  for (const e of groupEls) {
    if (e.id !== arr.id && typeof e.text === "string" && parseMarker(e.text.trim()) && !modifiedTextElements.has(e.id)) {
      clog(`  ✂️ Cleaning transfer marker from grouped text: "${e.text}"`);
      e.text = "";
      e.rawText = "";
      e.originalText = "";
      modifiedTextElements.add(e.id);
      safeCopyToEA([e]);
    }
  }
}

/* ---------- transfers with improved direction detection ---------- */
async function ensureTransfer(arr) {
  clog(`\n--- Processing arrow ${arr.id} ---`);
  const classification = classifyElement(arr);

  // Accept both explicit "transfer" markers AND existing links to transfer files
  let isExistingTransfer = false;
  let existingTransferPath = null;

  if (classification?.kind === "existing" && classification.existingPath) {
    // Check if the existing link points to a transfer file (in Transfers folder)
    const transferFolder = getTargetFolder("transfer");
    if (classification.existingPath.startsWith(transferFolder)) {
      clog(`  ✓ Existing link points to transfer file: ${classification.existingPath}`);
      isExistingTransfer = true;
      existingTransferPath = classification.existingPath;
    } else {
      clog(`  ✗ Existing link is not a transfer (not in ${transferFolder})`);
      return;
    }
  } else if (!classification || classification.kind !== "transfer") {
    clog(`  ✗ Not classified as transfer`);
    return;
  }

  clog(`  ✓ Classified as transfer, customName: ${classification.customName}, isExisting: ${isExistingTransfer}`);

  const direction = determineArrowDirection(arr);
  if (!direction.fromElementId || !direction.toElementId) {
    // v1.6.1: Improved error message - specify which binding is missing
    const missingStart = !direction.fromElementId;
    const missingEnd = !direction.toElementId;
    let errorDetail = "";
    if (missingStart && missingEnd) {
      errorDetail = "Arrow not connected to any shapes - snap both ends to shapes";
    } else if (missingStart) {
      errorDetail = "Arrow START not connected - snap start point to a shape";
    } else {
      errorDetail = "Arrow END not connected - snap end point to a shape";
    }
    clog(`  ✗ Could not determine arrow direction: ${errorDetail}`);
    clog(`  ℹ️ Tip: Click arrow endpoint and drag to shape until it snaps (shows blue highlight)`);
    note(`↯ ${errorDetail}`);
    return;
  }

  const startEl = byId[direction.fromElementId];
  const endEl = byId[direction.toElementId];
  if (!startEl || !endEl) {
    clog(`  ✗ Direction objects not found in element map`);
    note("↯ direction objects not found");
    return;
  }

  clog(`  ✓ Found objects: ${startEl.type} → ${endEl.type} (via ${direction.directionSource})`);

  const bidirectional = direction.isBidirectional;
  if (bidirectional) {
    clog(`  🔄 Detected bidirectional arrow (mode: ${BIDIRECTIONAL_MODE})`);
  }

  // Ensure both objects are linked - handle existing links properly
  const startClass = classifyElement(startEl) || { kind: "asset", customName: null };
  const endClass = classifyElement(endEl) || { kind: "asset", customName: null };

  const startPath = await ensureNodeLinked(
    startEl,
    startClass.kind === "existing" ? "existing" : startClass.kind,
    startClass.customName,
    startClass.existingPath,
    startClass.boundTextElementId
  );
  const endPath = await ensureNodeLinked(
    endEl,
    endClass.kind === "existing" ? "existing" : endClass.kind,
    endClass.customName,
    endClass.existingPath,
    endClass.boundTextElementId
  );

  if (!startPath || !endPath) {
    clog(`  ✗ Could not ensure object notes`);
    note("↯ could not ensure object notes");
    return;
  }

  clog(`  ✓ Objects: ${startPath} → ${endPath}`);

  // Handle existing transfer link (either from isExistingTransfer or from arrow.link)
  let reuseExistingPath = existingTransferPath;

  // Also check for existing arrow link (exclude marker links) - legacy path
  if (!reuseExistingPath && typeof arr.link === "string" && arr.link.includes("[[") && arr.link.includes("]]") && !parseMarker(arr.link)) {
    clog(`  Found existing arrow link: ${arr.link}`);
    const resolvedPath = resolveLink(arr.link, view.file.path);
    if (resolvedPath && exists(resolvedPath)) {
      reuseExistingPath = resolvedPath;
    }
  }

  // v1.7.2: Transfer reuse decision based on TRANSFER_REUSE_MODE and explicit markers
  const sourceWiki = shortWiki(view.file.path, view.file.path);

  // v1.7.2: Handle explicit "transfer=reuse" marker - always reuse
  if (classification.reuseMode === "auto") {
    clog(`  v1.7.2: Explicit reuse marker detected - searching for existing transfer`);
    // Find any existing transfer between these objects (regardless of diagram ownership)
    const transferFolder = getTargetFolder("transfer");
    const matchedPath = await findAnyTransferBetweenObjects(transferFolder, bn(startPath), bn(endPath));
    if (matchedPath) {
      reuseExistingPath = matchedPath;
      clog(`  v1.7.2: Found matching transfer to reuse: ${matchedPath}`);
    } else {
      clog(`  v1.7.2: No existing transfer found, will create new`);
    }
  } else if (classification.reuseMode === "specific") {
    // Specific transfer path requested
    const targetPath = resolveLink(classification.reuseTarget, view.file.path);
    if (targetPath && exists(targetPath)) {
      reuseExistingPath = targetPath;
      clog(`  v1.7.2: Using specific transfer: ${targetPath}`);
    } else {
      clog(`  v1.7.2: Specific transfer not found: ${classification.reuseTarget}`);
      note(`⚠️ Transfer not found: ${classification.reuseTarget}`);
    }
  }

  // v1.7.2: If we found an existing transfer from arrow.link, check diagram ownership
  if (reuseExistingPath && TRANSFER_REUSE_MODE === "per_diagram" && !classification.reuseMode) {
    const transferFile = app.vault.getAbstractFileByPath(reuseExistingPath);
    const transferCache = transferFile ? app.metadataCache.getFileCache(transferFile) : null;
    const existingFM = transferCache?.frontmatter || {};

    if (!isTransferOwnedByCurrentDiagram(existingFM, sourceWiki)) {
      clog(`  v1.7.2: Transfer from different diagram, TRANSFER_REUSE_MODE=${TRANSFER_REUSE_MODE}`);
      clog(`  v1.7.2: Clearing arrow link, will create new transfer for this diagram`);
      // Clear the existing link - we'll create a new transfer
      arr.link = "";
      reuseExistingPath = null;
    }
  }

  // v1.7.2: Check if THIS diagram already has a transfer for these objects (same-diagram reuse)
  // v1.7.6: Now uses UUID for rename-proof matching
  if (!reuseExistingPath) {
    const transferFolder = getTargetFolder("transfer");
    const ownedTransfer = await findExistingTransferForDiagram(transferFolder, bn(startPath), bn(endPath), sourceWiki, diagramUUID);
    if (ownedTransfer) {
      reuseExistingPath = ownedTransfer;
      clog(`  v1.7.6: Found existing transfer owned by this diagram: ${ownedTransfer}`);
    }
  }

  // If we have an existing transfer to reuse, update relationships and return
  if (reuseExistingPath) {
    clog(`  ✓ Reusing existing transfer: ${reuseExistingPath}`);
    const startWiki = shortWiki(startPath, view.file.path);
    const endWiki = shortWiki(endPath, view.file.path);
    // Note: sourceWiki already declared above at line 1452 for v1.7.2 reuse logic

    // Read current values to detect endpoint changes
    const transferFile = app.vault.getAbstractFileByPath(reuseExistingPath);
    const transferCache = transferFile ? app.metadataCache.getFileCache(transferFile) : null;
    const oldFM = transferCache?.frontmatter || {};

    const oldObjectA = oldFM.object_a || null;
    const oldObjectB = oldFM.object_b || null;
    const oldWikiLink = shortWiki(reuseExistingPath, view.file.path);

    clog(`  📖 Current endpoints: object_a=${oldObjectA}, object_b=${oldObjectB}`);
    clog(`  📝 New endpoints: object_a=${startWiki}, object_b=${endWiki}`);

    // SPECIAL CASE: Bidirectional + dual_transfers mode
    // Need to convert existing unidirectional transfer to forward/reverse pair
    if (bidirectional && BIDIRECTIONAL_MODE === "dual_transfers") {
      const existingFileName = bn(reuseExistingPath);
      // v1.7.4: Use includes() because diagram name may come after _forward/_reverse suffix
      const hasSuffix = existingFileName.includes("_forward") || existingFileName.includes("_reverse");

      if (!hasSuffix) {
        clog(`  🔄 Converting unidirectional to bidirectional (dual_transfers mode)`);

        // Step 1: Rename existing transfer to have _forward suffix
        const folder = reuseExistingPath.substring(0, reuseExistingPath.lastIndexOf("/"));
        const forwardPath = `${folder}/${existingFileName}${OBJECT_SEPARATOR}forward.md`;

        clog(`  📝 Renaming: ${reuseExistingPath} → ${forwardPath}`);
        await app.fileManager.renameFile(transferFile, forwardPath);

        // Step 2: Create reverse transfer
        const reversePath = await createTransferNote(endPath, startPath, classification, false, `${OBJECT_SEPARATOR}reverse`);
        clog(`  📝 Created reverse transfer: ${reversePath}`);

        const forwardWiki = shortWiki(forwardPath, view.file.path);
        const reverseWiki = shortWiki(reversePath, view.file.path);

        // Step 3: Update forward transfer properties
        await setFM(forwardPath, fm => {
          fm.object_a = startWiki;
          fm.object_b = endWiki;
          fm.paired_transfer = reverseWiki;
          fm.direction_source = direction.directionSource;

          if (INCLUDE_FROM_TO_PROPERTIES) {
            fm.from = startWiki;
            fm.to = endWiki;
          }
        });

        // Step 4: Update reverse transfer properties
        await setFM(reversePath, fm => {
          fm.object_a = endWiki;
          fm.object_b = startWiki;
          fm.paired_transfer = forwardWiki;
          fm.direction_source = direction.directionSource;

          if (INCLUDE_FROM_TO_PROPERTIES) {
            fm.from = endWiki;
            fm.to = startWiki;
          }
        });

        // v1.7.1: Add diagram to _source_diagrams array for both transfers
        await addSourceDiagram(forwardPath, sourceWiki, diagramUUID);
        await addSourceDiagram(reversePath, sourceWiki, diagramUUID);

        // Step 5: Update arrow link to new forward path
        arr.link = forwardWiki;

        // Step 6: Clean up old dfd_in/dfd_out references (old wikilink no longer valid)
        await removeFromArr(startPath, "dfd_out", oldWikiLink);
        await removeFromArr(startPath, "dfd_in", oldWikiLink);
        await removeFromArr(endPath, "dfd_out", oldWikiLink);
        await removeFromArr(endPath, "dfd_in", oldWikiLink);

        // Step 7: Add new references for both directions
        await pushArr(startPath, "dfd_out", forwardWiki);
        await pushArr(startPath, "dfd_in", reverseWiki);
        await pushArr(endPath, "dfd_in", forwardWiki);
        await pushArr(endPath, "dfd_out", reverseWiki);

        note(`✓ Converted to bidirectional: ${forwardWiki} ⟷ ${reverseWiki}`);
        clog(`  ✓ Converted to bidirectional: ${forwardWiki} ⟷ ${reverseWiki}`);
        cleanTransferMarkerText(arr);
        safeCopyToEA([arr]);
        return;
      } else {
        clog(`  ℹ️ Transfer already has _forward/_reverse suffix, checking for pair...`);
        // Already has suffix - make sure pair exists and arrays are correct
        // v1.7.0: Also re-link orphaned reverse if it exists

        const existingFileName = bn(reuseExistingPath);
        const folder = reuseExistingPath.substring(0, reuseExistingPath.lastIndexOf("/"));

        // Determine if we're the forward or reverse, and find the pair
        // v1.7.4: Use includes() and handle diagram name suffix
        let forwardPath, reversePath, forwardWiki, reverseWiki;
        const isForward = existingFileName.includes("_forward");
        const isReverse = existingFileName.includes("_reverse");

        if (isForward) {
          forwardPath = reuseExistingPath;
          // v1.7.4: First try to get paired path from frontmatter (more reliable)
          if (oldFM.paired_transfer) {
            reversePath = resolveLink(oldFM.paired_transfer, view.file.path);
            clog(`  🔍 Found reverse in frontmatter: ${reversePath}`);
          } else {
            // Derive the expected reverse path from the forward name
            // v1.7.4: Extract suffix that comes after "_forward" (e.g., diagram name)
            const forwardIdx = existingFileName.indexOf("_forward");
            const baseName = existingFileName.slice(0, forwardIdx);
            const afterSuffix = existingFileName.slice(forwardIdx + "_forward".length); // e.g., "_test-10-14"
            // Try to find existing reverse (may be orphaned)
            const expectedReverseName = baseName.replace(
              `${bn(startPath)}${OBJECT_SEPARATOR}${TRANSFER_DIRECTION_WORD}${OBJECT_SEPARATOR}${bn(endPath)}`,
              `${bn(endPath)}${OBJECT_SEPARATOR}${TRANSFER_DIRECTION_WORD}${OBJECT_SEPARATOR}${bn(startPath)}`
            ) + "_reverse" + afterSuffix;
            reversePath = `${folder}/${expectedReverseName}.md`;
            clog(`  🔍 Derived reverse path: ${reversePath}`);
          }
        } else if (isReverse) {
          reversePath = reuseExistingPath;
          // v1.7.4: First try to get paired path from frontmatter
          if (oldFM.paired_transfer) {
            forwardPath = resolveLink(oldFM.paired_transfer, view.file.path);
            clog(`  🔍 Found forward in frontmatter: ${forwardPath}`);
          } else {
            // Derive the expected forward path
            const reverseIdx = existingFileName.indexOf("_reverse");
            const baseName = existingFileName.slice(0, reverseIdx);
            const afterSuffix = existingFileName.slice(reverseIdx + "_reverse".length);
            const expectedForwardName = baseName.replace(
              `${bn(startPath)}${OBJECT_SEPARATOR}${TRANSFER_DIRECTION_WORD}${OBJECT_SEPARATOR}${bn(endPath)}`,
              `${bn(endPath)}${OBJECT_SEPARATOR}${TRANSFER_DIRECTION_WORD}${OBJECT_SEPARATOR}${bn(startPath)}`
            ) + "_forward" + afterSuffix;
            forwardPath = `${folder}/${expectedForwardName}.md`;
            clog(`  🔍 Derived forward path: ${forwardPath}`);
          }
        }

        // Check if the reverse exists and might be orphaned
        if (forwardPath && reversePath) {
          forwardWiki = shortWiki(forwardPath, view.file.path);
          reverseWiki = shortWiki(reversePath, view.file.path);

          if (exists(reversePath)) {
            const reverseFile = app.vault.getAbstractFileByPath(reversePath);
            const reverseCache = reverseFile ? app.metadataCache.getFileCache(reverseFile) : null;
            const reverseFM = reverseCache?.frontmatter || {};

            // v1.7.0: If the reverse is orphaned, re-link it!
            if (reverseFM._status === "orphaned") {
              clog(`  ✓ Found orphaned reverse transfer, re-linking: ${reversePath}`);

              // Clear orphan flags and restore paired relationship
              await setFM(reversePath, fm => {
                fm._status = "active";
                delete fm._orphan_detected;
                delete fm._orphan_reason;
                delete fm._last_linked_diagram;
                fm.paired_transfer = forwardWiki;
                fm.direction_source = direction.directionSource;
              });

              // Update forward's paired_transfer
              await setFM(forwardPath, fm => {
                fm.paired_transfer = reverseWiki;
                fm.direction_source = direction.directionSource;
              });

              // Update dfd_in/dfd_out arrays for both directions
              await pushArr(startPath, "dfd_out", forwardWiki);
              await pushArr(startPath, "dfd_in", reverseWiki);
              await pushArr(endPath, "dfd_in", forwardWiki);
              await pushArr(endPath, "dfd_out", reverseWiki);

              // v1.7.3: Set arrow link to forward transfer
              arr.link = forwardWiki;
              clog(`  ✓ Set arrow link: ${forwardWiki}`);

              note(`✓ Re-linked orphaned transfer: ${reverseWiki}`);
              clog(`  ✓ Re-linked orphaned reverse: ${reverseWiki}`);
              cleanTransferMarkerText(arr);
              safeCopyToEA([arr]);
              return;
            } else {
              // Reverse exists but isn't orphaned - just update arrays
              clog(`  ✓ Pair exists and is active, updating arrays`);
              await pushArr(startPath, "dfd_out", forwardWiki);
              await pushArr(startPath, "dfd_in", reverseWiki);
              await pushArr(endPath, "dfd_in", forwardWiki);
              await pushArr(endPath, "dfd_out", reverseWiki);

              // v1.7.3: Set arrow link to forward transfer
              arr.link = forwardWiki;
              clog(`  ✓ Set arrow link: ${forwardWiki}`);
              cleanTransferMarkerText(arr);
              safeCopyToEA([arr]);
              return;
            }
          } else {
            // Reverse doesn't exist - create it
            clog(`  ℹ️ Paired reverse not found, creating new one`);
            const newReversePath = await createTransferNote(endPath, startPath, classification, false, `${OBJECT_SEPARATOR}reverse`);
            const newReverseWiki = shortWiki(newReversePath, view.file.path);

            await setFM(forwardPath, fm => {
              fm.paired_transfer = newReverseWiki;
              fm.direction_source = direction.directionSource;
            });

            await setFM(newReversePath, fm => {
              fm.object_a = endWiki;
              fm.object_b = startWiki;
              fm.paired_transfer = forwardWiki;
              fm.direction_source = direction.directionSource;
              if (INCLUDE_FROM_TO_PROPERTIES) {
                fm.from = endWiki;
                fm.to = startWiki;
              }
            });

            // v1.7.1: Add diagram to _source_diagrams array for both transfers
            await addSourceDiagram(forwardPath, sourceWiki, diagramUUID);
            await addSourceDiagram(newReversePath, sourceWiki, diagramUUID);

            await pushArr(startPath, "dfd_out", forwardWiki);
            await pushArr(startPath, "dfd_in", newReverseWiki);
            await pushArr(endPath, "dfd_in", forwardWiki);
            await pushArr(endPath, "dfd_out", newReverseWiki);

            // v1.7.3: Set arrow link to forward transfer
            arr.link = forwardWiki;
            clog(`  ✓ Set arrow link: ${forwardWiki}`);

            note(`✓ Created paired reverse: ${newReverseWiki}`);
            clog(`  ✓ Created new paired reverse: ${newReversePath}`);
            cleanTransferMarkerText(arr);
            safeCopyToEA([arr]);
            return;
          }
        }
      }
    }

    // v1.7.0: ORPHAN DETECTION - Check for bidirectional → unidirectional conversion
    // v1.7.1: Now multi-diagram aware - only orphan if no other diagrams reference
    // v1.7.4: Updated to handle diagram name suffix (use includes instead of endsWith)
    const wasBidirectional = oldFM.direction_source === "bidirectional";
    if (wasBidirectional && !bidirectional && BIDIRECTIONAL_MODE === "dual_transfers") {
      clog(`  ⚠️ Detected bidirectional → unidirectional conversion!`);

      // Check if this is a _forward transfer with a paired _reverse
      // v1.7.4: Use includes() because diagram name may come after _forward/_reverse suffix
      const currentFileName = bn(reuseExistingPath);
      const pairedTransferWiki = oldFM.paired_transfer;
      const isForwardTransfer = currentFileName.includes("_forward");
      const isReverseTransfer = currentFileName.includes("_reverse");
      clog(`  Checking: isForward=${isForwardTransfer}, isReverse=${isReverseTransfer}, paired=${pairedTransferWiki}`);

      if (isForwardTransfer && pairedTransferWiki) {
        // The _reverse transfer might be orphaned
        const pairedPath = resolveLink(pairedTransferWiki, view.file.path);

        if (pairedPath && exists(pairedPath)) {
          // v1.7.1: Check if reverse transfer has multiple source diagrams
          const pairedFile = app.vault.getAbstractFileByPath(pairedPath);
          const pairedCache = pairedFile ? app.metadataCache.getFileCache(pairedFile) : null;
          const pairedFM = pairedCache?.frontmatter || {};
          const sourceDiagramCount = getSourceDiagramCount(pairedFM);

          if (sourceDiagramCount > 1) {
            // Multiple diagrams reference this transfer - don't auto-orphan
            clog(`  ⚠️ Reverse transfer has ${sourceDiagramCount} source diagrams - NOT auto-orphaning`);
            note(`⚠️ Warning: ${pairedTransferWiki} used by ${sourceDiagramCount} diagrams - review manually`);

            // Still clear paired_transfer since the bidirectional relationship is severed
            await setFM(reuseExistingPath, fm => {
              delete fm.paired_transfer;
            });
            await setFM(pairedPath, fm => {
              delete fm.paired_transfer;
            });
          } else {
            // Only one diagram (or none) - safe to orphan
            clog(`  🗑️ Flagging orphaned reverse transfer: ${pairedPath}`);

            // Flag the reverse transfer as orphaned
            await setFM(pairedPath, fm => {
              fm._status = "orphaned";
              fm._orphan_detected = new Date().toISOString();
              fm._orphan_reason = "bidirectional_to_unidirectional";
              fm._last_linked_diagram = sourceWiki;
              // Clear paired_transfer since the relationship is severed
              delete fm.paired_transfer;
            });

            // Also clear paired_transfer from the forward transfer
            await setFM(reuseExistingPath, fm => {
              delete fm.paired_transfer;
            });

            // Remove reverse transfer from dfd_in/dfd_out arrays
            const reverseWiki = shortWiki(pairedPath, view.file.path);
            await removeFromArr(startPath, "dfd_in", reverseWiki);
            await removeFromArr(startPath, "dfd_out", reverseWiki);
            await removeFromArr(endPath, "dfd_in", reverseWiki);
            await removeFromArr(endPath, "dfd_out", reverseWiki);

            note(`⚠️ Orphaned: ${pairedTransferWiki} (bidirectional → unidirectional)`);
          }
        }
      } else if (isReverseTransfer && pairedTransferWiki) {
        // This reverse transfer might be orphaned (arrow now links to _forward)
        // v1.7.1: Check multi-diagram status
        const sourceDiagramCount = getSourceDiagramCount(oldFM);

        if (sourceDiagramCount > 1) {
          clog(`  ⚠️ This reverse has ${sourceDiagramCount} source diagrams - NOT auto-orphaning`);
          note(`⚠️ Warning: ${currentFileName} used by ${sourceDiagramCount} diagrams - review manually`);
          await setFM(reuseExistingPath, fm => {
            delete fm.paired_transfer;
          });
        } else {
          clog(`  🗑️ This reverse transfer is being orphaned`);

          await setFM(reuseExistingPath, fm => {
            fm._status = "orphaned";
            fm._orphan_detected = new Date().toISOString();
            fm._orphan_reason = "bidirectional_to_unidirectional";
            fm._last_linked_diagram = sourceWiki;
            delete fm.paired_transfer;
          });

          note(`⚠️ Orphaned: ${currentFileName} (bidirectional → unidirectional)`);
          // Don't process further - this transfer is orphaned
          return;
        }
      }
    }

    // v1.7.0: Clear orphan flag if a previously orphaned transfer is being re-linked
    if (oldFM._status === "orphaned") {
      clog(`  ✓ Re-linking previously orphaned transfer`);
      await setFM(reuseExistingPath, fm => {
        fm._status = "active";
        delete fm._orphan_detected;
        delete fm._orphan_reason;
        delete fm._last_linked_diagram;
      });
      note(`✓ Re-linked: ${bn(reuseExistingPath)} (was orphaned)`);
    }

    // Standard case: Update existing transfer (non-bidirectional or single_bidirectional mode)
    let wikiLink = shortWiki(reuseExistingPath, view.file.path);

    // Check if endpoints changed
    const endpointsChanged = (oldObjectA && oldObjectA !== startWiki) ||
                             (oldObjectB && oldObjectB !== endWiki);

    if (endpointsChanged) {
      clog(`  🔄 Endpoints changed! Cleaning up old references...`);

      // Resolve old paths and clean up their dfd_in/dfd_out arrays
      if (oldObjectA && oldObjectA !== startWiki) {
        const oldAPath = resolveLink(oldObjectA, view.file.path);
        if (oldAPath) {
          await removeFromArr(oldAPath, "dfd_out", wikiLink);
          await removeFromArr(oldAPath, "dfd_in", wikiLink);
        }
      }

      if (oldObjectB && oldObjectB !== endWiki) {
        const oldBPath = resolveLink(oldObjectB, view.file.path);
        if (oldBPath) {
          await removeFromArr(oldBPath, "dfd_out", wikiLink);
          await removeFromArr(oldBPath, "dfd_in", wikiLink);
        }
      }

      // v1.6.0: Check if transfer filename needs to be updated to match new endpoints
      const currentFileName = bn(reuseExistingPath);
      const expectedBaseName = generateTransferBaseName(startPath, endPath, bidirectional && BIDIRECTIONAL_MODE === "single_bidirectional");

      // v1.7.4: Preserve suffix like _forward, _reverse (with any diagram name after)
      // Also handle numbered suffix
      let suffix = "";
      if (currentFileName.includes("_forward")) {
        const idx = currentFileName.indexOf("_forward");
        suffix = currentFileName.slice(idx); // Gets "_forward" and everything after (e.g., "_forward_test-10-14")
      } else if (currentFileName.includes("_reverse")) {
        const idx = currentFileName.indexOf("_reverse");
        suffix = currentFileName.slice(idx); // Gets "_reverse" and everything after
      } else {
        // Check for numbered suffix (e.g., _2, _3) - may have diagram name after
        const numMatch = currentFileName.match(/_(\d+)(_.*)?$/);
        if (numMatch) {
          suffix = `_${numMatch[1]}${numMatch[2] || ""}`;
        } else if (TRANSFER_INCLUDE_DIAGRAM) {
          // v1.7.4: May just have diagram name suffix
          const diagramSlug = slug(bn(view.file.path));
          if (currentFileName.endsWith(`_${diagramSlug}`)) {
            suffix = `_${diagramSlug}`;
          }
        }
      }

      const expectedFileName = `${expectedBaseName}${suffix}`;
      const fileNameMismatch = currentFileName !== expectedFileName;

      if (fileNameMismatch) {
        clog(`  📝 Transfer filename mismatch detected!`);
        clog(`    Current: ${currentFileName}`);
        clog(`    Expected: ${expectedFileName}`);

        // Rename the transfer file to match new endpoints
        const folder = reuseExistingPath.substring(0, reuseExistingPath.lastIndexOf("/"));
        const newPath = `${folder}/${expectedFileName}.md`;

        clog(`  📝 Renaming: ${reuseExistingPath} → ${newPath}`);
        await app.fileManager.renameFile(transferFile, newPath);

        // Update references
        reuseExistingPath = newPath;
        wikiLink = shortWiki(newPath, view.file.path);

        // Update arrow link to point to renamed file
        arr.link = wikiLink;

        note(`⚠️ Transfer renamed: ${currentFileName} → ${expectedFileName}`);
      }
    }

    // Update the transfer's own properties - ALWAYS overwrite (diagram is source of truth)
    clog(`  📝 Updating existing transfer properties (overwriting)`);
    await setFM(reuseExistingPath, fm => {
      // ALWAYS set endpoints - diagram is source of truth
      fm.object_a = startWiki;
      fm.object_b = endWiki;

      // Update from/to based on direction
      if (INCLUDE_FROM_TO_PROPERTIES) {
        fm.from = startWiki;
        fm.to = endWiki;
      }

      // Update direction_source
      fm.direction_source = direction.directionSource;
    });

    // v1.7.1: Add this diagram to _source_diagrams array (handles migration from legacy source_drawing)
    await addSourceDiagram(reuseExistingPath, sourceWiki, diagramUUID);

    // v1.7.3: CRITICAL - Set arrow link to the transfer (was missing!)
    arr.link = wikiLink;
    clog(`  ✓ Set arrow link: ${wikiLink}`);

    // Update object arrays based on bidirectional mode
    if (bidirectional && BIDIRECTIONAL_MODE === "single_bidirectional") {
      await pushArr(startPath, "dfd_out", wikiLink);
      await pushArr(startPath, "dfd_in", wikiLink);
      await pushArr(endPath, "dfd_out", wikiLink);
      await pushArr(endPath, "dfd_in", wikiLink);
    } else {
      await pushArr(startPath, "dfd_out", wikiLink);
      await pushArr(endPath, "dfd_in", wikiLink);
    }

    note(`✓ Transfer updated: ${wikiLink}`);
    cleanTransferMarkerText(arr);
    safeCopyToEA([arr]);
    return;
  }

  // Create transfer based on bidirectional mode
  if (bidirectional && BIDIRECTIONAL_MODE === "dual_transfers") {
    const path1 = await createTransferNote(startPath, endPath, classification, false, `${OBJECT_SEPARATOR}forward`);
    const path2 = await createTransferNote(endPath, startPath, classification, false, `${OBJECT_SEPARATOR}reverse`);

    const startWiki = shortWiki(startPath, view.file.path);
    const endWiki = shortWiki(endPath, view.file.path);
    const sourceWiki = shortWiki(view.file.path, view.file.path);

    await setFM(path1, fm => {
      fm.object_a = startWiki;
      fm.object_b = endWiki;
      fm.paired_transfer = shortWiki(path2, view.file.path);
      fm.direction_source = direction.directionSource;

      if (INCLUDE_FROM_TO_PROPERTIES) {
        fm.from = startWiki;
        fm.to = endWiki;
      }
    });

    await setFM(path2, fm => {
      fm.object_a = endWiki;
      fm.object_b = startWiki;
      fm.paired_transfer = shortWiki(path1, view.file.path);
      fm.direction_source = direction.directionSource;

      if (INCLUDE_FROM_TO_PROPERTIES) {
        fm.from = endWiki;
        fm.to = startWiki;
      }
    });

    // v1.7.1: Add diagram to _source_diagrams array for both transfers
    await addSourceDiagram(path1, sourceWiki, diagramUUID);
    await addSourceDiagram(path2, sourceWiki, diagramUUID);

    const transferWiki = shortWiki(path1, view.file.path);
    arr.link = transferWiki;

    const transfer1Wiki = shortWiki(path1, view.file.path);
    const transfer2Wiki = shortWiki(path2, view.file.path);

    await pushArr(startPath, "dfd_out", transfer1Wiki);
    await pushArr(startPath, "dfd_in", transfer2Wiki);
    await pushArr(endPath, "dfd_in", transfer1Wiki);
    await pushArr(endPath, "dfd_out", transfer2Wiki);

    note(`✓ bidirectional transfers → ${transfer1Wiki} ⟷ ${transfer2Wiki}`);
    clog(`  ✓ Created dual transfers: ${transfer1Wiki} ⟷ ${transfer2Wiki}`);

  } else if (bidirectional && BIDIRECTIONAL_MODE === "single_bidirectional") {
    const path = await createTransferNote(startPath, endPath, classification, true);

    const startWiki = shortWiki(startPath, view.file.path);
    const endWiki = shortWiki(endPath, view.file.path);
    const sourceWiki = shortWiki(view.file.path, view.file.path);

    await setFM(path, fm => {
      fm.object_a = startWiki;
      fm.object_b = endWiki;
      fm.direction_source = direction.directionSource;

      if (INCLUDE_FROM_TO_PROPERTIES) {
        if (BIDIRECTIONAL_FROM_TO_BOTH) {
          fm.from = [startWiki, endWiki];
          fm.to = [startWiki, endWiki];
        } else {
          fm.from = startWiki;
          fm.to = endWiki;
        }
      }
    });

    // v1.7.1: Add diagram to _source_diagrams array
    await addSourceDiagram(path, sourceWiki, diagramUUID);

    const transferWiki = shortWiki(path, view.file.path);
    arr.link = transferWiki;

    await pushArr(startPath, "dfd_out", transferWiki);
    await pushArr(startPath, "dfd_in", transferWiki);
    await pushArr(endPath, "dfd_out", transferWiki);
    await pushArr(endPath, "dfd_in", transferWiki);

    note(`✓ bidirectional transfer → ${transferWiki}`);
    clog(`  ✓ Created bidirectional transfer: ${transferWiki}`);

  } else {
    // Normal unidirectional transfer
    const path = await createTransferNote(startPath, endPath, classification, false);

    const startWiki = shortWiki(startPath, view.file.path);
    const endWiki = shortWiki(endPath, view.file.path);
    const sourceWiki = shortWiki(view.file.path, view.file.path);

    await setFM(path, fm => {
      fm.object_a = startWiki;
      fm.object_b = endWiki;
      fm.direction_source = direction.directionSource;

      if (INCLUDE_FROM_TO_PROPERTIES) {
        fm.from = startWiki;
        fm.to = endWiki;
      }
    });

    // v1.7.1: Add diagram to _source_diagrams array
    await addSourceDiagram(path, sourceWiki, diagramUUID);

    const transferWiki = shortWiki(path, view.file.path);
    arr.link = transferWiki;

    await pushArr(startPath, "dfd_out", transferWiki);
    await pushArr(endPath, "dfd_in", transferWiki);

    note(`✓ transfer → ${transferWiki}`);
    clog(`  ✓ Created unidirectional transfer: ${transferWiki}`);
  }

  // Add metadata to arrow
  arr.customData = {
    ...(arr.customData || {}),
    dfd: {
      kind: "transfer",
      bidirectional: bidirectional && BIDIRECTIONAL_MODE !== "ignore_bidirectional",
      mode: BIDIRECTIONAL_MODE,
      directionSource: direction.directionSource
    }
  };

  // v1.7.3: Clean "transfer" marker text from arrow (refactored to helper)
  cleanTransferMarkerText(arr);
  safeCopyToEA([arr]);
}

/* ---------- main execution ---------- */
// v1.7.6: Module-level diagram UUID (initialized in main block)
let diagramUUID = null;

(async () => {
  clog("\n🚀 Starting Linkify DFD v1.7.6");
  clog(`📋 Explicit markers required: ${REQUIRE_EXPLICIT_MARKER}`);
  clog(`📋 Smart custom name matching: ${SMART_CUSTOM_NAME_MATCHING}`);
  clog(`📋 Search all subfolders: ${SEARCH_ALL_SUBFOLDERS}`);
  clog(`📋 Direction determination: ${DIRECTION_DETERMINATION}`);
  clog(`📋 Bidirectional mode: ${BIDIRECTIONAL_MODE}`);
  clog(`📋 Transfer reuse mode: ${TRANSFER_REUSE_MODE}`);
  clog(`📋 Transfer naming: include_diagram=${TRANSFER_INCLUDE_DIAGRAM}, position=${TRANSFER_POSITION}, collision=${TRANSFER_COLLISION_MODE}`);
  clog(`📋 Transfer fuzzy match: ${TRANSFER_FUZZY_MATCH}`);
  clog(`📋 Clean marker text: ${CLEAN_MARKER_TEXT}`);

  // v1.7.6: Get or create stable diagram UUID for rename-proof transfer ownership
  diagramUUID = await ensureDiagramUUID(view.file);
  if (diagramUUID) {
    clog(`📋 Diagram UUID: ${diagramUUID}`);
  } else {
    clog(`📋 Diagram UUID: (not available - .excalidraw format)`);
  }

  // Process nodes first
  const nodeElements = els.filter(e => e.type !== "arrow");
  clog(`\n📦 Processing ${nodeElements.length} node elements`);

  for (const el of nodeElements) {
    const classification = classifyElement(el);
    if (classification && (["asset", "entity"].includes(classification.kind) || classification.kind === "existing")) {
      await ensureNodeLinked(el, classification.kind, classification.customName, classification.existingPath, classification.boundTextElementId);
    }
  }

  // Process arrows
  const arrowElements = els.filter(e => e.type === "arrow");
  clog(`\n🏹 Processing ${arrowElements.length} arrow elements`);

  for (const el of arrowElements) {
    await ensureTransfer(el);
  }

  await ea.addElementsToView(false, true, true, true);
  clog("\n✅ Linkify DFD v1.7.6: finished");
  note("Linkify DFD v1.7.6: finished");

  // Flush debug log to file
  await flushDebugLog();
})();
