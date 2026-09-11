---
name: navisworks-wf-viewpoints
description: Saved viewpoints, camera, section planes and generating images. Requires the HMK Pilot connector for Navisworks.
---

## What a saved viewpoint holds

Camera position AND visibility state — hidden items, overrides, section
planes. That is why restoring one can change what is visible, and why
overwriting someone's viewpoint loses more than a camera angle. Confirm before
overwriting.

## Camera

`get_camera` / `set_camera` move the view. Restoring a saved viewpoint is
usually what the user means by "şu görünüme dön" — prefer it over
reconstructing a camera by hand.

## Section planes

Clipping planes cut the model for inspection. They are part of the viewpoint
state, so a viewpoint saved with clipping on restores with it on. Tell the user
when you leave clipping enabled, or they will report the model as "missing
half its geometry" later.

## Images

`generate_image` renders the current view. Frame the shot first — isolate or
zoom to what matters — and say what the image shows. An image of the whole
federation from the default camera answers nothing.

For a specific clash, `get_clash_image` is better: it frames the pair for you.

## Reporting

Report the viewpoint NAME and folder so the user can find it in the tree.
