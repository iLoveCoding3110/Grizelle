# Grizelle Design System

## Design intent

Grizelle is warm, elegant, and premium. The interface should feel like a curated wedding-souvenir catalogue: calm, spacious, tactile, and editorial rather than corporate or overly modern.

## Visual principles

- Use generous whitespace, fine dividers, restrained microcopy, and large serif headings.
- Let product photography lead. Cards should be quiet and should not compete with the images.
- Use soft ivory backgrounds and dark brown text. Avoid strong blue navigation bars, harsh blocks, or high-saturation UI treatments.
- Keep calls to action concise and visually deliberate.

## Palette

| Token | Value | Use |
| --- | --- | --- |
| `--ivory` | `#f8f5f0` | Primary page background |
| `--ink` | `#292522` | Headings, buttons, primary text |
| `--cocoa` | `#5a3820` | Warm secondary accents and labels |
| `--gold` | `#bc8a4c` | Premium accent, brand dot, eyebrow labels |
| `--sand` | `#e9e1d5` | Image placeholders and subtle panels |
| `--line` | `rgba(41,37,34,.12)` | Borders and specification dividers |

## Typography

- **Display:** Cormorant Garamond, used for the brand, product titles, hero headings, and section headlines.
- **Body:** Inter, used for navigation, descriptions, specifications, prices, and buttons.
- Display typography is compact, large, and slightly tight in letter spacing. Body typography is clear, quiet, and readable.

## Page structure

### Catalog page (`index.html`)

1. A transparent-feeling ivory sticky header with the Grizelle wordmark and WhatsApp action.
2. A full-width image hero with a dark warm overlay, large Indonesian headline, and short explanatory copy.
3. A responsive product grid. Each card contains:
   - Product image
   - Small uppercase category label
   - Serif product name
   - `Harga mulai dari Rp ...`
4. Consultation CTA section, explanatory brand paragraph, and location-oriented heading.
5. ENZO Group footer with contacts and map.

### Detail page (`detail.html`)

Use the Pentone-inspired two-column product layout:

- **Left:** a large, vertically framed primary product image.
- **Right:** category eyebrow, prominent dynamic product `<h1>`, description, specification rows, pricing table, and WhatsApp CTA.
- Specifications use thin horizontal rules with labels aligned left and values aligned right.
- Pricing uses one packaging section per tier with four visible quantity columns: 100, 300, 500, and 1,000 pcs.
- Variant images appear below the main two-column section when available.

## Responsive behavior

- Product catalog: 4 columns on wide desktop, 2 columns on mobile.
- Detail page: 2 columns on desktop and one stacked column below 760px.
- Pricing table: 4 columns on desktop and 2 columns on mobile.
- Contact footer: 4 columns on desktop and 2 columns on mobile.
- Preserve image aspect ratios and reduce spacing, never typography hierarchy, on smaller screens.

