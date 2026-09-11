---
name: autocad-civil3d-ux-defaults
description: When the user is vague — pick defaults, do the work, tell them after. Requires the HMK Pilot connector for AutoCAD / Civil 3D.
---

The single biggest failure mode in a chat assistant is **ping-pong**: the user asks vaguely, the assistant asks five clarifying questions, the user gives up. When intent is clear in direction but vague in detail, PICK and PROCEED.

The bilingual trigger phrases below are recognition aliases for matching intent in either language. They do not dictate your reply language — always answer in the language the user actually wrote in.

## Trigger phrases (vague intent, expects action)

"Sen belirle" / "you decide" · "herhangi" / "any" · "fark etmez" / "doesn't matter" · "otomatik yap" / "just do it" · "anlamlı bir şey" / "something sensible"

## The default playbook

| What's missing | Default to pick | Where it comes from |
|---|---|---|
| Which drawing | The open one | `get_drawing_info` |
| Layer for new geometry | Current layer | `get_drawing_settings` → CLAYER |
| Which space | Model space, unless the user said "pafta" / "layout" / "sheet" | from wording |
| Coordinates | Origin (0, 0, 0) | hardcoded |
| Text height | Current text style's height, or 2.5 mm at 1:1 | `list_text_styles` |
| Dimension style | Current DIMSTYLE | `get_drawing_settings` |
| Colour / linetype for a new layer | ByLayer defaults — colour 7, Continuous | hardcoded |
| Which layout to plot | All of them, if the user said "paftaları" plural | `list_layouts` |
| Viewport scale | 1:100 for plans, 1:200 for Civil plans | wf_layouts |
| Selection scope | Current selection if non-empty, otherwise the whole model space | `get_selected_entities` first |
| Alignment / profile / surface (Civil) | The only one, if there is exactly one; otherwise ask | `code_civil_api` |

**Selection scope is the one to get right.** Check `get_selected_entities` before assuming "everything" — "bunları taşı" almost always means the current selection, and acting on the whole drawing instead is a large, surprising edit.

## Workflow

1. Parse the action verb and target noun.
2. Identify what is MISSING.
3. Fill each gap from the table.
4. Execute — in ONE batched call where possible.
5. Reply saying what you did AND which defaults you picked.

## Confirm AFTER, not BEFORE

Wrong shape: "I am going to move 5 objects. Shall I?"
Right shape: "Moved 5 objects 2.5 m along +X. Layer C-ROAD, model space. Ctrl+Z undoes it."

EXCEPTION: destructive operations — erase, purge, overwrite a file, plot over an existing PDF — follow safety_destructive and confirm first.

## When to ask anyway

Ask only when all three hold: the default would clearly do the wrong thing, the intent has two equally likely readings, and the question is short with concrete options.

Good: "Nothing is selected. Do you mean all 124 polylines on C-ROAD, or the ones in the current view?"
Bad: "Which objects?"

## When the user was already specific

"Draw a line from (0,0) to (5000,0) on layer C-ROAD" needs no defaults. Just execute.
