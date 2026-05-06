# Changelog

All notable changes to TimelineDuplicator are documented here.

---

## [1.2] — 2026-05-06

### Added
- **Content sanity check** — before duplicating any timeline, each proposed replacement resource is validated against the original clip for matching file extension and duration (tolerance: 0.001 s). All mismatches are logged.
- **Stop on sanity check fail** property (bool, default on) — when enabled, any timeline whose replacement resources fail validation is skipped entirely; in batch mode the remaining suffixes continue.
- **Preview** trigger slot — dry-run mode that logs all clip matches, timeline names, and sanity check results without creating or modifying any timelines.
- **Pre-flight validation** — both Preview and Execute now check that all required properties are set and non-empty before doing any work, listing all errors in the log if not.
- **Best-effort rollback** — in Suffix - Batch mode, if a duplicate timeline is created but layer processing subsequently fails, the partial timeline is automatically deleted.
- **Discover Suffixes** button — scans all resource filenames in the library, extracts the suffix segment (everything after the last `_`), deduplicates, sorts, and logs the result in a format ready to paste directly into **Suffix List**.
- **Execute output pin** — Execute's slot style changed from input-only to input+output; the output fires when the operation completes, enabling downstream node chaining.

### Changed
- **Resource Folder** renamed to **Resource Folder (ignored on batch)** to make it clear the property is unused in Suffix - Batch mode.
- **New Timeline Suffix** renamed to **New Timeline Suffix (ignored on batch)** for the same reason.
- Core replacement logic refactored into composable internal functions (`findFolderPairs`, `findSuffixPairs`, `validatePairs`, `applyPairs`) shared by both Preview and Execute.
- `getAllResources` now stores resource handles instead of IDs, enabling duration queries via `Pixera.Resources.Resource.getDuration`.

### Known limitations
- Rollback does not work if a Pixera API timeout interrupts the Lua runtime mid-batch. Partially created timelines must be deleted manually in that case. Use Preview and single-suffix testing before large batch runs.

---

## [1.1] — 2025

### Added
- `all Layers` option in Source Layer dropdown — applies replacement across every layer of the duplicated timeline.
- **Suffix - Batch** match mode — creates one new timeline per suffix, searching all resource folders for files named `originalClip_suffix`.
- **Suffix List** property — comma/semicolon/colon/dot separated suffix input.
- **CSV File Path** property and **Import CSV** button — loads suffixes from a CSV file; CSV suffixes override the Suffix List.

### Changed
- **Timeline** property renamed to **Source Timeline**.
- **Layer** property renamed to **Source Layer**.
- `BatchExecute` trigger removed; all operations run through a single **Execute** slot.
- `By Name` match mode renamed to **Match by same name**.
- Match Mode property moved above Source Timeline in the property list.
- Resource Folder List removed; replaced by Suffix List and all-library search.
- Description property updated.

---

## [1.0] — initial release

### Added
- **Execute** and **BatchExecute** trigger slots.
- **By Order** match mode — sequential clip-to-resource replacement.
- **By Name** match mode — replace clips whose resource name matches a file in the selected folder.
- **Resource Folder List** property for specifying batch folder paths.
