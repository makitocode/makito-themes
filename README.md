# makito-themes

One color engine, many apps. Each theme is a fixed **accent palette** set on a **base**
whose contrast you can change, and the same theme is generated for every terminal and
editor.

The first family is **Panda**, based on
[Panda Theme](https://marketplace.visualstudio.com/items?itemName=tinkertrain.theme-panda).
Its contrast variations are inspired by
[Dainty Panda](https://marketplace.visualstudio.com/items?itemName=alexanderte.dainty-panda-theme-vscode),
plus a new **pink** base (a dark rose close to black).

> **Status:** early design. No code yet. This document describes the architecture we are building toward.

---

## The core idea

A theme is not a file. It is a **palette plus a base**.

| Layer | What it holds | Changes per variant? |
|-------|---------------|----------------------|
| **Accents** | Syntax and ANSI colors (pink, cyan, orange, purple…) | No. They define the theme's identity |
| **Base** | Background, surfaces, borders, selection, muted text | Yes. It is generated |

You change the base with two controls:

| Control | Values | Example |
|---------|--------|---------|
| **Tint family** | `gray`, `blue`, `black`, `pink` | `pink` gives a dark rose base close to black |
| **Contrast level** | `0`–`4` (exact range to be decided) | `0` = deepest, `4` = lightest |

Each combination is one variant, for example `Panda Pink 2` or `Panda Blue 0`.

### Why OKLCH

Bases are generated in the **OKLCH** color space, not HEX or HSL.

- OKLCH is **perceptually uniform**. One step of lightness looks equally lighter
  whether the tint is gray, blue or pink. HSL does not work that way: "10% pink" and
  "10% blue" do not look equally dark.
- The result is that every tint family stays even across levels, and the pink base reads
  as elegant rather than loud.
- Every variant is **checked for contrast** (WCAG / APCA) so no accent gets lost
  on its background.

---

## Stack

| Piece | Choice | Why |
|-------|--------|-----|
| Language | TypeScript on Node | Typed tokens, simple build scripts, no framework |
| Color math | [culori](https://culorijs.org/) | OKLCH, gamut mapping, contrast helpers |
| Output | Generated files **committed** to the repo | Users download a theme without building anything |

---

## Architecture

The repo uses a hexagonal architecture with screaming top-level folders. Open the repo
and you see the themes.

```
makito-themes/
├── panda/                  # Theme family: accent palette + generated output
│   ├── palette.json        #   (planned) source of truth for accents
│   └── dist/
│       └── terminal-app/   #   (planned) e.g. "Panda Pink 2.terminal"
├── gentle-ai/              # (planned) second family, same shape
└── src/
    ├── core/               # Domain: palette, base scales, variants, contrast
    └── targets/            # Adapters: one per app
        ├── terminal-app/   #   macOS Terminal.app (first target)
        ├── iterm2/         #   (planned)
        └── vscode/         #   (planned) also covers Cursor
```

### How the layers talk

```
 panda/palette.json ─┐
                     ├─► core (tokens + variants) ─► port: Target ─► adapter ─► file
 tint × level ───────┘                                                (terminal-app, vscode, …)
```

- **Core** knows colors and contrast. It does not know about any app.
- **Target (port)** is the contract every adapter implements: it takes resolved tokens
  and returns files.
- **Adapters** translate tokens into one app's format.

### Extending

| You want to… | You touch | You don't touch |
|--------------|-----------|-----------------|
| Add a theme family (e.g. `gentle-ai`) | A new palette folder | Core, adapters |
| Add an app (e.g. Ghostty) | A new folder in `src/targets/` | Core, palettes |
| Add a tint family or level | Base scales in `src/core/` | Palettes, adapters |

---

## First target: Terminal.app

Two constraints decide how this adapter works:

1. **The format is binary-ish.** A `.terminal` file is a property list in which each color
   is an `NSColor` serialized with `NSKeyedArchiver`. The adapter builds that archive
   directly from TypeScript, with no Swift dependency.
2. **The palette is limited.** Terminal.app themes expose 16 ANSI colors plus background,
   foreground, cursor, selection and bold. Choosing which Panda accent goes in which ANSI
   slot is the key design decision for this target.

---

## Roadmap

- [ ] Panda accent palette and base scales (`gray`, `blue`, `black`, `pink`)
- [ ] Core variant generator with contrast checks
- [ ] Terminal.app adapter
- [ ] iTerm2 adapter
- [ ] VS Code / Cursor extension
- [ ] `gentle-ai` theme family
- [ ] Agent-native distribution (under exploration)

## License

To be decided.
