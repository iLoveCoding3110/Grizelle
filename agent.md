# Agent Rules of Engagement for Future Brand Projects

## Before building

1. Read every provided reference file, spreadsheet, screenshot, asset folder, and prior website reference that is relevant to the request.
2. Inspect the actual workspace before assuming file names, image paths, or project structure.
3. Keep the approved brand aesthetic unless the user explicitly asks for a redesign.
4. Treat screenshots as visual requirements, not merely inspiration.

## Product data rules

1. Every catalog item must have a complete data record before it is linked from the landing page.
2. Never send a standard product to a generic fallback record or unrelated image.
3. The catalog slugs and detail-dictionary keys must match exactly. Verify this programmatically before delivery.
4. Use the product's real root catalog image in its card and detail view.
5. Use local variant images only on the related product's detail view, never in the main catalog unless asked.

## Detail-page rules

1. Always render the dynamic main title using an explicit DOM lookup:

   ```js
   document.getElementById('name').textContent = item.name;
   ```

   Do not rely on implicit global element variables such as `name`, because they can collide with browser globals.
2. Verify that `<h1 id="name">` exists and is visibly populated for at least one standard and one newly added product URL.
3. Every product must render a specification sheet with an information field and a minimum-order field.
4. Every product must render a pricing table. If approved pricing is unavailable, use clearly replaceable placeholder tiers so the layout is never replaced by an empty state, a gray error box, or missing rows.
5. Preserve the same image, title, detail, price-table, and WhatsApp layout for all catalog products.

## Visual rules

1. Use the brand's approved palette, typography, spacing, and responsive patterns.
2. Do not replace a warm premium style with a stark, corporate, or unrelated visual system.
3. Do not remove start prices, tier tables, product detail interactions, or variant galleries while adding a new section.
4. Build mobile layouts intentionally; do not merely shrink desktop layouts.

## Verification before handoff

1. Check that every local asset path exists.
2. Check that every landing-page slug has a matching detail-page record.
3. Open representative URLs for all data states, including a core priced product and a placeholder-priced product.
4. Confirm titles, images, specification rows, tier rows, WhatsApp links, and variant galleries render correctly.
5. Do not state that a change is complete until the required UI behavior has been verified.
