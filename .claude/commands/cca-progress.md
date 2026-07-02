---
description: Show CCA-F study progress
---

Show the learner's CCA-F study progress.

1. Read `progress.md` in this folder. If it doesn't exist, say they
   haven't started tracking yet and suggest `/cca-start`.
2. Summarise:
   - Lessons completed vs total (the curriculum map in
     `cca-coach-material.md` lists all lessons)
   - Per-domain completion
   - Recent quiz scores with dates
   - Average quiz percentage
3. Recommend the natural next step: the first unticked lesson, or a
   quiz on a domain they finished but haven't tested.

If $ARGUMENTS is "reset", ask for explicit confirmation, and only after
they confirm, clear progress.md back to an empty template.
