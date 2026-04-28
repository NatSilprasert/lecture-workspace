---
name: active-call
description: Use when the user wants active recall practice from a specific chapter or summary file. Pull 10 short questions from that chapter's Q&A; if fewer than 10 exist, create additional non-duplicate Q&A from the chapter body and add them to the same summary file before asking the questions.
---

# Active Call

Use this skill when the user asks for active recall, oral drill, practice questions, or self-test from a specific chapter.

## Workflow

1. Identify the target summary file from the chapter number, title, or explicit path.
2. Read the summary body and its `## Q&A` section.
3. Count existing Q&A entries.
4. If there are fewer than 10 questions:
   - Create additional short Q&A items from the existing summary body.
   - Keep them consistent with `raw` and the chapter summary.
   - Do not duplicate or trivially rephrase existing Q&A.
   - Add the new Q&A to the same summary file, preserving existing entries.
5. Build a set of 10 questions from that file's Q&A.
6. Randomize question order before presenting them.

## Output Rules

- Ask one compact question per line.
- Do not reveal answers unless the user asks.
- If the user asks to check answers, grade against the same Q&A set.
- Keep question wording short and study-friendly.

## Guardrails

- Follow `AGENTS.md` rules for updating the chapter body and Q&A.
- Prefer the explicitly requested chapter; do not mix chapters unless the user asks.
- If the file already has 10 or more Q&A items, do not add new ones unless the user asks for more.
