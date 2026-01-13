# Data Flow Diagram System for Obsidian

> A graph-based approach to mapping, managing, and analyzing organizational data flows using Obsidian and Excalidraw.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Obsidian](https://img.shields.io/badge/Obsidian-7C3AED?logo=obsidian&logoColor=white)](https://obsidian.md)
[![Excalidraw](https://img.shields.io/badge/Excalidraw-6965DB?logo=excalidraw&logoColor=white)](https://excalidraw.com)

## Overview

The DFD (Data Flow Diagram) System transforms visual diagrams into queryable knowledge graphs. Unlike traditional diagramming tools, this system treats diagram elements as first-class database objects with relationships, metadata, and version control.

### Key Innovation

**Bidirectional Workflow**: Diagrams generate structured data → Data enables analysis and reporting → Analysis informs diagram updates.

```
Visual Diagrams (Excalidraw)
         ↓
   Automation Script
         ↓
Graph Database (Markdown)
         ↓
Analysis & Reporting (Dataview, CSV)
```

---

## Features

### Core Capabilities

- ✅ **Graph-Based Model**: Assets, entities, and transfers as nodes and edges
- ✅ **Smart Matching**: Reuses existing pages instead of creating duplicates
- ✅ **Shape-Based Classification** (v1.7.9): Rectangle→Asset, Ellipse→Entity - no markers needed
- ✅ **Stable UUIDs** (v1.7.6+): Rename-proof diagram identity tracking
- ✅ **Bidirectional Support**: Single transfer or dual transfer modes for ↔ arrows
- ✅ **Transfer Numbering**: Multiple transfers between same objects (prefix/suffix/random modes)
- ✅ **Configuration System**: Define custom object types, markers, and defaults
- ✅ **Direction Detection**: Arrowhead-based or binding-based flow direction
- ✅ **Explicit Markers**: Optional strict mode - only process marked elements
- ✅ **Version Control**: Plain text markdown files work with Git
- ✅ **Export Ready**: CSV export for DLP tools, GRC platforms, audits

### Use Cases

- **Data Governance**: Map data flows for GDPR, SOC 2, HIPAA compliance
- **Vendor Risk Management**: Track which vendors receive which data
- **Incident Response**: Quickly identify downstream impact of breaches
- **Shadow IT Discovery**: Compare diagrams against DLP tool detections
- **Internal Audits**: Document data flows for audit trails
- **DLP Policy Management**: Export allow-lists to tools like Cyberhaven

---

## Quick Start

### Prerequisites

- [Obsidian](https://obsidian.md) desktop app (v1.4.0+)
- [Excalidraw Plugin](https://github.com/zsviczian/obsidian-excalidraw-plugin) for Obsidian
- [Excalidraw Automate](https://github.com/zsviczian/obsidian-excalidraw-plugin) (included with Excalidraw plugin)

#### Excalidraw File Formats (v1.7.7+)

The DFD system works with Excalidraw's markdown-based file formats:

| Format | Frontmatter | UUID Support | Notes |
|--------|-------------|--------------|-------|
| `.md` | ✅ Yes | ✅ Yes | **2025 default** - recommended |
| `.excalidraw.md` | ✅ Yes | ✅ Yes | Original Obsidian format |
| `.excalidraw` | ❌ No | ❌ No | Pure JSON - legacy format |

> **Note**: The script detects Excalidraw files by checking for `excalidraw-plugin: parsed` in frontmatter, not by file extension. This means renamed files (e.g., `diagram.excalidraw.md` → `diagram.md`) continue to work.

#### Excalidraw Plugin Settings (Required)

| Setting | Value | Why |
|---------|-------|-----|
| Excalidraw file format | `.md` or `.excalidraw.md` | Both support UUID (frontmatter required) |
| Compress Excalidraw JSON | **OFF** | Script needs to read element data |
| Auto-export (Compatibility Mode) | Your preference | Optional - syncs `.excalidraw` for external tools |

**Recommended workflow**:
1. Use `.md` format (2025 default) or `.excalidraw.md`
2. Enable Compatibility Mode if you need `.excalidraw` exports for external tools
3. UUID is automatically stored in frontmatter as `dfd_diagram_id`

> **Note**: Pure `.excalidraw` files (JSON format) will still work but cannot store UUIDs. The script falls back to wiki link matching, which breaks if you rename the diagram.

### Installation

1. **Clone or download this repository**:
   ```bash
   git clone https://github.com/yourusername/dfd-system.git
   cd dfd-system
   ```

2. **Open in Obsidian**:
   - Launch Obsidian → Open folder as vault → Select `dfd-system` folder

3. **Install Excalidraw plugin**:
   - Settings → Community plugins → Browse → Search "Excalidraw"
   - Install and enable

4. **Verify installation**:
   - Open `Data Flow Diagrams/Example - Simple System.excalidraw`
   - Should render diagram with example assets

### Creating Your First Diagram

1. **Create a new diagram**:
   - Create file: `Data Flow Diagrams/My First Diagram.excalidraw`
   - Or use Excalidraw.com and import later

2. **Add shapes**:
   - Rectangle (`#2`) = Asset (system, app, database)
   - Circle (`#4`) = Entity (person, vendor, data subject)
   - Arrow (`#5`) = Transfer (data flow)

3. **Add markers**:
   - Double-click shape → Add text: `asset=Customer Database`
   - Double-click another → Add text: `asset=Analytics Platform`
   - Draw arrow between them → Add text: `transfer`

4. **Run the script**:
   - Open diagram in Obsidian
   - Command palette (Ctrl/Cmd+P) → "Excalidraw: Run Excalidraw Automate script"
   - Select: `Linkify DFD v1.7`
   - Wait for completion notice

5. **Verify results**:
   - Check `DFD Objects Database/Assets/` for new files
   - Check `DFD Objects Database/Transfers/` for transfer page
   - Diagram shapes should now have wikilinks

---

## System Architecture

### Data Model

**Three Object Types**:

| Type | Description | Storage | Example |
|------|-------------|---------|---------|
| **Assets** | Systems, apps, storage | `Assets/` | `crm-system.md` |
| **Entities** | People, vendors, subjects | `Entities/` | `hr-team.md` |
| **Transfers** | Data flows between objects | `Transfers/` | `transfer_crm_to_analytics.md` |

**Assets/Entities** (Nodes):
```yaml
---
schema: dfd-asset-v1
type: asset
name: customer-database
created: 2025-11-24T12:00:00Z
dfd_out:
  - "[[transfer_customer-database_to_analytics]]"
dfd_in:
  - "[[transfer_website_to_customer-database]]"
---
```

**Transfers** (Edges):
```yaml
---
schema: dfd-transfer-v1
type: transfer
object_a: "[[customer-database]]"
object_b: "[[analytics-platform]]"
source_drawing: "[[My First Diagram]]"
direction_source: end_arrowhead
from: "[[customer-database]]"
to: "[[analytics-platform]]"
---
```

### Why This Model?

**Assets/Entities**:
- Store lists of relationships (`dfd_out`, `dfd_in`)
- Have independent metadata (owner, classification)
- Exist across multiple diagrams

**Transfers**:
- Store relationship details (`from`, `to`, `direction_source`)
- Have context metadata (frequency, data types)
- Diagram-specific (tracked via `source_drawing`)

**Benefits**:
- Query: "Show all systems sending data to X" → Check X's `dfd_in`
- Query: "Show high-risk transfers" → Filter transfer pages
- Navigate: Asset → Transfer → Destination Asset

---

## Documentation

### Technical Documentation

- [DFD System Architecture](DFD%20System%20Architecture.md) - High-level design, data model, algorithms
- [DFD Developer Reference](DFD%20Developer%20Reference.md) - API reference, extension points, debugging
- [DFD v1.5 Testing Guide](DFD%20v1.5%20Testing%20Guide.md) - Test scenarios and validation

### Scripts

**Main Script**:
- [Linkify DFD v1.7](Linkify%20DFD%20v1.7.md) - Main automation script (Excalidraw Automate)

**Helper Scripts** (v1.7):
- [Mark All Arrows](Excalidraw/Scripts/Mark%20All%20Arrows.md) - Bulk add `transfer` marker to unmarked arrows
- [Clear All DFD Links](Excalidraw/Scripts/Clear%20All%20DFD%20Links.md) - Reset diagram by removing all wikilinks and markers
- [Validate DFD Diagram](Excalidraw/Scripts/Validate%20DFD%20Diagram.md) - Pre-flight checks before running Linkify

**Cleanup Script**:
- [Delete DFD Transfers](Excalidraw/Scripts/Delete%20DFD%20Transfers.md) - Bulk cleanup for testing

---

## Configuration

### Basic Configuration

Edit `Linkify DFD v1.7.md` (lines 21-59):

```javascript
// Debug mode
const DEBUG = true;                              // Enable logging

// Element processing
const REQUIRE_EXPLICIT_MARKER = true;            // Only process marked elements
const AUTO_CLASSIFY_BY_SHAPE = false;            // v1.7.9: Shape-based classification

// Smart matching
const SMART_CUSTOM_NAME_MATCHING = true;         // Reuse existing pages
const SEARCH_ALL_SUBFOLDERS = true;              // Search across folders

// Storage
const DB_PLACEMENT = "db_folder";                // Where to store pages
const DB_FOLDER_NAME = "DFD Objects Database";   // Database folder name

// Transfer numbering
const TRANSFER_NUMBERING_MODE = "prefix";        // "prefix" | "suffix" | "random"

// Direction detection
const DIRECTION_DETERMINATION = "arrowhead_priority";  // Use arrowheads first
const BIDIRECTIONAL_MODE = "single_bidirectional";     // How to handle ↔ arrows
```

#### Shape-Based Classification (v1.7.9)

Enable marker-free diagramming by classifying elements based on shape type:

```javascript
const REQUIRE_EXPLICIT_MARKER = false;  // Required
const AUTO_CLASSIFY_BY_SHAPE = true;    // Enable shape-based classification
```

| Shape | Object Type | Notes |
|-------|-------------|-------|
| Rectangle | Asset | Standard DFD process/data store |
| Ellipse | Entity | Standard DFD external entity |
| Diamond | Asset | Fallback for non-standard shapes |
| Arrow | Transfer | Data flow between objects |

**Workflow with shape-based classification**:
1. Draw shapes (no need to type `asset=` or `entity=`)
2. Add descriptive text inside shapes (becomes the object name)
3. Connect with arrows
4. Run Linkify DFD

Explicit markers (`asset=`, `entity=`, `transfer`) still override shape-based classification when present.

### Custom Object Types

Create configuration files in `DFD Object Configuration/`:

**Example**: `DFD Object Configuration/Asset - Website.md`

```yaml
---
DFD__KIND: asset
DFD__MARKER: website
DFD__SUBFOLDER: Assets

schema: dfd-asset-website-v1
type: asset
category: website
data_classification: Restricted
---

## Website Asset
[Custom body content for all website assets]
```

**Usage in diagram**: Add text to shape: `website=Customer Portal`

### Diagram Identity (v1.7.6+)

Each diagram gets a **stable UUID** that persists through file renames:

```yaml
# In diagram frontmatter (auto-generated)
---
dfd_diagram_id: "a7b2c3d4-e5f6-7890-abcd-ef1234567890"
---
```

**Why This Matters**:
- Transfers track which diagram created them via UUID
- When you rename a diagram, transfers are still found
- Without UUID (pure `.excalidraw` files), renaming breaks transfer lookup

**Transfer Tracking**:
```yaml
# In transfer frontmatter
---
_source_diagrams:
  - "[[My Diagram]]"           # Human-readable (for inspection)
_source_diagram_ids:
  - "a7b2c3d4-e5f6-..."        # Stable UUID (for lookup)
---
```

**Lookup Priority**:
1. UUID match (stable, survives renames)
2. Wiki link match (fallback for older transfers)
3. Legacy `source_drawing` field (backward compatibility)

> **Requirement**: Use `.excalidraw.md` format for UUID support. See [Prerequisites](#excalidraw-plugin-settings-required-for-v176).

---

## Examples

### Example 1: Simple Data Flow

**Diagram**: Member → Website → Database

**Markers**:
- Circle: `entity=Member`
- Rectangle: `asset=Website`
- Rectangle: `asset=Database`
- Arrow 1: `transfer` (Member → Website)
- Arrow 2: `transfer` (Website → Database)

**Generated Files**:
```
DFD Objects Database/
├── Entities/
│   └── member.md
├── Assets/
│   ├── website.md
│   └── database.md
└── Transfers/
    ├── transfer_member_to_website.md
    └── transfer_website_to_database.md
```

**Relationships**:
- `member.md`: `dfd_out: ["[[transfer_member_to_website]]"]`
- `website.md`: `dfd_in: ["[[transfer_member_to_website]]"]`, `dfd_out: ["[[transfer_website_to_database]]"]`
- `database.md`: `dfd_in: ["[[transfer_website_to_database]]"]`

### Example 2: Bidirectional Flow

**Diagram**: System A ↔ System B (double-headed arrow)

**Configuration**: `BIDIRECTIONAL_MODE = "single_bidirectional"`

**Generated**:
- `transfer_a_to_b.md` with:
  - `direction_source: bidirectional`
  - `from: ["[[a]]", "[[b]]"]`
  - `to: ["[[a]]", "[[b]]"]`

**Relationships**:
- `a.md`: `dfd_out: ["[[transfer_a_to_b]]"]`, `dfd_in: ["[[transfer_a_to_b]]"]`
- `b.md`: `dfd_out: ["[[transfer_a_to_b]]"]`, `dfd_in: ["[[transfer_a_to_b]]"]`

### Example 3: Multiple Transfers

**Diagram**: Three arrows from System A → System B

**Configuration**: `TRANSFER_NUMBERING_MODE = "prefix"`

**Generated**:
```
Transfers/
├── transfer_a_to_b.md        (first transfer)
├── transfer_2_a_to_b.md      (second transfer)
└── transfer_3_a_to_b.md      (third transfer)
```

**Use Case**: Different data types or processes using same connection.

---

## Querying Data

### Dataview Queries

**Show all assets sending data to a specific system**:

```dataview
LIST dfd_out
FROM "DFD Objects Database/Assets"
WHERE contains(dfd_out, [[target-system]])
```

**Show all high-risk transfers**:

```dataview
TABLE from, to, data_classification
FROM "DFD Objects Database/Transfers"
WHERE data_classification = "Restricted"
```

**Show transfers created from specific diagram**:

```dataview
TABLE object_a, object_b, direction_source
FROM "DFD Objects Database/Transfers"
WHERE source_drawing = [[My Diagram]]
```

### CSV Export (Obsidian Bases)

1. Install [Bases](https://github.com/SkepticMystic/obsidian-bases) plugin
2. Create view: Query `FROM "DFD Objects Database/Transfers"`
3. Export to CSV for external tools (DLP, GRC, Excel)

---

## Workflow Integration

### Data Governance Workflow

1. **Initial Setup**: ISRM team creates base diagrams
2. **Delegation**: Send diagrams to data stewards
3. **Update**: Stewards add/update flows using criteria
4. **Generation**: Run Linkify DFD script
5. **Enrichment**: Fill in metadata (classifications, owners)
6. **Analysis**: Query for compliance gaps, high-risk flows
7. **Export**: CSV to DLP tools, GRC platforms

### Vendor Risk Management

1. Create entity pages for vendors
2. Diagram transfers: Internal Assets → Vendor Entities
3. Tag transfers: `vendor_due_diligence: true`
4. Query: Show all vendor transfers with sensitive data
5. Export for vendor assessment questionnaires

### Incident Response

1. Identify compromised system (Asset X)
2. Navigate to Asset X page
3. Check `dfd_out`: All downstream systems
4. Follow transfer pages for data classification
5. Generate impact report
6. Notify downstream system owners

---

## Troubleshooting

### Script Creates Duplicates

**Issue**: `asset=Customer DB` creates new page even though `customer-db.md` exists

**Solution**:
```javascript
const SMART_CUSTOM_NAME_MATCHING = true;  // Enable smart matching
```

### Arrows Not Creating Transfers

**Issue**: Arrows don't generate transfer pages

**Causes**:
1. `REQUIRE_EXPLICIT_MARKER = true` but no `transfer` marker
2. Arrow not bound to shapes (ends not attached)
3. Connected shapes not classified as asset/entity

**Solution**: Add `transfer` marker to arrow or disable strict mode

### Wrong Direction Detected

**Issue**: Transfer shows reversed direction

**Solution**: Check arrowhead in Excalidraw or use:
```javascript
const DIRECTION_DETERMINATION = "binding_only";  // Ignore arrowheads
```

---

## Helper Scripts

Three utility scripts streamline the DFD workflow. All are run the same way as the main script:
Command Palette → "Excalidraw: Run Excalidraw Automate script" → Select script

### Mark All Arrows

**Purpose**: Bulk add `transfer` marker to all unmarked arrows in the diagram.

**When to use**:
- Starting a new diagram with many arrows already drawn
- Converting an existing diagram to use the DFD system
- After importing a diagram from Excalidraw.com

**Usage**:
1. Open diagram in Excalidraw
2. Run "Mark All Arrows" script
3. All arrows without markers or links get `transfer` added
4. Run "Linkify DFD" to create transfer files

**Options** (edit script):
```javascript
const SKIP_LABELED = false;   // Skip arrows that have label text
const DRY_RUN = false;        // Preview only, don't modify
```

### Clear All DFD Links

**Purpose**: Remove all wikilinks and markers from diagram elements (reset to pre-Linkify state).

**When to use**:
- Starting over on a diagram
- Debugging link issues
- Before re-running Linkify with different settings

**What it clears**:
- Wikilinks (`[[asset-name]]`)
- Markers (`asset=Name`, `entity=Name`, `transfer`)
- Custom data (DFD metadata)

**Usage**:
1. Open diagram with existing DFD links
2. Run "Clear All DFD Links" script
3. All links removed from diagram elements
4. Markdown files in database are NOT deleted

**Options** (edit script):
```javascript
const CLEAR_MARKERS = true;      // Also clear marker text
const CLEAR_CUSTOM_DATA = true;  // Clear DFD metadata
const DRY_RUN = false;           // Preview only
```

### Validate DFD Diagram

**Purpose**: Pre-flight checks before running Linkify - identifies common issues.

**What it checks**:
- **Disconnected arrows**: Arrows not snapped to any shapes
- **Partially connected arrows**: Only one end snapped
- **Unmarked shapes**: Shapes without `asset=` or `entity=` markers
- **Unmarked arrows**: Arrows without `transfer` marker
- **Duplicate names**: Multiple shapes with same custom name
- **Empty markers**: Markers like `asset=` with no name

**Usage**:
1. After drawing diagram, run "Validate DFD Diagram"
2. Check console for issues (Ctrl+Shift+I)
3. Fix issues (snap arrows, add markers)
4. Run validation again until clean
5. Run "Linkify DFD"

**Recommended workflow**:
```
Draw diagram → Validate → Fix issues → Mark All Arrows → Validate → Linkify DFD
```

---

## Contributing

We welcome contributions! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

### Development Setup

1. Fork the repository
2. Create feature branch: `git checkout -b feature/my-feature`
3. Make changes with tests
4. Update documentation
5. Submit pull request

### Reporting Issues

- Use [GitHub Issues](https://github.com/yourusername/dfd-system/issues)
- Include: Script version, Obsidian version, steps to reproduce
- Attach: Console logs (with `DEBUG = true`), diagram screenshot

### Roadmap

See [CHANGELOG.md → Roadmap](CHANGELOG.md#roadmap) for the detailed roadmap including:
- v1.7.x remaining improvements
- v2.0 plugin interactive mode (fuzzy matching, modals)
- v2.x advanced features (auto-styling, custom shape mapping)
- Documentation backlog

---

## Version History

### v1.7.x Series (2026-01-09 - 2026-01-12)
- **v1.7.9**: Shape-based auto-classification (no markers needed)
- **v1.7.8**: UUID support for assets/entities (`source_diagram_ids`)
- **v1.7.7**: Frontmatter-based Excalidraw detection (works with `.md` and `.excalidraw.md`)
- **v1.7.6**: Stable diagram UUID for rename-proof transfer ownership
- **v1.7.5**: Fuzzy matching for naming scheme transitions
- **v1.7.4**: Orthogonal transfer naming settings
- **v1.7.3**: Diagram name in transfer filenames, stale link detection
- **v1.7.2**: Per-diagram transfer isolation (breaking: new default)
- **v1.7.1**: Multi-diagram tracking for transfers
- **v1.7.0**: Orphan detection for bidirectional→unidirectional conversion

### Earlier Versions
- **v1.6.x**: Stale transfer detection, bound text fixes, multiline normalization
- **v1.5.x**: Transfer numbering modes, endpoint change detection
- **v1.4**: Explicit markers, smart matching
- **v1.0**: Initial release

**Helper Scripts** (v1.7):
- Mark All Arrows, Clear All DFD Links, Validate DFD Diagram

See [CHANGELOG.md](CHANGELOG.md) for detailed version history.

---

## License

MIT License - see [LICENSE](LICENSE) for details.

---

## Acknowledgments

- [Obsidian](https://obsidian.md) - Knowledge management platform
- [Excalidraw](https://excalidraw.com) - Diagramming tool
- [Zsolt Viczián](https://github.com/zsviczian) - Excalidraw Obsidian plugin author
- Data governance community for feedback and use cases

---

## Support

- **Documentation**: See [docs/](docs/) folder
- **Community**: [GitHub Discussions](https://github.com/yourusername/dfd-system/discussions)
- **Issues**: [GitHub Issues](https://github.com/yourusername/dfd-system/issues)

---

## Citation

If you use this system in research or publications:

```bibtex
@software{dfd_system_2025,
  author = {Your Name},
  title = {Data Flow Diagram System for Obsidian},
  year = {2025},
  url = {https://github.com/yourusername/dfd-system},
  version = {1.5}
}
```

---

**Made with ❤️ for the data governance community**
