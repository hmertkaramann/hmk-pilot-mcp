---
name: navisworks-ux-reporting
description: Output formatting rules — how to summarize tool results back to the user. Requires the HMK Pilot connector for Navisworks.
---

The user sees your message, not the tool JSON. Format for a person reading in a docked pane.

Example phrasings are Turkish to show STRUCTURE and LENGTH, not language. Reply in whatever language the user wrote in.

## Result size → format

| Size | Format |
|---|---|
| nothing found | One sentence: "Bu ölçütle eşleşen öğe yok." |
| 1 item | One sentence with the key facts and the path: "Duct (0/3/17, Revit Type: Rectangular Duct)." |
| 2–5 | Short bullets, each ≤ 80 chars |
| 6–20 | Bullets or a small table |
| 21–100 | Count + top-5 sample + "İstersen tam listeyi verebilirim." |
| 100+ | Count and distribution only: "4.812 öğe: 2.104 Duct, 1.330 Pipe, …" |

## Always include a count

The count is the user's trust hook — and in a federation it is often the
answer itself.

## Identity is the INDEX PATH

Report the path (`0/3/17`) when the user may want to act on the item again.
Paths are positional and valid only while the tree is unchanged — say so if
the scene was reloaded between turns.

## Group by source file when it helps

A coordinator thinks in files, not in tree nodes. "412 çakışmanın 280'i
MEP.nwc ile STR.nwc arasında" is worth more than a flat list.

## Don't echo JSON, don't show code

The user cares about the finding, not the traversal.

## Verify inside the same call

A count you report must be a count you MEASURED, in the same script that made
the change. A separate verification call is a whole extra round.

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

## Markdown discipline

Headings on their own line with blank lines around. Bullets start with `- `.
Two to four short paragraphs, or one list. No padding, no "hope this helps".
