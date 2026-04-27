# TimelineDuplicator

A Pixera Control module for duplicating timelines and replacing clip resources on a specified layer. Perfect for multi-language production workflows where you need to create variants of the same timeline with different audio/video tracks.

## Overview

TimelineDuplicator automates a common multi-language workflow in Pixera:
- **Duplicate** a programmed timeline in its entirety (all cues, clips, keyframes, layer settings)
- **Replace** clip resources on a designated layer with alternative content (different language audio, alternate video tracks, etc.)
- **Scale** to batch operations for creating multiple language variants in one go

Instead of manually recreating timeline programming for each language, this module preserves all timing and structure while swapping only the media content.

## Features

- 🎬 **Dynamic dropdowns** for Timeline, Layer, and Resource Folder selection
- 🔄 **Two match modes**: By Order (sequential) or By Name (filename-based)
- ⚡ **Single execution** (Execute) or **batch processing** (BatchExecute)
- 📝 **Detailed logging** for troubleshooting
- 🎯 **Recursive folder support** for nested resource structures
- ✅ **Style-compliant** with Pixera Module and Lua Style Guides

## Installation

1. Download the latest `TimelineDuplicator.json` from the [Releases](https://github.com/yourusername/TimelineDuplicator/releases)
2. Copy it to your Pixera Control user library folder:
   ```
   C:\ProgramData\AV Stumpfl\Pixera\control_library_user\
   ```
3. Open Pixera Control and drag the module onto a Control page
4. The module auto-initializes — verify in the Pixera log that initialization succeeded

## Quick Start

### Single Language Variant

1. **Select source Timeline** from the Timeline dropdown
2. **Select Layer** containing the clips to replace
3. **Select Resource Folder** with the replacement media
4. **Choose Match Mode**:
   - **By Order**: Clip 1 gets Resource 1, Clip 2 gets Resource 2, etc.
   - **By Name**: Each clip is matched to a resource with the same filename
5. **Trigger Execute** on the module node

Result: A new timeline appears (e.g., `MyShow_copy`) with all programming preserved and clips replaced.

### Batch Multi-Language

1. Configure Timeline, Layer, and Match Mode as above
2. In the **Resource Folder List** property, enter folder paths separated by semicolons:
   ```
   Audio/English;Audio/German;Audio/French;Audio/Spanish
   ```
3. **Trigger BatchExecute**

Result: Four new timelines are created (`MyShow_English`, `MyShow_German`, `MyShow_French`, `MyShow_Spanish`), each with content from the respective language folder.

## Configuration

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| **Timeline** | Dropdown | (empty) | Source timeline to duplicate |
| **Layer** | Dropdown | (empty) | Layer whose clips will be replaced (updates based on Timeline) |
| **Resource Folder** | Dropdown | (empty) | Folder with replacement resources |
| **Match Mode** | Dropdown | By Order | How clips are matched to resources |
| **New Timeline Suffix** | String | `_copy` | Appended to the duplicated timeline name |
| **Log Messages** | Bool | true | Enable detailed logging |
| **Resource Folder List** | String | (empty) | Semicolon-separated paths for batch mode |

## Recommended Folder Structure

### By Order Matching
```
Resources/
  Audio/
    English/
      Track_01.wav
      Track_02.wav
    German/
      Track_01.wav
      Track_02.wav
```

### By Name Matching
```
Resources/
  Audio/
    English/
      Intro_EN.wav
      Main_EN.wav
    German/
      Intro_DE.wav
      Main_DE.wav
```
(Filenames can differ; same count and order matters)

## Triggering

The module exposes two functions:

- **Execute** — Single duplication + replacement
- **BatchExecute** — Batch processing with folder list

Both appear as trigger-in slots on the module node and can be called via:
- Right-click → Trigger
- A connected Trigger node (button, hotkey, etc.)
- A Cue action during timeline playback
- Remote control (TCP, OSC, etc.)

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Dropdowns are empty | Ensure module is initialized. Check project has timelines and resource folders. |
| Layer dropdown empty | Select a Timeline first — Layer dropdown depends on Timeline selection. |
| 0 clips replaced | Check filenames match exactly (By Name mode). Verify resource folder has files. Enable Log Messages. |
| Wrong timeline name | Check the **New Timeline Suffix** property. In batch mode, suffix is derived from folder name. |
| BatchExecute does nothing | Verify **Resource Folder List** uses semicolons as separators. Folder paths are case-sensitive. |

## Documentation

Full user documentation is included in the [Docs](./docs/) folder:
- `TimelineDuplicator_Documentation.docx` — Complete module guide with screenshots and API reference

## API Reference

For developers integrating this module or building on it:

- **`ref()`** — Returns module handle for chaining
- **`Execute()`** — Performs single duplication with resource replacement
- **`BatchExecute()`** — Iterates folder list, creating multiple variants
- **Hidden functions**:
  - `getTimelineOptions()`, `getLayerOptions()`, `getResourceFolderOptions()`, `getMatchModeOptions()` — Populate dropdowns dynamically
  - `replaceClipsOnLayer()` — Core logic for clip replacement (reusable)

## Requirements

- **Pixera**: 2.0.172 or later
- **Pixera Control**: With API access to Timelines and Resources namespaces
- **Pixera API Revision**: 481 or compatible

## Style & Quality

This module adheres to:
- [Pixera Module Style Guide](https://help.pixera.one/en_US/module-style-guide)
- [Pixera Control Lua Style Guide](https://help.pixera.one/en_US/control-lua-style-guide)

All code is lowerCamelCase with nil-checks, proper error/warning logging, and clean slot styling.

## License

MIT License — See [LICENSE](./LICENSE) for details.

## Support

- 📖 See [docs](./docs/) for detailed documentation
- 🐛 Found a bug? [Open an issue](https://github.com/yourusername/TimelineDuplicator/issues)
- 💬 Have ideas? [Discussions](https://github.com/yourusername/TimelineDuplicator/discussions)

## Changelog

See [CHANGELOG.md](./CHANGELOG.md) for version history and updates.

---

**Created by**: AV Stumpfl GmbH  
**Module Version**: 1.0  
**Last Updated**: 2025  
**Pixera API Rev**: 481
