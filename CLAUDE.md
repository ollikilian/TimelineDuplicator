# TimelineDuplicator

A Pixera Control module that duplicates a source timeline and replaces clip resources on selected layers. See README.md for full usage documentation and ARCHITECTURE.md for the internal module reference.

---

## Pixera API Protocol

When working with Pixera systems, use JSON-RPC over TCP to the configured host:port (default 1400). Resource paths and clip assignment methods have specific formats — check persistent memory or TESTING.md for the learned protocol before writing new scripts.

---

## Environment Constraints

- No browser/localhost access available — if inspection of a running site is needed, ask the user to paste DevTools output.
- GitHub CLI (gh) may not be installed; check before relying on it and fall back to git remote + manual auth instructions.

---

## Conventions

### Deliverable Scope

When asked for a specific task (e.g., "set up tests"), produce only what was requested. Do not create supplementary docs (review summaries, extra guides) unless explicitly asked.
