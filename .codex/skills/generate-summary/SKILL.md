---
name: generate-summary
description: Use when the user wants summaries created for source files in raw that do not yet have matching files in summary. Follow the New Source Files Rule by preserving relative paths from raw to summary, including raw/test to summary/test.
---

# Generate Summary

Use this skill when new source material appears under `raw/` and needs a matching summary.

## New Source Files Rule

- When new files are added under `raw`, always create or update summaries from those files.
- Preserve relative folder structure from `raw` to `summary`.
- If files are in `raw/test`, place summaries in `summary/test`.

## Workflow

1. Scan `raw/` for source files.
2. Scan `summary/` for existing summaries.
3. Identify raw files that do not yet have an appropriate summary output.
4. For each missing item:
   - derive a clear markdown filename
   - place it in the mirrored path under `summary/`
   - summarize in Thai
   - cover the important points from the source
   - keep technical terms in English when useful
   - end with `## Q&A`
5. If an existing summary already corresponds to the source, update it instead of creating a duplicate.

## Matching Rules

- Prefer explicit chapter mappings already established in the repo.
- For new unmatched sources, create a new summary file rather than overwriting an unrelated one.
- Keep naming readable and stable.

## Guardrails

- Use `raw` as the primary source of truth.
- Do not contradict the slides or source document.
- Keep summaries concise and easy to study from.
