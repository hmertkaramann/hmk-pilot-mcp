---
name: revit-safety-destructive
description: Destructive operation policy — when to confirm, when to proceed, how to be reversible. Requires the HMK Pilot connector for Revit.
---

Destructive operations (delete, overwrite, bulk modify) are not equally risky. The key axis is REVERSIBILITY: in-Revit mutations (delete, bulk_set, modify) are Ctrl+Z undoable, so the default is **act, then report** — not confirm-first. Reserve confirmation for genuinely IRREVERSIBLE actions, and clarification for scope that is truly ambiguous.

## What counts as destructive

- **delete_elements** with N elements
- **bulk_set_parameter** that OVERWRITES non-empty values
- **execute_revit_code** with `doc.Delete(...)`, `Family.Delete`, `Workset.Delete`, etc.
- Any export to a path that may overwrite a file
- `doc.Save()` / `SaveAs()` — never call without explicit user ask

## Policy — reversible in-Revit mutations (delete / bulk_set / modify, all Ctrl+Z undoable)

| Scope | Policy |
|---|---|
| 1 element the user pointed at | Just do it. State the id + category in your reply. |
| A clear filter or bounded set ("Mark=TBD olan duvarları sil", "bu 40 kolonu güncelle") | Just do it in ONE call. Report count + categories + 1-3 sample ids + "Ctrl+Z ile geri alınır." No pre-confirmation — it's undoable. |
| Large (roughly 500+ elements) | Proceed, but LEAD your reply with the count + category breakdown so any surprise is obvious and one Ctrl+Z away. Pause to confirm only if the count is far larger than the user likely expected. |
| "Tümünü sil" / "delete all" with NO scope | Clarify scope first: "Tüm projede mi, aktif view'da mı?" — don't guess on an unbounded delete. |

Prefer ONE bulk call (bulk_set_parameter / delete_elements with a subset or id list) over many single-element calls — it's faster, a single undo step, and fewer round-trips. Don't fragment a bulk job into per-element calls.

## Irreversible actions — confirm FIRST even at small N

These are NOT Ctrl+Z undoable, so probe + confirm before acting:
- File writes: export / SaveAs to a path that may overwrite an existing file; `doc.Save()` / `SaveAs()` (also needs an explicit "save" from the user).
- Workset deletion, sync-with-central, closing without saving.

## Undo discipline

- Every Transaction in execute_revit_code has a description — make it descriptive ("HMK: Delete walls without Mark"). The user sees this in their Undo dropdown.
- Pilot Chat operations ARE undoable with Ctrl+Z (single transaction → single undo step).
- TransactionGroup with Assimilate() merges multiple ops into one undo step — use when several tool calls form a logical batch.
- Inform the user: "Geri almak için Ctrl+Z. Birden fazla adım birleştirildi tek undo oluyor."

## Things that are NOT undoable

- File system operations (export, save, write to disk)
- Workset deletion (sometimes — depends on Revit version)
- Sync-with-central
- Closing without saving

For these, the policy is stricter: probe + confirm even at small N. Save() needs an explicit user "save" word in the message.

## Refusal conditions

REFUSE (politely, with reasons) when:
- User says "Tümünü sil" with no scope and won't disambiguate after one ask.
- User wants to delete a workset that has unowned elements they don't own (will fail anyway, but warn first).
- User asks for SaveAs to a path that already exists (would overwrite a project file).
- User asks for "delete all warnings" — warnings can't be deleted directly; only by fixing the underlying issues.

## What to do AFTER the mass mutation

Reply with three things:
1. What was done + count
2. Sample of 1-3 affected element ids (so user can verify)
3. Undo path

Example:
> "47 duvarı sildim (OST_Walls, Mark=TBD filtresine uyanlar). Silinen örnek ID'ler: 12345, 12347, 12348. Geri almak için Ctrl+Z."

## Don't repeat the confirmation on every turn

If the user already confirmed once in the conversation, don't re-ask for similar-scoped follow-ups in the same session — annoying. Trust the running context.

## Auto-recovery

If the mutation partially succeeded (e.g. 47 of 50 elements deleted, 3 failed), DON'T retry the failing 3 automatically — show what failed, let the user decide.

## Edge case: "şu seçileni sil" with no selection

User selected nothing, says "sil". Don't delete the whole view. Reply:
> "Hiçbir element seçili değil. Aktif view'daki tüm X'i mi silmek istedin yoksa elle pick mi edeceksin?"
