---
name: autocad-civil3d-ux-reporting
description: Output formatting rules — how to summarize tool results back to the user. Requires the HMK Pilot connector for AutoCAD / Civil 3D.
---

The user sees your message, not the tool JSON. Format for a person reading in a docked panel.

Example phrasings below happen to be Turkish; they illustrate STRUCTURE and LENGTH, not language. Render the equivalent in whatever language the user wrote in.

## Result size → format

| Size | Format |
|---|---|
| nothing found | One sentence: "Bu çizimde hiç X yok." |
| 1 item | One sentence with the key facts and the handle: "Polyline (handle 2A4, layer C-ROAD) 124.5 m." |
| 2–5 | Short bullets, each ≤ 80 chars |
| 6–20 | Bullets or a small table — whichever reads faster |
| 21–100 | Count + top-5 sample + "İstersen tam listeyi verebilirim." |
| 100+ | Count and distribution only: "1.284 nesne: 612 LINE, 341 LWPOLYLINE, …" |

## Always include a count

The count is the user's trust hook. "47 nesne taşındı" — not "taşındı".

## Identity is the HANDLE

Report the hex handle when the user may want to act on the object again. Never invent one, never reformat one a tool gave you.

## Don't echo JSON, don't show code

Wrong: `Tool returned: {"success":true,"handle":"2A4",...}`
Wrong: "Ran `foreach (ObjectId id in ms) ...` and got 47."
Right: "C-ROAD katmanında 47 polyline var, toplam 3.2 km."

The user cares about the destination, not the journey — even when execute_civil_code did the work.

## Verify inside the same call

A count you report must be a count you MEASURED. Fold the check into the same script that made the change — the data is already there, so it costs nothing. A separate verification call is a whole extra round, and rounds are the latency the user feels.

> "100 kesit görünümü taşındı; yeniden ölçtüm, çakışan çift kalmadı."

Never claim a change you did not measure. "47 updated" must come from the result, not from the fact that you passed 47 handles.

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

Headings on their own line with a blank line around them. Bullets start with `- `. Bold as `**text**`, not glued to the next word. Two to four short paragraphs, or one list — no walls of text.

## Don't pad

No "Hope this helps", no "Let me know if you need anything else". End on the last informative sentence.
