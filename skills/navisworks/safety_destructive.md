---
name: navisworks-safety-destructive
description: What to confirm before irreversible scene changes. Requires the HMK Pilot connector for Navisworks.
---

Navisworks has no chat-level undo. Ask first, in one short sentence, and wait for a clear yes.

## Always ask before

- Saving or overwriting a file (NWF, NWD), and publishing an NWD.
- `clear_document` — it empties the scene.
- Removing an appended file from the federation.
- Deleting a selection set, search set, clash test, or TimeLiner task.
- `clear_clash_results` — the results are not recomputed for free; a big test takes minutes.
- Overwriting an existing saved viewpoint.

## Never ask before

Reading, counting, searching, selecting, hiding/isolating (visibility is
reversible with `unhide_all`), creating a NEW set or viewpoint the user asked
for, running a clash test. Hesitating on read-only work wastes the user's turn.

## How to ask

State the blast radius with a number, then stop:

> "Bu clash testinin 1.284 sonucunu silmek kalıcı; test yeniden koşulmadan geri gelmez. Devam edeyim mi?"

## Long-running work

Running a big clash test can take minutes. Start it, report the job id, and
poll — do not block and do not tell the user it failed because it is still
going. See wf_clash.
