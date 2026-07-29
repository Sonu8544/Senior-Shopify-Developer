# Shopify Product Page Custom Section — Submission

**Assignment:** Senior Shopify Developer – Technical Assignment (Pilgrim)
**Deliverable:** A custom, responsive Shopify Product Page section built with Liquid, HTML, CSS & vanilla JavaScript.

| | |
|---|---|
| **Main file** | [`sections/custom-product-info.liquid`](sections/custom-product-info.liquid) |
| **Lines of code** | ~909 (Liquid + scoped CSS + vanilla JS + JSON schema — single self-contained file) |
| **Dependencies** | None (no jQuery, no libraries for the section's own logic) |
| **Approx. time taken** | ~3-4 hours (build + iterations + polish + review) |

---

## 1. How to install & preview

1. The section file is already in `sections/custom-product-info.liquid`.
2. In the **Theme Editor** → open a Product template → **Add section** → **"Custom Product Info"**.
3. In the section settings, optionally pick a **Product** (leave empty on a product template to use the current product), set the **Accent color**, and edit the **"Why Customers Love This Product"** bullet points.
4. Save.

The section is **self-contained** — all markup, styles (`{% style %}`) and script (`<script>`) live in one file and are scoped to the section instance, so it can be dropped into any theme without side effects.

---

## 2. Requirements checklist

### ✅ Core requirements

| # | Requirement | Done | Where / How |
|---|---|:--:|---|
| 1 | Responsive product information section | ✅ | CSS Grid: 1 column on mobile, 2 columns ≥ 750px. Fluid type via `clamp()`. |
| 2 | Product **Title** | ✅ | `<h1 class="cpi__title">{{ product.title }}</h1>` |
| 2 | Product **Price** | ✅ | `{{ current_variant.price | money }}` — updates live on variant change |
| 2 | **Variant Selector** | ✅ | Accessible pill/button group (not a dropdown) with availability handling |
| 2 | **Quantity Selector** | ✅ | `− / +` stepper + typed input, clamped to min 1 |
| 2 | **Add-to-Cart** button | ✅ | AJAX submit to `routes.cart_add_url` |
| 3 | Block titled **"Why Customers Love This Product"** | ✅ | `.cpi__love` block with heading |
| 3 | **Exactly 3** editable bullet points | ✅ | Three settings + `limit: 3` guard so never more than 3 |
| 4 | Editable via **Metafields OR Section Settings** | ✅✅ | **Both** implemented — see §4 |
| 5 | Clean, responsive, optimised for mobile + desktop | ✅ | Section-scoped CSS, no layout shift, lazy images |

### ✅ Bonus requirements

| Bonus | Done | How |
|---|:--:|---|
| Loading state on Add-to-Cart | ✅ | Button shows animated **"Adding…"** → **"Added ✓"** → reverts |
| Validation if no variant selected | ✅ | Blocks submit + shows an inline error message (see §5) |
| Optimised CSS / avoid unnecessary JS | ✅ | One scoped `{% style %}`, ~200 lines of dependency-free JS |

### ⭐ Extra (senior-level additions)

- AJAX add-to-cart that **refreshes the theme's header cart count + slide-out drawer** (integrated with the Halo/Ella theme's own cart API).
- **Variant availability** — unavailable option combinations are auto struck-through and disabled.
- **Image gallery** with a responsive thumbnail slider (max 6 on desktop, 4 on mobile; extras scroll).
- **Accent colour** is a live, theme-editor-driven CSS variable.
- **Accessibility** — `role="radiogroup"`/`radio`, `aria-checked`, `aria-live` status region, labelled controls, `prefers-reduced-motion` support.
- Graceful states — single-variant products, no-image fallback, empty-product placeholder in the editor.

### 📋 Submission items (manual)

| Item | Status |
|---|:--:|
| GitHub repo link / ZIP | ⬜ push / zip the repo |
| Loom video / screenshots | ⬜ record a short walkthrough |
| Approximate time taken | ✅ ~3-4 hours (noted above) |

---

## 3. What is dynamic / configurable (Theme Editor)

All of the following are editable by the merchant **without touching code**, defined in the section's `{% schema %}`:

| Setting (id) | Type | Default | Purpose |
|---|---|---|---|
| `product` | product | current product | Which product to render (optional on product templates) |
| `accent_color` | color | `#000000` | Drives selected pills, Add-to-Cart button, and bullet check icons (live CSS variable `--cpi-accent`) |
| `love_title` | text | "Why Customers Love This Product" | Block heading |
| `love_point_1/2/3` | text | 3 sample lines | The three bullet points (fallback source) |
| `use_metafield` | checkbox | `true` | Prefer a product metafield over the text fields |
| `metafield_namespace` | text | `custom` | Metafield namespace to read |
| `metafield_key` | text | `love_points` | Metafield key to read |

**Dynamic runtime behaviour (JavaScript):**
- Price, availability and the "Option: value" labels update instantly when a variant pill is clicked.
- Unavailable combinations recompute on every selection.
- Cart count + drawer refresh after a successful add.
- Thumbnail slider shows/hides based on how many images fit.

---

## 4. Editable bullet points — dual source (Metafield + Settings)

The block reads its 3 points from **either** a product metafield **or** the section settings, with a safe fallback:

```liquid
assign mf_ns  = section.settings.metafield_namespace | default: 'custom'
assign mf_key = section.settings.metafield_key | default: 'love_points'
assign love_mf = product.metafields[mf_ns][mf_key]      # bracket access = dynamic key
if section.settings.use_metafield and love_mf != blank
  assign meta_points = love_mf.value
endif
```

Render logic:

```liquid
{%- if meta_points and meta_points.size > 0 -%}
  {%- for point in meta_points limit: 3 -%}<li>{{ point }}</li>{%- endfor -%}
{%- else -%}
  ... section.settings.love_point_1/2/3 (fallback) ...
{%- endif -%}
```

**Metafield to create in Admin** (Settings → Custom data → Products → Add definition):

| Field | Value |
|---|---|
| Name | *(anything, e.g. "love points")* |
| Namespace and key | `custom.love_points` |
| **Type** | **List of single line text** (`list.single_line_text_field`) |

Then add up to 3 entries on any product and tick **"Use product metafield for the bullet points"** in the section. If the metafield is empty or the checkbox is off, the three text-field settings are used instead.

> Note: Shopify's "dynamic source" (🔗) picker cannot expand a *list* metafield into separate bullets — that requires a Liquid loop, which is why the namespace/key approach is used here.

---

## 5. Validation handling

| Case | Behaviour |
|---|---|
| No valid variant id present | Submit blocked, inline error: *"Please select an available option before adding to cart."* |
| Selected combination is sold out / disabled | Submit blocked, inline error: *"Selected option is sold out."* |
| Combination does not exist | Add-to-Cart disabled, message: *"This combination is not available."* |
| Quantity typed as 0 / negative / non-numeric | Clamped to a minimum of **1** (on input change **and** on submit) |
| `/cart/add.js` returns an error | Caught; shows the API's error description (or a generic message) |
| Sold-out on load | Add-to-Cart rendered `disabled` with "Sold out" label |

The status message uses an `aria-live="polite"` region so screen readers announce it.

---

## 6. How the key features work

### Variant selector (pills)
- Rendered from `product.options_with_values`; each value is a `<button role="radio">`.
- A hidden `<select name="id">` holds every variant with `data-cpi-option1/2/3`, `data-cpi-price`, `data-cpi-available`.
- On click, JS collects the selected values, finds the matching variant, updates price/availability, and writes the variant id into the hidden select (which is what the cart form submits).

### Add-to-Cart (AJAX + loading)
- `submit` is intercepted (`preventDefault`), validated, then `fetch('…/cart/add.js')` is called.
- Button enters `.is-loading` ("Adding…"), then `.is-added` ("Added ✓"), then reverts after 1.8s.
- On success, `refreshThemeCart()` calls the theme's own `Shopify.getCart()` + `window.sharedFunctions.updateSidebarCart()` to update the header count and open the drawer (with a `/cart.js` fallback if those globals are unavailable).

### Image gallery
- Main image + thumbnail row (bottom on both mobile and desktop).
- Thumbnail widths are computed with `calc()` from a CSS variable `--cpi-thumb-count` (4 on mobile, 6 on desktop); extra images scroll with `scroll-snap`.
- Clicking a thumbnail cross-fades the main image (`srcset` for retina) and marks the active thumb.

### Styling & scoping
- Every rule is prefixed with `#cpi-{{ section.id }}` and colours come from CSS variables, so multiple instances on one page never clash and theming is one-line.

---

## 7. Files in this submission

| File | Role |
|---|---|
| [`sections/custom-product-info.liquid`](sections/custom-product-info.liquid) | The entire feature — Liquid markup, scoped `{% style %}`, vanilla `<script>`, and `{% schema %}` settings. Self-contained. |
| [`SUBMISSION.md`](SUBMISSION.md) | This document. |

No other theme files were modified — the section is fully portable.

---

## 8. Browser support & performance notes

- Vanilla ES5-safe JS (uses `fetch`, `Promise.finally`) — works in all modern evergreen browsers.
- No external requests, no libraries → zero added bundle weight.
- Images: `loading="lazy"`, explicit `width`/`height` and `srcset`/`sizes` → no layout shift, retina-ready.
- `prefers-reduced-motion` disables animations/transitions.
- CSS is scoped and minimal; no `!important` abuse (only the `[hidden]` utility).

---

## 9. Optional future enhancements (not bugs)

- Arrow-key navigation across variant pills (roving `tabindex`) for stricter radiogroup semantics.
- Auto-select the first available variant when the current combination becomes sold out.
- Spam-click guard on Add-to-Cart during the "Added ✓" confirmation window.
 
