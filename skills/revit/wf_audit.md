---
name: revit-wf-audit
description: Model audit / warnings cleanup workflows. Requires the HMK Pilot connector for Revit.
---

Revit projects accumulate warnings, orphans, duplicates. The user asks "modeli temizle" / "audit this model" — handle systematically.

## Trigger phrases
- "Uyarılar nedir", "show warnings", "kaç uyarı var"
- "Modeli temizle", "audit the model", "audit yap"
- "Duplicate'leri sil", "find duplicates"
- "Orphan element'ler var mı", "any orphans"
- "Boş element", "kullanılmayan tip"

## Workflow A — Warning overview

Trigger: "Bu projede ne kadar uyarı var" / "show warnings"

```
1. get_warnings(sampleSize=20)
2. Read the response: totalCount + groupedByMessage[] + sample[]
3. Reply with:
   - Total warning count
   - Top 3-5 groups by count (the groupedByMessage array is pre-sorted)
   - A line about which group to tackle first
4. Don't paste the raw JSON.
```

Format example:
```
1247 uyarı var. En kalabalık gruplar:
- Areas not in a properly enclosed region: 412
- Highlighted walls overlap: 218
- Identical instances in same place: 184
- ...

Geometri kaynaklı duplicate'lerden başlamak en hızlı kazançlı olur.
```

## Workflow B — Drill into one warning group

Trigger: "Duplicate'leri çöz" → after the overview

```
1. get_warnings(sampleSize=50)
2. Filter sample[] to the group user picked
3. Each sample has elementIds → user can decide manually OR
4. execute_revit_code: get FailureMessage objects, extract elementIds, optionally delete one of each pair
   (deletion is destructive — apply safety_destructive rules)
```

## Workflow C — Find duplicate element instances at same point

Trigger: "Üst üste binen elementler" / "duplicate instances"

```
1. get_warnings → look for "Identical instances in same place" group
2. If not in warnings list, use find_elements_at_point on suspect coords + radius
3. Identify pairs, ask user if they want one of each deleted (safety_destructive)
```

Or execute_revit_code: loop a category, group by LocationPoint with tolerance → duplicates.

## Workflow D — Unused types / orphan parameter values

This is **DataBuild Audit** territory. Redirect (ref_hmk_modules):
> "Model audit için DataBuild modülü kullan. Audit tab'da unused type, orphan element, parameter usage statistics gibi metrikler çıkar. Delete'e basmak gerekebilir 2-3 kez (orphan zinciri var)."

If user insists on chat-based, use execute_revit_code:
- Unused family symbols: FamilySymbol.IsActive==false instances → no usage in project
- Project parameters never set: scan all elements, count which parameter values are null

## Workflow E — Performance / view filter complexity

Trigger: "Model neden yavaş" → diag_perf_warnings template (it's a separate diagnostic flow)

## Workflow F — Warnings as a metric for QA

Trigger: "Modeli teslim için hazır mı?"

```
1. get_warnings → total count
2. get_links → external links count
3. get_workset_info (if workshared) → un-synced changes
4. get_project_info → unit system, project info filled
5. Reply with a QA scorecard:
   "Teslim hazırlık raporu:
   - 1247 uyarı (önerilen <500 — geometri grubu inceleyelim)
   - 4 link, hepsi up-to-date
   - 12 workset
   - Project Number: BOŞ ⚠
   - Client Name: BOŞ ⚠
   ..."
```

For final export → redirect to HMK Deliver (ref_hmk_modules).

## What NOT to do

- Don't auto-delete duplicate warnings without confirming with the user — safety_destructive applies.
- Don't enumerate all 1200 warnings into chat — too much noise, the user reads pattern not list.
- Don't claim "model is clean" with non-zero warning count.

## execute_revit_code recipe — quick warning summary

When the user wants a custom slice (e.g. only warnings on selected elements), use execute_revit_code with `Document.GetWarnings()`. Each FailureMessage has GetFailingElements() + GetDescriptionText() + Severity.
