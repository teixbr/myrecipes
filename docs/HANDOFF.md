# MyRecipes — hand-off

Everything decided so far, so any Claude session (cloud, VS Code, terminal) can continue.

- **Design canvas (all screens):** https://claude.ai/artifact/HFVFp9SjBvE9cnxjPu4k9V (private to Bryan's account)
- **Original planning session:** `claude --teleport session_01P5NAAnxHvBk57AaEhJgf98`

## 1. The goal
One place for her recipes. She mostly finds them online. Must work offline, cost nothing to run,
and feel personal: it's a gift, not a product.

## 2. Constraints
| Item | Decision |
|---|---|
| Phone | iPhone 15 (non-Pro): no Apple Intelligence |
| Test device | Bryan's iPad |
| Mac | Intel → Xcode 26.x max (iOS 26 SDK) |
| Distribution | Paid Apple Developer account (£79/yr) + TestFlight, or install from Xcode |
| Cost | Zero running costs. No server, no paid API |
| Gemini | Rejected: Gemini API terms allow only paid services for apps serving UK/EEA/CH users |
| AWS/Oracle server | Rejected: costs after credits, breaks offline, needs upkeep |

## 3. Visual design: direction D
B's layouts (photo grid, cook mode, scan review) in C's soft colours. See the "D" rows on the canvas.

| Token | Light | Use |
|---|---|---|
| background | `#FAF6F2` | linen page |
| card | `#FFFFFF` | cards, sheets |
| ink | `#3D3632` | text |
| muted | `#756C66` | secondary text |
| accent | `#5E7A64` | sage: primary buttons, active tab |
| accentInk | `#4A6350` | sage text on light |
| soft (sage) | `#DDE6DA` | tags, chips |
| blush | `#F1DCD5` | tints |
| oat | `#EFE6D8` | tints |
| lavender | `#E4E0EC` | tints |
| butter | `#F7EEDB` | tips, "not sure" flags |
| line | `#F0E9E2` | dividers |
| cookBg / raised / light | `#2E3A31` / `#3E4B41` / `#B9CFB8` | dark cook mode |

- **Type:** Fraunces with SOFT=100 for headings (serif), DM Sans for body.
- **Shape:** pill buttons, 20–24 pt card radius, soft shadows instead of borders, ≥44 pt touch targets.
- **Themes (Settings → Look & feel):** Sage (default), Blush, Lavender, Clay, Ocean, Oat, each with
  light and dark variants, plus a Light / Dark / Auto switch. All colours come from one `Theme` object.

## 4. Features

### v1 (target: Christmas)
1. **Web import:** Share Extension from Safari + paste a URL. Parse schema.org `Recipe` JSON-LD;
   fall back to readable text plus the review screen.
2. **Photo scan:** camera or Photos → Vision OCR → heuristic parser → **scan review screen**
   (tap a tag to move a line between ingredients and method; uncertain lines highlighted).
   The parser sits behind a `RecipeParser` protocol so an on-device LLM can be swapped in later.
3. **Recipe builder (block editor, "like a ClickUp doc"):** blocks are text (size, bold, italic,
   underline, lists), heading, photo, timer, tip, ingredients, step, tools, divider and voice memo.
   Drag to reorder. Imported recipes are blocks too.
4. **Timers:** times in steps ("28 minutes", "1 hr 10 min") are detected and become tappable
   buttons. Running timers sit in a tray. Use AlarmKit so they ring in silent mode and show on
   the lock screen / Dynamic Island.
5. **Tools:** a third recipe tab (Ingredients · Tools · Method) and a checklist ("4 of 6 ready").
   Catalogue of 205 tools in 11 groups: `Resources/tools.json`. She can add her own.
   Detection: keyword + alias matching (and temperature/size notes); optional local AI suggests
   implied tools, shown as "Suggested: tap to keep".
6. **Scaling & conversions:** servings stepper; g/ml ↔ cups/oz; oven °C fan ↔ conventional ↔ gas ↔ °F.
7. **Shopping list:** add recipes, merge duplicates ("3 onions · Risotto + Thai curry"), group by aisle.
8. **"I made this" log:** date, photo, 1–5 stars, note, per recipe.
9. **Voice memo block:** record inside a recipe; on-device transcription (Speech framework).
10. **JSON export / import:** see §5.
11. **Personal touches:** a first-launch welcome screen with Bryan's note, 5–10 pre-loaded recipes
    and a custom app icon.
12. **Hands-free cook mode** (Bryan wants this early): offline speech commands "next", "back",
    "repeat", "start timer", "ingredients"; read steps aloud. Needs tuning for kitchen noise.
13. **TikTok / Instagram import** (Bryan wants this early): share a post to the app.
    TikTok's public oEmbed returns the caption. Meta's oEmbed is tokenless since June 2026 but has
    no caption field, so the caption must be pulled from the embed HTML (needs testing). Fallback:
    paste the caption or OCR a screenshot. Only the caption is read, never the video.

### Later
Meal planner · "What can I make?" from the cupboard · iCloud sync · share as a card/PDF ·
"My kitchen" (tools she owns, flag recipes needing missing tools) · library filters by tool.

### Out of scope
Nutrition and calories. Any cloud AI.

## 5. Data model & export
A recipe is metadata plus an ordered list of blocks. Images and audio are files referenced by name.

Export is a zip with its own file type (`.recipes`), shareable via AirDrop or Files and openable
straight into the app. Import merges by `id`: an existing recipe is updated, not duplicated.

```
MyRecipes-2026-12-25.recipes   (zip)
├── recipes.json
├── images/<uuid>.jpg
└── audio/<uuid>.m4a
```

```json
{
  "version": 1,
  "exportedAt": "2026-12-25T10:00:00Z",
  "recipes": [{
    "id": "3f2a…",
    "title": "Mushroom risotto",
    "source": { "type": "web", "url": "https://…" },
    "servings": 4, "prepMinutes": 10, "cookMinutes": 30,
    "tags": ["Dinner"], "favourite": true,
    "images": ["images/3f2a….jpg"],
    "ingredients": [{ "quantity": 300, "unit": "g", "name": "Arborio rice", "raw": "300g arborio rice" }],
    "tools": [{ "kind": "oven", "note": "200°C fan" }],
    "blocks": [
      { "type": "heading", "text": "Custard tarts" },
      { "type": "text", "runs": [{ "text": "Make the custard ", "bold": true }, { "text": "the night before." }] },
      { "type": "image", "file": "images/9b7e.jpg", "caption": "How the tops should look" },
      { "type": "timer", "label": "Bake", "seconds": 1080 },
      { "type": "tip", "text": "Rotate the tray halfway." },
      { "type": "step", "text": "Soften the onion…" },
      { "type": "voice", "file": "audio/77c1.m4a", "transcript": "…" }
    ],
    "log": [{ "date": "2026-10-12", "rating": 5, "note": "Added walnuts", "photo": "images/…" }],
    "createdAt": "…", "updatedAt": "…"
  }]
}
```

## 6. Screens (design canvas, row D and "More screens")
Library · Recipe (Ingredients / Tools / Method with timer buttons) · Cook mode · Hands-free cook mode ·
Add recipe · Scan review · Recipe builder · Voice memo block · Add a tool · Scaling & conversions ·
Shopping list · I made this · TikTok/Instagram import · Look & feel · Welcome.

## 7. Suggested build order
1. Xcode project, Theme, SwiftData models, library + recipe screens with sample data.
2. Web import (JSON-LD) + Share Extension.
3. Block editor + timers (AlarmKit).
4. Photo scan (Vision) + review screen.
5. Tools, scaling, shopping list, log.
6. Export / import zip.
7. Voice memos, hands-free cook mode, social import.
8. Welcome screen, pre-loaded recipes, app icon → TestFlight.

## 8. Still to tell Claude
- Exact Mac model and macOS version; iOS/iPadOS versions on her iPhone and his iPad.
- Her name and Bryan's welcome note (placeholders on the welcome screen).
- Which recipes to pre-load.
