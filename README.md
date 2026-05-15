# MIDGE 2.0 UI — Working Folder

**Location:** `~/Desktop/MIDGE-2.0-UI/`
**Status:** Working v1 — wired to Railway, customization works, training modals work.
**Owner:** Brent Jones

---

## What's in this folder

| File | What it is |
|---|---|
| `midge.html` | **The working UI.** Single self-contained file. Double-click to open. |
| `mockup.html` | The original design mockup (side-by-side dark/light preview, color swatches). Reference only — not the production page. |
| `MIDGE_2.0_UI_PLAN.md` | The full spec for this UI. Read this first if you're a new session picking up work. |
| `assets/` | Logo PNGs (white / black / silver) that `midge.html` references. Must stay alongside the HTML for the logo to render. |

---

## How to open the working UI

Double-click `midge.html`, or in Terminal:
```
open ~/Desktop/MIDGE-2.0-UI/midge.html
```

It hits the live Railway backend at `https://midge-backend-production.up.railway.app/ask` — no local server needed.

---

## What this UI does (the short version)

- **Halo + logo** welcome screen with "Talk to MIDGE / What do we need to tackle today?"
- **Hamburger menu (top right by default)** opens "My Midge" drawer with:
  - My Logo (white / black / silver)
  - My Background (color picker)
  - My Words / Text (color picker)
  - Behind My Logo (color picker for the disc)
  - My Halo (on/off)
  - Light / Dark (theme preset)
  - My Hand (L/R — mirrors menu and Back-to-Vault button to the other side)
  - Reset
- **Back to the Vault** button (top left by default, with circular vault-door icon) → flyvault.net
- **First-tap training modals** on every menu item — explain in plain English what the item does, dismiss with X
- All choices persist via `localStorage`
- Sends `session_id` with every `/ask` call so the backend's sticky-tyer scope works across turns

---

## The frame (why this exists)

This is **the stopgap UI that retires Pickaxe** ($100/mo bleed). It is NOT the full FlyVault rebuild — that's separate work scheduled after Pickaxe dies.

Decisions baked into this UI that **carry forward into the rebuild unchanged:**

1. **"My ___" language pattern** — every label that points to the user's stuff uses "My." Matches the existing FlyVault site nav ("My Flies," "My Flybox").
2. **Customization-as-identity** — users color their own MIDGE. Says "this is yours" without words.
3. **Training-modal infrastructure** — keyed by feature in the `TRAINING` object. Becomes the Vault Door onboarding mechanism later.
4. **3rd-grade-level copy everywhere** — no dev-speak. Geezer-friendly. Forever rule.
5. **Possession + delivery rule** — if a label says "My X," the link MUST actually go to *their* X.

These are architectural decisions, not cosmetic ones.

---

## Where everything else lives

- **Backend repo:** `~/Desktop/midge-backend/` (also on GitHub as `brenttiesflies-sys/midge-backend`)
- **The repo has these plan docs:**
  - `ROADMAP.md` — the vision
  - `RETRIEVAL_FIX_PLAN.md` — Plan 1, ✅ shipped (commit `8e4f684`)
  - `HONESTY_FIX_PLAN.md` — Plan 2, ✅ shipped (commit `37a5192`)
  - `MIDGE_2.0_COSTS.md` — cost analysis & pricing
  - `MIDGE_2.0_UI_PLAN.md` — the UI spec (same file copied here)
  - `midge.html` — also lives in the repo
- **Backend (production):** `https://midge-backend-production.up.railway.app/`
- **Supabase project:** `vflpeacbpbicxkoemfvm` ("Fly Tying Technique Library")
- **Live FlyVault site:** `https://flyvault.net/`

---

## What's left before Pickaxe dies

1. Delete `/raw-vault` debug endpoint from `main.py` (hygiene)
2. Rotate API keys (Groq, Supabase service, Tavily, Formidable)
3. Stripe embed of this UI on FlyVault
4. Migrate the ~6 Midge 1.5 paying members
5. Kill Pickaxe ($100/mo saved)

No deadline beyond Pickaxe bleed. Build correct, not fast.

---

## Pointer for a new chat picking this up

If you're a fresh Claude session reading this — start by reading:

1. This README
2. `MIDGE_2.0_UI_PLAN.md` in this folder (the full UI spec)
3. `~/Desktop/midge-backend/ROADMAP.md` (the bigger vision)
4. `~/Desktop/midge-backend/RETRIEVAL_FIX_PLAN.md` and `HONESTY_FIX_PLAN.md` (what the backend does)

That gives you the full picture in about 10 minutes of reading. The codebase is on GitHub at `brenttiesflies-sys/midge-backend`.

Brent's priority right now is **getting off Pickaxe** — not feature breadth. Keep that frame.
