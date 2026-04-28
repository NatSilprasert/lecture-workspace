# Lecture Summary Workspace

This workspace is used to maintain summaries of my lectures.

## Project Structure

```text
.
├── AGENTS.md       Repo-specific rules for updating summaries, Q&A, and workflow behavior. 
├── README.md       Quick guide for anyone continuing work in this workspace.
├── raw/            Source files such as lecture PDFs and reading materials.
├── summary/        Thai summaries derived from files in raw/. Each file should end with a Q&A section.
└── skills/         Reusable local skills for common summary and homework workflows.
```

## How To Use

Just ask the AI a question about your lecture through an AI agent such as Codex, Claude Code, or an IDE agent.

## How It Works

1. The AI answers by reading content from `raw/` and `summary/`, and may add a small amount of extra explanation to improve understanding.
2. The AI updates its explanation in the relevant chapter summary.
3. The AI adds your question and a short answer to the `Q&A` section at the end of that summary file.

## Skills

| Skill | Purpose |
| --- | --- |
| `active-call` | Generate 10 active-recall questions from a chapter Q&A, creating more Q&A first if needed. |
| `add-qna` | Add 5 new non-duplicate Q&A items to a specified summary file. |
| `refactor-summary` | Rewrite a summary to be shorter, smoother, and easier to review before exams. |
| `generate-summary` | Find files in `raw/` that do not yet have summaries and create matching summary files. |
