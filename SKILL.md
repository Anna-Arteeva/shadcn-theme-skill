---
name: shadcn-theme
description: >
  Convert raw design tokens (colors, typography, shadows, radii) into a single,
  drop-in ShadCN/Tailwind v4 globals.css file with :root, .dark, and
  @theme inline blocks. Use this skill any time the user provides design tokens —
  from Figma variables, Tokens Studio JSON, a brand style guide, a list of hex
  values, a Markdown table, or even a screenshot of a palette — and wants a
  ShadCN-ready theme file. Trigger even if the user doesn't say "shadcn"
  explicitly: phrases like "turn these brand colors into a theme",
  "generate globals.css from these tokens", "build me a Tailwind theme from
  this palette", "make these design tokens work with shadcn",
  or "I have a color system, give me the CSS" should all invoke this skill.
---

# shadcn-theme

Take raw design tokens in any shape and emit one `globals.css` that drops into a fresh ShadCN/Tailwind v4 project with no further edits.

## What you produce

A **single `css` code block** containing three sections, in this order:

1. `:root { … }` — light-mode token values
2. `.dark { … }` — dark-mode token values
3. `@theme inline { … }` — Tailwind v4 theme bindings

Nothing else — no preamble, no trailing prose, no separate files. The user copies the block into their `app/globals.css` and is done.

## Workflow

1. **Read the default template.** Open `assets/globals.template.css`. This is the canonical ShadCN starter — the source of every default value and the exact ordering/structure your output must follow.

2. **Parse the user's tokens.** Inputs vary widely (Tokens Studio JSON, a Figma export, a Markdown table, a casual list like "background #FAFAFA, brand #FF5722"). Extract: color values, font stacks, radii, shadows, plus any **original token name** and **description** the format carries.

3. **Apply mapping rules** (next section) to slot the user's tokens into the right ShadCN semantic variables.

4. **Fill the rest from defaults.** Any slot the user didn't cover keeps its value from `assets/globals.template.css` — including `--destructive`, charts, fonts, and the full shadow scale if the user didn't provide one.

5. **Tint shadows in light mode** with the brand primary (see Shadow handling). Dark-mode shadows stay default black.

6. **Emit one fenced `css` code block** — see Output format.

## Mapping rules

| User-provided token | ShadCN slot(s) |
|---|---|
| Background color | `--background` |
| Primary text color | `--foreground`, `--card-foreground`, `--popover-foreground` |
| Main brand / action color | `--primary`, `--ring`, `--sidebar-primary` |
| Light tint of brand color | `--secondary` |
| Subtle / muted text color | `--muted-foreground` |
| Border color | `--border`, `--input` |
| Highlight / accent color | `--accent` |
| Body / sans font | `--font-sans` |
| Serif font | `--font-serif` |
| Mono font | `--font-mono` |
| Base corner radius | `--radius` |
| Anything with no clean semantic equivalent | Keep as a custom variable; **also expose it inside `@theme inline`** so it's available as a Tailwind utility |

If the user only gives a brand primary (no explicit secondary tint), derive a light tint by mixing the primary with white at ~90% white.

## Dark mode

- For surfaces (`--background`, `--card`, `--popover`, `--sidebar`): keep the template's dark values unless the user explicitly provides dark-mode equivalents.
- For `--primary` in `.dark`: lighten the brand primary so it stays visible on dark surfaces (mix with white at ~15–25%).
- For `--secondary`, `--muted`, `--accent` in `.dark`: keep the template's neutral dark grays unless overridden.
- For dark-mode `--ring` and `--sidebar-primary`: derive from the lightened primary.

## Shadow handling

- **Light mode shadows** (`--shadow-2xs` through `--shadow-2xl` in `:root`): replace the `hsl(0 0% 0% / …)` color with an HSL representation of the brand primary, keeping the existing alpha values. Convert the primary hex to HSL channels for this. Also set `--shadow-color` to the primary in `oklch()` form (or `hsl()` if oklch conversion isn't straightforward — both are valid).
- **Dark mode shadows** (in `.dark`): leave the template defaults unchanged. Black-on-dark reads as a real shadow; tinted shadows on dark backgrounds usually don't.

## Naming rules

- **kebab-case only** — no spaces, no camelCase, no underscores.
- Map unconventional input names (e.g. `Brand/Primary500`, `colors.surface.default`, `text/secondary`) to the **nearest ShadCN equivalent** rather than inventing new variable names.
- Never introduce a custom naming convention that isn't already in the template.
- Custom variables that don't fit any semantic slot use kebab-case too: `--brand-coral`, `--surface-elevated`, etc.

## Comments in the output

Two kinds of comments, both short:

1. **Group headers** — one inline comment per logical group of variables, explaining what the group is for. Examples:
   ```css
   --background: #fafafa;
   --foreground: #1a1a1a; /* page surfaces and primary text */
   ```
   ```css
   --primary: #ff5722;
   --primary-foreground: #ffffff; /* brand action color */
   ```
   Place the comment on the **last line** of the group, not on every line.

2. **Provenance comments** — when the input token carries an original name and/or description, append it to the relevant variable:
   ```css
   --primary: #ff5722; /* original: brand.primary.500 — main brand action color */
   ```
   Format: `/* original: <original-name> — <description> */`. Drop either part if it's missing. Skip entirely for variables that came from defaults (no input token, no comment needed).

Don't comment every line. Comments exist to help a human pick the file up later, not to narrate the CSS.

## Output format

ALWAYS use this exact structure:

````
```css
:root {
  /* … light-mode values, in the same order as the template … */
}

.dark {
  /* … dark-mode values, in the same order as the template … */
}

@theme inline {
  /* … theme bindings, including any custom variables you added … */
}
```
````

- One single fenced `css` block. No prose before or after.
- Variable order matches `assets/globals.template.css` — colors first, then fonts, then radius, then shadows, then spacing.
- Custom variables (tokens with no semantic slot) go at the end of their respective section in `:root` / `.dark`, and are added to `@theme inline` as `--color-<name>: var(--<name>);` (or the appropriate Tailwind theme prefix: `--font-<name>`, `--radius-<name>`, `--shadow-<name>`).

## Example

**Input from user:**
```json
{
  "color": {
    "bg":      { "$value": "#FAFAFA", "$description": "page background" },
    "text":    { "$value": "#1A1A1A", "$description": "primary text" },
    "brand":   { "$value": "#FF5722", "$description": "main brand action" },
    "border":  { "$value": "#E0E0E0" }
  },
  "font":  { "sans": { "$value": "Inter, system-ui, sans-serif" } },
  "radius":{ "base": { "$value": "0.5rem" } }
}
```

**Output:**
````
```css
:root {
  --background: #fafafa; /* original: color.bg — page background */
  --foreground: #1a1a1a; /* original: color.text — primary text */
  --card: #fafafa;
  --card-foreground: #1a1a1a;
  --popover: #fafafa;
  --popover-foreground: #1a1a1a; /* surface + text on cards and popovers */
  --primary: #ff5722; /* original: color.brand — main brand action */
  --primary-foreground: #ffffff;
  --secondary: #ffe5db;
  --secondary-foreground: #1a1a1a; /* brand + light tint */
  --muted: #f5f5f5;
  --muted-foreground: #737373; /* subtle backgrounds and secondary text */
  --accent: #f5f5f5;
  --accent-foreground: #1a1a1a; /* hover / highlight */
  --destructive: #e7000b;
  --destructive-foreground: #ffffff;
  --border: #e0e0e0; /* original: color.border */
  --input: #e0e0e0;
  --ring: #ff5722; /* borders, inputs, focus rings */
  --chart-1: #91c5ff;
  --chart-2: #3a81f6;
  --chart-3: #2563ef;
  --chart-4: #1a4eda;
  --chart-5: #1f3fad; /* default chart palette */
  --sidebar: #fafafa;
  --sidebar-foreground: #1a1a1a;
  --sidebar-primary: #ff5722;
  --sidebar-primary-foreground: #ffffff;
  --sidebar-accent: #f5f5f5;
  --sidebar-accent-foreground: #1a1a1a;
  --sidebar-border: #e0e0e0;
  --sidebar-ring: #ff5722; /* sidebar surfaces and accents */
  --font-sans: Inter, system-ui, sans-serif; /* original: font.sans */
  --font-serif: ui-serif, Georgia, Cambria, "Times New Roman", Times, serif;
  --font-mono: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
  --radius: 0.5rem; /* original: radius.base */
  --shadow-x: 0;
  --shadow-y: 1px;
  --shadow-blur: 3px;
  --shadow-spread: 0px;
  --shadow-opacity: 0.1;
  --shadow-color: oklch(0.65 0.20 35);
  --shadow-2xs: 0 1px 3px 0px hsl(14 100% 57% / 0.05);
  --shadow-xs: 0 1px 3px 0px hsl(14 100% 57% / 0.05);
  --shadow-sm: 0 1px 3px 0px hsl(14 100% 57% / 0.10), 0 1px 2px -1px hsl(14 100% 57% / 0.10);
  --shadow: 0 1px 3px 0px hsl(14 100% 57% / 0.10), 0 1px 2px -1px hsl(14 100% 57% / 0.10);
  --shadow-md: 0 1px 3px 0px hsl(14 100% 57% / 0.10), 0 2px 4px -1px hsl(14 100% 57% / 0.10);
  --shadow-lg: 0 1px 3px 0px hsl(14 100% 57% / 0.10), 0 4px 6px -1px hsl(14 100% 57% / 0.10);
  --shadow-xl: 0 1px 3px 0px hsl(14 100% 57% / 0.10), 0 8px 10px -1px hsl(14 100% 57% / 0.10);
  --shadow-2xl: 0 1px 3px 0px hsl(14 100% 57% / 0.25); /* shadows tinted with brand primary */
  --tracking-normal: 0em;
  --spacing: 0.25rem;
}

.dark {
  --background: #0a0a0a;
  --foreground: #fafafa;
  --card: #171717;
  --card-foreground: #fafafa;
  --popover: #262626;
  --popover-foreground: #fafafa;
  --primary: #ff8a65; /* lightened brand for dark surfaces */
  --primary-foreground: #1a1a1a;
  --secondary: #262626;
  --secondary-foreground: #fafafa;
  --muted: #262626;
  --muted-foreground: #a1a1a1;
  --accent: #404040;
  --accent-foreground: #fafafa;
  --destructive: #ff6467;
  --destructive-foreground: #fafafa;
  --border: #282828;
  --input: #343434;
  --ring: #ff8a65;
  --chart-1: #91c5ff;
  --chart-2: #3a81f6;
  --chart-3: #2563ef;
  --chart-4: #1a4eda;
  --chart-5: #1f3fad;
  --sidebar: #171717;
  --sidebar-foreground: #fafafa;
  --sidebar-primary: #ff8a65;
  --sidebar-primary-foreground: #1a1a1a;
  --sidebar-accent: #262626;
  --sidebar-accent-foreground: #fafafa;
  --sidebar-border: #282828;
  --sidebar-ring: #525252;
  --font-sans: Inter, system-ui, sans-serif;
  --font-serif: ui-serif, Georgia, Cambria, "Times New Roman", Times, serif;
  --font-mono: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
  --radius: 0.5rem;
  --shadow-x: 0;
  --shadow-y: 1px;
  --shadow-blur: 3px;
  --shadow-spread: 0px;
  --shadow-opacity: 0.1;
  --shadow-color: oklch(0 0 0);
  --shadow-2xs: 0 1px 3px 0px hsl(0 0% 0% / 0.05);
  --shadow-xs: 0 1px 3px 0px hsl(0 0% 0% / 0.05);
  --shadow-sm: 0 1px 3px 0px hsl(0 0% 0% / 0.10), 0 1px 2px -1px hsl(0 0% 0% / 0.10);
  --shadow: 0 1px 3px 0px hsl(0 0% 0% / 0.10), 0 1px 2px -1px hsl(0 0% 0% / 0.10);
  --shadow-md: 0 1px 3px 0px hsl(0 0% 0% / 0.10), 0 2px 4px -1px hsl(0 0% 0% / 0.10);
  --shadow-lg: 0 1px 3px 0px hsl(0 0% 0% / 0.10), 0 4px 6px -1px hsl(0 0% 0% / 0.10);
  --shadow-xl: 0 1px 3px 0px hsl(0 0% 0% / 0.10), 0 8px 10px -1px hsl(0 0% 0% / 0.10);
  --shadow-2xl: 0 1px 3px 0px hsl(0 0% 0% / 0.25);
}

@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-card: var(--card);
  --color-card-foreground: var(--card-foreground);
  --color-popover: var(--popover);
  --color-popover-foreground: var(--popover-foreground);
  --color-primary: var(--primary);
  --color-primary-foreground: var(--primary-foreground);
  --color-secondary: var(--secondary);
  --color-secondary-foreground: var(--secondary-foreground);
  --color-muted: var(--muted);
  --color-muted-foreground: var(--muted-foreground);
  --color-accent: var(--accent);
  --color-accent-foreground: var(--accent-foreground);
  --color-destructive: var(--destructive);
  --color-destructive-foreground: var(--destructive-foreground);
  --color-border: var(--border);
  --color-input: var(--input);
  --color-ring: var(--ring);
  --color-chart-1: var(--chart-1);
  --color-chart-2: var(--chart-2);
  --color-chart-3: var(--chart-3);
  --color-chart-4: var(--chart-4);
  --color-chart-5: var(--chart-5);
  --color-sidebar: var(--sidebar);
  --color-sidebar-foreground: var(--sidebar-foreground);
  --color-sidebar-primary: var(--sidebar-primary);
  --color-sidebar-primary-foreground: var(--sidebar-primary-foreground);
  --color-sidebar-accent: var(--sidebar-accent);
  --color-sidebar-accent-foreground: var(--sidebar-accent-foreground);
  --color-sidebar-border: var(--sidebar-border);
  --color-sidebar-ring: var(--sidebar-ring);

  --font-sans: var(--font-sans);
  --font-mono: var(--font-mono);
  --font-serif: var(--font-serif);

  --radius-sm: calc(var(--radius) - 4px);
  --radius-md: calc(var(--radius) - 2px);
  --radius-lg: var(--radius);
  --radius-xl: calc(var(--radius) + 4px);

  --shadow-2xs: var(--shadow-2xs);
  --shadow-xs: var(--shadow-xs);
  --shadow-sm: var(--shadow-sm);
  --shadow: var(--shadow);
  --shadow-md: var(--shadow-md);
  --shadow-lg: var(--shadow-lg);
  --shadow-xl: var(--shadow-xl);
  --shadow-2xl: var(--shadow-2xl);
}
```
````

## Reminders

- One code block. No prose around it.
- All three sections: `:root`, `.dark`, `@theme inline`.
- Defaults from `assets/globals.template.css` for anything the user didn't provide.
- Light-mode shadows tinted with the brand primary; dark-mode shadows untouched.
- Original names + descriptions preserved as `/* original: name — description */` comments where they exist.
