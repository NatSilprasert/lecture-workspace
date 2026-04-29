# Lecture Summary Workspace

This workspace is used to maintain summaries of my lectures.

This repo also includes project-local Codex skills for summary maintenance and study workflows.

## Project Structure

```text
.
├── AGENTS.md       Repo-specific rules for updating summaries, Q&A, and workflow behavior. 
├── README.md       Quick guide for anyone continuing work in this workspace.
├── .codex/skills/  Project-local Codex skills used only in this repository.
├── raw/            Source files such as lecture PDFs and reading materials.
├── summary/        Thai summaries derived from files in raw/. Q&A is added only when asked.
```

## How To Use

Just ask the AI a question about your lecture through an AI agent such as Codex, Claude Code, or an IDE agent.

## How It Works

1. The AI answers by reading content from `raw/` and `summary/`, and may add a small amount of extra explanation to improve understanding.
2. The AI updates its explanation in the relevant chapter summary.
3. If you ask a question or request review questions, the AI adds a `Q&A` section to the relevant summary file when needed.
4. New Q&A items must be grounded in `raw/` or the chapter body in `summary/` only, not generic outside knowledge.

## Skills

| Skill | Purpose |
| --- | --- |
| `active-recall` | Generate 10 active-recall questions from a chapter Q&A, creating more Q&A first if needed, and prefer an interactive multiple-choice artifact when supported. |
| `add-qna` | Add 5 new non-duplicate Q&A items to a specified summary file, grounded only in `raw/` or the matching chapter summary. |
| `refactor-summary` | Rewrite a summary to be shorter, smoother, and easier to review before exams. |
| `generate-summary` | Find files in `raw/` that do not yet have summaries and create matching summary files. |
