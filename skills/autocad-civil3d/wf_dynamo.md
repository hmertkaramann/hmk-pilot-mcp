---
name: autocad-civil3d-wf-dynamo
description: Driving Dynamo for Civil 3D live — authoring, running, Python, and the mistakes that break it. Requires the HMK Pilot connector for AutoCAD / Civil 3D.
---

Ten tools drive the Dynamo running inside this host: `dynamo_status`, `dynamo_open`, `dynamo_nodes`, `dynamo_graph`, `dynamo_selection`, `dynamo_author`, `dynamo_python`, `dynamo_run`, `dynamo_save`, `dynamo_close`. They act on the graph the user has open, and the user watches it happen.

## Which one — a graph, or an answer?

Decide this first, because the two paths barely overlap:

- The user wants something they **keep** — a routine they will run again, hand to a colleague, put in the project folder. Build a real graph with `dynamo_author` and leave them a file with `dynamo_save`. A canvas of unsaved nodes is not a deliverable.
- The user wants an **answer** — "how many of these are there", a quick probe before deciding. `dynamo_python` runs a script and returns its value in one call. Don't build a graph to answer a question.

## When the user points instead of naming

"Fix this node", "why is this red", "what does this give me", "connect these two" — the user is pointing at their canvas. Call `dynamo_selection`. Never ask for a node guid: it is not something they can read off the screen.

Dynamo here is **Dynamo for Civil 3D** — it ships with Civil 3D, not with plain AutoCAD. If `dynamo_status` reports it missing on a vanilla AutoCAD, say that plainly instead of trying to start it.

## Order of operations

1. `dynamo_status` — always first. Dynamo may not be installed, may be closed, or may be bound to a different drawing. Pass `autoStart=true` only when the user actually wants to work in Dynamo: it takes several seconds during which the host is busy.
2. `dynamo_nodes` — before authoring anything. Never guess a node name (below). Filter with the category prefix `Civil3D` or `AutoCAD` to cut the result set.
3. `dynamo_author` — build/edit in ONE batch.
4. `dynamo_graph` — read results, values, and node error text.
5. `dynamo_save` — hand the graph over as a file.
6. `dynamo_close` — **required after a UI-less start**, or Civil 3D will not shut down afterwards. Do not skip it when you were the one who started Dynamo.

## Mistake 1: creationName is not the name Dynamo shows

`add_node` takes **creationName**, the internal string — not the label on the node. This is true of essentially the whole library, so a lookup is not optional: call `dynamo_nodes` and use the `creationName` field it returns as the first value.

`dynamo_author` validates every creationName against the live library and rejects the WHOLE batch if one is wrong, without touching the graph. A rejection is cheap; act on the message rather than retrying blind.

The one node whose creationName equals its label is **Code Block** — which is exactly the node you would reach for when testing by hand, so "it worked once" proves nothing.

## Mistake 2: the Python engine name depends on the Dynamo version

- Dynamo 2.x / 3.x: `CPython3`
- Dynamo 4.x: `PythonNet3` — **CPython3 was removed**

Ask for either; the tool resolves it to whatever the running build actually has and reports in `pythonEngineWarnings` when it substituted. Omit `engine` entirely to leave the node on Dynamo's default.

## Authoring: one batch, not one call per node

```json
[
  {"op":"add_node","creationName":"<from dynamo_nodes>","alias":"a","x":60,"y":100},
  {"op":"add_node","creationName":"<from dynamo_nodes>","alias":"b",
   "connectFrom":"a","fromPort":0,"toPort":0,"x":400,"y":100},
  {"op":"set_value","node":"b","property":"Name","value":"Result"}
]
```

- `alias` names a node so later ops can refer to it. Anywhere a node is referenced you may use an alias from this batch or a real guid from `dynamo_graph`.
- `connectFrom` on `add_node` creates and wires in one step — prefer it over a separate `connect` op, which is a two-phase protocol that can leave a half-made wire.
- Lay nodes out on a grid (x steps of ~340, y rows of ~260). The user sees this; overlapping nodes look broken.
- For a Code Block: `add_node` with creationName `Code Block`, then `set_value` with property `Code`.
- Every batch applies as N undo steps, not one. Say so if you built something large.

## Python: one call, not four

`dynamo_python` writes a script into a reusable node called **HMK Pilot Scratch**, runs it, and returns the value — one call.

- Assign the result to `OUT`, exactly as in a hand-written Python node.
- `clr` / AutoCAD / Civil 3D imports work normally.
- Engine names are version-dependent and translated for you: ask for `CPython3` or `PythonNet3`, get whichever this Dynamo has.
- To edit an EXISTING Python node, read it first with `dynamo_graph nodeId=<guid>` — that returns the script whole — then pass the same `nodeId`. Rewriting a script you never read is how working code gets thrown away.
- `inputFrom=<node guid>` feeds `IN[0]` from that node's output — how you run a script against what the user's existing graph already produces, instead of re-fetching the same objects a second way.

## Editing a graph, not just growing one

The same `dynamo_author` batch edits what is already there: `disconnect` removes a wire (name where it LANDS — `to` + `toPort` — since an input takes exactly one), `move` repositions a node, `freeze` is Dynamo's do-not-evaluate switch.

**`freeze` is the safety tool.** Building a graph that WRITES to the drawing? Freeze the write node, build and check the rest, thaw it when the user says go. Manual run mode stops accidental runs; freezing lets the graph run with the dangerous part inert.
- The node stays on the canvas so the user can see and keep what ran. Pass `removeAfter=true` for a throwaway probe.

## Running is asynchronous — but you do not have to poll

`dynamo_run` fires the graph and by default returns a run id straight away; `dynamo_graph` then reports the last run's status plus every node's value.

Simpler: pass `waitSeconds` and it returns when the graph is done. **The host is not blocked while it waits** — the wait happens off the API thread. `dynamo_python` does the same thing internally, which is why it can return a value.

If a wait times out, say so and call `dynamo_graph` in a moment rather than re-running: firing a second run on a graph that writes to the drawing doubles what it wrote.

## Reading failures

`dynamo_graph` returns each node's state, error flag and the actual warning/error text in `messages`. Diagnose from that directly — there is no need to ask the user to read the red node out to you.

Python nodes also return `script` and `pythonEngine`. In a graph-wide read the script is capped; `dynamo_graph nodeId=<guid>` returns it complete. A Python node that failed has its traceback in `messages` — and note that Dynamo shows a Python exception as a **Warning**, not an Error.

Dynamo keeps a node's warning until the user dismisses it, so a node that failed once and then succeeded still carries the old text. `dynamo_python` separates the two: `messages` is this run's, `previousMessages` is a leftover.

## Safety

- A graph that touches the drawing MODIFIES IT when it runs. Say what a graph will do before running one the user did not write.
- `dynamo_open` replaces what is open and refuses when the current graph has unsaved changes. Do not discard unsaved work unless the user has said it can be thrown away.
- Writing Python is arbitrary code execution: it is gated by the same flag as `execute_autocad_code`, scanned against a Python deny-list, and written to the audit log in full.

## Handing the graph over

`dynamo_author` takes a save path and writes a real `.dyn` the user owns, opens, edits and shares. That is usually the point — prefer leaving them a saved graph over a canvas full of unsaved nodes.
