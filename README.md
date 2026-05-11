# TimelineDuplicator

A Pixera Control module that duplicates a source timeline and replaces clip resources on selected layers — supporting single-timeline and multi-language batch workflows.

**Version:** 1.3  
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
| **Resource Folder (ignored on batch)** | *(By Order / Match by same name only)* Folder containing replacement media |
| **New Timeline Suffix (ignored on batch)** | *(By Order / Match by same name only)* Appended to the new timeline name |
| **Suffix List (no underscore needed)** | *(Suffix - Batch only)* Comma/semicolon/colon/dot separated list, e.g. `DE,EN,FR`. Leading underscores are stripped automatically — enter `DE`, not `_DE`. |
| **CSV File Path** | *(Suffix - Batch only)* Full path to a CSV file containing suffixes |
| **Import CSV** | Button — reads the CSV file path and loads suffixes into memory |
| **Discover Suffixes** | Button — scans the resource library and extracts suffixes from filenames; results logged for copy/paste |
| **Log Messages** | Toggle verbose clip-level logging in the Pixera log |
| **Stop on sanity check fail** | When checked (default), skips any timeline whose replacement resources fail the sanity check |

---

## Match Modes

### By Order
Duplicates the source timeline and replaces clips on the target layer with resources from the selected folder in sequence (clip 1 → resource 1, clip 2 → resource 2, etc.).

**Required properties:** Source Timeline, Source Layer, Resource Folder (ignored on batch), New Timeline Suffix (ignored on batch)

### Match by same name
Duplicates the source timeline and replaces only the clips whose assigned resource filename exactly matches a resource filename in the selected folder.

**Required properties:** Source Timeline, Source Layer, Resource Folder (ignored on batch), New Timeline Suffix (ignored on batch)

### Suffix - Batch
Creates one new timeline per suffix. For each suffix, searches **all resource folders** for media files named `OriginalClipName_suffix` (e.g. source clip `Video.mp4` → replacement `Video_DE.mp4` for suffix `DE`).

New timelines are named `SourceTimeline_suffix` (e.g. `MyShow_DE`, `MyShow_EN`).

**Required properties:** Source Timeline, Source Layer, Suffix List (no underscore needed) or CSV import

> **Resource Folder (ignored on batch)** and **New Timeline Suffix (ignored on batch)** are not used in Suffix - Batch mode.

#### Suffix naming rules — important

| Suffix List entry | What the module looks for | Result |
|---|---|---|
| `_EN` ✗ | `Video_EN.mp4` | Use underscore in your media naming scheme!  |

**Versioned files:** when a file carries both a language suffix and a version suffix, the **language must come before the version**:

```
myfile_en_v2.mp4   ← correct — suffix "en" matches myfile_en_v2
myfile_v2_en.mp4   ← incorrect — suffix "en" would look for myfile_en_v2_en
```

The module extracts everything after the **last** underscore as the suffix, so version tags appended after the language tag will break the match. Always order: `name_language_version.ext`.

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
> - Use **Preview** first to verify all matches without creating any timelines.
> - Test with a single suffix first to estimate per-suffix duration.
> - Reduce the resource library to only the files needed for the operation (move others to a separate folder temporarily).
> - Use a specific **Source Layer** instead of `all Layers` when only one layer carries language-versioned content.
> - Run during a maintenance window when the Pixera system is not live.

---

## Sanity Check

Before duplicating any timeline, the module validates each proposed replacement resource against the original clip's resource:

| Check | Description |
|---|---|
| **Format** | File extensions must match (e.g. `.mp4` → `.mp4`). A mismatch is logged. |
| **Duration** | Resource durations must match within 0.001 s. A mismatch is logged with both durations. |

All issues are logged regardless of the **Stop on sanity check fail** setting.

**If "Stop on sanity check fail" is checked (default):**
- Single mode: the entire timeline duplicate is skipped if any issue is found.
- Suffix - Batch: that suffix's timeline is skipped; the batch continues with the next suffix.

**If unchecked:** issues are logged but execution continues.

Use **Preview** to run the sanity check in read-only mode before committing to any changes.

---

## Preview (Dry-Run)

Trigger the **Preview** slot to simulate the full operation without creating or modifying any timelines.

Preview logs:
- Which timeline(s) would be created
- For each layer: which clips would be replaced and with which resource
- Any sanity check issues found for each clip pair
- A summary count of clips and timelines

No data is written to Pixera during Preview.

---

## Pre-flight Validation

Both **Preview** and **Execute** run a pre-flight check before doing any work:

- Source Timeline must be set and exist
- Source Layer must be set
- Resource Folder and New Timeline Suffix must be set (non-batch modes)
- At least one suffix must be defined via Suffix List or CSV import (Suffix - Batch)

If any check fails, the operation aborts and all errors are listed in the log.

---

## Rollback on Partial Failure

In Suffix - Batch mode, if a duplicate timeline is created but subsequent layer or clip processing fails (e.g. a layer is not found on the duplicate), the module automatically deletes that partial timeline and continues with the next suffix.

> **Limitation:** If a Pixera API timeout occurs mid-batch, the Lua runtime is interrupted and no cleanup code can run. Any partially created timelines must be deleted manually. To reduce this risk, use Preview first and consider testing with a single suffix before running the full batch.

---

## Suffix Auto-Discovery

Click **Discover Suffixes** to automatically extract suffixes from the resource library:

- Scans all resource filenames for the `name_SUFFIX.ext` pattern
- Deduplicates and sorts the result
- Logs the output in a format ready to paste into **Suffix List**

Example log output:
```
[module]: discovered 3 suffix(es) - copy/paste into Suffix List: DE,EN,FR
```

Copy `DE,EN,FR` directly into the **Suffix List (no underscore needed)** property.

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

## Trigger Slots

| Slot | Description |
|---|---|
| **Preview** | Simulates the operation and logs matches; no changes made |
| **Execute** | Runs the operation in whichever mode is currently selected; fires an output signal on completion |

The **Execute** output pin can be connected to downstream Control nodes to trigger follow-up actions (e.g. activating the first created timeline).

---

## Logging

All results are written to the Pixera log:

- `[module]: === PREVIEW START ===` — preview mode begin
- `[module]:   layer [LayerName]: 4 clip(s) would be replaced` — per-layer preview count
- `[module]:     Video.mp4 -> Video_DE.mp4` — clip-level match detail
- `[module]:   [SANITY] duration mismatch: Video.mp4 (10.000s) -> VideoAlt.mp4 (12.500s)` — sanity issue
- `[module]: preview summary: 4 clip(s) would be replaced in MyShow_copy`
- `[module]: === PREVIEW END ===`
- `[module]: execute aborted - pre-flight check failed:` — missing required property
- `[module]: execute aborted - sanity check failed (uncheck 'Stop on sanity check fail' to proceed anyway)`
- `[module]: done - replaced 4 clips in MyShow_copy` — single mode success
- `[module]: [2/3] MyShow_EN - 4 clips replaced` — batch progress
- `[module]: [2/3] skipping suffix 'EN' - sanity check failed` — batch skip
- `[module]: [2/3] removed partial timeline for suffix 'EN'` — rollback
- `[module]: batch complete` — batch finished
- `[module]: imported 3 suffixes from CSV: DE, EN, FR` — CSV import success
- `[module]: discovered 3 suffix(es) - copy/paste into Suffix List: DE,EN,FR` — discovery result

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

### v1.3
- **Suffix List** renamed to **Suffix List (no underscore needed)** — the property name now makes the convention explicit
- Leading underscores entered in the Suffix List are automatically stripped before processing, preventing the double-underscore (`__`) mismatch that caused no files to be found
- README expanded with suffix naming rules and language-before-version ordering guidance

### v1.2
- Renamed **Resource Folder** to **Resource Folder (ignored on batch)** and **New Timeline Suffix** to **New Timeline Suffix (ignored on batch)** for clarity
- Added **content sanity check**: validates format (extension) and duration of each replacement resource before duplicating; controlled by new **Stop on sanity check fail** property (default: on)
- Added **Preview** trigger slot — full dry-run that logs all matches and sanity issues without touching any timeline
- Added pre-flight validation in both Preview and Execute: reports all missing/invalid properties before attempting any work
- Added best-effort **rollback**: partial timelines created during a failed batch iteration are automatically deleted
- Added **Discover Suffixes** button — scans the resource library and logs found suffixes ready for copy/paste into Suffix List
- **Execute** now fires an output signal on completion for downstream node chaining
- Refactored core logic into composable `findFolderPairs`, `findSuffixPairs`, `validatePairs`, and `applyPairs` functions shared by Preview and Execute

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
