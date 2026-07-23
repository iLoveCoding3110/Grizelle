# company.md — [Client Name]

> **Audience:** AI coding agents building or modifying this client's site.
> **Purpose:** This file holds everything that's specific to *this* client — the facts `design.md` and `security.md` tell you to ask for or infer, collected in one place instead of scattered across a chat history.
> **Status:** TEMPLATE — every `[ ]` and `___` below must be filled in before an agent starts writing code for a real client. If a section is still blank when you reach it, halt and ask the human rather than guessing or inventing a placeholder.
>
> This file is the **adaptive layer**. `design.md` and `security.md` are fixed agency DNA and take precedence — if anything below conflicts with them (see "Deviations from DNA" at the bottom), that conflict gets flagged, not silently resolved either way.

---

## 1. Client Identity

- **Business name:** ___
- **Industry / product category:** ___ (e.g. furniture — jati wood, minimalist, artisan; or wedding souvenirs, lifestyle goods, etc.)
- **Tagline / one-line positioning:** ___
- **Logo file:** ___ (path, or "not yet provided — flag before shipping")
- **Target customer:** ___ (e.g. urban Indonesian homeowners, mid-to-premium segment)
- **Market:** ☐ Domestic (Bahasa Indonesia) ☐ International (English) — per `design.md` §9

---

## 2. Contact & Conversion

- **WhatsApp number:** ___ (format: `62xxxxxxxxxx`, no `+` or leading `0`)
- **Email:** ___
- **Physical address / showroom:** ___
- **Business hours:** ___
- **Instagram / other social:** ___

---

## 3. Brand Color & Typography

*(Fill once, then treat as locked for the project — see `design.md` §2.1–2.2)*

- **Primary brand color (hex):** ___ (source: logo / signage / category convention — note which)
- **Full 11-step ramp generated:** ☐ yes ☐ not yet
- **Typography pattern chosen:** ☐ Display + body ☐ Single sans, two weights
  - Display/heading font: ___
  - Body font: ___
- **Tone of copy:** ___ (e.g. warm heritage-craft vs. modern-clean — per `design.md` §9)

---

## 4. Product Catalog Scope

- **Number of SKUs at launch:** ___
- **Categories:** ___ (e.g. Kursi, Meja, Lemari, Rak, Sofa, Dekorasi; or Pouch, Handsoap, Keramik, etc.)
- **Currency / price format:** `Rp 1.250.000` style unless international client
- **Badges in use:** ___ (e.g. Terlaris, Baru, Premium — or client-equivalents)

---

## 5. Project Tier

*(Per `security.md` §2 — confirm before writing any backend-adjacent code)*

- **Current tier:** ☐ Tier A (static + Cloudflare edge proxy) ☐ Tier B (backend-enabled)
- **If Tier B, reason for graduating:** ___ (login dashboard / client-managed catalog with an admin UI / order tracking / ERP integration / other — name it; "we need to hide an API key" is *not* by itself a valid reason — see `security.md` §2.2)
- **Domain:** ___ (custom domain if any, in addition to the `*.pages.dev` domain in §6)

---

## 6. Infrastructure (Cloudflare)

*(Per `security.md` §2.1 and §5a — every Tier A project has these three identifiers. Do not proceed with code that references any of these until the real values are filled in here.)*

- **Cloudflare Pages project name:** ___
- **Cloudflare Pages URL:** ___ (e.g. `https://[client].pages.dev`)
- **Custom production domain (if any):** ___
- **Deployment method:** Direct Upload (per `agent.md` §4 / `security.md` §2.1) — confirm this hasn't drifted to a git-linked auto-build before assuming it: ☐ confirmed Direct Upload
- **Cloudflare R2 bucket name:** ___ (convention: `[client]-3d-asset`)
- **Cloudflare Worker proxy URL:** ___ (e.g. `https://[client]-3d.[account].workers.dev`)
- **Worker CORS scope:** ☐ restricted to this client's Pages/custom domain(s) per `security.md` §5a ☐ still wildcard `*` — flag if so, this is a gap to close, not the intended state for a new project

---

## 7. 3D Asset Status

*(Per `security.md` §6–7 — one row per product, or a link to wherever this is tracked if the catalog is large)*

| Product | .glb ready | .usdz ready | Decimated (Layer 2) | Watermarked (Layer 3) | 2D fallback image set |
|---|---|---|---|---|---|
| ___ | ☐ | ☐ | ☐ | ☐ | ☐ |

- **Client informed of HAKI / IP registration option (Layer 1)?** ☐ yes ☐ not yet
- Per `design.md` §8: every row above needs a 2D fallback image regardless of 3D status, since the detail view must degrade gracefully if a `.glb`/`.usdz` fails to load or hasn't been uploaded yet.

---

## 8. Special Requirements / Deviations from DNA

*(Only fill this in if the client needs something that doesn't fit `design.md` or `security.md` as written. Every row here should have been an explicit halt-and-flag conversation with the human, not a quiet workaround.)*

| Requested deviation | Which DNA rule it touches | Resolution |
|---|---|---|
| — | — | — |

---

## 9. Revision History

| Date | Change | By |
|---|---|---|
| ___ | Initial template filled | ___ |
