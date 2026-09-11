---
name: revit-wf-sheets
description: Sheet creation + viewport layout workflows. Requires the HMK Pilot connector for Revit.
---

Sheets are paper layouts. The workflow is always: create sheet → find views → place viewports → arrange.

## Trigger phrases
- "Yeni sayfa oluştur", "create a new sheet"
- "Bu view'ı bir sayfaya bas", "put this view on a sheet"
- "Her plan için ayrı sayfa", "one sheet per plan view"
- "Pafta düzeni yap", "lay out a sheet set"

## Workflow A — New sheet with a view

Trigger: "A101 sayfasını aç, Level 1 planını yerleştir"

```
1. create_sheet(sheetName="...", sheetNumber="A101", titleBlockType="")
   → returns sheetId
2. get_views_list(viewTypeFilter="FloorPlan") → find the level-1 plan, grab viewId
3. place_view_on_sheet(sheetId, viewId, x=0, y=0)
   → tool centers it on the sheet by default if x,y are 0
4. (optional) move_viewport_on_sheet(viewportId, x, y) to tidy position
5. Reply: "A101 oluşturuldu, Level 1 planı yerleştirildi."
```

## Workflow B — Bulk: every plan view to its own sheet

Trigger: "Her plan view'a ayrı sayfa aç" / "one sheet per floor plan"

```
1. get_views_list(viewTypeFilter="FloorPlan", maxResults=200)
2. Loop each non-template plan:
     create_sheet(sheetNumber=auto-incremented like "A100", "A101", ..., sheetName=view.name)
     place_view_on_sheet(sheetId, viewId, x=0, y=0)
3. Reply with count + numbering scheme
```

Sheet numbering convention if user didn't specify: A100 → A199 for plans, A200 → 299 for elevations, A300 → 399 for sections, A500 → 599 for details. Standard arch numbering. If user provides a pattern (like "K-1", "K-2"), follow theirs.

## Workflow C — Reposition existing viewport

Trigger: "Viewport sağa kaydır" / "move the viewport"

```
1. The user often has the sheet active → get_current_view_info to confirm
2. get_selected_elements → if a viewport is selected, use its id
   OR execute_revit_code to find viewports on the active sheet
3. move_viewport_on_sheet(viewportId, x, y) — coords in SHEET SPACE, feet
```

## Workflow D — Schedule on a sheet

Trigger: "Schedule'ı sayfaya yerleştir"

```
1. get_views_list(viewTypeFilter="Schedule") → find schedule
2. place_view_on_sheet(sheetId, viewId, x, y)
3. Schedules behave like viewports for placement purposes.
```

## Workflow E — Replace title block on existing sheets

`create_sheet` accepts `titleBlockType`. For changing it on an existing sheet, use execute_revit_code:

```
sheet = doc.GetElement(new ElementId(sheetId)) as ViewSheet
// find the FamilySymbol of the desired title block...
// then create new instance / delete old / re-host
```

Tell the user this is fragile and DataBuild / a fresh export might be easier.

## Coordinate frames

| Tool | Frame | Origin |
|---|---|---|
| place_view_on_sheet (x,y) | Sheet space, feet | Bottom-left corner of sheet |
| move_viewport_on_sheet (x,y) | Sheet space, feet | Viewport center is placed at (x,y) |
| place_column, place_wall, find_elements_at_point | Model space, feet | Project origin (0,0,0) |

Don't mix these — placing a viewport at "model coordinates" will go off the paper.

## Common pitfalls

- **No title block defined** → `create_sheet` with titleBlockType="" may still work if a default title block family is loaded. If it fails, list available with execute_revit_code: `FilteredElementCollector(doc).OfCategory(BuiltInCategory.OST_TitleBlocks).OfClass(typeof(FamilySymbol))`.
- **Sheet number collision** → Revit refuses duplicate sheet numbers. Check existing via get_sheets_list before bulk creating.
- **View can only be on ONE sheet** → trying to place_view_on_sheet a view already placed elsewhere fails. Duplicate the view first (duplicate_view, mode="WithDetailing"), place the copy.
- **Schedule rotation** → schedules placed on sheets can be rotated; that's only available via execute_revit_code on the ScheduleSheetInstance.
