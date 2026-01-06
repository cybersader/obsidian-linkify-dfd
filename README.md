# Linkify DFD - Excalidraw Automate Script

An [Excalidraw Automate](https://zsviczian.github.io/obsidian-excalidraw-plugin/) script that transforms visual Data Flow Diagrams into a queryable markdown-based graph database in Obsidian.

## What It Does

- **Converts diagram shapes** into linked markdown notes (Assets, Entities)
- **Converts arrows** into Transfer notes that track data flow relationships
- **Maintains bidirectional links** via `dfd_in` and `dfd_out` arrays
- **Supports bidirectional arrows** with paired transfer files
- **Auto-detects and fixes** stale transfers when endpoints change

## Installation

1. Install [Obsidian](https://obsidian.md/) and [Excalidraw plugin](https://github.com/zsviczian/obsidian-excalidraw-plugin)
2. Copy `Linkify DFD v1.6.md` to your vault's `Excalidraw/Scripts/` folder
3. Open an Excalidraw diagram
4. Run via Command Palette: `Excalidraw: Run Excalidraw Automate script`

## Quick Start

1. Draw rectangles for your systems/assets
2. Add text markers: `asset=Customer Database` or `entity=User`
3. Connect with arrows
4. Add `transfer` marker to arrows (Ctrl+K on arrow, type `transfer`)
5. Run the script

## Generated Structure

```
DFD Objects Database/
├── Assets/
│   ├── customer-database.md
│   └── analytics-system.md
├── Entities/
│   └── user.md
└── Transfers/
    └── transfer_customer-database_to_analytics-system.md
```

## Configuration

Edit settings at the top of the script:

```javascript
const DEBUG = true;                              // Console logging
const REQUIRE_EXPLICIT_MARKER = true;            // Only process marked elements
const DB_FOLDER_NAME = "DFD Objects Database";   // Output folder
const BIDIRECTIONAL_MODE = "dual_transfers";     // or "single_bidirectional"
const TRANSFER_NUMBERING_MODE = "random";        // "prefix" | "suffix" | "random"
```

## Markers

| Marker | Creates |
|--------|---------|
| `asset=Name` | Asset note in Assets/ |
| `entity=Name` | Entity note in Entities/ |
| `transfer` | Transfer note linking connected shapes |

## Bidirectional Modes

- **`single_bidirectional`**: One transfer file with `direction: bidirectional`
- **`dual_transfers`**: Two files (`_forward` and `_reverse`) with `paired_transfer` property

## Frontmatter Schema

**Assets/Entities:**
```yaml
schema: dfd-asset-v1
type: asset
name: customer-database
dfd_out: ["[[transfer_customer-database_to_analytics]]"]
dfd_in: ["[[transfer_user_to_customer-database]]"]
source_drawings: ["[[My Diagram]]"]
```

**Transfers:**
```yaml
schema: dfd-transfer-v1
type: transfer
object_a: "[[customer-database]]"
object_b: "[[analytics-system]]"
from: "[[customer-database]]"
to: "[[analytics-system]]"
source_drawing: "[[My Diagram]]"
```

## Querying with Dataview

```dataview
TABLE object_a, object_b, source_drawing
FROM "DFD Objects Database/Transfers"
WHERE type = "transfer"
```

## Version History

See changelog in script header for full history.

- **v1.6.2** - Multiline marker text normalization fix
- **v1.6.1** - Bound text duplication fix, improved error messages
- **v1.6.0** - Stale transfer detection & auto-rename
- **v1.5.6** - Bidirectional conversion fixes
- **v1.5.0** - Transfer numbering modes, bound text support

## License

MIT

## Contributing

Issues and PRs welcome at [github.com/cybersader/obsidian-linkify-dfd](https://github.com/cybersader/obsidian-linkify-dfd)
