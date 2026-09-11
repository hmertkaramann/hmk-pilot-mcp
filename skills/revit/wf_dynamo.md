---
name: revit-wf-dynamo
description: Driving Dynamo live - graph authoring, running, Python, and the two mistakes that break it. Requires the HMK Pilot connector for Revit.
---

Ten tools drive the Dynamo running inside this Revit: `dynamo_status`, `dynamo_nodes`, `dynamo_graph`, `dynamo_selection`, `dynamo_author`, `dynamo_python`, `dynamo_run`, `dynamo_open`, `dynamo_save`, `dynamo_close`. They act on the graph the user has open, and the user watches it happen.

## When the user points instead of naming

"Fix this node", "why is this red", "what does this give me", "connect these two" — the user is pointing at their canvas. Call `dynamo_selection`. Never ask for a node guid: it is not something they can read off the screen.

## Which one — a graph, or an answer?

Decide this first, because the two paths barely overlap:

- The user wants something they **keep** — a routine they will run again, hand to a colleague, put in the project folder. Build a real graph with `dynamo_author` and leave them a file with `dynamo_save`. A canvas of unsaved nodes is not a deliverable.
- The user wants an **answer** — "how many of these are there", "does this hold", a quick probe before deciding. `dynamo_python` runs a script and returns its value in one call. Don't build a graph to answer a question.

When it is genuinely unclear, prefer the graph: an unwanted graph is a node the user deletes, a missing one is work they have to redo by hand.

## Order of operations

1. `dynamo_status` — always first. Dynamo may not be installed, may be closed, or may be bound to a different Revit document. Pass `autoStart=true` only when the user actually wants to work in Dynamo: it takes 7-45 seconds and Revit is busy throughout.
2. `dynamo_nodes` — before authoring anything. Never guess a node name (see below).
3. `dynamo_author` — build/edit in ONE batch.
4. `dynamo_graph` — read results, values, and node error text.
5. `dynamo_save` — hand the graph over as a file.

If Dynamo was started with `showUi=false`, close it with `dynamo_close` when the work is done: a Dynamo with no window has nothing to run its own teardown and keeps its engine alive for the rest of the session.

## Mistake 1: creationName is not the name Dynamo shows

`add_node` takes **creationName**, the internal string — not the label on the node.

| What the user says / sees | creationName to pass |
|---|---|
| All Elements of Category | `DSRevitNodesUI.ElementsOfCategory` |
| Categories | `DSRevitNodesUI.Categories` |
| All Elements of Class | `DSRevitNodesUI.ElementsOfType` |
| Element By Id | `DSRevitNodesUI.ElementById` |
| Watch | `CoreNodeModels.Watch` |
| Number | `CoreNodeModels.Input.DoubleInput` |
| Python Script | `PythonNodeModels.PythonNode` |
| Code Block | `Code Block` |

On a stock install **1575 of 1576 nodes** have a creationName different from their display name. The single exception is Code Block — which is exactly the node you would reach for when testing by hand, so "it worked once" proves nothing. Always look it up with `dynamo_nodes` first; it returns creationName as the first field.

`dynamo_author` validates every creationName against the live library and rejects the WHOLE batch if one is wrong, without touching the graph. A rejection is cheap; act on the message rather than retrying blind.

## Mistake 2: the Python engine name depends on the Dynamo version

- Dynamo 2.x / 3.x (Revit 2023-2026): `CPython3`
- Dynamo 4.x (Revit 2027): `PythonNet3` — **CPython3 was removed**

Ask for either; the tool translates to whatever the running build has and tells you it did. Omit `engine` entirely to leave the node on Dynamo's default.

Opening an older graph on Revit 2027 converts its Python nodes automatically and Dynamo writes a backup file. `dynamo_open` reports the engines it found, so mention it if the user cares about the file changing.

## Authoring: one batch, not one call per node

```json
[
  {"op":"add_node","creationName":"DSRevitNodesUI.Categories","alias":"cats","x":60,"y":100},
  {"op":"add_node","creationName":"DSRevitNodesUI.ElementsOfCategory","alias":"els",
   "connectFrom":"cats","fromPort":0,"toPort":0,"x":400,"y":100},
  {"op":"add_node","creationName":"CoreNodeModels.Watch","alias":"out",
   "connectFrom":"els","fromPort":0,"toPort":0,"x":760,"y":100},
  {"op":"set_value","node":"out","property":"Name","value":"Result"}
]
```

- `alias` names a node so later ops can refer to it. Anywhere a node is referenced you may use an alias from this batch or a real guid from `dynamo_graph`.
- Editing an EXISTING graph uses the same batch: `disconnect` removes a wire (name where it LANDS — `to` + `toPort` — since an input takes exactly one), `move` repositions a node, `freeze` is Dynamo's do-not-evaluate switch.
- **`freeze` is the safety tool.** Building a graph that WRITES to the model? Freeze the write node, build and check the rest, then thaw it when the user says go. Manual run mode stops accidental runs; freezing lets the graph run with the dangerous part inert.
- `connectFrom` on `add_node` creates and wires in one step — prefer it over a separate `connect` op, which is a two-phase protocol that can leave a half-made wire.
- Lay nodes out on a grid (x steps of ~340, y rows of ~260). The user sees this; overlapping nodes look broken.
- For a Code Block: `add_node` with creationName `Code Block`, then `set_value` with property `Code`.
- Every batch applies as N undo steps, not one. Say so if you built something large.

## Python: one call, not four

`dynamo_python` writes a script into a reusable node called **HMK Pilot Scratch**, runs it, and returns the value — one call.

```
dynamo_python(code="OUT = len([e for e in IN[0]])")
```

- Assign the result to `OUT`, exactly as in a hand-written Python node.
- `clr` / RevitAPI imports work normally, so this reaches anything the Revit API reaches.
- Engine names are version-dependent and translated for you: ask for `CPython3` or `PythonNet3`, get whichever this Dynamo has.
- To edit an EXISTING Python node, read it first with `dynamo_graph nodeId=<guid>` — that returns the script whole — then pass the same `nodeId` to `dynamo_python`. Rewriting a script you never read is how working code gets thrown away.
- `inputFrom=<node guid>` feeds `IN[0]` from that node's output. That is how you run a script against what the user's existing graph already produces, instead of re-fetching the same elements a second way.
- The node stays on the canvas so the user can see and keep what ran. Pass `removeAfter=true` for a genuinely throwaway probe.

## Running is asynchronous — but you do not have to poll

`dynamo_run` fires the graph and by default returns a `runId` straight away; `dynamo_graph` then reports `lastRun.status` (`completed` / `running` / `unknown`) plus every node's value.

Simpler: pass `waitSeconds` (e.g. `dynamo_run(waitSeconds=60)`) and it returns when the graph is done. **Revit is not blocked while it waits** — the wait happens off Revit's thread. `dynamo_python` does the same thing internally, which is why it can return a value.

If a wait times out, say so and call `dynamo_graph` in a moment rather than re-running: firing a second run on a graph that writes to Revit doubles what it wrote.

## Reading failures

`dynamo_graph` returns each node's `state`, `isInErrorState` and the actual warning/error text in `messages`. Diagnose from that directly — there is no need to ask the user to read the red node out to you. Nodes carrying errors are always included even when a large graph is truncated.

Python nodes also return `script` and `pythonEngine`. In a graph-wide read the script is capped; `dynamo_graph nodeId=<guid>` returns it complete. A Python node that failed has its traceback in `messages`.

## Safety

- A graph that touches Revit MODIFIES THE MODEL when it runs. Say what a graph will do before running one the user did not write.
- `dynamo_open` replaces what is open and refuses when the current graph has unsaved changes. Do not pass `discardUnsavedChanges=true` unless the user has said the open graph can be thrown away.
- Graphs are opened in Manual run mode on purpose, so nothing evaluates on arrival. Run it deliberately with `dynamo_run`.
- Writing Python is arbitrary code execution: it is gated by the same flag as `execute_revit_code`, scanned against a Python deny-list, and written to the audit log in full.

## Handing the graph over

`dynamo_save` writes a real `.dyn` the user owns, opens, edits and shares. Called bare it saves over the file the graph came from; with `filePath` it saves somewhere new, and it refuses to write over a DIFFERENT existing file unless you pass `overwrite=true`. (`dynamo_author` also takes `savePath`, which does the same thing at the end of a batch.)

That is usually the point — prefer leaving them a saved graph over a canvas full of unsaved nodes.
