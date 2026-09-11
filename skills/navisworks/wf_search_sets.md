---
name: navisworks-wf-search-sets
description: Finding items by property, and turning a find into a set that keeps working. Requires the HMK Pilot connector for Navisworks.
---

## Properties are category + property + value

Every item carries PROPERTIES grouped into CATEGORIES — "Item", "Element",
"Revit Type", and whatever the source authoring tool exported. A search names
all three parts. Guessing the category is the usual reason a search returns
nothing; `list_property_categories` on a known-good item is the cheap way to
learn the real names before searching the whole federation.

## Recipe

1. Take ONE representative item the user points at (or find one).
2. Read its property categories and values.
3. Build the search from the exact category and property names you just saw.
4. Run it, report the count, and show a handful of samples.

That is four steps but usually two rounds — steps 1-2 are one tool call and
3-4 are the next.

## Selection set vs search set

- **Selection set** — a frozen list of items. Correct when the user means
  "these specific ones, as they are now".
- **Search set** — stores the QUERY and re-evaluates every time it is opened.
  Correct when the user means "everything that matches this rule", and the
  right default for coordination work where the model keeps changing.

If the user says "kaydet" without specifying, a search set is almost always
what they actually want. Say which one you made.

## Reporting

Give the count and group it by source file — a coordinator thinks in files.
Report the set NAME back so they can find it in the tree.
