---
name: autocad-civil3d-safety-destructive
description: What to confirm before irreversible drawing changes. Requires the HMK Pilot connector for AutoCAD / Civil 3D.
---

Some operations cannot be undone from the chat. Ask first, in one short sentence, and wait for a clear yes.

## Always ask before

- Saving or overwriting a file (`SAVE`, `SAVEAS`, export over an existing path).
- Purging (`purge_unused`, `purge_blocks`) — it deletes definitions the user may still want.
- Erasing entities the user did not explicitly point at.
- `audit_drawing` with fix enabled, and anything that rewrites the drawing database wholesale.
- Deleting or merging layers — merging moves every entity and cannot be reversed selectively.
- Civil: deleting an alignment, profile, corridor or surface. Downstream objects depend on them and will break or rebuild empty.
- Running a script or `run_command` whose effect you cannot predict.

## Never ask before

Reading anything. Counting. Listing. Zooming. Selecting. Creating a NEW layer, block reference, or annotation the user asked for. Hesitating on read-only work wastes the user's turn.

## How to ask

State the blast radius with a number, then stop:

> "Bu 3 katmanı birleştirmek 1.284 nesneyi kalıcı olarak taşır ve geri alınamaz. Devam edeyim mi?"

Do not bury the question at the end of a long explanation, and do not ask twice for the same approval in one turn.

## After a destructive action

Report what actually changed, measured — see ux_reporting. "Purged 214 unused blocks" is a report; "purge complete" is not.
