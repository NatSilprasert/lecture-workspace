---
name: add-qna
description: Use when the user wants more Q&A for a specific chapter. Add 5 new short, non-duplicate Q&A items to the chapter's summary file, grounded only in the chapter body or raw source material, while preserving style and existing entries.
---

# Add QnA

Use this skill when the user asks to add more review questions to a chapter.

## Workflow

1. Identify the target summary file from the chapter number, title, or explicit path.
2. Read the chapter body and current `## Q&A` section.
3. Read the corresponding `raw` source for the same chapter when available, and use it as the primary source of truth.
4. Generate exactly 5 new Q&A entries.
5. Ensure each new item:
   - is short
   - is not already in the file
   - covers meaningful concepts, not trivia
   - matches the chapter's current tone and wording
   - is directly supported by either the chapter body in `summary` or the matching file in `raw`
6. Before appending, verify that each question-answer pair can be traced back to a specific statement in `raw` or `summary`.
7. Append the new entries to the end of the `## Q&A` section.

## Content Rules

- Use `raw` as the primary source of truth and `summary` as secondary context.
- Use only `raw` and `summary` as sources. Do not add facts from general background knowledge unless that fact is explicitly stated in one of those sources.
- Keep answers concise.
- Prefer conceptual questions that help with exam review.
- If the user asked about a confusing topic, also refine the main chapter body to clarify that topic.

## Guardrails

- Do not delete old Q&A unless the user explicitly asks.
- Do not add duplicate questions with minor wording changes.
- Keep Q&A in the same file, not split across chapters.
- If a candidate Q&A cannot be supported clearly from `raw` or `summary`, do not add it.
