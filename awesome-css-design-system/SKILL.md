---
name: awesome-css-design-system
description: Design system reference for building WordPress Gutenberg blocks and sections using the Awesome CSS (awx) token system. Use this skill whenever creating block CSS, section CSS, page templates, or any styling within the Awesome CSS / Awesome XP / Monomyth FSE ecosystem. Triggers on mentions of awx tokens, Awesome CSS, Monomyth theme, Gutenberg block styling, or CSS custom properties with the --awx- prefix.
---

## Architecture Overview

Awesome CSS is a design token system built for WordPress content platforms. It uses CSS `@layer` for cascade control and CSS custom properties for all design values.

### Layer Order (declared once, never changed)

```css
@layer tokens, base, blocks, sections;
```

- **tokens**: CSS custom properties on `:root`. Design decisions.
- **base**: Scoped classless editorial content styles inside `:where(.entry-content)`.
- **blocks**: Individual Gutenberg block styles. Self-contained.
- **sections**: Full-page section compositions (hero, pricing, CTA, footer layouts).
- **Unlayered CSS**: WordPress core, plugin CSS, project overrides. Sits above all layers.

Later layers always beat earlier layers regardless of specificity.

### Platform Independence

Blocks ONLY use `--awx-*` variables. NEVER reference `--wp--preset--*` in block or section CSS. An alias layer (generated automatically by the plugin) maps `--awx-*` to WordPress's `--wp--preset--*` variables. This means block CSS works unchanged on WordPress, headless setups, or static sites.

---

## Token Naming Contract

**Every token follows: `--awx-{category}-{name}`**

### Categories and Their Tokens

#### Colors (`--awx-color-{name}`)
Blocks use role-based color tokens. NEVER reference primitive palette values.

| Token | Purpose |
|-------|---------|
| `--awx-color-primary` | Primary brand color |
| `--awx-color-primary-light` | Light tint of primary |
| `--awx-color-primary-dark` | Dark shade of primary |
| `--awx-color-on-primary` | Text color on primary background |
| `--awx-color-secondary` | Secondary brand color |
| `--awx-color-secondary-light` | Light tint of secondary |
| `--awx-color-secondary-dark` | Dark shade of secondary |
| `--awx-color-on-secondary` | Text color on secondary background |
| `--awx-color-accent` | Accent / highlight color |
| `--awx-color-accent-light` | Light tint of accent |
| `--awx-color-accent-dark` | Dark shade of accent |
| `--awx-color-on-accent` | Text color on accent background |
| `--awx-color-background` | Page background |
| `--awx-color-surface-sunken` | Sunken/nested content (comment cards, code blocks, sidebars) |
| `--awx-color-surface` | Card / container background |
| `--awx-color-surface-raised` | Elevated surface (modals, popovers) |
| `--awx-color-surface-brand-subtle` | Light brand tint (blockquotes, asides, highlight strips) |
| `--awx-color-surface-brand` | Full brand-colored surface (CTA banners, callout boxes) |
| `--awx-color-on-background` | Text on page background |
| `--awx-color-on-surface` | Text on surface |
| `--awx-color-text` | Primary body text |
| `--awx-color-text-secondary` | Secondary / supporting text |
| `--awx-color-text-muted` | Muted / placeholder text |
| `--awx-color-text-inverse` | Text on dark backgrounds |
| `--awx-color-border` | Default border color |
| `--awx-color-border-strong` | Emphasized borders |
| `--awx-color-divider` | Subtle dividers / separators |
| `--awx-color-link` | Link text color |
| `--awx-color-link-hover` | Link hover state |
| `--awx-color-link-visited` | Link visited state |
| `--awx-color-focus-ring` | Focus outline color |
| `--awx-color-success` | Success state |
| `--awx-color-success-light` | Success background tint |
| `--awx-color-on-success` | Text on success background |
| `--awx-color-warning` | Warning state |
| `--awx-color-warning-light` | Warning background tint |
| `--awx-color-on-warning` | Text on warning background |
| `--awx-color-error` | Error state |
| `--awx-color-error-light` | Error background tint |
| `--awx-color-on-error` | Text on error background |
| `--awx-color-info` | Info state |
| `--awx-color-info-light` | Info background tint |
| `--awx-color-on-info` | Text on info background |

#### Font Sizes (`--awx-font-size-{step}`) — Fluid Modular Scale (1.25 ratio)

Steps -2 to 0 are fixed. Steps 1–8 are fluid via `clamp()`.

| Token | Desktop Value | Use |
|-------|--------------|-----|
| `--awx-font-size--2` | 0.64rem | Fine print, legal |
| `--awx-font-size--1` | 0.8rem | Captions, labels |
| `--awx-font-size-0` | 1rem | Body text |
| `--awx-font-size-1` | 1.25rem | Lead text |
| `--awx-font-size-2` | 1.563rem | H6, H5 |
| `--awx-font-size-3` | 1.953rem | H4 |
| `--awx-font-size-4` | 2.441rem | H3 |
| `--awx-font-size-5` | 3.052rem | H2 |
| `--awx-font-size-6` | 3.815rem | H1 |
| `--awx-font-size-7` | 4.768rem | Display |
| `--awx-font-size-8` | 5.96rem | Display XL |

#### Font Families (`--awx-font-{name}`)

| Token | Default |
|-------|---------|
| `--awx-font-heading` | Inter, system-ui, sans-serif |
| `--awx-font-body` | Inter, system-ui, sans-serif |
| `--awx-font-mono` | JetBrains Mono, ui-monospace, monospace |

#### Font Weights (`--awx-font-weight-{name}`)

| Token | Value |
|-------|-------|
| `--awx-font-weight-light` | 300 |
| `--awx-font-weight-regular` | 400 |
| `--awx-font-weight-medium` | 500 |
| `--awx-font-weight-semibold` | 600 |
| `--awx-font-weight-bold` | 700 |

#### Line Heights (`--awx-leading-{name}`)

| Token | Value | Use |
|-------|-------|-----|
| `--awx-leading-none` | 1 | Decorative text |
| `--awx-leading-tight` | 1.2 | Large headings |
| `--awx-leading-snug` | 1.35 | Small headings |
| `--awx-leading-normal` | 1.5 | Default |
| `--awx-leading-relaxed` | 1.65 | Body text (long-form) |
| `--awx-leading-loose` | 1.8 | Spacious text |

#### Letter Spacing (`--awx-tracking-{name}`)

| Token | Value |
|-------|-------|
| `--awx-tracking-tighter` | -0.04em |
| `--awx-tracking-tight` | -0.02em |
| `--awx-tracking-normal` | 0em |
| `--awx-tracking-wide` | 0.025em |
| `--awx-tracking-wider` | 0.05em |
| `--awx-tracking-widest` | 0.1em |

#### Spacing (`--awx-space-{step}`) — 8px Base Unit

Steps 0–6 are fixed. Steps 8–20 are fluid.

| Token | Value | Px |
|-------|-------|----|
| `--awx-space-05` | 0.25rem | 4 |
| `--awx-space-1` | 0.5rem | 8 |
| `--awx-space-15` | 0.75rem | 12 |
| `--awx-space-2` | 1rem | 16 |
| `--awx-space-25` | 1.25rem | 20 |
| `--awx-space-3` | 1.5rem | 24 |
| `--awx-space-4` | 2rem | 32 |
| `--awx-space-5` | 2.5rem | 40 |
| `--awx-space-6` | 3rem | 48 |
| `--awx-space-8` | 2.5→4rem | 40→64 (fluid) |
| `--awx-space-10` | 3→5rem | 48→80 (fluid) |
| `--awx-space-12` | 3.5→6rem | 56→96 (fluid) |
| `--awx-space-16` | 4→8rem | 64→128 (fluid) |
| `--awx-space-20` | 5→10rem | 80→160 (fluid) |

#### Semantic Spacing Aliases

| Token | References | Use |
|-------|-----------|-----|
| `--awx-space-block-padding` | `--awx-space-4` | Inner padding for blocks |
| `--awx-space-block-gap` | `--awx-space-3` | Gap between elements in a block |
| `--awx-space-section-padding` | `--awx-space-8` | Vertical section padding (fluid) |
| `--awx-space-section-gap` | `--awx-space-6` | Gap between sections |
| `--awx-space-stack` | `--awx-space-2` | Vertical rhythm between items |
| `--awx-space-inline` | `--awx-space-1` | Horizontal inline gaps |

#### Border Radius (`--awx-radius-{name}`)

| Token | Value | Use |
|-------|-------|-----|
| `--awx-radius-none` | 0 | Sharp corners |
| `--awx-radius-sm` | 4px | Inputs, small elements |
| `--awx-radius-md` | 8px | Cards, containers |
| `--awx-radius-lg` | 12px | Large cards, modals |
| `--awx-radius-xl` | 16px | Prominent containers |
| `--awx-radius-2xl` | 24px | Hero elements |
| `--awx-radius-full` | 9999px | Pills, avatars |

#### Shadows (`--awx-shadow-{level}`)

| Token | Use |
|-------|-----|
| `--awx-shadow-none` | No shadow |
| `--awx-shadow-1` | Cards at rest |
| `--awx-shadow-2` | Hover state, dropdowns |
| `--awx-shadow-3` | Popovers, sticky headers |
| `--awx-shadow-4` | Modals, dialogs |
| `--awx-shadow-5` | Toasts, notifications |
| `--awx-shadow-inner` | Pressed / input states |

#### Motion (`--awx-duration-{name}` + `--awx-ease-{name}`)

| Token | Value | Use |
|-------|-------|-----|
| `--awx-duration-fast` | 120ms | Hover, micro-interactions |
| `--awx-duration-normal` | 200ms | General transitions |
| `--awx-duration-slow` | 320ms | Modals, panels |
| `--awx-ease-standard` | cubic-bezier(0.4, 0, 0.2, 1) | General purpose |
| `--awx-ease-emphasized` | cubic-bezier(0.2, 0, 0, 1) | Entrances, emphasis |

Durations auto-zero under `prefers-reduced-motion: reduce`.

#### Border Widths (`--awx-border-{name}`)

| Token | Value |
|-------|-------|
| `--awx-border-thin` | 1px |
| `--awx-border-default` | 1.5px |
| `--awx-border-thick` | 2px |

#### Z-Index (`--awx-z-{name}`)

| Token | Value | Use |
|-------|-------|-----|
| `--awx-z-deep` | -1 | Behind content |
| `--awx-z-base` | 0 | Default |
| `--awx-z-raised` | 1 | Cards |
| `--awx-z-dropdown` | 10 | Dropdowns, tooltips |
| `--awx-z-sticky` | 20 | Sticky headers |
| `--awx-z-overlay` | 30 | Overlays, backdrops |
| `--awx-z-modal` | 40 | Modals, dialogs |
| `--awx-z-toast` | 50 | Toast notifications |

#### Layout Widths (`--awx-width-{name}`)

| Token | Value | Use |
|-------|-------|-----|
| `--awx-width-content` | 48rem (768px) | Default content width |
| `--awx-width-wide` | 72rem (1152px) | Wide blocks |
| `--awx-width-max` | 90rem (1440px) | Max container |
| `--awx-width-measure` | 65ch | Optimal reading width |
| `--awx-width-measure-wide` | 80ch | Wide reading width |
| `--awx-width-measure-narrow` | 45ch | Narrow reading width |

---

## Rules for Building Blocks

### MUST

1. **Wrap all styles in `@layer blocks`**:
   ```css
   @layer blocks {
     .wp-block-awx-card { ... }
   }
   ```

2. **Only use `--awx-*` variables**. Never hardcode colors, sizes, spacing, or any design value:
   ```css
   /* ✅ CORRECT */
   .wp-block-awx-card {
     background: var(--awx-color-surface);
     padding: var(--awx-space-block-padding);
     border-radius: var(--awx-radius-md);
   }

   /* ❌ WRONG */
   .wp-block-awx-card {
     background: #f8fafc;
     padding: 2rem;
     border-radius: 8px;
   }
   ```

3. **Use semantic color tokens** (role-based), not structural names:
   ```css
   /* ✅ CORRECT — intent-based */
   color: var(--awx-color-text-secondary);
   background: var(--awx-color-surface);

   /* ❌ WRONG — structural/primitive */
   color: var(--awx-neutral-600);
   background: var(--awx-neutral-50);
   ```

4. **Use semantic spacing aliases** when the intent matches:
   ```css
   /* ✅ CORRECT — intent is clear */
   padding: var(--awx-space-block-padding);
   gap: var(--awx-space-block-gap);

   /* Also fine — scale step when no semantic alias fits */
   margin-inline: var(--awx-space-3);
   ```

5. **Each block's CSS must be fully self-contained**. No block can leak styles into another block. Adding block #101 must not break blocks #1–100.

6. **Prefix block class names with `wp-block-awx-`**:
   ```css
   .wp-block-awx-card { }
   .wp-block-awx-testimonial { }
   .wp-block-awx-pricing-table { }
   ```

7. **Use motion tokens for all transitions**:
   ```css
   transition: background var(--awx-duration-fast) var(--awx-ease-standard),
               box-shadow var(--awx-duration-fast) var(--awx-ease-standard);
   ```

8. **One CSS file per block**, placed in `blocks/{block-name}.css`.

### MUST NOT

1. **Never reference `--wp--preset--*` variables** in block or section CSS.
2. **Never use utility classes** (no `.p-4`, `.text-primary`, `.bg-blue-500`). Blocks use tokens directly.
3. **Never use `!important`**. The `@layer` system handles cascade priority.
4. **Never style bare HTML elements** in block CSS. That belongs in `@layer base`. Blocks only style their own class-scoped elements.
5. **Never use arbitrary values**. If a value doesn't exist as a token, question whether it should. If yes, request it as a new token.

---

## Rules for Building Sections

### Sections wrap in `@layer sections`

```css
@layer sections {
  .wp-section-hero { ... }
}
```

### Section naming: `wp-section-{name}`

```css
.wp-section-hero { }
.wp-section-pricing { }
.wp-section-cta { }
.wp-section-testimonials { }
.wp-section-footer { }
```

### Sections use fluid section-level spacing

```css
@layer sections {
  .wp-section-hero {
    padding-block: var(--awx-space-section-padding);
  }

  .wp-section-hero .hero-heading {
    font-size: var(--awx-font-size-8);
    font-family: var(--awx-font-heading);
    font-weight: var(--awx-font-weight-bold);
    line-height: var(--awx-leading-tight);
    letter-spacing: var(--awx-tracking-tighter);
    color: var(--awx-color-on-primary);
  }

  .wp-section-hero .hero-subheading {
    font-size: var(--awx-font-size-2);
    color: var(--awx-color-text-secondary);
    max-width: var(--awx-width-measure);
  }
}
```

### Sections define their own background context

When a section sets a colored background, it must also set appropriate text and link colors for that context:

```css
@layer sections {
  .wp-section-cta {
    background: var(--awx-color-primary);
    color: var(--awx-color-on-primary);
    padding-block: var(--awx-space-section-padding);
  }

  .wp-section-cta a {
    color: var(--awx-color-on-primary);
    text-decoration: underline;
  }
}
```

---

## Block Examples

### Card Block

```css
@layer blocks {
  .wp-block-awx-card {
    background: var(--awx-color-surface-raised);
    border: var(--awx-border-thin) solid var(--awx-color-border);
    border-radius: var(--awx-radius-md);
    padding: var(--awx-space-block-padding);
    box-shadow: var(--awx-shadow-1);
    transition: box-shadow var(--awx-duration-fast) var(--awx-ease-standard);
  }

  .wp-block-awx-card:hover {
    box-shadow: var(--awx-shadow-2);
  }

  .wp-block-awx-card .card-title {
    font-family: var(--awx-font-heading);
    font-size: var(--awx-font-size-3);
    font-weight: var(--awx-font-weight-semibold);
    line-height: var(--awx-leading-snug);
    color: var(--awx-color-text);
    margin-block-end: var(--awx-space-1);
  }

  .wp-block-awx-card .card-body {
    font-size: var(--awx-font-size-0);
    line-height: var(--awx-leading-relaxed);
    color: var(--awx-color-on-surface);
  }

  .wp-block-awx-card .card-footer {
    margin-block-start: var(--awx-space-3);
    padding-block-start: var(--awx-space-2);
    border-top: var(--awx-border-thin) solid var(--awx-color-divider);
    font-size: var(--awx-font-size--1);
    color: var(--awx-color-text-muted);
  }
}
```

### Button Block

```css
@layer blocks {
  .wp-block-awx-button {
    display: inline-flex;
    align-items: center;
    gap: var(--awx-space-1);
    padding: var(--awx-space-15) var(--awx-space-4);
    font-family: var(--awx-font-body);
    font-size: var(--awx-font-size-0);
    font-weight: var(--awx-font-weight-semibold);
    line-height: var(--awx-leading-none);
    border-radius: var(--awx-radius-sm);
    cursor: pointer;
    transition: background var(--awx-duration-fast) var(--awx-ease-standard),
                box-shadow var(--awx-duration-fast) var(--awx-ease-standard);
  }

  /* Variant: Primary */
  .wp-block-awx-button.is-primary {
    background: var(--awx-color-primary);
    color: var(--awx-color-on-primary);
    border: none;
  }

  .wp-block-awx-button.is-primary:hover {
    background: var(--awx-color-primary-dark);
  }

  /* Variant: Secondary (outline) */
  .wp-block-awx-button.is-secondary {
    background: transparent;
    color: var(--awx-color-primary);
    border: var(--awx-border-default) solid var(--awx-color-primary);
  }

  .wp-block-awx-button.is-secondary:hover {
    background: var(--awx-color-primary-light);
  }

  /* Focus state (all variants) */
  .wp-block-awx-button:focus-visible {
    outline: 2px solid var(--awx-color-focus-ring);
    outline-offset: 2px;
  }
}
```

### Alert / Notice Block

```css
@layer blocks {
  .wp-block-awx-alert {
    display: flex;
    gap: var(--awx-space-2);
    padding: var(--awx-space-3);
    border-radius: var(--awx-radius-md);
    font-size: var(--awx-font-size-0);
    line-height: var(--awx-leading-normal);
    border-left: 4px solid;
  }

  .wp-block-awx-alert.is-success {
    background: var(--awx-color-success-light);
    color: var(--awx-color-success);
    border-color: var(--awx-color-success);
  }

  .wp-block-awx-alert.is-warning {
    background: var(--awx-color-warning-light);
    color: var(--awx-color-warning);
    border-color: var(--awx-color-warning);
  }

  .wp-block-awx-alert.is-error {
    background: var(--awx-color-error-light);
    color: var(--awx-color-error);
    border-color: var(--awx-color-error);
  }

  .wp-block-awx-alert.is-info {
    background: var(--awx-color-info-light);
    color: var(--awx-color-info);
    border-color: var(--awx-color-info);
  }
}
```

---

## Section Example

### Hero Section

```css
@layer sections {
  .wp-section-hero {
    display: grid;
    place-items: center;
    min-height: 80vh;
    padding-block: var(--awx-space-section-padding);
    padding-inline: var(--awx-space-3);
    text-align: center;
    background: var(--awx-color-primary);
    color: var(--awx-color-on-primary);
  }

  .wp-section-hero .hero-content {
    max-width: var(--awx-width-wide);
  }

  .wp-section-hero .hero-heading {
    font-family: var(--awx-font-heading);
    font-size: var(--awx-font-size-8);
    font-weight: var(--awx-font-weight-bold);
    line-height: var(--awx-leading-tight);
    letter-spacing: var(--awx-tracking-tighter);
    margin-block-end: var(--awx-space-3);
  }

  .wp-section-hero .hero-subheading {
    font-size: var(--awx-font-size-2);
    line-height: var(--awx-leading-relaxed);
    opacity: 0.9;
    max-width: var(--awx-width-measure);
    margin-inline: auto;
    margin-block-end: var(--awx-space-5);
  }

  .wp-section-hero .hero-actions {
    display: flex;
    gap: var(--awx-space-2);
    justify-content: center;
    flex-wrap: wrap;
  }
}
```

### Pricing Section

```css
@layer sections {
  .wp-section-pricing {
    padding-block: var(--awx-space-section-padding);
    padding-inline: var(--awx-space-3);
    background: var(--awx-color-surface);
  }

  .wp-section-pricing .pricing-header {
    text-align: center;
    max-width: var(--awx-width-measure);
    margin-inline: auto;
    margin-block-end: var(--awx-space-8);
  }

  .wp-section-pricing .pricing-title {
    font-family: var(--awx-font-heading);
    font-size: var(--awx-font-size-6);
    font-weight: var(--awx-font-weight-bold);
    line-height: var(--awx-leading-tight);
    color: var(--awx-color-text);
  }

  .wp-section-pricing .pricing-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
    gap: var(--awx-space-4);
    max-width: var(--awx-width-max);
    margin-inline: auto;
  }

  .wp-section-pricing .pricing-card {
    background: var(--awx-color-surface-raised);
    border: var(--awx-border-thin) solid var(--awx-color-border);
    border-radius: var(--awx-radius-lg);
    padding: var(--awx-space-5);
  }

  .wp-section-pricing .pricing-card.is-featured {
    border-color: var(--awx-color-primary);
    box-shadow: var(--awx-shadow-3);
    position: relative;
  }

  .wp-section-pricing .pricing-price {
    font-family: var(--awx-font-heading);
    font-size: var(--awx-font-size-6);
    font-weight: var(--awx-font-weight-bold);
    color: var(--awx-color-text);
    line-height: var(--awx-leading-none);
  }

  .wp-section-pricing .pricing-price span {
    font-size: var(--awx-font-size-0);
    font-weight: var(--awx-font-weight-regular);
    color: var(--awx-color-text-muted);
  }

  .wp-section-pricing .pricing-features {
    list-style: none;
    padding: 0;
    margin-block: var(--awx-space-4);
    display: flex;
    flex-direction: column;
    gap: var(--awx-space-15);
  }

  .wp-section-pricing .pricing-features li {
    font-size: var(--awx-font-size-0);
    color: var(--awx-color-on-surface);
    padding-inline-start: var(--awx-space-4);
    position: relative;
  }

  .wp-section-pricing .pricing-features li::before {
    content: "✓";
    position: absolute;
    left: 0;
    color: var(--awx-color-success);
    font-weight: var(--awx-font-weight-bold);
  }
}
```

---

## File Structure

```
awesome-css/
├── awesome.css              ← @layer declaration + imports
├── tokens/                  ← Generated by the token module (do not edit)
│   └── tokens.css           ← All --awx-* custom properties
├── base/
│   ├── reset.css            ← Global reset (box-sizing, defaults)
│   └── classless.css        ← :where(.entry-content) editorial styles
├── blocks/
│   ├── card.css             ← @layer blocks { .wp-block-awx-card { } }
│   ├── button.css
│   ├── alert.css
│   ├── testimonial.css
│   └── ...                  ← One file per block
└── sections/
    ├── hero.css             ← @layer sections { .wp-section-hero { } }
    ├── pricing.css
    ├── cta.css
    └── ...                  ← One file per section
```

---

## Modifier Convention

Use `is-*` and `has-*` prefixes for readable modifiers (inspired by Bulma):

```css
/* State variants */
.wp-block-awx-button.is-primary { }
.wp-block-awx-button.is-secondary { }
.wp-block-awx-button.is-small { }
.wp-block-awx-button.is-large { }
.wp-block-awx-button.is-loading { }
.wp-block-awx-button.is-disabled { }

/* Feature flags */
.wp-block-awx-card.has-shadow { }
.wp-block-awx-card.has-border { }
.wp-block-awx-card.has-image { }
```

---

## Responsive Approach

DO NOT write media query breakpoints for font sizes or section spacing. These are already fluid via `clamp()` in the token values. The tokens handle responsiveness automatically.

Use media queries ONLY for layout changes (grid columns, flex direction):

```css
@layer blocks {
  .wp-block-awx-feature-grid {
    display: grid;
    gap: var(--awx-space-4);
    grid-template-columns: 1fr; /* Mobile default */
  }

  @media (min-width: 48rem) {
    .wp-block-awx-feature-grid {
      grid-template-columns: repeat(2, 1fr);
    }
  }

  @media (min-width: 64rem) {
    .wp-block-awx-feature-grid {
      grid-template-columns: repeat(3, 1fr);
    }
  }
}
```

Breakpoint reference values (use raw values in media queries — CSS variables cannot be used in `@media`):
- `40rem` (640px) — Large phones
- `48rem` (768px) — Tablets
- `64rem` (1024px) — Laptops
- `80rem` (1280px) — Desktops
- `96rem` (1536px) — Large screens

---

## Common Patterns

### On-Color Pattern
When setting a background, always pair it with the matching `on-` color:

```css
.wp-block-awx-badge.is-primary {
  background: var(--awx-color-primary);
  color: var(--awx-color-on-primary);  /* ensures readable contrast */
}
```

### Surface Two-Axis Model
Surfaces have two axes: **elevation** (neutral) and **fill intent** (brand). Pick the right surface token based on what the block IS, not what shade you want:

```
ELEVATION (neutral):
  surface-sunken → background → surface → surface-raised
       ↑              ↑           ↑            ↑
  nested content    page       cards      modals

BRAND FILL:
  surface-brand-subtle → surface-brand
         ↑                    ↑
  blockquotes/asides    CTA banners
```

**Text pairing rules for surfaces:**
```css
/* Neutral surfaces: use regular text tokens */
.wp-block-awx-comment {
  background: var(--awx-color-surface-sunken);
  color: var(--awx-color-text);  /* dark text on light neutral */
}

/* Brand-subtle surface: use regular text tokens (tint is subtle enough) */
.wp-block-awx-aside {
  background: var(--awx-color-surface-brand-subtle);
  color: var(--awx-color-text);
}

/* Full brand surface: use on-primary (high contrast needed) */
.wp-block-awx-callout {
  background: var(--awx-color-surface-brand);
  color: var(--awx-color-on-primary);
}
```

### Hover Shadow Escalation
Cards go up one shadow level on hover:

```css
.wp-block-awx-card {
  box-shadow: var(--awx-shadow-1);
  transition: box-shadow var(--awx-duration-fast) var(--awx-ease-standard);
}
.wp-block-awx-card:hover {
  box-shadow: var(--awx-shadow-2);
}
```

### Focus Visible Ring
All interactive elements need focus state:

```css
.wp-block-awx-button:focus-visible {
  outline: 2px solid var(--awx-color-focus-ring);
  outline-offset: 2px;
}
```

### Content Width Constraint
Text blocks should constrain their width for readability:

```css
.wp-block-awx-text-block p {
  max-width: var(--awx-width-measure);
}
```

### Vertical Rhythm Inside Blocks
Use the stack token for consistent spacing between child elements:

```css
.wp-block-awx-card > * + * {
  margin-block-start: var(--awx-space-stack);
}
```

---

## Checklist Before Shipping a Block

- [ ] Wrapped in `@layer blocks` (or `@layer sections` for sections)
- [ ] Class prefixed with `wp-block-awx-` (or `wp-section-` for sections)
- [ ] Zero hardcoded color, size, spacing, radius, or shadow values
- [ ] All colors use role tokens (not primitives)
- [ ] All transitions use `--awx-duration-*` and `--awx-ease-*`
- [ ] Focus-visible state defined for interactive elements
- [ ] On-color pattern used for colored backgrounds
- [ ] No `!important` anywhere
- [ ] No `--wp--preset--*` references anywhere
- [ ] No bare element selectors (everything scoped to block class)
- [ ] One CSS file, one block
- [ ] Works without JavaScript (CSS-only styling)
