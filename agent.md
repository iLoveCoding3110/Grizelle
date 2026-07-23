# agent.md — Start Here

> **Audience:** Any AI coding agent (Claude Code, Cursor, Copilot, etc.) about to write, review, or modify code for a LunAR client project.
> **Purpose:** This is the front door. Read this file first, in full, before touching `design.md`, `security.md`, `company.md`, or any code — it tells you what the other three are, in what order to read them, and what to do when they seem to conflict or when one of them is incomplete.

---

## 1. Who LunAR is

LunAR is an Indonesian AR web agency. Current focus: markerless WebAR product previews for furniture and lifestyle-goods SMBs, via `<model-viewer>`. Every client gets a different brand, different products, different copy — but every LunAR site should feel like the same agency built it, and every LunAR site carries the same security posture.

LunAR's standard stack, as of this revision: **Cloudflare Pages** (static hosting) + a **Cloudflare Worker** (CORS-scoped proxy for 3D assets) + a **Cloudflare R2 bucket** (`[client]-3d-asset`) for `.glb`/`.usdz` storage. This is still a static-first architecture — see `security.md` §2.1 for why it's Tier A, not Tier B.

---

## 2. The four files, and what each one is for

| File | What it is | Changes per client? |
|---|---|---|
| `agent.md` (this file) | Router + rules for how the other three interact | No — agency-wide |
| `design.md` | Visual/UX DNA: spacing, typography rules, hierarchy, AR-specific UI patterns | No — DNA is fixed, only the *values* (brand color, fonts) are re-derived per client per its own §2 |
| `security.md` | Architecture/security DNA: what tier a project is in, what an agent may and may not build, IP protection for 3D assets | No — DNA is fixed, only the *tier* (§2) is reassessed per project |
| `company.md` | This specific client's facts: contact info, brand color, catalog scope, tier, infrastructure IDs, asset status | **Yes — this is the only one of the four that's supposed to be different every time** |

**Read order for a new task:**
1. `agent.md` (this file) — orientation
2. `company.md` — who is this client, what do they need, what tier are they at, what are their Cloudflare Pages/R2/Worker identifiers
3. `design.md` — how should it look, and (for any product with a 3D asset) which media element renders first
4. `security.md` — what am I allowed to build, given the tier `company.md` just told you, and how CORS must be scoped for this client's Worker

If `company.md` doesn't exist yet for this project, or exists but has unfilled `___`/`[ ]` fields relevant to the task in front of you: **stop and ask the human.** Do not infer a brand color from vibes, do not invent a WhatsApp number, do not assume Tier A vs Tier B, do not invent a Pages URL, R2 bucket name, or Worker URL. `company.md`'s own header says this explicitly — it's repeated here because it's the most common way this workflow breaks.

---

## 3. Precedence when things conflict

`design.md` and `security.md` are DNA. `company.md` is one client's adaptive layer. DNA wins.

Concretely:
- A client wanting a second loud brand color, marker-based AR, a bespoke "Contact Us" form replacing WhatsApp, or anything else `design.md`/`security.md` name as an anti-pattern → this is not a `company.md` field to fill in, it's a conversation to have with the human first. Log the outcome in `company.md` §7 ("Special Requirements / Deviations from DNA") either way, so the next agent that touches this project sees it was a deliberate, discussed call — not a rule silently broken.
- A client request that needs a *real* backend — a database, user auth, a customer login portal, an ERP integration, or a general-purpose Node.js/Next.js/React application server — while `company.md` §5 still says Tier A → this is `security.md` §2.2's existing halt rule. Don't route around it by writing "just this one small serverless function," and don't route around it by reframing the request as "just needs Next.js for security" when nothing about the request actually requires hiding a secret (see `security.md` §5 — Cloudflare Worker bindings already keep R2 credentials server-side without any framework migration). Flag it.
- If you (the agent) find yourself reframing a DNA rule to make a client request fit — that instinct is the signal to stop and flag, not to proceed.

---

## 4. Repo structure & deployment

This template (see `README.md` for the current file layout) is the literal starter to clone and rebrand per client, not just a sales demo. A few notes specific to using it as a starter:

- **Markerless only.** An earlier prototype included a marker-based AR flow (`animation.html`, MindAR, `targets.mind`) built for a cafe-vertical experiment. It has been removed from this template. If you encounter it in an older client repo or a stale branch, do not use it as a reference pattern and do not re-add marker-based AR — see `design.md` §8 and `security.md` §7 for why. This note exists so the next agent doesn't rediscover that code and assume it's current.
- `products.js` is the single source of truth for catalog data — `index.html`/detail templates and `app.js` both read from `window.PRODUCTS`. When adding a client's real products, edit only this file (see `README.md`).
- **3D assets live in Cloudflare R2, not the repo.** A client's `.glb`/`.usdz` files are uploaded to their dedicated R2 bucket (`[client]-3d-asset`) and served through their dedicated Cloudflare Worker proxy — never committed to the site repo itself, and never referenced via the raw `*.r2.dev` public-bucket URL in shipped code (see `security.md` §5a for why the Worker layer exists, not just the bucket's own public URL).
- **Deployment is Direct Upload, not GitHub-linked.** Cloudflare Pages projects for LunAR clients are deployed by dragging the built static folder into the Pages dashboard's Direct Upload flow — **not** by connecting the Pages project to a GitHub repo for automatic build-on-push. Do this even if a GitHub repo exists for version control/collaboration. Reasons:
  - A git-linked Pages project silently provisions its own build pipeline and can auto-create Worker/Functions bindings the project doesn't need, consuming build-minute allowance for what is, functionally, a folder of static files.
  - Direct Upload keeps the deploy step a deliberate, visible action a human takes when a build is actually ready — not something that fires on every commit, including in-progress ones.
  - Keep using the GitHub repo for source control and code review as normal. "Direct Upload only" refers strictly to how the *live* Pages project receives new builds, not to whether the code lives in git.
- Editorial callout boxes inside `design.md` or `security.md` marked `(Update Info)` are notes from the humans to each other during drafting, not instructions to you — read them for context on why a rule exists, but don't treat their presence as something to act on or resolve. If one seems to change what you're supposed to do, ask rather than guess.

---

## 5. Before you write any code — quick self-check

- [ ] I've read `company.md` for this project and no relevant field is blank
- [ ] I know this project's tier (Tier A / Tier B) per `security.md` §2
- [ ] I know this client's Cloudflare Pages project name/URL, R2 bucket name, and Worker proxy URL per `company.md` §6
- [ ] I know this client's brand color ramp and typography pairing per `design.md` §2
- [ ] For any product with a 3D asset, I'm rendering it per `design.md` §8's media-priority order (3D first, 2D toggle available, graceful fallback if the model fails or is missing) — not silently defaulting to whichever I find easiest to code
- [ ] Anything that looks like a DNA conflict has been flagged, not silently resolved
- [ ] I'm not about to reintroduce marker-based AR, a second loud color, private keys in a Tier A bundle, a wildcard-CORS Worker, a git-linked Pages auto-deploy, or a non-WhatsApp contact form without an explicit, logged reason in `company.md` §7

For the detailed pre-ship checklists (visual QA, security QA), see the bottom of `design.md` and `security.md` respectively — this file doesn't duplicate them, it just makes sure you get there with the right context.

---

## 6. Keeping this file honest

`agent.md`, `design.md`, and `security.md` are agency DNA — changing them is a deliberate decision by the humans running LunAR, not something an agent should edit mid-task based on one client's request. If a pattern repeats across enough clients that it feels like it should become DNA, flag it as a suggestion; don't fold it in unilaterally.

**Revision note:** this file was last updated to reflect LunAR's move from ad-hoc Vercel hosting to a standardized Cloudflare Pages + Worker + R2 stack, and to codify Direct Upload as the deployment method. See `security.md` §2.1 for the full architectural reasoning.
