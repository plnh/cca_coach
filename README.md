# CCA Architect Coach

A folder of Markdown files that turns Claude Code into a study coach for the **Claude Certified Architect — Foundations
(CCA-F)** exam, run entirely inside Claude Code. .

Teaches the full curriculum, runs exam-style quizzes, and tracks your progress.

---

## Setup

You need [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) installed. Then:

```bash
cd cca-coach
claude
```

Once Claude Code is open in this folder, **restart it** (commands load on startup), then type `/` to see the coaching commands. Start with:

```
/cca-start
```

That's it. There's nothing to `pip install` — the whole thing is Markdown.

---

## Commands

| Command | What it does | Example |
|---------|--------------|---------|
| `/cca-start` | Friendly welcome + picks up where you left off | `/cca-start` |
| `/cca-curriculum` | Lists every lesson across all domains | `/cca-curriculum` |
| `/cca-lesson` | Teaches one lesson, plain-English first | `/cca-lesson domain_1 1.3` |
| `/cca-quiz` | Runs an exam-style quiz, one question at a time | `/cca-quiz domain_1 10` |
| `/cca-quiz mock_exam` | Mixed questions across all domains | `/cca-quiz mock_exam 20` |
| `/cca-glossary` | Defines a term in plain language | `/cca-glossary stop_reason` |
| `/cca-progress` | Shows what you've completed and your quiz scores | `/cca-progress` |

If a command doesn't show up when you type `/`, restart Claude Code — commands are only loaded at startup.

---

## What's covered

| Domain | Topic | Exam weight |
|--------|-------|-------------|
| Foundations | How Claude works: Messages API, tokens, tool use | — |
| Domain 1 | Agentic Architecture & Orchestration | 27% |
| Domain 2 | Claude Code Configuration & Workflows | 20% |
| Domain 3 | Prompt Engineering & Structured Output | 20% |
| Domain 4 | Tool Design & MCP Integration | 18% |
| Domain 5 | Context Management & Reliability | 15% |

Code examples throughout use **Python** — no other languages required.

---

## How it's built (for anyone who wants to edit it)

Everything is plain Markdown. No code to maintain.

```
cca-coach/
├── README.md                 ← you're reading it
├── CLAUDE.md                 ← coaching behaviour rules (tone, pacing)
├── cca-coach-material.md     ← all curriculum, quiz questions, glossary
├── .gitignore                ← keeps personal progress.md out of git
└── .claude/
    └── commands/             ← each file here becomes a /command
        ├── cca-start.md      → /cca-start
        ├── cca-lesson.md     → /cca-lesson
        ├── cca-quiz.md       → /cca-quiz
        ├── cca-progress.md   → /cca-progress
        ├── cca-glossary.md   → /cca-glossary
        └── cca-curriculum.md → /cca-curriculum
```

**To add or edit lessons or quiz questions:** edit `cca-coach-material.md`.
The commands read from it, so you never touch the command files for content
changes.

**To change how the coach behaves** (tone, how it quizzes): edit `CLAUDE.md`.

**To add a new command:** create a new Markdown file in `.claude/commands/`.
The filename (without `.md`) becomes the command name. Whatever you write in
the file is the instruction Claude follows when the command runs.

After any change, push it and your teammates get it on their next `git pull`.

---
