# Grizelle Frontend Architecture

## Overview

This is a static website with no server, database, or build system. It is deployable to GitHub Pages, Netlify, or Vercel as static files.

## Files

| File | Responsibility |
| --- | --- |
| `index.html` | Catalog landing page, hero, 21-card product grid, consultation CTA, and ENZO footer. |
| `detail.html` | Dynamic product-detail view with specifications, per-piece pricing, variants, and WhatsApp CTA. |
| `asset/` | Main catalog images, one image per product card. |
| `asset/Hand Soap/`, `asset/kramik/`, `asset/Totebag/` | Variant images used only on product-detail views. |

## Catalog to detail routing

`index.html` has a `products` array containing the slug, category, title, start price, and image for every card. Each card routes to the shared detail template with a URL parameter:

```html
<a href="detail.html?item=keramik">...</a>
```

The URL parameter is read in `detail.html`:

```js
const q = new URLSearchParams(location.search).get('item') || 'handsoap';
```

This lets a single page render every product without making 21 separate HTML detail files.

## Lightweight product database

The `const items = { ... }` object in `detail.html` is the website's lightweight static database. Every slug in the catalog must have a matching key in `items`.

Each item contains:

```js
{
  cat: 'Tableware',
  name: 'Keramik Motif Mandarin',
  img: 'asset/Keramik.jpeg',
  description: '...',
  info: '...',
  minOrder: '100 pcs',
  spec: [['Material', 'Keramik'], ['Minimum order', '100 pcs']],
  tiers: [
    ['Plastik (Standar)', ['22.000', '20.000', '18.900', '17.500']]
  ],
  variants: ['asset/kramik/Piring Motif Mandarin.png']
}
```

The `tiers` values map in order to 100, 300, 500, and 1,000 pcs.

## Dynamic DOM rendering

After selecting an item, `detail.html` dynamically updates the existing DOM elements:

```js
document.getElementById('name').textContent = p.name;
document.getElementById('category').textContent = p.cat;
document.getElementById('mainImage').src = p.img;
```

Specification and pricing markup are created from `spec` and `tiers` with `map(...).join('')`. Variant markup is rendered only when an item has a non-empty `variants` array.

## Pricing behavior

- Products with approved source data use their real packaging tiers.
- Products without approved source data still provide a standard placeholder tier so the pricing-table layout always renders.
- The placeholder is visual continuity only and must be replaced when the price source is available.

## WhatsApp integration

The product-detail CTA creates a product-specific message and routes to the customer-service WhatsApp account using `wa.me`.

