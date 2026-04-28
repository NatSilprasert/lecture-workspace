# AGENTS.md

## Purpose
- This folder is for Computer System Architecture summaries based on source files in `raw`.
- All summary files must be stored in `summary`.

## Source of Truth
- Use files in `raw` as the primary source.
- Use `summary` as the secondary source for quick context and consistency.
- Do not add content that contradicts the original slides.
- Extra explanation is allowed only to improve clarity without changing the original meaning.

## Response Style
- Reply in a natural conversational style, similar to normal ChatGPT chat.
- Prioritize direct answers first, then add short supporting context when useful.
- Keep responses concise by default, and expand only when the user asks for more detail.
- Base explanations on `raw` first, then align wording with `summary` when appropriate.
- If details are uncertain or not present in the slides, say so clearly and avoid over-claiming.

## Summary Rules
- Write summaries in Thai.
- Cover all six chapters and all important points from the slides.
- Keep the writing concise, clear, and easy to study.
- Use real-world examples when they improve understanding.
- Keep important technical terms in English when useful.

## File Layout
- Create and maintain these files:
- `summary/01-contemporary-architecture.md`
- `summary/02-cache-memory-organization-1.md`
- `summary/03-virtual-memory-memory-organization-2.md`
- `summary/04-storage-architecture.md`
- `summary/05-quantum-computing.md`
- `summary/06-gpu.md`

## Q&A Rules
- Every summary file must always end with a `## Q&A` section.
- When the user asks a new question, answer it and add it to the `Q&A` section of the most relevant chapter.
- Q&A entries must always stay short.
- If one question could fit multiple chapters, choose the single most relevant file unless the user says otherwise.

## Clarification Update Rules
- If the user asks about a topic, assume that part of the current summary is not clear enough.
- After answering, update the main summary content of the relevant chapter so it matches the latest explanation.
- Keep refining the summary whenever the user keeps asking about the same topic.
- If the user stops asking about that topic, treat the latest revised summary as clear enough.
- Do not only append Q&A; also improve the main explanation in the body when needed.

## Editing Rules
- Keep the summary structure and writing style consistent.
- Do not delete existing Q&A entries unless the user explicitly asks.
- When revising a summary, preserve both the main content and earlier useful Q&A unless they need correction.
