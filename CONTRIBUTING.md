# Contributing to TimelineDuplicator

Thanks for your interest in improving TimelineDuplicator! This document provides guidelines for contributing.

## Code of Conduct

- Be respectful and constructive
- Focus on the code, not the person
- Help others learn and grow

## How to Contribute

### Reporting Bugs

1. **Check existing issues** — avoid duplicates
2. **Use the bug template** (if available) or provide:
   - Pixera version (e.g., 2.0.172)
   - Steps to reproduce
   - Expected vs. actual behavior
   - Relevant logs (from Pixera Log window)
3. **Attach screenshots** if helpful
4. Open an issue on GitHub

### Suggesting Features

1. **Describe the feature** — what problem does it solve?
2. **Provide examples** — use cases or workflow scenarios
3. **Check dependencies** — does it require new Pixera APIs?
4. Open a GitHub Discussion or Issue with the `enhancement` label

### Submitting Code

1. **Fork** the repository
2. **Create a branch** for your feature:
   ```bash
   git checkout -b feature/your-feature-name
   ```
3. **Follow style guides**:
   - [Pixera Module Style Guide](https://help.pixera.one/en_US/module-style-guide)
   - [Pixera Lua Style Guide](https://help.pixera.one/en_US/control-lua-style-guide)
4. **Test thoroughly** — the module must work end-to-end
5. **Document changes**:
   - Update README.md if user-facing
   - Update CHANGELOG.md with your changes
   - Add inline comments for complex logic
6. **Commit with clear messages**:
   ```bash
   git commit -m "Add: feature description"
   git commit -m "Fix: bug description"
   git commit -m "Docs: update installation steps"
   ```
7. **Push to your fork**:
   ```bash
   git push origin feature/your-feature-name
   ```
8. **Open a Pull Request**:
   - Reference related issues (e.g., "Closes #5")
   - Describe your changes
   - Mention any breaking changes

## Development Workflow

### Setting Up Locally

```bash
git clone https://github.com/your-username/TimelineDuplicator.git
cd TimelineDuplicator

# Make your changes to TimelineDuplicator.json
# Test in Pixera Control
# Update docs if needed
```

### Testing

1. **Copy the module** to your Pixera Control user library
2. **Reload Control** to load the updated module
3. **Test all scenarios**:
   - Single Execute with different match modes
   - BatchExecute with multiple folders
   - Error handling (missing properties, invalid folders)
   - Logging output
4. **Check the Pixera log** for errors or warnings

### Code Review Checklist

Before submitting a PR, ensure:
- ✅ Code follows Pixera style guides
- ✅ All nil-checks are in place
- ✅ pixc.log() is used appropriately (not cluttered)
- ✅ Variable names are descriptive (lowerCamelCase)
- ✅ No global variables (use `local` or `self.`)
- ✅ Early returns for validation
- ✅ Functions are well-commented
- ✅ Module tested end-to-end in Pixera Control

## Branching Strategy

- **main** — Stable, released code
- **develop** — Integration branch for features (if used)
- **feature/*** — Feature branches (merge to develop, then main)
- **fix/*** — Bug fix branches (merge to develop, then main)

## Release Process

Maintainers will:
1. Merge PRs into main
2. Update CHANGELOG.md with version and changes
3. Create a GitHub Release with tag (e.g., v1.1.0)
4. Attach TimelineDuplicator.json and documentation
5. Announce in Pixera community channels (if applicable)

## Documentation

- **README.md** — User-facing quick start and feature overview
- **TimelineDuplicator_Documentation.docx** — Comprehensive guide for end users
- **Inline comments** — Explain the "why", not the "how"
- **CHANGELOG.md** — User-visible changes per release

Example comment:
```lua
-- Check if folder exists; if not, warn and skip processing
local resFolder = Pixera.Resources.getResourceFolderWithNamePath(folderPath)
if resFolder == nil then
    pixc.warning(...)
    return 0
end
```

## Getting Help

- 📖 [Pixera Help Center](https://help.pixera.one)
- 🎓 [Pixera Control API](https://help.pixera.one/api)
- 💬 [Pixera Forum](https://help.pixera.one/forum)
- 🐛 Open a GitHub Issue

---

Thank you for contributing! 🙏
