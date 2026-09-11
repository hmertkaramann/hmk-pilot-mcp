---
name: autocad-civil3d-wf-layers
description: Layer operations — listing, isolating, merging, freeze/thaw, and what breaks. Requires the HMK Pilot connector for AutoCAD / Civil 3D.
---

## Reach for the tool first

`list_layers`, `create_layer`, `set_current_layer`, `set_layer_properties`,
`isolate_layers`, `delete_layer` cover almost everything. Drop to code only
for bulk logic no tool expresses.

## Facts that change the answer

- A layer's state is several independent flags: On/Off, Frozen/Thawed,
  Locked/Unlocked, Plot/NoPlot. "Not visible" can mean any of the first two,
  and they behave differently — frozen layers are excluded from regeneration
  and selection, layers that are merely off are not.
- Layer 0 and Defpoints are special. Do not delete, rename or merge them.
- The CURRENT layer cannot be deleted or frozen. Switch first.
- A layer holding entities cannot be deleted until they are moved or erased —
  `delete_layer` will report this rather than silently dropping geometry.
- Xref-dependent layers (`xrefname|layername`) cannot be renamed or deleted in
  the host drawing. Say so; do not try to work around it.

## Merging

Merging is destructive and not selectively reversible: every entity moves to
the target layer and the source is removed. Count the affected entities and
confirm with that number before doing it — see safety_destructive.

## Isolating

`isolate_layers` changes visibility state, which the user will see and may not
know how to undo. Tell them how to restore ("tüm katmanları geri açmak için
söylemen yeterli") in the same reply.
