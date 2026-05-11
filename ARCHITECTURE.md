# TimelineDuplicator Module Architecture

**Module Version:** 1.3  
**Type:** Pixera Control Module (embedded Lua in JSON)  
**Purpose:** Duplicates a source timeline and replaces clip resources on selected layer(s) using three distinct matching modes

---

## 1. Public Properties

### Control

| Name | Type | Default | optionsSourceFunc | optionsAction | readOnly | showName |
|------|------|---------|-------------------|---------------|----------|----------|
| Auto Init | bool | true | — | — | false | true |
| Match Mode | string | "By Order" | self.getMatchModeOptions | — | false | true |

### Configuration

| Name | Type | Default | optionsSourceFunc | optionsAction | readOnly | showName |
|------|------|---------|-------------------|---------------|----------|----------|
| Source Timeline | string | "" | self.getTimelineOptions | — | false | true |
| Source Layer | string | "" | self.getLayerOptions | — | false | true |
| Resource Folder (ignored on batch) | string | "" | self.getResourceFolderOptions | — | false | true |
| New Timeline Suffix (ignored on batch) | string | "_copy" | — | — | false | true |

### Batch (Suffix - Batch mode)

| Name | Type | Default | optionsSourceFunc | optionsAction | readOnly | showName |
|------|------|---------|-------------------|---------------|----------|----------|
| Suffix List (no underscore needed) | string | "" | — | — | false | true |
| CSV File Path | string | "" | — | — | false | true |
| Import CSV | button | "Import CSV" | — | self.importCsv | false | false |
| Discover Suffixes | button | "Discover Suffixes" | — | self.discoverSuffixes | false | false |

### Display & Execution

| Name | Type | Default | optionsSourceFunc | optionsAction | readOnly | showName |
|------|------|---------|-------------------|---------------|----------|----------|
| Log Messages | bool | true | — | — | false | true |
| Stop on sanity check fail | bool | true | — | — | false | true |
| Suffix Discovery Hint | string | "Results are logged - copy/paste into Suffix List (no underscore needed)" | — | — | true | false |
| Description | string | (module description text) | — | — | true | true |
| Author | string | "AV Stumpfl GmbH" | — | — | true | false |
| Logo | image | (base64 PNG) | — | — | true | false |
| Url | url | "www.pixera.one" | — | — | true | true |
| Version | string | "1.3" | — | — | true | true |

---

## 2. Internal Functions

### Options / Helper Functions (all hidden slots)

#### `getTimelineOptions()` → array
Returns all timeline names from `Pixera.Timelines.getTimelineNames()`. Empty array on nil.

#### `getLayerOptions()` → array
Reads "Source Timeline" property, fetches timeline object, returns `{"all Layers"}` when missing or empty, otherwise prepends `"all Layers"` to the timeline's layer name list via `Pixera.Timelines.Timeline.getLayerNames()`.

#### `getResourceFolderOptions()` → array
Recursively traverses all resource folders via `Pixera.Resources.ResourceFolder.getResourceFolders()`, building full hierarchical paths separated by `/` (e.g. `"folder/subfolder"`).

#### `getMatchModeOptions()` → array
Returns hardcoded `{"By Order", "Match by same name", "Suffix - Batch"}`.

#### `ref()` → self
Returns `self()`. Slot style `leftNone;rightOut;triggerHidden` — exposes a module reference output pin.

#### `getAllResources()` → map `{resName → resHandle}`
Recursively walks all resource folders. For every resource found, stores `{resName → resHandle}` in a flat map. Used by Suffix - Batch mode and discoverSuffixes.

---

### Data Gathering & Matching Functions (all hidden slots)

#### `findFolderPairs(layer, folder, matchMode)` → array of pair tables

Builds clip↔resource pairs within a single resource folder.

**By Order:** pairs clip[i] with resource[i] sequentially (up to `min(#clips, #resources)`).  
**Match by same name:** builds a `{resName → resHandle}` map for the folder; for each clip, looks up its assigned resource name in the map; only clips with an exact match are paired.

Each pair entry: `{index, origResName, origHandle, replHandle, replName}`

#### `findSuffixPairs(layer, allResources, suffix)` → array of pair tables

For each clip on the layer:
1. Gets assigned resource name (e.g. `clip.mov`)
2. Strips extension → `baseName` (e.g. `clip`)
3. Constructs target: `baseName .. "_" .. suffix` (e.g. `clip_EN`)
4. Iterates the flat `allResources` map; on first resource whose base name matches the target, creates a pair and breaks

Each pair entry: `{index, origResName, origHandle, replHandle, replName}`

#### `validatePairs(pairList)` → `{hasIssues, issues}`

For each pair:
- Compares file extensions (case-insensitive) — mismatch logged as "format mismatch"
- Queries both resource durations via `Pixera.Resources.Resource.getDuration()` — absolute difference > 0.001 s logged as "duration mismatch"

Returns `{hasIssues = bool, issues = {array of strings}}`.

#### `preflightCheck()` → array of error strings

Validates required properties before execution:
- Source Timeline and Source Layer must be non-empty
- By Order / Match by same name: Resource Folder and New Timeline Suffix required
- Suffix - Batch: at least one suffix must exist (via `self._csvSuffixes` or Suffix List property)

Returns empty array if all checks pass.

---

### Action Functions (hidden slots / button handlers)

#### `applyPairs(layer, pairList, doLog)` → integer (replaced count)

For each pair: calls `Pixera.Resources.Resource.getId(replHandle)` then `Pixera.Timelines.Clip.assignResource(clip, resId, false)`. Logs each replacement if `doLog` is true. Returns count of successful replacements.

#### `importCsv()` → void

Opens the file at CSV File Path, parses each line splitting on `,.:;` and whitespace, trims entries, stores the result in `self._csvSuffixes`. Logs imported count or error. Sets `self._csvSuffixes = nil` on failure.

#### `discoverSuffixes()` → void

Calls `getAllResources()`, then for each resource strips the extension and extracts the suffix segment (`_([^_]+)$` — everything after the last underscore). Deduplicates, sorts alphabetically, logs the result as a comma-separated list ready to paste into Suffix List.

---

### Main Slots

#### `Preview()` — slot style: `leftIn;rightNone`

Full dry-run. Runs preflightCheck, resolves layers, then for each suffix/folder/layer calls the appropriate find function and validatePairs. Logs all planned replacements and sanity issues. No timelines are created or modified.

Logs `=== PREVIEW START ===` / `=== PREVIEW END ===` markers.

#### `Execute()` — slot style: `leftIn;rightOut`

Same logic as Preview but actually duplicates and modifies timelines:
1. preflightCheck
2. (Suffix - Batch) validatePairs across all layers — skip suffix if issues and stopOnFail
3. `duplicateThis(sourceTimeline, false)` → new timeline handle
4. `setName(newTimeline, newName)`
5. For each layer: findXxxPairs → applyPairs
6. If a layer is missing on the duplicate in batch mode: `removeThis(newTimeline)` (rollback)

Fires the rightOut trigger signal on completion.

---

## 3. Pixera API Calls

### Timelines

| Call | Returns | Used in |
|------|---------|---------|
| `Pixera.Timelines.getTimelineNames()` | `{string}` | getTimelineOptions |
| `Pixera.Timelines.getTimelineFromName(name)` | timeline handle or nil | getLayerOptions, Preview, Execute |
| `Pixera.Timelines.Timeline.getLayerNames(tl)` | `{string}` | getLayerOptions, Preview, Execute |
| `Pixera.Timelines.Timeline.getLayerFromName(tl, name)` | layer handle or nil | Preview, Execute |
| `Pixera.Timelines.Timeline.duplicateThis(tl, false)` | timeline handle or nil | Execute |
| `Pixera.Timelines.Timeline.setName(tl, name)` | void | Execute |
| `Pixera.Timelines.Timeline.removeThis(tl)` | void | Execute (batch rollback) |

### Layers

| Call | Returns | Used in |
|------|---------|---------|
| `Pixera.Timelines.Layer.getClips(layer)` | `{clip handle}` or nil | findFolderPairs, findSuffixPairs, applyPairs |

### Clips

| Call | Returns | Used in |
|------|---------|---------|
| `Pixera.Timelines.Clip.getAssignedResourceName(clip)` | filename string or nil | findFolderPairs, findSuffixPairs |
| `Pixera.Timelines.Clip.getAssignedResource(clip)` | resource handle or nil | findFolderPairs, findSuffixPairs |
| `Pixera.Timelines.Clip.assignResource(clip, resId, false)` | void | applyPairs |

### Resources

| Call | Returns | Used in |
|------|---------|---------|
| `Pixera.Resources.getResourceFolders()` | `{folder handle}` or nil | getResourceFolderOptions, getAllResources |
| `Pixera.Resources.getResourceFolderWithNamePath(path)` | folder handle or nil | Preview, Execute |
| `Pixera.Resources.ResourceFolder.getName(folder)` | string or nil | getResourceFolderOptions |
| `Pixera.Resources.ResourceFolder.getResourceFolders(folder)` | `{folder handle}` or nil | getResourceFolderOptions, getAllResources |
| `Pixera.Resources.ResourceFolder.getResources(folder)` | `{resource handle}` or nil | getAllResources, findFolderPairs |
| `Pixera.Resources.Resource.getName(resource)` | filename string or nil | getAllResources, findFolderPairs |
| `Pixera.Resources.Resource.getId(resource)` | ID or nil | applyPairs |
| `Pixera.Resources.Resource.getDuration(resource)` | float (seconds) or nil | validatePairs |

---

## 4. Clip/Resource Assignment Flow

### By Order

```
Execute()
  preflightCheck()
  getTimelineFromName(tlName) → sourceTimeline
  for each targetLayerName:
    getLayerFromName(sourceTimeline, layerName) → srcLayer
    findFolderPairs(srcLayer, resFolder, "By Order")
      getClips(srcLayer) → clips[]
      getResources(resFolder) → resources[]
      clip[i] ↔ resource[i]  (sequential, min length)
    validatePairs(pairList)
  duplicateThis(sourceTimeline) → newTimeline
  setName(newTimeline, tlName + timelineSuffix)
  for each targetLayerName:
    getLayerFromName(newTimeline, layerName) → dupLayer
    findFolderPairs(srcLayer, resFolder, "By Order") → pairList
    applyPairs(dupLayer, pairList)
      for each pair: assignResource(clips[pair.index], getId(replHandle))
```

### Match by Same Name

```
Execute()
  preflightCheck()
  getTimelineFromName(tlName) → sourceTimeline
  for each targetLayerName:
    getLayerFromName(sourceTimeline, layerName) → srcLayer
    findFolderPairs(srcLayer, resFolder, "Match by same name")
      getClips(srcLayer) → clips[]
      getResources(resFolder) → {resName → resHandle} map
      for each clip:
        getAssignedResourceName(clip) → origResName
        if origResName in resMap → pair created
    validatePairs(pairList)
  duplicateThis / setName / applyPairs  (same as By Order)
```

### Suffix - Batch

```
Execute()
  preflightCheck()
  getAllResources() → allRes {resName → resHandle}  ← loaded once
  for each suffix:
    duplicateThis(sourceTimeline) → newTimeline
    setName(newTimeline, tlName .. "_" .. suffix)
    for each targetLayerName:
      srcLayer = getLayerFromName(sourceTimeline, layerName)
      dupLayer = getLayerFromName(newTimeline, layerName)
      if dupLayer == nil → removeThis(newTimeline); break  ← rollback
      findSuffixPairs(srcLayer, allRes, suffix)
        for each clip:
          origResName → strip ext → baseName
          target = baseName .. "_" .. suffix
          scan allRes map → first resource whose base name == target
      applyPairs(dupLayer, pairList)
```

---

## 5. Module State

### `self._csvSuffixes`

| Attribute | Detail |
|-----------|--------|
| Type | Array of strings, or nil |
| Set by | `importCsv()` on success |
| Cleared by | `importCsv()` on failure (sets nil) |
| Read by | `preflightCheck()`, `Preview()`, `Execute()` |
| Priority | Checked before Suffix List property; if populated, Suffix List is ignored |
| Lifetime | Persists across multiple Preview/Execute calls until importCsv is called again or module reloads |

### `self.helper`

| Attribute | Detail |
|-----------|--------|
| Type | PixcHelper object (requires `pixcHelper` library) |
| Initialized | Lazily, on first call in any major function; `if self.helper == nil then require "pixcHelper"; self.helper = createPixcHelper(pixc, self()) end` |
| Used for | `self.helper:getProperty(name, default)` and `self.helper.toBool(str)` |
| Lifetime | Created once, reused for all subsequent calls |

---

## 6. Slot Wiring Summary

### Exposed Slots

| Function | Slot Style | Input | Output | Description |
|----------|-----------|-------|--------|-------------|
| `ref` | leftNone;rightOut;triggerHidden | — | module ref | Module reference output (hidden trigger) |
| `Preview` | leftIn;rightNone | trigger | — | Dry-run; logs planned replacements, no changes |
| `Execute` | leftIn;rightOut | trigger | trigger | Duplicates timeline(s) and replaces clip resources; fires on completion |

### Hidden (Internal Only)

All other functions — options providers, pair finders, validators, helpers — use `slotStyle: "hidden"` and are called by Preview/Execute or bound to property optionsSourceFunc/optionsAction.

---

## 7. Summary

**Three matching strategies, one shared execution path:**

| Mode | Pair source | Scope | Timeline name |
|------|-------------|-------|---------------|
| By Order | Position in folder | Single folder | `sourceName + suffix property` |
| Match by same name | Exact filename match in folder | Single folder | `sourceName + suffix property` |
| Suffix - Batch | `baseName_SUFFIX` pattern | All resources globally | `sourceName_SUFFIX` |

**Core invariants:**
- Pairs are always `{index, origResName, origHandle, replHandle, replName}`
- Validation (extension + duration) runs before any timeline is created
- Suffix - Batch loads all resources once, then reuses for every suffix
- Partial timelines from failed batch iterations are removed automatically
- `self._csvSuffixes` takes priority over Suffix List property when both present
- Underscore stripping: Suffix List entries have leading `_` stripped via `gsub("^_+", "")` before use
