# MIDGE 2.0 — UI Plan (the stopgap that retires Pickaxe)

**Owner:** Brent Jones
**Status:** Spec locked. Not yet built.
**Last updated:** 2026-05-13

---

## The frame (read this first)

**This is not the rebuild.** It is the stopgap chat page that retires Pickaxe ($100/mo bleed) so the team can move the small Midge 1.5 cohort (~6 paying members at $10/mo) onto Midge 2.0 — running on our own stack (Railway + Supabase + Groq), which we own.

**The rebuild comes after.** That's where Formidable Forms, PeepSo, and WordPress get replaced. Whole different scope. Don't conflate.

**No deadline. No sprint. Build it correctly.** The work tonight isn't urgent in days — it's urgent in dollars (Pickaxe bleed). Done right means we never have to redo the brand voice, the My-language, or the training pattern in the rebuild. Done wrong means a retrofit.

**Scope discipline.** Materials work, hatch engine, FlyCad, voice — all rebuild scope. Out of scope here. The retrieval fixes already shipped (commits 8e4f684, 37a5192) cover the brain. This UI is the face members see.

---

## What ships in 2.0 (the punch list)

A new HTML chat page that replaces `midge-2.0-test.html`, wired to the same Railway `/ask` endpoint, embedded in FlyVault behind Stripe.

### Visual identity
- Lift the design system from `~/Desktop/midge-2.0-design/mockup.html`: fonts (Inter + Fraunces), halo gradient, color tokens, dark/light theme tokens.
- Single panel — not the side-by-side mockup preview. Full screen on mobile, side-panel on desktop.
- Welcome state (logo + halo + "What do we need to tackle today?" + chip suggestions) → Conversation state (chat log + textarea + send) on first message.

### Layout (locked)
- **Top left:** "Back to the Vault" button + round vault-door icon. → `https://flyvault.net/`. Always visible. Big on mobile (it was the #1 complaint pre-fix). Smaller on desktop where Midge opens beside the vault anyway.
- **Top right:** Hamburger menu icon (3 lines). Tapping drops a menu DOWN, covering content rather than pushing it. Standard mobile drawer pattern. Closes on outside tap or X.
- **Center:** Halo + logo (welcome) or chat log (conversation).
- **Bottom:** Textarea + send button. Has to stay anchored when mobile keyboard opens.

### The Menu — "My Midge"
- Header: **My Midge**
- Customization items (each opens a color picker / image selector):
  - **My Logo** → pick from 3 PNGs (white / black / silver — already in `assets/`)
  - **My Background** → page color
  - **My Words / Text** → text color (intentional dual label to anchor the geezer-friendly word; this is the only label that gets a slash, on purpose)
  - **Behind My Logo** → the disc color inside the halo
  - **My Halo** → on/off toggle (steady ring of light, slow color cycle, NO wobble)
  - **Light / Dark** → starting preset (both load as a base; user customizes from there)
  - **My Hand: L — R** → side toggle. Green half of the pill indicates the active side. R is the default (most users are right-handed). When flipped to L, the menu AND the Back to the Vault button mirror sides together (menu goes left, vault button goes right). One toggle = full horizontal flip. Persists via localStorage.
  - **Reset** → put everything back to default
- Navigation items (below customization, separated):
  - **Back to the Vault** → `https://flyvault.net/`
  - (deferred: a "My Flies" link to `/fly-listing/my-flybox/` — confirmed fine to ship if trivial; check before adding)

### Training-wheels modal (the disposable teaching layer)
- Every menu item shows a modal on FIRST tap. Modal explains in plain English what the item does ("Change your text color"). Has an **X in the opposite corner** to dismiss.
- After dismissing the modal, the SECOND tap on the same item opens the actual control (color picker etc.).
- Globally toggle-able via one flag (e.g. `window.MIDGE_TRAINING = true`). Future-self can flip it off in one place once members are walking each other through it.
- Modal text is stored in a SINGLE JS object keyed by feature, e.g. `TRAINING['my-words'] = "Change your text color"`. New features = new key. This object becomes the **onboarding infrastructure** that propagates forward into the Vault Door experience and Super MIDGE feature intros.

### The "My" language pattern (architectural, not cosmetic)
- FlyVault already uses "My Flies" and "My Flybox" in production nav (verified live on flyvault.net).
- Every label that points to one of the user's things uses "My."
- "My" is a **possession word**, not branding. It does psychological work: tells the user the space belongs to them. Old-school anglers get it instantly.
- The principle: if you say "My X" the link MUST actually go to *their* X. The word collapses if the platform doesn't deliver.
- This pattern propagates forward into the rebuild unchanged. It is a brand-architectural decision, not a cosmetic one. Lock it in 2.0 so it never has to be retrofit.

### Customization persistence
- Save user choices in `localStorage`. Survives page refresh / browser restart on the same device.
- This is "stopgap correct" — in the rebuild, preferences will be tied to FlyVault account so they sync across devices. For now, per-device is fine.

### Safety nets
- Color contrast: if a user picks a combo where text becomes unreadable (white on white), don't auto-fix it (their space, their call) — but make Reset prominent so they can recover in one tap.
- Mobile keyboard: textarea anchored to bottom, "Back to the Vault" button doesn't move when keyboard opens.
- `session_id` must continue flowing through to the backend (already in test page — preserve when porting).

---

## What is explicitly OUT of scope

- Account-tied preferences (sync across devices) → rebuild
- Color presets ("Trout Stream," "Sunset") → after launch, designed with real members
- Real undo/redo → Reset is enough for now
- Voice / voice-to-voice → rebuild
- Materials catalog, hatch engine, FlyCad, visual ID → rebuild
- RLS policies on `patterns` / `midge_knowledge` → pre-public-launch task, separate session
- Removing `/raw-vault` debug endpoint → pre-public-launch hygiene, do before Stripe embed
- Rotating API keys → To-Do had this listed; do before Stripe embed

---

## The 3.0 / Rebuild thread that runs THROUGH this work

The following decisions made tonight survive the rebuild verbatim:

1. **The "My" language pattern.** Every label everywhere. Vault Door inherits it.
2. **The training-modal infrastructure.** Becomes the Vault Door onboarding mechanism and the Super MIDGE new-feature introducer.
3. **Customization as identity signal.** The "this is yours" feel transfers to the rebuilt platform — same controls, deeper persistence (account-level), more options (presets, theme sharing, etc.).
4. **3rd-grade-level language as a discipline.** Every label, every modal, every button. No dev-speak. This is a forever rule.
5. **Possession + delivery rule.** "My X" only works if the link actually goes to their X. Architectural rule across both code and copy.

These are NOT to be redesigned in the rebuild. They are inherited as decided.

---

## Implementation order (when we pick this up)

1. **Create `midge.html`** in `~/Desktop/midge-2.0-design/` (or wherever lives well alongside the existing assets). Single file, no framework.
2. **Lift the design system** from `mockup.html` (CSS variables, fonts, halo, layout). Drop the side-by-side preview frame. Drop the swatch picker / theme designer (those were for the designer, not production).
3. **Build the welcome → conversation transition** (small JS state change on first send).
4. **Wire to `/ask`** — fetch call with `question`, `history`, `session_id`. Include the markdown-image rendering for vault pattern images, same as the current test page.
5. **Build the My Midge drawer** (hamburger top-right, drops down covering content).
6. **Wire the customization controls** — each one hits a CSS variable on the root element, persisted to localStorage.
7. **Add the training-modal layer** — one modal component, fed by a `TRAINING` object keyed by feature.
8. **Add the "Back to the Vault" button** top-left with the vault-door icon.
9. **Mobile responsive pass** — verify keyboard behavior, button reachability, drawer feel.
10. **Test against the live `/ask`** endpoint end-to-end with real queries (Jaselyn flies, Greensleeves, foam beetle, etc.) to confirm nothing regresses.

---

## Where things live (for the next session that picks this up)

- **Mockup (design source):** `~/Desktop/midge-2.0-design/mockup.html`
- **Logo assets:** `~/Desktop/midge-2.0-design/assets/` (white, black, silver PNGs)
- **Current beta page:** `~/Desktop/midge-2.0-test.html` (has `session_id` wiring to copy)
- **Backend repo:** `~/Desktop/midge-backend/` (on GitHub, brenttiesflies-sys/midge-backend)
- **Backend prod URL:** `https://midge-backend-production.up.railway.app/ask`
- **Supabase project:** `vflpeacbpbicxkoemfvm` ("Fly Tying Technique Library")
- **Live FlyVault:** `https://flyvault.net/` (where this embeds via Stripe gate eventually)
- **Pre-2.0 punch list:** ROADMAP.md (this repo), MIDGE_2.0_COSTS.md (cost analysis), RETRIEVAL_FIX_PLAN.md and HONESTY_FIX_PLAN.md (both ✅ shipped)

---

## Why this is the right shape

- **It retires Pickaxe.** That's the only deadline.
- **It feels like 2.0.** The halo, the My language, the customization — members open it and know "something big happened."
- **It teaches gently.** Training modals carry first-time users through.
- **It points home.** The Back to the Vault button is bigger than necessary. On purpose.
- **It survives the rebuild.** Brand decisions don't get redone. Code can be replaced; the language and pattern can't.

Build this when ready. No rush. No deadline beyond Pickaxe. Get it right.
