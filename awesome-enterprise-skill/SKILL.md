---
name: awesome-enterprise-shortcodes
description: Programming guidelines, Single-File Component (SFC) dynamic blocks, Website Assembly Language (WAL) engine, pattern bridge, and Agent MCP reference for the Awesome XP WordPress plugin.
---

# Awesome Enterprise & Dynamic Gutenberg Blocks (DGB) Master Skill

Awesome XP is an enterprise-grade WordPress application development platform and agentic authoring environment. It provides a low-code shortcode processor, a Single-File Component (SFC) dynamic block compiler, a declarative Website Assembly Language (WAL) engine, an unsynced pattern bridge, and a 31-ability Model Context Protocol (MCP) registry for autonomous agents.

---

## 1. Core Processor & Shortcode Execution Lifecycle

The Awesome Processor parses low-code templates in four sequential stages:

1. **Comment Stripping**: Multi-line comments `//* ... *//` are removed.
2. **Conditions (`c.*`)**: Evaluates conditional attributes (`c.cond`, `c.eq`, `c.empty`, `c.user_can`, `c.device`). If false, skips execution.
3. **Service Execution**: Executes the service tag (e.g. `[mysqli.multi.fetch.rows]`, `[template.get]`, `[curl.api.get]`).
4. **Modifiers (`m.*`)**: Chains data transformations sequentially (`m.json_decode`, `m.trim`, `m.implode_on`, `m.words`).
5. **Outputters (`o.*`)**: Directs the final value (`o.set='template.var'`, `o.merge_with`, `o.destroy`, `o.echo`). If omitted, prints directly to the output stream.

### Scoping Hierarchy
- **Module Scope (`module.*`)**: Active during module execution. Read/write with `[module.get]`, `[module.set]`.
- **Template Scope (`template.*`)**: Local variables within template files. Set with `o.set="template.x"`, get with `[template.get x /]`.
  > [!IMPORTANT]
  > Since `template.get` automatically prepends `template.`, calling `[template.get template.foo /]` is invalid because it resolves to `template.template.foo`. Always write `[template.get foo /]`.
- **Block Scope (`do.*`)**: Scoped to `[do.@local_var] ... [/do.@local_var]`. The variable `@local_var` is destroyed immediately on block exit.
- **Loop Scoping (`loop.@iterator`)**: Array iteration items MUST be accessed via `.item`: `[env.@iterator.item.field /]` or `{@iterator.item.field}`. Direct access like `{@iterator.field}` is strictly forbidden.

### Syntax, Typecasting & Attribute Boundaries
- **Attribute Typecasting Prefixes**:
  - `int:<val>`: Casts to integer (`int:42`).
  - `num:<val>` / `float:<val>`: Casts to float (`num:19.99`).
  - `str:<val>`: Casts to string (`str:active`).
  - `bool:<val>`: Converts `"false"` to `false`, else boolean (`bool:false`).
  - `comma:<a,b,c>`: Converts comma list to string array (`comma:admin,editor`).
  - `get:<path>`: Fetches env value while preserving native PHP type.
  - `x:<service>`: Executes inline service, preserving return type (`x:request2.get id`).
- **Curly Braces Parsing Boundary**:
  `{path.key}` and `{{service.name}}` are evaluated **strictly within shortcode attributes**, never inside shortcode body content. Inside content, write full tags (e.g. `[env.get path /]`).
- **Disallowed Syntax**:
  - No nested curly braces: `{user.{field}}` is invalid.
  - No nested brackets in attributes: `[env.set x=[esc.str y] /]` fails parsing. Store the escaped value in a temporary variable first.
  - No single quotes in unnamed main attributes: `[str.create 'text' /]` is invalid. Use double quotes `[str.create "text" /]` or bare words `[str.create text /]`.

### Core Safety, Escaping & Stream Trapping
- **Zero-Tolerance SQL Escaping**: Never interpolate raw variables or curly braces directly into SQL. Always sanitize inputs via the `esc` library and assign to MySQL session variables:
  ```sql
  [mysqli.multi.fetch.rows o.set="template.posts"]
    SET @cat = [esc.str request.category /];
    SET @status = 'publish';
    SELECT ID, post_title FROM [env.get db.prefix /]posts
    WHERE post_status = @status AND post_name = @cat;
  [/mysqli.multi.fetch.rows]
  ```
- **MySQL LIMIT Clause**: MySQL session variables (e.g. `@limit`) cannot be used in `LIMIT`. Use hardcoded integer literals in the query template.
- **Table Prefixes**: Never hardcode `wp_posts`. Always use `[env.get db.prefix /]posts`.
- **Untrapped Output Stream Leaks**: Any service returning data (`mysqli.fetch.rows`, `curl.api.get`, `arr.create`) prints raw PHP debug strings directly to output unless captured via `o.set="path"` or suppressed via `o.destroy=yes`.
- **Agent Allowlist Guardrails**: Enforced via `Allowlist::is_service_allowed`. In block templates (`context='block'`), write queries (`mysqli.cud`, `mysqli.multi.cud`), raw network calls (`curl.*`), and sensitive authentication (`wp.signon`) are disallowed.

### Complete Shortcodes & Limitations Reference
For the exhaustive catalog of all 450+ registered services across Core Lang (`arr.*`, `str.*`, `int.*`, `num.*`, `date.*`, `cond.*`, `while.*`, `arr_each.*`), Utilities (`request2.*`, `clean.*`, `unique.*`, `cookie.*`), Caching (`global_cache.*`, `redis.*`, `mysql.cache.*`), and Network/Integrations:
- Consult: **[Awesome Shortcodes & Limitations Reference](references/shortcodes.md)**
- Quick Syntax Reference: **[Syntax Cheat Sheet](references/syntax_cheat_sheet.md)**

---

## 2. Dynamic Gutenberg Blocks (DGB) Architecture

Dynamic blocks are stored as Single-File Component (SFC) posts in the `aw_gb_blocks` post type and compiled into cascade-layered Gutenberg blocks.

### Single-File Component (SFC) Template Standard
Each block SFC defines up to five template sections:

```html
[templates.add register]
  [gt_blocks.register]
    [config]
      [name]block-slug[/name]
      [title]Display Title[/title]
      [icon]dashicon-name-or-svg[/icon]
      [category]layout[/category]
      [description]Short description[/description]
      [sidebar_tabs]section_style,section_advanced[/sidebar_tabs]
    [/config]
    [render_service   _value="gb_blocks.block-slug.render" /]
    [controls_service _value="gb_blocks.block-slug.controls" /]
    [styles_service   _value="gb_blocks.block-slug.styles" /]
    [scripts_service  _value="gb_blocks.block-slug.scripts" /]
  [/gt_blocks.register]
[/templates.add]

[templates.add controls]
  [arr.create o.set='template.controls']
    [sections new]
      [name _value='section_content' /]
      [title _value="Content" /]
      [fields new]
        {
          "type": "text",
          "name": "title",
          "label": "Heading",
          "attr_name": "content.title",
          "default": "Default Heading"
        }
      [/fields]
    [/sections]
    [sections new]
      [name _value='section_style' /]
      [title _value="Appearance" /]
      [sidebar _value="true" /]
      [fields new]
        {
          "type": "select",
          "name": "surface",
          "label": "Surface",
          "attr_name": "style.surface",
          "default": "surface",
          "options": [
            { "label": "Default Surface", "value": "surface" },
            { "label": "Raised", "value": "surface-raised" },
            { "label": "Sunken", "value": "surface-sunken" }
          ]
        }
      [/fields]
    [/sections]
  [/arr.create]
  [template.return template.controls /]
[/templates.add]

[templates.add styles]
@layer blocks {
  .wp-block-dgb-block-slug {
    /* Scoped CSS using --awx design tokens */
  }
}
[/templates.add]

[templates.add render]
<div [template.get wp.wrapper_attributes /]>
  <h2>[template.get content.title /]</h2>
  [template.get content.innerblocks /]
</div>
[/templates.add]
```

### Essential Block Authoring Rules
1. **Dynamic Root Wrapper**: The root HTML tag must ALWAYS receive `[template.get wp.wrapper_attributes /]`. Never write hardcoded `class="..."` on root tags.
2. **Cascade Layering (`@layer blocks`)**: All block CSS must be wrapped inside `@layer blocks { .wp-block-dgb-<slug> { ... } }`.
3. **Flat Attribute Namespaces**:
   - `content.*`: Text, badges, media, lists, query parameters (`content.title`, `content.badge`, `content.items`).
   - `style.*`: Surfaces, padding, alignment, column counts, layout variants (`style.surface`, `style.padding`, `style.columns`, `style.align`, `style.layout`).
4. **Automatic Modifier Classes**:
   The runtime automatically computes and injects modifier classes into `wp.wrapper_attributes`:
   - `style.surface` -> `has-surface-{value}`
   - `style.padding` -> `has-padding-{value}`
   - `style.columns` -> `has-cols-{value}`
   - `style.align` -> `{value}` (e.g. `is-aligned-left`)
   - `style.layout` -> `is-bento`, `is-reversed`
5. **Dynamic Sidebar Routing**:
   - Add `[sidebar _value="true" /]` to any controls section to place it in the Inspector Sidebar.
   - Set `"show_in_sidebar": true` on any individual field definition.
6. **InnerBlocks**: Render nested child blocks with `[template.get content.innerblocks /]`.

---

## 3. The Core 12 Blocks Catalog

Awesome XP provides 12 built-in foundation blocks seeded in `aw_gb_blocks` (`_awx_core_block = 1`):

| Block Slug | Title | Key Flat Attributes | Structural Elements |
| :--- | :--- | :--- | :--- |
| `awx-section` | Section Wrapper | `style.surface`, `style.width`, `style.padding` | Semantic `<section>` with inner content container |
| `awx-hero` | Hero Header | `content.badge`, `content.title`, `content.text`, `style.align`, `style.level` | Header lead section with CTA InnerBlocks slot |
| `awx-split` | 50/50 Split Showcase | `content.badge`, `content.title`, `content.text`, `style.layout` (`reversed`) | Two-column grid with content and media slots |
| `awx-feature-grid` | Feature Grid | `style.columns` (2, 3, 4), `style.layout` (`standard`, `bento`) | CSS Grid container for feature cards |
| `awx-post-grid` | Dynamic Post Query | `content.post_type`, `content.count`, `style.columns` | SSR database query using `mysqli.multi.fetch.rows` |
| `awx-pricing` | Pricing Matrix | `content.items` (repeater), `style.columns` | Multi-tier cards with pricing, badge, and feature lists |
| `awx-accordion` | FAQ Accordion | `content.items` (repeater) | Semantic HTML5 `<details>` and `<summary>` (zero-JS) |
| `awx-testimonials`| Testimonial Showcase | `content.items` (repeater), `style.columns` | Social proof quote cards with author attribution |
| `awx-cta` | Call to Action | `content.badge`, `content.title`, `content.text`, `style.surface`, `style.align` | High-conversion focal banner with actions slot |
| `awx-stats` | KPI Stat Highlights | `content.items` (repeater), `style.columns` | Metric number values and descriptive labels |
| `awx-logo-cloud` | Logo Cloud | `content.items` (repeater), `style.columns`, `style.grayscale` | Grayscale-filtered partner/client logo grid |
| `awx-steps` | Process Steps | `content.items` (repeater), `style.columns` | Numbered sequential roadmap items |

Core blocks are automatically seeded on activation/upgrade at `init` priority 20 (`AWX_CORE_BLOCKS_VERSION = '1.0.0'`).

---

## 4. Block Patterns & Pattern Bridge

Managed block patterns are stored in the WordPress `wp_block` post type and bridged through `AWX_Pattern_Bridge`.

### Metadata Contracts
- `wp_pattern_sync_status = 'unsynced'`: Required so instances in pages can be modified independently without altering the global master pattern.
- `_awx_pattern_managed = 1`: Identifies patterns managed by AWX.
- `_awx_pattern_slug`: Explicit pattern identifier. Supports dotted notation (e.g. `pricing.3_tier_standard`, `hero.standard_conversion`).
- Taxonomy: Categorized in `wp_pattern_category`.

### Import & Export Tooling
- **Admin UI Import Screen**: `?page=awesome-gt-block-import` handles JSON file uploads or pasted JSON bundles.
- **Admin UI Export Screen**: `?page=awesome-app-import-export` provides one-click export of `awx-patterns.json`.
- **WP-CLI**:
  ```bash
  wp awx patterns list
  wp awx patterns import /path/to/patterns.json
  wp awx patterns export [--output=/path/to/patterns.json]
  ```

---

## 5. Website Assembly Language (WAL) Engine

Website Assembly Language (WAL) is the declarative Intermediate Representation (IR) used by autonomous agents and LLMs to create, compose, and modify WordPress pages deterministically.

### WAL Document Schema

```yaml
page:
  title: "Modern Enterprise Compute"
  slug: "enterprise-compute"
  status: "publish" # or "draft"
  sections:
    - block: "awx-section"
      attrs:
        style.surface: "surface"
        style.padding: "default"
      inner:
        - block: "awx-hero"
          attrs:
            content.badge: "Next-Gen Cloud"
            content.title: "Deterministic Compute for AI Teams"
            content.text: "Deploy workloads globally without infrastructure drift."
            style.align: "is-aligned-left"
            style.level: "h1"
          inner:
            - block: "core/button"
              attrs:
                text: "Get Started"
                url: "/signup"
                className: "is-style-fill"

    - block: "awx-section"
      attrs:
        style.surface: "surface-sunken"
      inner:
        - pattern: "pricing.3_tier_standard"
          overrides:
            content.badge: "Transparent Tiers"
            style.columns: "3"
```

### Deterministic Transpilation (`WAL_Transpiler`)
- Compiles WAL documents into native Gutenberg block AST array nodes.
- Auto-prefixes blocks with `dgb/` if no namespace is specified (`awx-section` -> `dgb/awx-section`).
- Core blocks (e.g. `core/button`, `core/paragraph`) retain their `core/` prefix without double-prefixing.
- Resolves `pattern` nodes by querying `wp_block` via `_awx_pattern_slug`, parses blocks, applies `overrides` to root attributes, and splices child nodes directly into the AST.
- Serializes blocks via `serialize_blocks()` and creates the page via `wp_insert_post()` using `wp_slash()`.

### Post-Publish Round-Trip Decompilation (`WAL_Decompiler`)
- Reads published page `post_content` via `parse_blocks()`.
- Converts AST nodes back into clean WAL representation: strips `dgb/` prefixes, preserves flat attribute dictionaries, and retains nested block hierarchies for agent editing.

---

## 6. Agent MCP Abilities Reference (31 Abilities)

Awesome XP registers 31 standardized operations via `Abilities_Registry`:

| Capability Name | Ability Endpoint | Tier | Description & Payload Contract |
| :--- | :--- | :--- | :--- |
| **Compile WAL Page** | `awesome/wal-compile` | `publish` | Compiles a declarative WAL document into a published WordPress Page. `{ "wal": { "page": { ... } } }` |
| **Decompile Page to WAL** | `awesome/wal-decompile` | `read` | Decompiles an existing WordPress Page post_content into clean WAL. `{ "post_id": 104 }` |
| **Export Pattern Pack** | `awesome/pattern-export` | `read` | Exports all managed patterns into a JSON manifest bundle. `{ "path": "optional/file/path" }` |
| **Import Pattern Pack** | `awesome/pattern-import` | `author` | Imports a JSON pattern manifest into unsynced `wp_block` posts. `{ "manifest": [ ... ] }` |
| **Flush Block Cache** | `awesome/blocks-flush` | `author` | Invalidates block registry cache and recompiles CSS stylesheets. `{}` |
| **Read Guide** | `awesome/guide-read` | `read` | Returns reference guides (`overview`, `lifecycle`, `syntax`, `scopes`, `safety`, `css-tokens`, `block-css`, `wal-engine`, `patterns`, `pitfalls`). |

### WP-CLI Tooling Commands
```bash
wp awx blocks seed [--force]       # Seed or update the Core 12 blocks
wp awx blocks recompile [--slug=X] # Recompile cascade-layered CSS stylesheets
wp awx patterns list               # List all AWX-managed patterns
wp awx patterns import <file>      # Import pattern bundle manifest
wp awx patterns export [--output]  # Export managed patterns to JSON
```

---

## 7. Common Pitfalls & Lint Checklist

Before publishing templates or dynamic blocks, verify:
1. **Root Wrapper Check**: Root HTML element in `render` template outputs `[template.get wp.wrapper_attributes /]`.
2. **Cascade Layers**: All CSS in `styles` is wrapped in `@layer blocks { .wp-block-dgb-<slug> { ... } }`.
3. **Template Scope Prefix**: Calls to `template.get` use `[template.get var /]`, NOT `[template.get template.var /]`.
4. **Loop Iterator Scoping**: Iteration items are accessed via `.item` (`{@iterator.item.field}`).
5. **No Attribute Brackets**: Never nest square brackets in attributes (`[env.set x=[esc.str y] /]`); use a temp variable.
6. **SQL Escaping & LIMIT**: All query variables are passed via `[esc.*]` to MySQL session variables; `LIMIT` uses literals; queries use `[env.get db.prefix /]posts`.
7. **Untrapped Streams**: Capture data-returning services with `o.set="path"` or suppress with `o.destroy=yes`.
