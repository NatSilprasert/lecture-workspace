---
name: refactor-summary
description: Use when the user wants a chapter summary to be shorter, smoother, clearer, or easier to study before an exam. Reorganize headings, tighten wording, and improve flow without changing meaning or contradicting the raw source.
---

# Refactor Summary

Use this skill when a summary feels too wordy, uneven, repetitive, or hard to revise from quickly.

## Goals

- Make the summary more concise.
- Improve reading flow and heading structure.
- Preserve correctness and original meaning.
- Keep the writing style consistent with the rest of `summary/`.

## Workflow

1. Read the target summary file.
2. Identify:
   - repeated ideas
   - sections that are too long
   - awkward transitions
   - headings that should be merged or split
3. Rewrite the body for clarity and study speed.
4. Preserve all useful content unless it is redundant or misleading.
5. Keep `## Q&A` at the end.
6. If the refactor changes how a concept is explained, make sure the new wording still agrees with `raw`.

## Editing Heuristics

- Prefer short bullets over dense paragraphs.
- Put definitions before edge cases.
- Keep examples only when they improve understanding.
- Move advanced notes after the core idea.
- Make the "summary" section feel like a real pre-exam recap.

## Guardrails

- Do not invent new facts.
- Do not remove useful Q&A.
- Do not turn the summary into lecture notes that are longer than before unless the user explicitly wants more detail.
