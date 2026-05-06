# TimelineDuplicator

A Pixera Control module that duplicates a source timeline and replaces clip resources on selected layers — supporting single-timeline and multi-language batch workflows.

**Version:** 1.1  
**Author:** AV Stumpfl GmbH  
**Minimum Pixera Version:** 2.0.172  
**Pixera API Revision:** 481

---

## What it does

TimelineDuplicator copies a source timeline and swaps out media on one or all layers. Three match modes cover different workflows:

| Mode | Use case |
|---|---|
| **By Order** | Replace clips sequentially from a resource folder |
| **Match by same name** | Replace only clips whose filename matches a resource in the folder |
| **Suffix - Batch** | Create one timeline per language/version, auto-matching files by suffix |

---

## Installation

1. Download `TimelineDuplicator.json` from [this repository](https://github.com/ollikilian/TimelineDuplicator/blob/main/TimelineDuplicator.json)
2. Copy it to your Pixera Control user library folder:
   ```
   C:\ProgramData\AV Stumpfl\Pixera\control_library_user\
   ```
3. Reload the library in Pixera Control — the module appears under user modules.

---

## Properties

| Property | Description |
|---|---|
| **Auto Init** | Initialise the module automatically on Pixera startup |
| **Match Mode** | Select `By Order`, `Match by same name`, or `Suffix - Batch` |
| **Source Timeline** | The timeline to duplicate |
| **Source Layer** | The layer to apply resource replacement on, or `all Layers` |
| **Resource Folder** | *(By Order / Match by same name only)* Folder containing replacement media |
| **New Timeline Suffix** | *(By Order / Match by same name only)* Appended to the new timeline name |
| **Suffix List** | *(Suffix - Batch only)* Comma/semicolon/colon/dot separated list, e.g. `DE,EN,FR` |
| **CSV File Path** | *(Suffix - Batch only)* Full path to a CSV file containing suffixes |
| **Import CSV** | Button — reads the CSV file path and loads suffixes into memory |
| **Log Messages** | Toggle verbose clip-level logging in the Pixera log |

---

## Match Modes

### By Order
Duplicates the source timeline and replaces clips on the target layer with resources from the selected folder in sequence (clip 1 → resource 1, clip 2 → resource 2, etc.).

**Required properties:** Source Timeline, Source Layer, Resource Folder, New Timeline Suffix

### Match by same name
Duplicates the source timeline and replaces only the clips whose assigned resource filename exactly matches a resource filename in the selected folder.

**Required properties:** Source Timeline, Source Layer, Resource Folder, New Timeline Suffix

### Suffix - Batch
Creates one new timeline per suffix. For each suffix, searches **all resource folders** for media files named `OriginalClipName_suffix` (e.g. source clip `Video.mp4` → replacement `Video_DE.mp4` for suffix `DE`).

New timelines are named `SourceTimeline_suffix` (e.g. `MyShow_DE`, `MyShow_EN`).

**Required properties:** Source Timeline, Source Layer, Suffix List or CSV import

> **Resource Folder** and **New Timeline Suffix** are not used in Suffix - Batch mode.

---

## Source Layer: all Layers

When `all Layers` is selected, resource replacement runs on every layer of the duplicated timeline. In Suffix - Batch mode, each suffix timeline gets all its layers processed.

> **⚠ Performance warning — Suffix - Batch + all Layers**
>
> This combination is the most resource-intensive operation in the module. For every suffix, the module iterates over every layer, and for every clip on every layer it scans the entire resource library for a matching file. The workload scales as:
>
> **suffixes × layers × clips per layer × total resources in library**
>
> On large projects this can take a long time and **may cause Pixera to become unresponsive or trigger an API timeout**, which can leave partially-created timelines behind that need to be deleted manually.
>
> **Recommendations before running at scale:**
> - Test with a single suffix first to estimate per-suffix duration.
> - Reduce the resource library to only the files needed for the operation (move others to a separate folder temporarily).
> - Use a specific **Source Layer** instead of `all Layers` when only one layer carries language-versioned content.
> - Run during a maintenance window when the Pixera system is not live.

---

## CSV Import

1. Prepare a CSV file where suffixes are listed — one per row or comma/semicolon/colon/dot separated.  
   Example:
   ```
   DE,EN,FR
   ES
   IT
   ```
2. Paste the full file path into **CSV File Path** (e.g. `C:\Projects\suffixes.csv`).
3. Click **Import CSV**.
4. Check the Pixera log — the module reports how many suffixes were loaded, or an error if the file could not be read.

Imported CSV suffixes take priority over the **Suffix List** property. To revert to the Suffix List, clear the CSV File Path and click Import CSV again (which will fail and clear the stored CSV data).

---

## Trigger Slot

| Slot | Description |
|---|---|
| **Execute** | Runs the operation in whichever mode is currently selected |

Both single-timeline and batch operations are triggered from the single **Execute** slot.

---

## Logging

All results are written to the Pixera log:

- `[module]: duplicated timeline as MyShow_copy` — single mode success
- `[module]: [2/3] MyShow_EN - 4 clips replaced` — batch progress
- `[module]: batch complete` — batch finished
- `[module]: imported 3 suffixes from CSV: DE, EN, FR` — CSV import success
- `[module]: failed to open file: C:\bad\path.csv` — CSV import failure

Set **Log Messages** to `false` to suppress per-clip detail and keep only summary lines.

---

## Example: multi-language batch

**Setup:**
- Source Timeline: `MainShow`
- Source Layer: `all Layers`
- Match Mode: `Suffix - Batch`
- Suffix List: `DE,EN,FR`

**Media in resource library:**
```
Video_DE.mp4   Video_EN.mp4   Video_FR.mp4
Intro_DE.mp4   Intro_EN.mp4   Intro_FR.mp4
```

**Result after Execute:**
```
MainShow_DE  ->  Video_DE.mp4, Intro_DE.mp4
MainShow_EN  ->  Video_EN.mp4, Intro_EN.mp4
MainShow_FR  ->  Video_FR.mp4, Intro_FR.mp4
```

---

## Changelog

### v1.1
- Renamed **Timeline** to **Source Timeline**, **Layer** to **Source Layer**
- Added `all Layers` option to Source Layer dropdown
- Added **Suffix - Batch** match mode with full-library suffix search
- Added **Suffix List** property and **CSV import** workflow
- Merged `BatchExecute` into a single **Execute** trigger slot
- Renamed `By Name` to `Match by same name`
- Match Mode moved above Source Timeline in the property list
- Updated Description property

### v1.0
- Initial release
- Single Execute and BatchExecute triggers
- By Order and By Name match modes
- Resource Folder List for batch folder paths
