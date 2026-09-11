---
name: navisworks-wf-federation
description: Appending, refreshing, merging and publishing the federated scene. Requires the HMK Pilot connector for Navisworks.
---

## What a Navisworks scene is

A FEDERATION: several source files (RVT, IFC, DWG, NWC…) appended into one
tree. Each appended file is a node whose index path has one component. The
authoring files stay where they are — Navisworks holds references, not copies,
until an NWD is published.

## Append vs merge vs refresh

- **Append** adds a file as another node. The usual operation.
- **Merge** combines into the current scene without keeping the separate node
  structure — rarely what a coordinator wants; ask before assuming it.
- **Refresh** re-reads the appended files from disk. This is the answer to
  "the model looks out of date", and it is cheap compared to re-appending.

## NWF vs NWD

- **NWF** stores the file LIST plus your sets, viewpoints, clash tests and
  TimeLiner data — it re-reads the sources every time it opens. This is the
  working file.
- **NWD** bakes the geometry in. It is a snapshot for sharing, and it is why
  `doc.Models.Count` returns 1 on one (see code_navis_api).

Publishing an NWD of a large federation takes minutes and writes a large file.
Confirm the path before overwriting anything.

## Reporting the scene

List source files by name with their index, not just a count — the index is
the first component of every path the user will act on afterwards. Cap a long
list and offer the rest rather than dumping 158 lines.
