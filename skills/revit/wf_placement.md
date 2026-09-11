---
name: revit-wf-placement
description: Multi-step placement workflows — rows / grids of columns, walls, components. Requires the HMK Pilot connector for Revit.
---

When the user wants to place multiple elements following a pattern (a row of columns, a grid, a wall enclosure), don't ask 10 clarifying questions. Pick defaults, do it, report.

## Trigger phrases
- "X tane kolon yerleştir", "place X columns"
- "Şu noktalara duvar çek", "draw walls between these points"
- "Her aksta bir kolon", "one column per grid"
- "5 metrelik kareye kapat", "enclose in a 5m square"
- "Sıra halinde", "row of", "grid of"

## Default playbook (when user gave no specifics)

1. **Origin** = (0, 0, 0). Coords in feet.
2. **Level** = first level from get_levels (lowest elevation).
3. **Type** = first available type from get_elements_by_category(category, maxResults=1) → read its Type Name. If none, fall back to execute_revit_code with FilteredElementCollector.OfClass(typeof(FamilySymbol)).
4. **Spacing** = 3 meters (≈ 9.84 ft) for columns, 1 grid bay if grids exist.
5. **Wall height** = 3 m (~9.84 ft) unless user specified.

## Workflow A — N columns in a row (X axis)

Trigger: "5 tane kolon yerleştir" / "row of 5 columns"

```
1. get_levels                       → pick first
2. get_elements_by_category(category="OST_StructuralColumns", maxResults=1)
                                    → read the existing column's type name
                                    (fallback: leave columnType empty → place_column picks first)
3. Loop i = 0..N-1:
     place_column(x = i * 9.84, y = 0, levelName, columnType, structural=true)
4. Collect successful ids
5. Reply: "5 kolonu (0,0) — (12.3 m, 0) arasında 3m aralıkla yerleştirdim. Level: <level>, Type: <type>."
```

## Workflow B — Wall between two points

Trigger: "(0,0) ve (5,0) arasına duvar çek"

```
1. get_levels                       → first level
2. place_wall(x1, y1, x2, y2, heightFeet=9.84, levelName, structural=false)
3. If error → execute_revit_code fallback (Wall.Create with explicit Line)
4. Reply: "(0,0) — (5m, 0) arası 5 metrelik duvar (h=3m) — Type X, Level Y."
```

## Workflow C — N×M column grid

Trigger: "3x4 kolon grid'i" / "3 sıra 4 sütun kolon"

```
1. get_levels → first
2. existing column type lookup
3. Loop ix = 0..N-1, iy = 0..M-1:
     place_column(x = ix * 9.84, y = iy * 9.84, levelName, columnType, structural=true)
4. Reply: "12 kolonu 3×4 grid'inde (0,0)'dan başlayıp 3m aralıkla yerleştirdim."
```

## Workflow D — Walls around a rectangular footprint

Trigger: "5x5 m oda duvarı çek" / "enclose a 5×5 area"

```
1. get_levels → first
2. heightFeet from user or default 9.84
3. Four place_wall calls — corners (0,0), (5m, 0), (5m, 5m), (0, 5m) → close the loop
4. If any fails, execute_revit_code fallback with Wall.Create on a CurveLoop
5. Reply with the four wall ids and dimensions
```

## Workflow E — Place at picked XYZ points (multiple)

If the user wants to pick points first → that's NOT chat territory, redirect to **Place Family at XYZ** module (ref_hmk_modules). Pilot Chat can't drive Revit's pick gesture.

## Tuzaklar

- **"Sen belirle" + no params**: use defaults above, DO NOT ask. Apply ux_defaults policy.
- **"Sıra" vs "sütun" (TR)**: "sıra" = row = X axis, "sütun" = grid column = Y axis. Don't confuse with OST_Columns!
- **place_column with no levelName**: Tool's default is first level — OK to omit if you didn't query get_levels.
- **place_wall with structural=true**: That creates a structural wall, even on OST_Walls. Default false unless user said "structural" / "taşıyıcı".
- **Wall height is in feet** despite the param name `heightFeet` — already in feet. Don't double-convert.
- **3-3 attempts policy**: if place_column / place_wall fail 3 times for the same reason (e.g. "no matching type"), fall through to execute_revit_code with explicit FamilySymbol activation + NewFamilyInstance. Code recipe in code_transactions.

## What NOT to do

- Ask the user "which level?" → no, pick first level.
- Ask the user "which type?" → no, query the category, pick first.
- Ask the user "which coordinates?" → no, use origin + spacing default.
- Place ONE column and then ask "should I continue?" → no, do the whole batch.
