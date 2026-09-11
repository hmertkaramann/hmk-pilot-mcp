---
name: revit-ux-reporting
description: Output formatting rules — how to summarize tool results back to the user. Requires the HMK Pilot connector for Revit.
---

How you report a result matters as much as getting the result. The user sees the message; they don't see the tool JSON. Format intelligently.

The example phrasings below happen to be written in Turkish. They illustrate STRUCTURE and LENGTH, not language choice — render the equivalent in whatever language the user wrote in. A user who asked in English gets the English equivalent of "Kolon ID 12345 yerleştirildi.", never the Turkish text itself.

## Result size → format mapping

| Result size | Format |
|---|---|
| 0 items / nothing happened | Single sentence: "Hiç X bulunamadı." / "X yapılmadı, çünkü..." |
| 1 item | Single sentence with the key facts: "Kolon ID 12345 (Level 1, Type X) yerleştirildi." |
| 2–5 items | Short bullet list. Each bullet ≤ 80 chars. |
| 6–20 items | Bullet list OR a small table (3-4 cols). Pick whichever reads faster. |
| 21–100 items | Sayı + top-5 örneği + "İstersen detaylı liste verebilirim." |
| 100+ items | Sadece sayı ve dağılım: "1247 elementi işledim. 412 OST_Walls, 218 OST_Floors, ..." |

## Always include a count

Even for 1-item results: "1 kolon yerleştirildi: ID 12345." not just "Yerleştirildi."

The count is the user's hook into trust. They scan it, decide it matches expectations, move on.

## Numbers with units

Convert and include both when the user is metric:
- "Wall length: 5.00 m (16.4 ft)"
- "Area: 24.3 m² (262 sq.ft)"

Imperial user → just feet/inches.

## Don't echo JSON

Wrong:
> Tool returned: {"success": true, "wallId": 12345, "wallType": "Generic - 200mm", ...}

Right:
> Generic 200mm duvarı (Level 1) (0,0)–(5m, 0) arası yerleştirildi (ID 12345).

## Don't show code

Even when execute_revit_code did the work, describe the OUTCOME:

Wrong:
> Ran `var coll = new FilteredElementCollector(doc).OfCategory(...).Count();` and got 47.

Right:
> Bu projede 47 OST_StructuralColumn var.

## Tables — when and how

Use a table when results have ≥ 3 dimensions per row AND ≥ 5 rows:

```
| Level | Walls | Doors | Windows |
|---|---|---|---|
| L1 | 124 | 18 | 22 |
| L2 | 118 | 16 | 22 |
| L3 | 89 | 12 | 14 |
```

For 2-column data, bullet lines with `:` separator is cleaner:
- L1: 124 walls
- L2: 118 walls

## Markdown discipline (renderer-friendly)

- Headings (## ###) on their own line with blank before and after
- Bullets start with `- ` (dash + space)
- Numbered lists `1. ` (number + dot + space) — never `1.text`
- Bold with `**text**` not stuck to next word
- Keep replies to 2-4 short paragraphs OR one short list — no walls of text

## Action confirmations

Mutating ops should confirm what happened:

For single mutation:
> "Kolon Mark'ı 'C-101' olarak ayarlandı."

For batch mutation:
> "47 duvarın Comments parametresi 'Demir Donatılı' olarak güncellendi (toplam 47 başarılı, 0 başarısız)."

If some failed, surface the count + 1 sample reason:
> "50 elementten 47'sinin parametresi güncellendi. 3'ü başarısız — örnek hata: 'parameter not found on type X'."

## Verify inside the same call — never as a second round

A count you report should be a count you MEASURED, not one you assumed from
the arguments you sent. But where you measure it decides whether it is free
or expensive.

Fold the check into the SAME call that made the change — the data is already
in hand there, so it costs nothing:

> Right (one execute_revit_code call): move the elements, then re-read their
> positions in the same script and print "moved 91, 0 still overlapping".

A separate verification call is a whole extra LLM round. Rounds are the
latency the user actually feels — a tool call is milliseconds, a round is
seconds to minutes. So:

- Bulk / mutating work → do the check in the same script, report its number.
- A dedicated tool that already returns what changed → that IS the check;
  report it, don't call anything else to confirm.
- Only spend a separate round when the change genuinely cannot be observed
  from where it was made.

Never claim a change you did not measure. "47 walls updated" must come from
the result, not from the fact that you passed 47 ids.

### Measure with a DIFFERENT instrument than the one you acted with

This is where a confident report goes wrong, and it is not rare. If you compute
positions and then test those same computed positions for overlap, you have
proved your arithmetic — not the result. The check passes, the report says
"0 problems", and the user opens the drawing and sees the problem.

Re-read the state from the host after the change and measure THAT. If the tool
you used to act cannot also read back independently, say the check was indirect
rather than claiming a clean result.

**Sanity-check the magnitude before you believe your own number.** A gap, offset
or tolerance has to make sense against the SIZE of the things it separates. A
2.5-unit gap between objects that are 50 units wide is not a tight fit — it is a
unit or scale mistake, and "0 overlaps" alongside it is a false negative. When a
spacing turns out to be a small fraction of the object's own extent, stop and
re-derive it from the measured size instead of reporting success.

When you report zero of anything, say what you measured and in what units:
"re-read 101 views from the drawing, min gap 20 m horizontal / 25 m vertical,
overlapping pairs 0" — not a bare "no overlaps".

## Errors and recovery

If you tried and failed, say what you tried:

Wrong:
> "Kolonları silemedim, üzgünüm."

Right:
> "OST_Columns kategorisinde tarama yaptım — 0 element döndü. OST_StructuralColumns'a baktım — orada da 0. Bu projede silinecek kolon yok."

## Multilingual switching

The Rules section already mandates: match the LAST user message's language. Reports should follow same rule. If the user types Turkish, reply Turkish (including labels in the table). Switch back when they switch.

## Don't pad the reply

Cheaper models add "Hope this helps!" / "Let me know if you have questions" — strip those. Pilot Chat is task-focused. End on the last informative sentence.

## When you ran multiple tools

Don't describe each tool call. Synthesize the end result:

Wrong:
> "Önce get_levels çağırdım, sonra get_elements_by_category yaptım, sonra place_column'u 5 kez döndürdüm..."

Right:
> "5 kolonu Level 1 üzerinde (0,0)'dan başlayarak 3m aralıkla yerleştirdim. Type: Concrete-Rectangular-Column 300x300."

The user cares about the destination, not the journey.
