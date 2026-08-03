# Surface reconciliation — live FlyVault vs. the rebuild HTML

Date: 2026-08-03
Owner: Brent Jones
Scope: `midge-ui/rebuild/` (12 HTML pages + 3 video loops) measured against the
live flyvault.net WordPress site (40 published pages).

## Why this document exists

The rebuild UI has been kept **deliberately separate** from the database work.
That separation is correct and intentional, not an oversight:

> The new front end has to mirror the knowledge graph. It cannot mirror a graph
> that has not been rebuilt yet. That constraint was established roughly nine
> months ago and is the reason Pickaxe had to go.

So the order is fixed — database first, then the join. This document is the
snapshot of where the two sides stand at the moment the merge question became
live, so it can be found again when the merge actually starts.

**This is not the current focus.** Current focus is the database. This is the
map for when we come back.

---

## Method

Live surface pulled from WordPress admin: 40 published pages, plus the active
Code Snippets, which reveal what is genuinely wired rather than merely present —
Pattern Search & Filter, My Fly Boxes, Community Calendar, Pattern Count on
Profile, Vault Count shortcode, FlyVault Midge Connector, and "change blog to My
Flies". The logged-in homepage resolves to the PeepSo Activity Stream.

Rebuild surface: the 12 HTML files in `midge-ui/rebuild/`, read by title and
structure.

---

## What is covered

Eight live surfaces have something standing in the rebuild folder:

| Live page | Rebuild file | Note |
|---|---|---|
| Fly Shop · My Fly Shop™ | `flyvaultshopspine.html` | single-shop view; the shop *directory* is not built |
| Groups | `flyvaultclubspine.html` | |
| Activity Stream · Members | `flyvaultcommunityspine.html` | partial — one page standing in for several |
| Blog Editor | `flyvaultblogcompose3.html` | Blog Feed and Blog Manager not built |
| My Account Profile | `flyvaultprofilefull.html` | |
| User Profile | `flyvaultryanstorchdemo.html` | public tyer view |
| Submit A Fly | `submitaflyv22.html` | |
| Fly Vault™ (browse) | `flynorthcountryspidersv2.html` | **partial** — a curated collection view, NOT the search-and-filter grid the live site runs |

## What is net-new

Four rebuild pages have no live counterpart at all. These are new product, not
ports:

- `materialrealv3.html` — Golden Plover material detail. The live site has no
  materials surface of any kind.
- `flyvaultlogentry.html` — The Log. Fishing logbook with condition autofill and
  MIDGE recall over the member's own history.
- `flyvaultimmersioncockpit.html` — Immersion cockpit.
- `midge_living_hero_auto.html` — Immersion living-water hero.

---

## What is missing, ranked by what blocks a launch

### 1. The money stack — nothing built

Seven live pages, zero rebuilt: Membership Levels, Membership Checkout,
Membership Confirmation, Membership Account, Membership Billing, Membership
Invoice, Membership Cancel. Paid Memberships Pro is carrying all of it today.
Nothing ships without this tier.

### 2. Identity — nothing built

App Login, Site Registration, Recover Password, Reset Password, Delete Account.
The roadmap's answer is the **Vault Door**, which does not exist in any file yet.

### 3. The "My ___" tier — nothing built, and this is the sharp one

My Flies, My Fly Boxes, Photos. All live. All have active code snippets behind
them. None rebuilt.

`midge-ui/README` calls the "My ___" language pattern an **architectural**
decision, not a cosmetic one, with a hard rule: *if a label says "My X," the link
MUST actually go to their X.* The rebuild currently has a Profile but nowhere for
a member's own flies or boxes to live. That is the emotional center of the
product and it is empty.

### 4. Knowledge surfaces — nothing built

Encyclopedia, PDF Vault. Both live. Both feed the **library track**, which was
deliberately deferred (see `inbound/TRACKS.md`) until the book corpus finishes
re-processing and dressing attribution closes out.

### 5. Communications — nothing built

Messages, Notifications, Community Calendar, Members directory. The Community
Layer page covers some of the activity/feed idea but none of these specifically.

### 6. Public / pre-login

About Fly Vault™, Landing Page. Nothing for a visitor who is not signed in.

### 7. MIDGE herself

`MIDGE AI` is a live page. `midge.html` exists in the `midge-ui` repo root — the
current chat UI, wired to Railway. It is NOT in the rebuild folder and has no
Immersion treatment.

---

## Immersion — the priority read

Immersion ranks above everything else on the UI side. The honest assessment:

**What exists is a hero and a cockpit. What the roadmap promises is a skin.
Those are different things.**

A skin is a system applied across every surface. Today the other ten rebuild
pages share a single look, and not one of them has an Immersion variant.

Three specific gaps:

1. **No theme system.** The roadmap names four themes — Dark Water, Chalk Stream,
   High Desert, Tailwater. Zero theme files exist. The good news: the twelve
   pages already run a shared palette (`--ink`, `--paper`, `--gold`,
   `--gold-deep`, `--oxblood`, `--green`, `--dark`, `--hair`), so the token layer
   is half-written whether or not it has been named as such.
2. **No Classic FlyVault skin.** The roadmap promises two skins. There is one
   look. With nothing to switch between, a skin selector has nothing to select.
3. **No Vault Door.** Pick your colors → MIDGE asks your name → she introduces
   herself → door animation, lock turns → "Welcome to the Vault" → tour or skip →
   returning users get "Welcome back, [name]." That sequence is the *doorway into*
   Immersion. Without it the living-water hero is a beautiful screen with nothing
   in front of it.

### Shortest path when Immersion work resumes

1. **Vault Door** — it is the entry point and the onboarding mechanism at once.
   The training-modal infrastructure in `midge.html` (the `TRAINING` object) was
   explicitly designed to become this.
2. **Extract the theme tokens** from what the twelve pages already share, and
   name the four themes against them.
3. **My Flies and My Fly Boxes** as the first surfaces to get both skins — they
   are what a member opens every day.

---

## The merge question

This is the part that is coming up next and it is a statement of fact, not a
plan: at some point the rebuilt UI has to join the rebuilt graph. Neither side
can be finished in isolation, because the UI has to know what shapes the graph
can express, and tonight's vault-vocabulary work (62 → 114 category terms, both
sides mirrored) is exactly the kind of thing that changes what a page can show.

Preparing for the incoming and seeing what is missing — which is what this
document is — is groundwork for that join. It is not the current task.

**Current task remains the database.**
