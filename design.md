# LunAR Design System — Agent Guide

> **Audience:** AI coding agents (Claude Code, Cursor, Copilot, etc.) generating or modifying client-facing websites for LunAR, an Indonesian AR web agency currently focused on furniture and lifestyle-goods businesses (markerless WebAR product previews).
>
> **Purpose:** Every LunAR client gets a different brand color, different products, different copy — but every LunAR site should *feel* like it was made by the same agency. This document is the constant "DNA" that travels with every project, plus the rules for how the adaptive layer (client's own colors, fonts, imagery) plugs into it. Read this before writing any UI code for a LunAR client site.
>
> This is a design/frontend reference only. It does not cover security, hosting, 3D pipeline, or backend architecture — those live elsewhere (see `security.md`, including its Cloudflare Pages/Worker/R2 stack description).

---

## 1. How to use this file

- **DNA (Section 3–7):** Never changes between clients. If a client's brand seems to conflict with a DNA rule, adapt the client's expression of it — don't break the rule.
- **Adaptive layer (Section 2):** Re-derived per client from their one brand color and product photography. This is what actually changes site-to-site.
- **AR-specific patterns (Section 8):** LunAR's real differentiator. Every client site touches these — treat them as house signature, not optional flourish.
- When in doubt: an unfamiliar visitor should be able to tell "this looks aesthetic, I want to know more about the agency behind it" — that reaction is the actual success metric, not pixel-perfection to any one client's mockup.

---

## 2. Adaptive layer — set once per client, at project start

For every new client project, an agent should establish these four things before writing a single component, then treat them as fixed tokens for that project:

### 2.1 One primary brand color → full ramp
Ask for or infer **one** brand color from the client (their logo, existing signage, or category convention — e.g. warm wood tones for furniture, sit within that world unless told otherwise). Generate an 11-step ramp from it:

```
50   100   200   300   400   500   600   700   800   900   950
lightest ─────────────────────────────────────────→ darkest
```

- `500`/`600` = primary actions, links, focus rings
- `50`/`100` = tinted backgrounds, subtle section dividers
- `900`/`950` = body text on light backgrounds, dark headings
- Never introduce a second competing "loud" color. A second color is allowed only as a **semantic** color (see 2.3).

### 2.2 Typography pairing (one decision, then lock it)
Pick **exactly one** of these two patterns per project — never more than 2 font families total:
- **Display + body**: one serif/display font for H1/H2 (character, warmth — e.g. Playfair Display) + one clean sans for everything else (e.g. Nunito, DM Sans, Plus Jakarta Sans, Montserrat).
- **Single sans, two weights**: one sans family used everywhere, differentiated by weight/size only. Safer default for dashboards, admin views, or clients who want a more modern/tech feel rather than "heritage craft."

Font sizes: **max 6 sizes** for a marketing/landing site (roughly 64/42/32/20/16/14). Dense, data-heavy views (admin, order tracking) shrink that range further — nothing larger than ~24px, because information density matters more than drama there.

Typography "pro" tip to always apply on large display text: tighten letter-spacing by about **-2% to -3%** and set line-height to **~110–120%**. Loose default line-height on big headlines reads as unpolished.

### 2.3 Semantic colors (fixed meaning, independent of brand color)
- Blue → trust / info
- Red → danger / error / urgency
- Yellow/amber → warning
- Green → success **and** WhatsApp/contact actions (Indonesian SMB customers expect the WhatsApp-green CTA; don't fight this convention even if it clashes slightly with brand color)
- Use these only for their meaning, never as decoration.

### 2.4 Imagery style
Real product photography > illustration > emoji placeholders, in that order of priority for anything client-facing. Emoji/gradient placeholders (as seen in the furniture demo) are acceptable only for early prototypes shown to a prospective client before real assets exist — flag them as temporary in code comments if used.

---

## 3. Spatial system (fixed)

- **4-point base grid.** All spacing, padding, and gaps are multiples of 4px (4, 8, 12, 16, 20, 24, 28, 32, 36, 40...). Not because multiples look inherently better — because it keeps every screen visually consistent with every other screen.
- Default gap between distinct stacked elements (sections, cards in a list): **32px**.
- Elements that conceptually belong together (a label + its value, an announcement bar + its headline) get **tighter** spacing than elements that don't — spacing itself is a hierarchy signal, not just tidiness.
- Grid columns are a *guideline*, not a cage: 12-column desktop / 8-column tablet / 4-column mobile is the default for structured, repeating content (product grids, blog listings). Custom hero/landing sections are allowed to break the grid intentionally. Don't force alignment where it fights the design.
- Button padding rule of thumb: horizontal padding ≈ **2× the vertical padding** (e.g. 16px vertical / 32px horizontal).
- Icon sizing: match the icon's box size to the font's line-height it sits next to (commonly 24px for body text, ~15–16px font / 24px line-height contexts). Icons should never dominate or float independently of the text baseline they accompany.

---

## 4. Hierarchy & signifiers (fixed principle, always apply)

Every screen should be readable without instructions, because the UI itself signifies what's related, selected, active, or inert:

- **Grouping = relationship.** A shared container tells the user two things belong together. A container-within-a-container (e.g. a highlighted pill inside a segmented control) tells the user "this one is currently selected."
- **Graying out = disabled.** Never gray out something that's actually clickable, and never leave a genuinely disabled action looking fully active.
- **Size, position, and color drive importance**, in that order of usual impact. The single most important piece of information on a card (price, product name, primary CTA) should be the largest/boldest/highest-contrast element, positioned first (usually top or top-right for price/value figures).
- **Contrast creates hierarchy**, not decoration for its own sake. If everything is bold and colorful, nothing is.
- Prefer icons + alignment over spelled-out labels when the relationship is spatial or directional (e.g. an origin/destination pair shown as two pin icons with a connecting line, rather than "From: X, To: Y").

---

## 5. Depth & surfaces (fixed rules, values adapt to brand ramp)

### Light mode
- Shadows are the primary depth tool. Default to **low opacity + generous blur** — if a user notices the shadow before the content, the shadow is too strong.
- Shadow strength scales with elevation: cards/list items need the least; anything floating above other content (popovers, modals, toasts) needs progressively more.
- Inner + outer shadow combinations can simulate tactile, "pressed" buttons where that fits the brand (e.g. a heritage/craft furniture brand may want tactile buttons; a modern tech-forward client may not).

### Dark mode
- No shadows for depth (they don't read on dark backgrounds). Depth instead comes from **surfaces being lighter than the page background** as they elevate (card lighter than page, popover lighter than card).
- Avoid a light/bright border around cards — it fights this technique and adds harsh contrast. A subtle, dimmed border (matching the surface tone) reads better than a bright one.
- Any chip, badge, or accent that was designed for light mode must be desaturated/dimmed for dark mode, then flip the surrounding text to lighter for contrast.
- Dark mode is not just "invert the light mode colors" — it needs its own pass through this section.

---

## 6. Interactive states (fixed, mandatory)

Every clickable element ships with, at minimum:
1. Default
2. Hover (desktop) / equivalent tap feedback (mobile)
3. Active / pressed
4. Disabled

Add as needed:
- Loading (spinner, disables interaction, no layout shift)
- Success (transient — auto-dismiss, don't require a user action to clear a confirmation)

Inputs additionally require:
- Focus (visible ring/border in the brand's primary color)
- Error (red border + inline message)
- Warning (amber border + inline message, for non-blocking issues, e.g. "this email already exists — log in instead?")

**Rule:** if the user does anything, something visibly responds. No dead taps, no clicks that give zero feedback while a background action is happening.

---

## 7. Microinteractions & overlays (fixed principle)

- A state change alone (e.g. a button darkening on click) confirms the *click*, not the *outcome*. When an action has a real result (copied to clipboard, added to cart, message sent), add a distinct microinteraction for the outcome itself — a toast, a sliding chip, a checkmark — not just a button state.
- Any text laid over a photo needs a legibility treatment, not just "add text and hope": prefer a **linear gradient** from transparent to the surface/dark tone behind the text, and for extra polish add a **progressive blur** on top of the gradient. Never leave a full-bleed photo with raw text stacked on top of unpredictable image contrast — this is a recurring failure mode to explicitly check for in hero sections and promo cards.
- Full-screen dark overlays are the blunt-instrument fallback; prefer the gradient/blur approach whenever the image itself should stay visible and premium-feeling (which is most of the time for furniture — the product photo is the sales pitch).

---

## 8. AR-specific UI patterns (LunAR's signature — apply on every project)

These are what should make a visitor recognize "an AR agency built this," independent of client branding:

- **Markerless only.** LunAR's product is markerless WebAR via `<model-viewer>`. Do not build marker-based/image-tracking AR flows (MindAR, targets.mind, etc.) — that experiment was for a different vertical and is out of scope going forward.

- **Media-priority rendering logic on the detail view — mandatory, applies to every product.** A product's detail page can be in one of four states, and each has exactly one correct default render:

  | Product has | Default view | Secondary access |
  |---|---|---|
  | 3D asset **and** 2D image(s) | 3D/AR viewer loads and renders first | 2D image(s) available via a secondary toggle or thumbnail strip alongside the viewer (right-hand side on desktop, below on mobile) — never hidden behind more than one tap/click |
  | 2D image(s) only | Standard 2D image gallery viewer | — |
  | 3D asset only | 3D viewer renders directly | — |
  | 3D asset present but fails to load, or is missing from the bucket at request time | Falls back to the product's primary 2D image | No raw error text, no broken layout, no indefinitely-spinning loader — see below |

  A visitor should never have to know or care whether a given product currently has a working 3D asset; the page should just show the best available thing. This means the fallback path in row 4 is not an edge case to handle "if there's time" — it's a required state, because assets move in and out of R2 over a catalog's life (new products added before their model is ready, a re-export in progress, a temporarily broken upload). Ship the fallback and the happy path together, in the same pass, per `security.md` §7.

- **AR CTA gets its own visual treatment**, distinct from primary/secondary buttons: an outlined pill in the brand accent color, never filled solid like the primary CTA. This visually communicates "this is a different *kind* of action" (launching a 3D/camera experience) rather than a normal navigation click. This CTA only appears once the model has actually loaded — don't show an "AR" affordance for a model that isn't ready yet, since that's a dead tap (see §6).
- **Dual-asset requirement stays invisible to the user.** Every AR-enabled product needs both a `.glb` (Android/Chrome) and a `.usdz` (iOS Quick Look) asset behind a single button — the visitor never sees or chooses a platform, the button just works.
- **Loading/transition states around camera or 3D launches are mandatory**, not optional polish: a branded loading screen (logo + short reassuring copy, e.g. "menyiapkan pratinjau 3D..." style guidance) covers the gap while the AR/3D experience initializes. Never let a user stare at a blank or frozen viewport — and if loading fails outright, that state must resolve into the 2D fallback above, not stay frozen on the loading copy forever.
- **Product cards are the entry point to AR, not an afterthought.** Card hierarchy: product photo (largest, top) → name (bold) → price (top-right or near the image, brand accent color, distinct from body text — same "price should stand out" rule as any e-commerce card) → AR/preview CTA clearly visible without scrolling.
- **Contact conversion stays WhatsApp-first** for Indonesian SMB clients: green, recognizable WhatsApp iconography, pre-filled message text per product. Don't reinvent this into a generic "Contact us" form — it will underperform culturally.
- **Badges** ("Terlaris," "Baru," "Premium," or client-equivalent) are small pill chips, semantic-colored only when they carry real meaning (new/promo), otherwise neutral/brand-tinted.

---

## 9. Language & copy defaults

- Default microcopy language: **Bahasa Indonesia**, matching the client's own customers, unless a client specifically serves an international/English-speaking market.
- Currency formatting: `Rp 1.250.000` style (period as thousands separator), not `$` unless the client is non-Indonesian.
- Tone: warm and craft-forward for furniture/artisan clients (premium but approachable, not corporate-cold) — adjust tone per client's actual positioning, but avoid generic stock-agency copy ("Solutions for your business needs").

---

## 10. Anti-patterns — check before shipping

- ❌ More than 2 font families, or more than 6 distinct font sizes on a single marketing page.
- ❌ A second "loud" color competing with the brand color for attention.
- ❌ Any interactive element with no hover/active/disabled state.
- ❌ Text over an unprocessed photo with no gradient/blur/overlay treatment.
- ❌ A shadow strong enough to be the first thing noticed on a card.
- ❌ Dark mode built by simply inverting light-mode values.
- ❌ An AR button visually identical to a normal navigation/primary button, or one that's visible before the model has actually loaded.
- ❌ Marker-based AR flows.
- ❌ A product with a 3D asset defaulting to showing the 2D image first, with 3D buried as the secondary option (the priority is reversed — 3D leads when it exists).
- ❌ A failed or missing 3D asset leaving the detail view stuck on a loading state, a blank canvas, or a raw error instead of the 2D fallback.
- ❌ A disabled-looking element that is actually clickable, or vice versa.
- ❌ Spacing values that aren't multiples of 4px.
- ❌ A generic "Contact Us" form replacing the WhatsApp CTA without client instruction to do so.

---

## 11. Starter token template (fill per client)

```css
:root {
  /* --- adaptive: regenerate per client from their one brand color --- */
  --brand-50:  #___;
  --brand-100: #___;
  --brand-200: #___;
  --brand-300: #___;
  --brand-400: #___;
  --brand-500: #___;   /* primary actions */
  --brand-600: #___;
  --brand-700: #___;
  --brand-800: #___;
  --brand-900: #___;   /* body text on light bg */
  --brand-950: #___;

  /* --- fixed semantic colors, do not change --- */
  --success:  #16A34A;
  --warning:  #D97706;
  --danger:   #DC2626;
  --info:     #2563EB;
  --whatsapp: #25D366;

  /* --- fixed spatial system --- */
  --space-1: 4px;  --space-2: 8px;  --space-3: 12px; --space-4: 16px;
  --space-5: 20px; --space-6: 24px; --space-8: 32px; --space-10: 40px;
  --radius: 16px;
  --shadow-sm: 0 4px 24px rgba(0,0,0,.10);
  --shadow-lg: 0 8px 40px rgba(0,0,0,.16);

  /* --- adaptive: pick one pairing, fill in chosen fonts --- */
  --font-head: '___', serif;   /* or drop this line for single-sans pattern */
  --font-body: '___', sans-serif;
}
```

---

## 12. Pre-ship checklist for agents

- [ ] One brand color, full ramp generated, no second loud color introduced
- [ ] ≤2 font families, ≤6 font sizes (marketing) / tighter range (dense views)
- [ ] Large headings: letter-spacing ~-2–3%, line-height ~110–120%
- [ ] Spacing audited to 4px multiples
- [ ] Every button/input has default, hover, active, disabled (+ focus/error for inputs)
- [ ] Any photo-with-text has gradient or blur legibility treatment
- [ ] Dark mode (if built) uses lighter-surface elevation, not inverted shadows
- [ ] AR CTA visually distinct from primary/secondary buttons, markerless only, only shown once loaded
- [ ] For every product: correct default media per §8's priority table (3D-first when both exist, 2D-only when that's all there is, verified 2D fallback when 3D fails/missing) — tested against at least one product in each state, not just the happy path
- [ ] WhatsApp CTA present and pre-filled per product
- [ ] Copy in Bahasa Indonesia with `Rp` formatting, unless client is international
