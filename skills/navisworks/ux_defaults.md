---
name: navisworks-ux-defaults
description: When the user is vague — pick defaults, do the work, tell them after. Requires the HMK Pilot connector for Navisworks.
---

The single biggest failure mode in a chat assistant is **ping-pong**: the user asks vaguely, the assistant asks five clarifying questions, the user gives up. When intent is clear in direction but vague in detail, PICK and PROCEED.

The bilingual trigger phrases below are recognition aliases for matching intent in either language. They do not dictate your reply language — always answer in the language the user actually wrote in.

## Trigger phrases (vague intent, expects action)

"Sen belirle" / "you decide" · "herhangi" / "any" · "fark etmez" / "doesn't matter" · "otomatik yap" / "just do it" · "anlamlı bir şey" / "something sensible"

## The default playbook

| What's missing | Default to pick | Where it comes from |
|---|---|---|
| Which model | The whole open federation | `get_model_context` |
| Search scope | Everything, unless the user named a discipline or file | from wording |
| Selection scope | Current selection if non-empty, otherwise the search you were asked for | `get_selection` first |
| Clash test tolerance | 10 mm for hard clashes | wf_clash |
| Clash test type | Hard | wf_clash |
| Clash A / B selections | The two models or sets the user named; if they named one, ask for the other | wf_clash |
| Set name | Descriptive, from the query itself ("Ducts > 300mm") | from wording |
| Where a new set goes | Root of Selection Sets, unless a folder was named | `list_selection_sets` |
| Viewpoint name | The task it captures, plus the date the user gave | wf_viewpoints |
| Report format | HTML for clash reports, CSV for tabular data | wf_clash |
| Task type (TimeLiner) | Construct | wf_timeliner |

**Selection scope is the one to get right.** Check `get_selection` before assuming "everything" — "bunları gizle" almost always means the current selection, and hiding the whole model instead is a large, surprising change.

**Counts are not free here.** `get_model_context` deliberately omits the item count because counting a real federation takes seconds. If a number matters, call `count_items` — do not guess one, and do not silently skip it.

## Workflow

1. Parse the action verb and target noun.
2. Identify what is MISSING.
3. Fill each gap from the table.
4. Execute — in ONE batched call where possible.
5. Reply saying what you did AND which defaults you picked.

## Confirm AFTER, not BEFORE

Wrong shape: "I am going to create 4 search sets. Shall I?"
Right shape: "Created 4 search sets under Selection Sets: Ducts>300 (812 items), Pipes>150 (1,204), Cable Tray (96), Sprinkler (2,331)."

EXCEPTION: destructive operations — deleting a set, test or task, clearing results, clearing the scene, publishing or overwriting a file — follow safety_destructive and confirm first.

## When to ask anyway

Ask only when all three hold: the default would clearly do the wrong thing, the intent has two equally likely readings, and the question is short with concrete options.

Good: "There are two clash tests named ARCH vs MEP. Do you mean the one from 12 Aug (1,204 results) or the one from today (86)?"
Bad: "Which test?"

## When the user was already specific

"Run the ARCH vs MEP test and export the HTML report to C:\Reports" needs no defaults. Just execute.
