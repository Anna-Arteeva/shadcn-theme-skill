# ShadCN Theme Generator

Convert raw design tokens into a production-ready `globals.css` for ShadCN/Tailwind v4 projects.

## Overview

This skill takes design tokens in any format (JSON, Markdown tables, loose hex lists, screenshots) and generates a single, drop-in `globals.css` file with:

- `:root` block for light-mode token values
- `.dark` block for dark-mode variants
- `@theme inline` block for Tailwind v4 theme bindings

No modifications needed — copy the output directly into your ShadCN project.

## Features

✨ **Smart token mapping** — automatically maps your tokens to ShadCN semantic slots (primary, secondary, muted, accent, destructive, etc.)

🌓 **Dark mode ready** — derives dark-mode primaries by lightening the brand color; preserves dark-mode shadows as pure black

🎨 **Shadow tinting** — light-mode shadows tint with your brand primary for visual cohesion; dark-mode shadows stay neutral

📝 **Provenance tracking** — preserves original token names and descriptions as comments in the output

🔄 **Flexible input** — accepts JSON (Tokens Studio, Design Tokens format), Markdown tables, plain hex lists, or even casual natural language

## Installation

### Via Claude Code

1. Download `shadcn-theme.skill`
2. Double-click to install, or drag into Claude Code
3. Restart Claude Code (if prompted)

### Via CLI

```bash
unzip shadcn-theme.skill -d ~/.claude/skills/
```

### Via Git

```bash
git clone https://github.com/annaarteeva/shadcn-theme-skill.git ~/.claude/skills/shadcn-theme
```

## Usage

In any Claude Code session, describe your design tokens and what you want:

```
Here are my brand colors from Figma:
- background: #F7F5F0
- text: #1B1B1B
- brand: #2E5BFF
- border: #E2DFD6
- font: Söhne, system-ui, sans-serif

Generate a globals.css for shadcn.
```

Or paste a JSON snippet:

```json
{
  "color": {
    "surface": { "$value": "#F7F5F0" },
    "ink": { "$value": "#1B1B1B" },
    "primary": { "$value": "#2E5BFF" }
  },
  "font": {
    "body": { "$value": "Söhne, system-ui, sans-serif" }
  }
}
```

The skill will output a single `css` code block ready to paste into your `app/globals.css`.

## Mapping Rules

| Your Token | → | ShadCN Variables |
|---|---|---|
| Background color | | `--background` |
| Primary text | | `--foreground`, `--card-foreground`, `--popover-foreground` |
| Main brand / action color | | `--primary`, `--ring`, `--sidebar-primary` |
| Light tint of brand | | `--secondary` |
| Subtle / muted text | | `--muted-foreground` |
| Border color | | `--border`, `--input` |
| Accent / highlight | | `--accent` |
| Body font | | `--font-sans` |
| Serif font | | `--font-serif` |
| Monospace font | | `--font-mono` |
| Corner radius | | `--radius` |

Tokens with no semantic match become custom variables and are exposed in `@theme inline` for use as Tailwind utilities.

## Dark Mode

The skill automatically:

- **Lightens the primary** by ~20% in `.dark` for readability on dark surfaces
- **Keeps dark-mode shadows pure black** — no tinting needed
- **Preserves all dark-mode defaults** from ShadCN unless you provide overrides

## Example

**Input:**

```
background #FAFAFA, text #1A1A1A, brand #FF5722, border #E0E0E0, font Inter
```

**Output:**

```css
:root {
  --background: #fafafa;
  --foreground: #1a1a1a;
  /* ... brand colors, borders, fonts, all three sections ... */
}

.dark {
  --primary: #ff8a65; /* lightened for dark surfaces */
  /* ... */
}

@theme inline {
  /* ... all color, font, radius, and shadow bindings ... */
}
```

## Triggering Phrases

The skill auto-triggers on requests like:

- "Generate a globals.css from these design tokens"
- "Turn these brand colors into a shadcn theme"
- "Build me a Tailwind theme from this palette"
- "Make these design tokens work with shadcn"
- "I have a color system, give me the CSS"
- "Convert these tokens to globals.css"

No need to say "shadcn" explicitly — the skill recognizes design-token requests broadly.

## What's Included

- **SKILL.md** — Full mapping rules, dark-mode logic, naming conventions, and a worked example
- **assets/globals.template.css** — The canonical ShadCN Tailwind v4 default template

## Requirements

- Claude Code (any recent version)
- A ShadCN/Tailwind v4 project

## License

MIT

## Support

For issues, suggestions, or contributions:

- **GitHub:** [annaarteeva/shadcn-theme-skill](https://github.com/annaarteeva/shadcn-theme-skill)
- **Issues:** [Report a bug or request a feature](https://github.com/annaarteeva/shadcn-theme-skill/issues)

## Changelog

### v1.0.0 (2026-05-10)

- Initial release
- Full ShadCN/Tailwind v4 support
- Light/dark mode variants
- Shadow tinting with brand primary
- Provenance comment tracking
- Flexible token input (JSON, hex lists, natural language)

---

**Made with 🎨 for designers and developers who want theming without the fuss.**
