# SECURITY.md — LunAR AI Agent Security Directives

> **Audience:** AI coding agents (Claude Code, Cursor, GitHub Copilot, etc.) writing, refactoring, or reviewing code for LunAR client projects — AR web experiences for Indonesian SMBs, currently focused on furniture and lifestyle goods (markerless WebAR product previews).
>
> **Purpose:** Every LunAR client gets different products, different branding, and eventually a different tech tier (static MVP vs. backend-enabled). But every LunAR site should carry the same security posture and the same honesty about what that posture actually protects against. This document is the constant "DNA" — plus the adaptive layer for what changes as a project scales.
>
> This is a security/architecture reference only. For visual/brand rules, see `design.md`.

---

## 1. How to use this file

- **DNA (Sections 3–7):** Applies to every LunAR project regardless of client or project stage. If a client request seems to conflict with a DNA rule, find the version of the rule that fits their situation — don't silently break it.
- **Adaptive layer (Section 2):** Re-assessed per project, mainly around one axis: **is this project still a static MVP, or has it graduated to a backend-enabled tier?** Almost everything else in this doc branches off that one fact.
- **When in doubt:** favor telling the client the truth about what a protection does and doesn't do, over shipping something that *looks* secure. LunAR's credibility with business owners matters more than any single feature.

---

## 2. Adaptive layer — assess once per project, re-check at scale-up

Before writing security-relevant code for a project, an agent should confirm which tier it's in:

### 2.1 Tier A — Static MVP (current default for all clients)

Tier A now has a standardized shape: **Cloudflare Pages** for the static site, plus a **Cloudflare Worker** acting as a thin, CORS-scoped proxy in front of a **Cloudflare R2 bucket** (`[client]-3d-asset`) that stores `.glb`/`.usdz` files.

This is still Tier A, in full, because:
- There is no application database — R2 is object storage for static files, not a queryable data store the site reads structured records from.
- There is no user authentication, no session, no login.
- The Worker performs no business logic — it streams bytes from R2 and attaches response headers (notably CORS, see §5a). It does not validate input against a data model, does not branch on a logged-in user, and does not write anything back to storage on a visitor's behalf.
- If a Worker for a given client ever grows beyond "stream this object + set these headers" — e.g. it starts checking a database, applying auth, or accepting writes from visitors — that specific project has crossed into Tier B and must be flagged per §2.2, even though other Tier A projects keep using the plain proxy pattern.

**Deployment method:** Cloudflare Pages projects deploy via **Direct Upload** of the built static folder, not via a GitHub-linked auto-build pipeline. See `agent.md` §4 for the full reasoning (avoiding unintended Worker/Functions provisioning and build-minute consumption on what is, functionally, a folder of static files). Git remains the source-of-truth for code history; it's just not wired to trigger production deploys automatically.

All rules in Section 3 (Architecture) apply in full, unmodified, to this stack. This is the default assumption unless a project has explicitly moved to Tier B.

### 2.2 Tier B — Backend-enabled (future, per-client opt-in)

Triggered when a client needs things this static-plus-edge-proxy stack genuinely can't do: a real application backend, user login/customer portals, client-managed product catalogs edited through an admin UI, order tracking with write-backs, ERP/database integrations, or a general-purpose application framework (Node.js/Express, Next.js, React with server-side routes, etc.) running actual business logic.

**Important distinction an agent must not blur:** wanting to hide a secret is *not*, by itself, a reason to migrate to Next.js or any other framework — Cloudflare Worker environment bindings already keep credentials server-side inside the Tier A stack (see §5). If a proposed reason for "we need Tier B" turns out to reduce to "so the API key isn't public," stop and point out that the Tier A stack already solves that; ask what the *actual* Tier B requirement is (a login system? a database of orders? something a static site structurally cannot do?) before treating it as settled.

When a project genuinely moves to Tier B:
- Hosting moves to a platform with real serverless functions and a serverless-friendly database (e.g. Neon, Supabase Postgres, MongoDB Atlas), separate from the Tier A Pages/Worker/R2 pattern.
- **Secrets now live server-side only** — in the host's environment variable / secrets manager, never committed to the repo, never sent to the client bundle. Section 5's "no private keys in the client bundle" rule doesn't relax — it just gets a legitimate server-side place to live for the first time, beyond what a Worker binding alone covers.
- Any endpoint that accepts client data (forms, uploads, login) needs input validation and rate limiting at the server layer, not just client-side checks (client-side validation is UX, not security — it's trivially bypassed).
- Auth, if added, uses an established provider/library (e.g. Supabase Auth, Auth.js) rather than hand-rolled session/password logic. Do not write custom password hashing, session tokens, or JWT signing from scratch.
- If the app stores customer personal data (names, phone numbers, addresses) server-side, treat it as covered by Indonesia's **UU PDP** (Personal Data Protection Law) — data minimization, a stated retention period, and no sharing with third parties beyond what's disclosed to the customer.
- **An agent should never silently upgrade a project to Tier B.** If a requested feature needs backend/database code and the project is still Tier A, halt and flag it to the user explicitly (same as the existing API-key halt rule in Section 5) — this is a scope and cost decision for the human, not something to route around client-side.

---

## 3. Architectural constraints (Tier A — static MVP)

This is unchanged from LunAR's original MVP philosophy, and applies to the Cloudflare Pages/Worker/R2 stack described in §2.1 exactly as it applied to the earlier plain-static version:

- **DO NOT** generate or suggest general-purpose server-side application code (Node.js/Express app servers, Python, PHP, Ruby, Next.js API routes doing business logic) for a Tier A project. A byte-streaming, CORS-header-setting Worker per §2.1/§5a is the one narrow exception, not a precedent for adding more logic to it.
- **DO NOT** generate or suggest database integrations (MongoDB, PostgreSQL, Firebase/Supabase *databases*) for a Tier A project. R2 object storage for static `.glb`/`.usdz` files is not a database and doesn't count as one.
- **DO NOT** create API routes for a Tier A project beyond the single-purpose asset-proxy Worker.
- All application state, product data, and logic stay client-side — hardcoded in `products.js`, manipulated via standard DOM APIs.
- **Why this matters, not just what:** a static site (plus a stateless asset-streaming edge proxy) has no attack surface for SQL injection, auth bypass, server misconfiguration, or credential leaks — because there's no application server and no data store to misconfigure or breach. This constraint isn't a limitation to work around, it's the reason Tier A projects are cheap to host and hard to breach. Preserve it until a project has a real, discussed reason to graduate to Tier B.

---

## 4. Frontend asset deterrents ("speed bump" protocol) — cosmetic only

If asked to write UI protection scripts, these are the standard deterrents:

```js
// Block right-click context menu
window.addEventListener('contextmenu', (e) => e.preventDefault());

// Intercept common devtools shortcuts
window.addEventListener('keydown', (e) => {
  if (
    e.key === 'F12' ||
    (e.ctrlKey && e.shiftKey && ['I', 'J', 'C'].includes(e.key)) ||
    (e.ctrlKey && e.key === 'u')
  ) {
    e.preventDefault();
  }
});
```

**This section carries a mandatory disclaimer, both in code comments and in any client-facing conversation:**

> These are UX-level speed bumps against casual copy-pasting, not a security control. Any visitor can still open devtools via the browser's own menu, use a different keyboard layout, view `view-source:`, or just read the page's network requests. They do not stop a motivated competitor, and they should never be described to a client as "protecting" their assets — that claim belongs to Section 6 (IP protection), not this section.

Practical implications for agents:
- Always include the disclaimer comment block above the script when generating it.
- Never suggest this protocol as a substitute for the real IP protections in Section 6.
- Be aware of the UX cost: disabling right-click also disables legitimate uses (copying an address, opening a link in a new tab from long-press on mobile). Apply narrowly — e.g. only around AR/3D viewer containers or product images — rather than globally across the whole page, unless the client explicitly wants the blunt global version.

---

## 5. Secrets & API key management

Because Tier A is 100% frontend (plus one stateless edge proxy), **everything shipped to the browser is public** — this is not a matter of obfuscation, it's physics (anyone can read a browser's downloaded JS, and anyone can inspect what URL a `<model-viewer>` tag points at).

- **NEVER** embed private API keys (OpenAI, Stripe, AWS Secret Access Keys, private Google Maps/Places keys, etc.) in a Tier A repo or in client-side JS.
- **NEVER** connect directly to cloud storage (e.g. AWS S3, private GCS buckets) using embedded IAM credentials from client-side code.
- Public, rate-limited, no-auth-required third-party APIs (e.g. the QR code generator, Google Fonts, the `<model-viewer>` CDN script) are fine as-is — they carry no secret and no client liability.
- If a requested feature needs a private key (e.g. a paid AI API, a private maps tier, a payment provider secret), **halt and explicitly tell the user** that doing this in a static frontend exposes the key to anyone who views page source or the network tab — then point them at the Tier B path (Section 2.2) where the key can live server-side.
- This rule doesn't loosen at Tier B — it's satisfied there for the first time via environment variables on the server/hosting platform, never in a client bundle.

### 5a. Cloudflare Worker bindings and CORS scoping (Tier A 3D-asset proxy)

The Worker that proxies a client's R2 bucket does **not** need, and must never contain, an R2 Access Key ID / Secret Access Key in its code. Instead, the R2 bucket is attached to the Worker via a **binding** (configured in the Worker's settings, e.g. `MY_BUCKET`), which Cloudflare resolves server-side at request time. This is the correct pattern and satisfies this section's "no embedded credentials" rule by construction — an agent should never "fix" a binding by replacing it with an inline Access Key.

**CORS must be scoped to the client's own domain, not left as a wildcard.** A Worker serving 3D assets should set:

```js
// Correct: restrict to this client's actual Pages domain(s)
const ALLOWED_ORIGIN = "https://[client-project].pages.dev"; // plus custom domain if one exists

response.headers.set("Access-Control-Allow-Origin", ALLOWED_ORIGIN);
response.headers.set("Access-Control-Allow-Methods", "GET, HEAD");
```

rather than:

```js
// Avoid: allows any site on the internet to hotlink this client's 3D assets
response.headers.set("Access-Control-Allow-Origin", "*");
```

**Why this matters:** a wildcard-CORS Worker doesn't leak anything secret (the files are meant to be publicly viewable in AR), but it does let any third-party site embed and hotlink the client's `.glb`/`.usdz` files directly — consuming the client's Worker/R2 bandwidth for someone else's page, and making it trivial to embed a competitor's stolen catalog on another domain without even downloading the file first. Scoping `Access-Control-Allow-Origin` to the client's actual Pages/custom domain(s) closes that off while changing nothing about how the client's own site functions. If a client has both a `*.pages.dev` staging domain and a custom production domain, list both explicitly rather than falling back to a wildcard for convenience.

If an older project is still running a wildcard-CORS Worker from before this rule was written, that's a known gap to tighten opportunistically, not an emergency to interrupt other work for — but any *new* Worker written from this point forward must scope its origin from day one.

---
## (Update Info) BELOW IS INFORMATION ONLY FOR READ AND CONTEXT. NOTE NUMBER 6 BELOW ARE ENTIRELY HANDLED BY US HUMAN AND NOT AI AGENT, BECAUSE THERE'S SO MANY TECHNICALITY THAT IS BEYOND THE SCOPE OF THIS security.md SO DON'T MIND ANYTHING
## 6. IP protection for 3D/AR assets — the "3-Layer Defense"

This is LunAR's actual value proposition for furniture clients worried about design theft, and it deserves to be treated as a first-class deliverable, not an afterthought bolted onto the AR section.

**The problem:** a `.glb`/`.usdz` file served for AR preview can be downloaded by anyone who inspects network traffic — client-side deterrents (Section 4) cannot stop this technically, and neither can CORS scoping (§5a): CORS controls which *websites* can fetch the file via a browser script, not whether a person looking at Network tab requests can save it directly. The defense has to work *even after* the file is taken.

**Layer 1 — Legal (the only defense with real teeth).**
Advise clients to register their furniture designs with Indonesia's design/copyright system (Direktorat Jenderal Kekayaan Intelektual — HAKI). This is the only layer that lets them actually act against a competitor who copies the physical product, not just the file. LunAR's role here is to advise and connect clients to this step, not to provide legal services directly.

**Layer 2 — Decimation / "Hollow Shell."**
Every `.glb`/`.usdz` exported for a client-facing AR preview must be:
- Heavily decimated (low polygon count) relative to the source/manufacturing model.
- Stripped of any CAD precision, internal structure, or manufacturing-relevant geometry.
- Treated as a disposable "skin" — if stolen, it's a photograph of the product, not a blueprint. It cannot be sent to a CNC machine or a factory to reproduce the item.

**Layer 3 — Baked watermarking.**
Bake the client's logo or mark permanently into the 3D model's texture (not an on-screen overlay) — e.g. on the underside of a chair seat or the back panel of a cabinet. Practical rules for agents:
- The watermark must be part of the texture map, not a DOM/CSS overlay — an overlay is trivial to crop out, a baked texture isn't.
- Placement should be inconspicuous in normal AR viewing (underside, back panel) but unmistakable and undeletable if the raw file is extracted and re-used elsewhere.
- This is a production/3D-pipeline step (for the human 3D artist or Meshy AI cleanup pass), not something an agent generates in code — but agents should flag if a project's asset pipeline is missing this step before an asset ships to a client's live site.

**What this section is not:** it is not a claim that theft becomes impossible. It's an honest three-layer story — one legal, two technical — that meaningfully raises the cost and reduces the value of theft. Represent it to clients in those terms.

---

## 7. 3D asset handling & cross-platform AR

- Exclusively use Google's `<model-viewer>` component for rendering 3D assets in Tier A/B projects.
- **Mandatory iOS handling:** iOS does not support in-browser WebXR AR; it hands off to native **AR Quick Look**. Whenever generating or updating a `<model-viewer>` tag, always supply both paths:
  1. `src="path/to/model.glb"` — Android/Chrome WebXR, served through the client's Worker proxy.
  2. `ios-src="path/to/model.usdz"` — iPhone/Safari Quick Look, also served through the Worker proxy.
- **The iOS security exception:** once an iPhone enters native Quick Look, all client-side JS restrictions from Section 4 are bypassed by iOS itself — there is no way to intercept or block anything inside native Quick Look, and CORS scoping (§5a) has no bearing there either, since Quick Look's asset fetch is not a same-origin browser fetch subject to CORS. This makes the `.usdz` file's own protection (Layers 2 and 3 in Section 6) the *only* thing standing between that asset and casual extraction on iOS. Never treat Section 4's deterrents, or §5a's CORS scoping, as covering the iOS AR path — they structurally cannot.
- **Graceful degradation is mandatory, not optional.** Per `design.md` §8, if a `.glb`/`.usdz` fails to load (missing file, network error, malformed asset) the detail view must fall back to the product's static 2D image — never a raw error, a blank canvas, or an indefinitely-spinning loader. An agent shipping a `<model-viewer>` without a tested failure path has not finished the task.
- **Scope constraint:** per `design.md` Section 8, LunAR's product is **markerless-only** via `<model-viewer>`. Do not build marker-based/image-tracking AR flows (MindAR, `targets.mind`, etc.) — that pattern was an earlier experiment for a different vertical (cafe) and is out of scope. If a future client genuinely needs marker-based AR, treat it as a distinct project track with its own security review, not a variant of the standard product.

---

## 8. Dependencies & script inclusion

- No `npm` packages for backend utilities in Tier A projects (there is no application backend to use them in; the Worker is a thin proxy, not a Node runtime for arbitrary packages).
- Prefer lightweight, CDN-based script tags for frontend libraries to preserve the static nature of Tier A projects.
- Where the CDN supports it, use `<script src="..." crossorigin="anonymous">`; prefer providers/versions that support Subresource Integrity (`integrity="sha384-..."`) so a compromised CDN can't silently swap out a script — flag in code review if a currently-used CDN script lacks a pinned version or SRI hash.
- Pin CDN library versions explicitly (as the current codebase already does, e.g. `model-viewer@3.4.0`) rather than using `@latest` — an unpinned dependency can change behavior or introduce a vulnerability without any code change on LunAR's side.

---

## 9. Interactive/data-collection surfaces

Applies once a project includes any form, chat widget, or input beyond a static `wa.me` link:

- Client-side input validation is a UX nicety, never a security boundary. Any Tier B endpoint receiving that input must re-validate server-side regardless of what the frontend already checked.
- Pre-filled WhatsApp messages (the current pattern) are safe by construction — they're just a URL, no data leaves the browser until the customer chooses to send it via WhatsApp. Don't replace this with a custom form that silently collects and stores data without the client (and their customers) knowing where it goes.

---

## 10. Pre-ship security checklist for agents

- [ ] Confirm project tier (A/static-plus-edge-proxy or B/backend-enabled) before writing anything server- or database-adjacent.
- [ ] No private API keys, IAM credentials, or secrets anywhere in a Tier A repo or client bundle — R2 access goes through a Worker binding, never an inline Access Key.
- [ ] The Worker's `Access-Control-Allow-Origin` is scoped to the client's actual Pages/custom domain(s), not `*`.
- [ ] Cloudflare Pages deploys via Direct Upload, not a GitHub-linked auto-build pipeline.
- [ ] Any feature needing a private key or a real backend → halted and flagged to the user, not silently worked around by reaching for Next.js or another framework.
- [ ] Devtools/right-click deterrents (if used) carry the cosmetic-only disclaimer and are scoped narrowly, not blanket-applied without discussion.
- [ ] Every AR-enabled product's `.glb`/`.usdz` pair is decimated ("hollow shell") before it ships to a live client site.
- [ ] Watermarking (Layer 3) is baked into textures, not a removable on-screen overlay.
- [ ] `<model-viewer>` tags include both `src` and `ios-src`, both served through the Worker proxy.
- [ ] The detail view has a tested, non-broken fallback to the 2D image if the 3D asset fails to load or is missing.
- [ ] No marker-based AR (MindAR/`targets.mind`) introduced into a client project.
- [ ] CDN scripts are version-pinned; SRI hashes added where the CDN supports them.
- [ ] Any new form/input surface re-validates server-side if the project is Tier B — not just client-side.
- [ ] If customer personal data is collected and stored (Tier B only), UU PDP-style data minimization and retention are considered, not just "does it work."

---

**END OF DIRECTIVES.** Acknowledge these constraints internally — and confirm the project's current tier — before executing any code generation tasks for LunAR.
