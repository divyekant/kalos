---
name: kalos
description: >-
  Use when starting a design phase, setting up design tokens, or validating
  design artifacts. Triggers on '/kalos', 'design tokens', 'design standards',
  'design check', or 'design rules'.
argument_hint: "[init|check|sync]"
allowed_tools:
  - mcp__pencil__batch_get
  - mcp__pencil__search_all_unique_properties
  - mcp__pencil__replace_all_matching_properties
  - mcp__pencil__get_variables
  - mcp__pencil__set_variables
  - mcp__pencil__get_editor_state
  - mcp__pencil__get_screenshot
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
---

# Kalos — Design Governance

Format-agnostic design governance tool. Define design tokens and rules once,
enforce them across Pencil, Tailwind, CSS, and more.

## Sub-Command Routing

Detect the user's intent:

| User says | Route to |
|-----------|----------|
| `/kalos init` or "set up design tokens" or "design standards" | **Init** section |
| `/kalos check` or "check design" or "validate design" | **Check** section |
| `/kalos sync` or "sync tokens" or "push tokens" | **Sync** section |
| `/kalos` (bare) or "kalos" | **What Next** section |

## Config Resolution

Kalos uses three-tier config resolution. Later tiers override earlier ones.

**Resolution order:**
1. `~/.kalos/defaults.yaml` — global user preferences
2. `~/.kalos/templates/<name>.yaml` — template overrides (via `extends` key)
3. `.kalos.yaml` (project root) — per-project overrides

**How to resolve:**
1. Read `~/.kalos/defaults.yaml`. This is the base config.
2. If context has an `extends` key, read `~/.kalos/templates/<extends>.yaml`
   and deep merge on top of defaults.
3. If `.kalos.yaml` exists in current working directory, read it and deep merge
   on top. Ignore `extends` and `version` keys during merge.

**Merge strategy:** Deep merge. Nested keys override individually, not entire
objects. Arrays replace (not concatenate).

## Instruction Injection Procedure

Called by `/kalos init`, `/kalos` (bare, when drift detected), and after any
config change.

### Steps:

1. **Resolve config** using three-tier resolution above.

2. **Generate instructions** from resolved config. Translate each value:

   | Config | Instruction |
   |--------|------------|
   | `tokens.colors.primary` | "Primary: {value}" |
   | `tokens.colors.secondary` | "Secondary: {value}" |
   | `tokens.colors.semantic.*` | "Semantic colors: success={s}, warning={w}, error={e}" |
   | `tokens.typography.font_family` | "Font: {family}" |
   | `tokens.typography.scale` + `base_size` | "Type scale: {ratio} from {base}px base" |
   | `tokens.spacing.base` | "Spacing: {base}px base unit, use multiples only" |
   | `rules.colors.max_unique` | "Max {n} unique colors" |
   | `rules.colors.require_semantic: true` | "All colors must map to design tokens" |
   | `rules.typography.max_font_families` | "Max {n} font families" |
   | `rules.spacing.require_base_unit: true` | "All spacing must be multiples of base unit" |
   | `rules.accessibility.min_contrast` | "Min contrast ratio: {n} (WCAG AA)" |
   | `rules.accessibility.require_alt_text: true` | "Require alt text on all images" |
   | `rules.components.naming` | "Component naming: {convention}" |

   Only include instructions for config values that are set and meaningful.

3. **Write to CLAUDE.md** using marker-based injection:

   ```markdown
   <!-- KALOS:START - Do not edit this section manually -->
   ## Design Standards (managed by Kalos)
   - <instruction 1>
   - <instruction 2>
   <!-- KALOS:END -->
   ```

   Write strategy:
   - If file exists and has KALOS markers: replace content between markers
   - If file exists but no markers: append managed section at end
   - If file doesn't exist: create with managed section only

   **Important:** Never touch content outside the markers. Never touch Apollo's
   `<!-- APOLLO:START -->` section.

---

## /kalos init — Interactive Onboarding

Set up design tokens and rules for the current project.

### First-Run Detection

Before handling init, check if `~/.kalos/defaults.yaml` exists.

If it does NOT exist:
- Say: "Welcome to Kalos! Let's set up your design defaults first."
- Create `~/.kalos/` directory structure:
  ```
  ~/.kalos/
    defaults.yaml
    templates/
      modern.yaml
      minimal.yaml
  ```
- Copy template files from this skill's source: `../../config/templates/`
- Copy defaults from: `../../config/defaults.example.yaml`
- Then continue with project-level init below.

### Flow:

Ask questions ONE AT A TIME using AskUserQuestion.

1. **Template choice**
   - "Which design template?"
   - Options: list template names from `~/.kalos/templates/`
     (typically: modern, minimal)
   - This sets the `extends` value

2. **Primary color**
   - "Primary brand color? (hex value or 'use template default')"
   - Show the template default as reference

3. **Secondary color**
   - "Secondary/accent color? (hex value or 'use template default')"

4. **Font family**
   - "Primary font family?"
   - Options: Inter, system-ui, Custom (ask for value)
   - Default from template

5. **Spacing base unit**
   - "Base spacing unit?"
   - Options: 4px (recommended), 8px, Custom
   - Default from template

6. **Adapters**
   - "Which adapters to enable?"
   - Multi-select: Pencil (recommended for v0.1.0)
   - Note: Tailwind adapter coming in v0.2.0

7. **Strictness**
   - "How strict should design rules be?"
   - Options:
     - Relaxed (max 16 colors, contrast 3.0)
     - Standard (max 12 colors, contrast 4.5 WCAG AA) — recommended
     - Strict (max 8 colors, contrast 7.0 WCAG AAA)

### After questions:

1. Write `.kalos.yaml` to project root with `extends`, `version: 0.1.0`,
   and any overrides from user answers
2. Run the **Instruction Injection Procedure** to write KALOS section
   to CLAUDE.md
3. Confirm: "Design standards set up. Run `/kalos check` after creating
   designs to validate them, or `/kalos sync` to push tokens to Pencil."

---

## /kalos check — Validate Design Artifacts

(See Task 5)

---

## /kalos sync — Push Tokens to Adapters

(See Task 6)

---

## /kalos (bare) — What Next

(See Task 7)
