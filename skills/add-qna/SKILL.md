---
name: add-qna
description: Use when the user wants more Q&A for a specific chapter. Add 5 new short, non-duplicate Q&A items to the chapter's summary file, based on the chapter body and raw source material, while preserving style and existing entries.
---

# Add QnA

Use this skill when the user asks to add more review questions to a chapter.

## Workflow

1. Identify the target summary file from the chapter number, title, or explicit path.
2. Read the chapter body and current `## Q&A` section.
3. Generate exactly 5 new Q&A entries.
4. Ensure each new item:
   - is short
   - is not already in the file
   - covers meaningful concepts, not trivia
   - matches the chapter's current tone and wording
5. Append the new entries to the end of the `## Q&A` section.

## Content Rules

- Use `raw` as the primary source of truth and `summary` as secondary context.
- Keep answers concise.
- Prefer conceptual questions that help with exam review.
- If the user asked about a confusing topic, also refine the main chapter body to clarify that topic.

## Guardrails

- Do not delete old Q&A unless the user explicitly asks.
- Do not add duplicate questions with minor wording changes.
- Keep Q&A in the same file, not split across chapters.
