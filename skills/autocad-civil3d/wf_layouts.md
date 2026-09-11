---
name: autocad-civil3d-wf-layouts
description: Layouts, viewports, plotting and PDF output. Requires the HMK Pilot connector for AutoCAD / Civil 3D.
---

## Model space vs layouts

Geometry is drawn once, at full size, in **model space**. A **layout** (paper space) is a sheet: it holds a title block plus **viewports**, each a window onto model space at a fixed scale. "Pafta" in Turkish means the layout/sheet, not the drawing file.

A viewport's scale is what makes annotation sizes work out. Changing it is a viewport property, never a rescale of the model geometry — never `scale_entities` model space to fit a sheet.

## Tools

| Task | Call |
|---|---|
| What sheets exist? | `list_layouts` |
| Make a sheet | `create_layout(name=…)` |
| Put a window on it | `create_viewport(layout=…, …)` |
| Fix the scale | `set_viewport_scale(handle=…, scale=…)` |
| Produce PDF | `plot_to_pdf(layout=…, outputPath=…)` |

## Plotting

`plot_to_pdf` prints ONE layout per call. For "plot all sheets", call `list_layouts` first, then issue the plot calls — and say in the reply how many sheets were produced and where they went.

Writing files is not reversible in the user's eyes: if the target path already exists, confirm before overwriting (safety_destructive).

Common failure: plotting a layout that has no plot configuration assigned yields an empty or wrongly-sized PDF. If the output looks wrong, check the layout's page setup with `execute_autocad_code` (`LayoutManager` → `Layout.PlotSettingsName`) before re-plotting.

## Viewport scale reference

| Scale text | Ratio (paper : model, mm) |
|---|---|
| 1:1 | 1.0 |
| 1:10 | 0.1 |
| 1:20 | 0.05 |
| 1:50 | 0.02 |
| 1:100 | 0.01 |
| 1:200 | 0.005 |
| 1:500 | 0.002 |

Civil sheets are usually 1:200 / 1:500 for plans and 1:100 vertical / 1:1000 horizontal (exaggerated) for profiles — see wf_sections for section-view sheets.

## Batched work

Anything touching more than a couple of sheets — renaming, re-scaling every viewport, stamping a revision into each title block — belongs in ONE `execute_autocad_code` call that iterates `LayoutManager.Current.LayoutDictionary` and prints a compact result table. Chaining per-sheet tool calls is the slow path the user feels.
