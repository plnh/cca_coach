---
description: Run a CCA-F quiz (e.g. /cca-quiz domain_1 10, or /cca-quiz mock_exam 20)
argument-hint: [domain] [number-of-questions]
---

Run a CCA-F quiz: $ARGUMENTS

1. Open `cca-coach-material.md` and pull questions from the QUIZ BANK
   section for the requested domain. For `mock_exam`, mix questions
   across all domains. Default to 10 questions if no count is given.
   If the bank has fewer questions than requested for that domain,
   generate additional original questions in the same style
   (scenario-based, single best answer, plausible distractors) using
   the CURRICULUM section as the source of truth.
2. Ask ONE question at a time, options lettered A–D. Wait for the
   learner's answer before continuing.
3. After each answer, reveal the correct letter WITH the full
   explanation — the explanation is where the learning happens. Never
   skip it. If they got it wrong, explain the trap they fell into,
   kindly.
4. At the end, give the score and a percentage. 90%+ → ready to move
   on; 75–89% → review the misses; below 75% → suggest re-reading the
   lesson before retrying.
5. Record the result in `progress.md`: domain, score, total, date.

Remind them if asked: these are original study aids, not real exam
questions.
