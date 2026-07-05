<p align="center">
  <a href="https://hmktools.com/revit-mcp">
    <img src="assets/banner.png" alt="HMK Pilot — MCP for Revit, AutoCAD & Civil 3D" width="820">
  </a>
</p>

<h1 align="center">HMK Pilot — MCP for Revit, AutoCAD &amp; Civil 3D</h1>

<p align="center">
  Drive your Autodesk apps from Claude, in plain language, over the Model Context Protocol.
</p>

<p align="center">
  <a href="https://hmktools.com"><img alt="Website" src="https://img.shields.io/badge/website-hmktools.com-021E45"></a>
  <a href="https://hmktools.com/revit-mcp"><img alt="Guide" src="https://img.shields.io/badge/guide-Revit%20MCP-5b9bf0"></a>
  <img alt="Apps" src="https://img.shields.io/badge/Autodesk-Revit%20%C2%B7%20AutoCAD%20%C2%B7%20Civil%203D-021E45">
  <img alt="License" src="https://img.shields.io/badge/license-commercial%20(free%20trial)-informational">
</p>

---

**HMK Pilot** runs a secure local MCP server **inside** your Autodesk application
and exposes **190+ real actions**, so an AI assistant like **Claude Desktop** or
**Claude Code** can read and edit the *live* model in plain language — not a stale
export.

> Part of **[HMK Tools](https://hmktools.com)**.
> Full guide: **[hmktools.com/revit-mcp](https://hmktools.com/revit-mcp)** ·
> Comparison: **[vs other Revit MCP options](https://hmktools.com/revit-mcp-comparison)**

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
  (toggle off in Settings → Security).

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
- Compare the options: <https://hmktools.com/revit-mcp-comparison>
- AI in Revit (guide): <https://hmktools.com/blog/ai-in-revit>
- Revit connector: <https://hmktools.com/toolbox/pilot-connector>
- AutoCAD & Civil 3D connector: <https://hmktools.com/toolbox/pilot-cad>

## More from HMK Tools

HMK Pilot is one part of a 25+ tool suite for Revit, AutoCAD and Civil 3D —
parameter automation, rebar, modeling, batch export and more. Browse them all at
**[hmktools.com/toolbox](https://hmktools.com/toolbox)**.

## License

Commercial — included with an **HMK Tools** license (free 30-day trial). This
repository is the public overview for the HMK Pilot MCP server; the server itself
ships inside the HMK Tools add-in.

---

<sub>Revit, AutoCAD and Civil 3D are trademarks of Autodesk, Inc. HMK Tools is not
affiliated with, endorsed by, or sponsored by Autodesk.</sub>
