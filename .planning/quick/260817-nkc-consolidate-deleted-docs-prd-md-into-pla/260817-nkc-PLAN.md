---
quick_id: 260817-nkc
status: planned
created: 2026-08-17
---

# Consolidate the deleted PRD into the project contract

## Objective

Keep the legacy product document deleted while making `.planning/PROJECT.md` the canonical product/PRD contract, preserving product information that is not already recorded in the planning set, and removing live links to the deleted file.

## Tasks

1. Compare the deleted product document with `.planning/PROJECT.md` and `.planning/REQUIREMENTS.md`; add the missing product motivation and adoption success target to the project contract.
2. Replace all links to the deleted PRD with links to `.planning/PROJECT.md`, and correct Phase 1 context so it describes the current contract accurately.
3. Verify the deleted path has no remaining Markdown references, the intended deletion is tracked, and the documentation diff is clean.

## Verification

- A repository-wide Markdown search returns no references to the deleted product-document path.
- `git diff --check` passes.
- `git status --short` shows the intentional legacy-document deletion and only the scoped documentation changes.
