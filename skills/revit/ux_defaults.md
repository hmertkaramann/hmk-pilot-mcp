---
name: revit-ux-defaults
description: When the user is vague — pick defaults, do the work, tell them after. Requires the HMK Pilot connector for Revit.
---

The single biggest failure mode in chat-based AI assistants is **ping-pong**: user asks vaguely, AI asks 5 clarifying questions, user gives up. Pilot Chat is supposed to be a force multiplier — when intent is clear in direction but vague in detail, PICK and PROCEED.

The bilingual trigger phrases below ("Sen belirle" / "you decide") are recognition aliases for matching intent in either language. They do not dictate your reply language — always answer in the language the user actually wrote in.

## Trigger phrases (vague intent, expects action)

- "Sen belirle", "you decide", "you pick"
- "Herhangi", "any", "anything"
- "Fark etmez", "doesn't matter"
- "Bir tane", "one of them"
- "Otomatik yap", "just do it"
- "Anlamlı bir şey", "something sensible"

## The default playbook

When the user gives an ACTION but no specifics, pick from these defaults and execute. Tell them what you picked AFTER, in the reply.

| What's missing | Default to pick | Where it comes from |
|---|---|---|
| Level | First level by elevation | get_levels()[0] |
| FamilySymbol / type | First available | get_elements_by_category(maxResults=1) → read type from result |
| Coordinates | Origin (0, 0, 0) | hardcoded |
| Wall/column height | 3 meters (9.84 ft) | hardcoded |
| Wall/column count | 1 (single element) unless user said "row" / "grid" | from user wording |
| Spacing in a row/grid | 3 meters (9.84 ft) | hardcoded |
| Section depth | 10 ft (3 m) | hardcoded |
| Section height | 20 ft (6 m) | hardcoded |
| View scale | 50 (1:50) for plans/sections, 1 for drafting | hardcoded |
| Sheet title block | First available, or "" → tool's default | hardcoded |
| Sheet number | Auto-increment from existing range (A100, A101, ...) | get_sheets_list + scan |
| Sheet name | View's own name | from view |
| Filter parameter (which one?) | "Mark" | hardcoded |
| Tag head position | View center / element center | hardcoded |
| Phase Created | Current phase (last in get_phases) | get_phases() last |
| Workset | Active workset | doc.GetWorksetTable().ActiveWorksetId |

## Workflow

```
1. Parse the user's action verb and target noun.
2. Identify what's MISSING (level? type? coords?).
3. For each missing param, pick from the table above.
4. Execute the tool call(s) with the defaults filled in.
5. Reply: "<action did> with <defaults: level=X, type=Y, ...>".
```

## Example dialogues

### Good
> User: "Bir kolon yerleştir"
> Pilot: [calls get_levels, get_elements_by_category(OST_StructuralColumns, maxResults=1), then place_column(x=0, y=0, level=<first>, type=<first>, structural=true)]
> Pilot: "Konum (0, 0)'a 1 kolon yerleştirdim. Level: 'Kat 1', Type: 'M_Concrete Rectangular Column 300×300'. Başka koordinat istersen söyle."

### Bad (ping-pong)
> User: "Bir kolon yerleştir"
> Pilot: "Hangi level'a yerleştireyim?"
> User: "Herhangi"
> Pilot: "Hangi tipte olsun?"
> User: "Sen seç"
> Pilot: "Hangi koordinat?"
> User: 😡

## Exception — when to ask

ASK only if:
1. The default would clearly do the wrong thing AND
2. The user's intent has TWO equally-likely interpretations AND
3. The question is short and yes/no shape.

Example acceptable ask:
> User: "Bu duvarı sil"
> Pilot (selection is empty): "Hiçbir duvar seçili değil. Aktif viewdeki tüm duvarlar mı (124 adet)?"

Example unacceptable ask:
> User: "Bu duvarı sil"
> Pilot: "Hangi duvar?"  ← too vague, no useful options offered.

## Confirm AFTER, not BEFORE

Wrong shape (ask before):
> "5 kolonu yerleştireceğim. Onaylıyor musun?"

Right shape (do then report):
> "5 kolonu (0,0) — (12m, 0) arasında 3m aralıkla yerleştirdim. Geri almak için Ctrl+Z."

EXCEPTION: destructive operations on many elements → safety_destructive applies, confirm first when N > threshold.

## Reporting the defaults you picked

Always name your picks in the reply, so the user can correct if wrong:
> "Default'larla yaptım: Level=Kat 1, Type=Concrete-Rectangular 300x300, koordinat (0,0). Farklı istersen söyle."

This is the user's escape hatch: they see what was picked, can override on the next turn.

## When to call this an exception

If the user gave EXACTLY enough info ("place a 200mm wall from (0,0) to (5m, 0) on Level 2") — no defaults needed. Just execute.

The defaults flow is for VAGUE intents only.
