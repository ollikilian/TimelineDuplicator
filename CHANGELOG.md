# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2025-01-17

### Added
- Initial release of TimelineDuplicator module
- Single execution mode (Execute) for duplicating one timeline with resource replacement
- Batch execution mode (BatchExecute) for multi-language workflows
- Dynamic dropdown selection for Timelines, Layers, and Resource Folders
- Two match modes: By Order (sequential) and By Name (filename-based)
- Recursive folder support for nested resource structures
- Detailed logging with pixc.log(), pixc.warning(), and pixc.error()
- ref() action for module handle chaining
- Compliant with Pixera Module Style Guide and Lua Style Guide
- Comprehensive user documentation (DOCX format)
- MIT License

### Technical Details
- Pixera API Revision: 481
- Minimum Pixera Version: 2.0.172
- Module Version: 1.0

---

## Future Roadmap

- [ ] Preview mode (show what would be replaced without executing)
- [ ] Undo/Rollback functionality
- [ ] Resource validation before replacement (format checking)
- [ ] Progress indicator for batch operations
- [ ] Custom rename patterns (e.g., auto-increment suffixes)
- [ ] Integration with Cue system for automated workflows
