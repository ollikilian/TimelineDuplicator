# TimelineDuplicator — Testing Strategy

## Static Review Checklist

- [ ] Orphan timeline cleanup: if duplication succeeds but a later lookup fails, `removeThis` is called and the log says "removing duplicate"
- [ ] `BatchExecute` continues to the next folder after a per-iteration failure (no early `return`)
- [ ] By Name matching is case-sensitive — document this or add `string.lower()` normalization
- [ ] Folder list parsing handles trailing semicolons, leading/trailing spaces, and single-entry lists correctly

---

## Manual Integration Tests

### Test project setup

- One source timeline: **TestTL** with 3 clips on a layer named **Video**
- Resource folders: `Animals/` (3 resources), `Buildings/` (2 resources)

### `Execute` — single operation

| # | Setup | Expected |
|---|-------|----------|
| 1 | All properties valid, Match Mode = By Order | New timeline `TestTL_copy`, 3 clips replaced in order, log: "replaced 3 clips" |
| 2 | Match Mode = By Name, resource names match clip assignments | 3 clips replaced by name |
| 3 | Match Mode = By Name, only 2 of 3 resource names match | 2 clips replaced, count reflects actual matches |
| 4 | Folder has 2 resources, layer has 3 clips, By Order | 2 clips replaced (min logic), log: "replaced 2 clips" |
| 5 | Timeline property blank | Log: "no timeline selected", no timeline created |
| 6 | Layer property blank | Log: "no layer selected", no timeline created |
| 7 | Resource Folder blank | Log: "no resource folder selected", no timeline created |
| 8 | Timeline name does not exist in project | Log: "timeline not found", no timeline created |
| 9 | Layer name does not exist on source timeline | Log: "layer not found … - removing duplicate", no orphan left |
| 10 | Resource folder path does not exist | Log: "resource folder not found … - removing duplicate", no orphan left |

### `BatchExecute` — multi-folder batch

| # | Setup | Expected |
|---|-------|----------|
| 11 | Resource Folder List = `Animals;Buildings` | Two new timelines: `TestTL_Animals`, `TestTL_Buildings` |
| 12 | List = `Animals ; Buildings ; ` (spaces + trailing semicolon) | Same as #11 — whitespace trimmed |
| 13 | List = `Animals;NonExistent;Buildings` | Animals and Buildings succeed; NonExistent logs "folder not found - removing duplicate" and continues |
| 14 | Nested path `Media/Animals` | Suffix derived as `_Animals`, timeline named `TestTL_Animals` |
| 15 | Resource Folder List blank | Log: "no folder list", nothing created |

### Option source functions

| # | Test | Expected |
|---|------|----------|
| 16 | Open Timeline dropdown | Populated with all timeline names in project |
| 17 | Select a timeline, open Layer dropdown | Shows only layers on that timeline |
| 18 | Open Resource Folder dropdown | Flat list including nested paths (e.g. `Media/Animals`) |

---

## Regression Checklist

Run these before releasing any modified version:

- [ ] `Execute` golden path — By Order, all properties valid
- [ ] `Execute` with blank Timeline property — must not crash
- [ ] `BatchExecute` with 3 folders including one invalid path
- [ ] Layer dropdown repopulates when Timeline selection changes
- [ ] `Log Messages = false` suppresses per-clip logs but not the summary log
- [ ] No orphan timelines left after any failure in `Execute` or `BatchExecute`
