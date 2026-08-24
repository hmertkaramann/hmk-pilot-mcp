<p align="center">
  <a href="https://hmktools.com/revit-mcp">
    <img src="assets/banner.png" alt="HMK Pilot — MCP for Revit, AutoCAD, Civil 3D & Navisworks" width="820">
  </a>
</p>

<h1 align="center">HMK Pilot — MCP for Revit, AutoCAD, Civil 3D &amp; Navisworks</h1>

<p align="center">
  Drive your Autodesk apps from Claude, in plain language, over the Model Context Protocol.
</p>

<p align="center">
  <a href="https://hmktools.com"><img alt="Website" src="https://img.shields.io/badge/website-hmktools.com-021E45"></a>
  <a href="https://hmktools.com/revit-mcp"><img alt="Guide" src="https://img.shields.io/badge/guide-Revit%20MCP-5b9bf0"></a>
  <img alt="Apps" src="https://img.shields.io/badge/Autodesk-Revit%20%C2%B7%20AutoCAD%20%C2%B7%20Civil%203D%20%C2%B7%20Navisworks-021E45">
  <img alt="License" src="https://img.shields.io/badge/license-commercial%20(free%20trial)-informational">
  <a href="https://glama.ai/mcp/servers/hmertkaramann/hmk-pilot-mcp"><img alt="Listed on Glama" src="https://img.shields.io/badge/listed%20on-Glama-5b9bf0"></a>
</p>

---

**HMK Pilot** runs a secure local MCP server **inside** your Autodesk application
and exposes **330+ real actions**, so an AI assistant like **Claude Desktop** or
**Claude Code** can read and edit the *live* model in plain language — not a stale
export.

> Part of **[HMK Tools](https://hmktools.com)**.
> Full guide: **[hmktools.com/revit-mcp](https://hmktools.com/revit-mcp)** ·
> Comparison: **[vs other Revit MCP options](https://hmktools.com/revit-mcp-comparison)**

## What it does

- **Revit** — 100+ actions: select elements, edit parameters, create sheets, tag
  views, run clash checks, export IFC/NWC, drive Dynamo, and more. Also a
  built-in chat panel.
- **AutoCAD** — 85+ actions: draw & modify geometry, manage layers, blocks &
  xrefs, layouts and plotting to PDF.
- **Civil 3D** — 50+ additional actions: alignments, profiles, surfaces,
  corridors, pipe networks, sample lines & section views, COGO points.
- **Navisworks** — 90+ actions: search sets, clash detection, TimeLiner 4D,
  quantification, viewpoints, federation (append / merge / refresh / publish).
- Runs **100% locally** (bound to `localhost`) — your models and drawings never
  leave your machine.
- **One-click** setup for Claude Desktop and Claude Code (writes the config and
  preserves any other MCP servers you already have).
- Optional Roslyn escape hatch — `execute_revit_code`, `execute_autocad_code`,
  `execute_civil_code`, `execute_navisworks_code` (toggle off in
  Settings → Security).

### Built-in domain knowledge

The connectors ship a knowledge base the assistant can pull on demand through
`get_knowledge_template` — verified API notes, workflow recipes, unit tables and
safety rules per application. It means the assistant reaches for a checked
pattern instead of guessing an API member and burning a round on the error.

It also **learns from your machine**. When a code call fails and a later one
succeeds, the working snippet and the error it stopped hitting are saved
locally, then offered back on similar work — including in a *later* session, and
across both the MCP server and the in-app chat panel. Nothing about this leaves
your computer.

## Requirements

- Windows (x64)
- Autodesk **Revit 2023–2027**, **AutoCAD / Civil 3D 2021–2027**, and/or
  **Navisworks Manage / Simulate 2023–2027**
- **Claude Desktop** or **Claude Code** (bring your own Claude account)
- An **HMK Tools license** — free 30-day trial, no card

> No Node.js, no Python, no separate runtime. The local bridge is a single
> native executable that ships with the add-in.

## Install

1. Download & install **HMK Tools** from **[hmktools.com](https://hmktools.com)**.
2. In Revit / AutoCAD / Civil 3D / Navisworks, open the **HMK Tools → Pilot AI**
   ribbon and toggle the **Connector** on.
3. Click **Connect Claude Desktop** (or **Connect Claude Code**) — done.

## Learn more

- How Revit MCP works: <https://hmktools.com/revit-mcp>
- AutoCAD & Civil 3D MCP: <https://hmktools.com/autocad-mcp>
- Navisworks + AI: <https://hmktools.com/navisworks-ai>
- Compare the options: <https://hmktools.com/revit-mcp-comparison>
- AI in Revit (guide): <https://hmktools.com/blog/ai-in-revit>
- Revit connector: <https://hmktools.com/toolbox/pilot-ai-revit>
- AutoCAD & Civil 3D connector: <https://hmktools.com/toolbox/pilot-cad>
- Navisworks connector: <https://hmktools.com/toolbox/navis-pilot>

## More from HMK Tools

HMK Pilot is one part of a 25+ tool suite for Revit, AutoCAD, Civil 3D and
Navisworks — parameter automation, rebar, modeling, batch export and more.
Browse them all at **[hmktools.com/toolbox](https://hmktools.com/toolbox)**.

## License

Commercial — included with an **HMK Tools** license (free 30-day trial). This
repository is the public overview for the HMK Pilot MCP server; the server itself
ships inside the HMK Tools add-in.

---

<sub>Revit, AutoCAD, Civil 3D and Navisworks are trademarks of Autodesk, Inc. HMK
Tools is not affiliated with, endorsed by, or sponsored by Autodesk.</sub>
