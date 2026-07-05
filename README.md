# HMK Pilot — MCP for Revit, AutoCAD & Civil 3D

Drive **Autodesk Revit, AutoCAD, and Civil 3D** from an AI assistant
(**Claude Desktop** / **Claude Code**) over the **Model Context Protocol (MCP)**.

HMK Pilot runs a secure local MCP server **inside** your Autodesk application and
exposes **190+ real actions**, so your LLM can read and edit the *live* model in
plain language — not a stale export.

> Part of **[HMK Tools](https://hmktools.com)**.
> Full guide: **[hmktools.com/revit-mcp](https://hmktools.com/revit-mcp)**

## What it does

- **Revit** — 90+ actions: select elements, edit parameters, create sheets, tag
  views, run clash checks, export IFC/NWC, and more. Also a built-in chat panel.
- **AutoCAD** — ~70 actions: draw & modify geometry, manage layers & blocks,
  plot to PDF.
- **Civil 3D** — ~30 actions: query alignments, surfaces, pipe networks,
  corridors, COGO points.
- Runs **100% locally** (bound to `localhost`) — your models and drawings never
  leave your machine.
- **One-click** setup for Claude Desktop and Claude Code (writes the config and
  preserves any other MCP servers you already have).
- Optional Roslyn `execute_autocad_code` / `execute_civil_code` escape hatch
  (can be turned off in Settings → Security).

## Requirements

- Windows (x64)
- Autodesk **Revit 2023–2027** and/or **AutoCAD / Civil 3D 2021–2026**
- **Node.js** (v20 LTS) — for the local bridge
- **Claude Desktop** or **Claude Code** (bring your own Claude account)
- An **HMK Tools license** — free 30-day trial, no card

## Install

1. Download & install **HMK Tools** from **[hmktools.com](https://hmktools.com)**.
2. In Revit / AutoCAD / Civil 3D, open the **HMK Tools → Pilot AI** ribbon and
   toggle the **Connector** on.
3. Click **Connect Claude Desktop** (or **Connect Claude Code**) — done.

## Learn more

- How Revit MCP works: <https://hmktools.com/revit-mcp>
- AI in Revit (guide): <https://hmktools.com/blog/ai-in-revit>
- Revit connector: <https://hmktools.com/toolbox/pilot-connector>
- AutoCAD & Civil 3D connector: <https://hmktools.com/toolbox/pilot-cad>
- In-Revit chat panel: <https://hmktools.com/toolbox/pilot-chat>

## License

Commercial — included with an **HMK Tools** license (free 30-day trial). This
repository is the public overview for the HMK Pilot MCP server; the server itself
ships inside the HMK Tools add-in.

---

Revit, AutoCAD and Civil 3D are trademarks of Autodesk, Inc. HMK Tools is not
affiliated with, endorsed by, or sponsored by Autodesk.
