# MIDGE 2.0 — Project Status
Last updated: May 18, 2026 (USGS subagent — shipped, debugged in prod, verified live)

---

## ⚡ STATUS AT END OF DAY

USGS Water Conditions subagent is **live, deployed, and verified end-to-end in production.** Got it working through three iterations today: initial ship → caught real parser bugs against the live API → caught a follow-up turn bug from Railway logs → final prompt fix landed and a live 3-message conversation returned real Madison River numbers (1010 CFS, 50.7°F at Ennis) with no hallucinations.

**Latest commit:** `b6148cc` (Tighten USGS tool guidance + add INFO logging)

Backend health check:
```
curl -s https://midge-backend-production.up.railway.app/health
```
Returns `{"status":"ok"}`.

To pull Railway logs from `~/Desktop/midge-backend`:
```
railway logs --service midge-backend | tail -80
```

---

## ✅ DONE TODAY (May 18 — USGS subagent: ship → debug → verify)

### USGS Water Conditions subagent (4th tool in the chain)
- New module `usgs_water.py` — fault-isolated USGS Water Services client. Any timeout/parse error returns `{"status": "error", ...}` instead of bubbling into the tool loop. **A USGS outage cannot break Midge.**
- Three entry points: `get_conditions(site_id)`, `find_sites(river, state)`, `resolve_and_fetch(query)` — the model can pass either a site number OR a free-text river name.
- **Solves issue #4 (White River disambiguation):** when a river name matches multiple gauges, returns `status="ambiguous"` with candidates labeled by state. The system prompt now treats this like the FlyVault "possible matches (ask for clarification)" path — Midge asks the user before answering.
- **Replaces Tavily for flow/temp/gauge questions.** TOOLS_GUIDANCE updated: "use get_water_conditions for raw numbers, search_web for hatch reports / shop intel only." Tavily was unreliable for live numerics; USGS is the authority.
- **39 unit tests** in `tests/test_usgs_water.py` — all mocked, run offline, cover happy path, malformed responses, timeouts, sentinel values (-999999), ambiguous matches, location parsing, sticky-gauge, and a fault-isolation guarantee test that confirms no exception can escape the module.
- Live smoke-test script at `tools/smoke_test_usgs.py` plus three debug scripts (`debug_usgs_raw.py`, `debug_usgs_expanded.py`, `debug_test4.py`) that proved out the real API contract.
- **No env vars needed.** USGS is free and unauthenticated.

### Real bugs caught against the live USGS API (these would have shipped without the smoke test)
- `state_cd` only returns when `siteOutput=expanded` is passed — without it every candidate had empty state. **Fixed.**
- `state_cd` is a FIPS numeric code (`30` = MT), not a 2-letter abbreviation. Added a full 50-state FIPS→abbrev map.
- USGS requires a "major filter" (state, HUC, bbox, or county) alongside `siteName` — bare-name searches return HTTP 400. Now we catch this BEFORE firing and return `status="needs_state"` with a hint for the model to ask the user.
- HTTP 404 means "no matches," not "service down." Was being misreported as a network error. Now returns `not_found` cleanly.
- Noise-word filter was leaking function words like `for`, `of`, `in`. USGS rejects `siteName=for madison` with 404 even though `siteName=madison` works. Expanded the stopword list to cover function words.

### Geographic location labels (no more bare site numbers in clarification questions)
- Parser extracts the human-readable location from station_nm (e.g. `Madison River near West Yellowstone, MT` → `Near West Yellowstone, MT`).
- USGS abbreviations expanded: `bl` → below, `nr` → near, `R` → River, `ck` → Creek, etc.
- Candidate output is structured `LOCATION: ... | SITE_ID: ...` so the model can't miss what to show the user vs. what to pass on the next call.
- Renders for users as "Are you fishing near West Yellowstone, below Hebgen Lake, at Kirby Ranch, near Cameron, or at Ennis?" — natural fishing language, never site numbers.

### Sticky-gauge per session (mirrors sticky-tyer pattern)
- `_STICKY_GAUGES` dict in `usgs_water.py` pins a site to a session_id with 30-minute TTL — matches main.py's existing `_SESSION_STATE` TTL.
- `session_id` is threaded `main.py → ask_with_tools → execute_tool → resolve_and_fetch`.
- Auto-pins when (a) the model passes an explicit site number, or (b) find_sites returns exactly one match.
- Frontend was already sending `session_id` in the POST body — verified in `midge.html` line 678 (`SESSION_ID` from `sessionStorage`).

### Prod debugging round (caught after first deploy)
First live test through the UI showed the model failing on turn 2: "Couldn't find any current flow data" for a Madison gauge that the smoke test had just pulled cleanly. Worse, on turn 3 it hallucinated "mid to high 50s as we transition into fall" — in May.

**Root causes from Railway logs:**
1. Model was re-searching by river name on follow-ups instead of using the SITE_ID from the previous turn's tool result. Cycle: name search → 404 (because "Madison River at Ennis" with all those words doesn't match USGS's strict tokenizer) → max_iterations exhausted → "no data."
2. Anti-hallucination rules in the system prompt covered books and patterns, NOT live numeric data. The model treated tool failure as a license to fabricate.
3. Python's default logging level was WARNING, which dropped the `MIDGE debug | tools_used=...` lines. We were debugging blind.

**Fixes (commit b6148cc):**
- Added `logging.basicConfig(level=INFO)` at the top of `main.py` so debug lines reach Railway logs.
- Tightened `get_water_conditions` tool description: explicit "on follow-up turns, pass the 8-digit SITE_ID, not the river name."
- Restructured the ambiguous-candidates output: each line now reads `LOCATION: Near West Yellowstone, MT  |  SITE_ID: 06037500  |  (state=MT, full USGS name: "...")`.
- Added a new anti-hallucination rule for live data: "Never invent flow numbers, water temps, or 'typical seasonal' values. Phrases like 'typically in the mid-50s for this time of year' are HALLUCINATIONS unless pulled from a tool result. NEVER guess at the season."
- Added query/state context to the 404 warning so future failures show us the exact siteName the model passed.

### Live verification (passed 100%)
After the b6148cc deploy, replayed the same 3-message conversation through the UI:
1. *"What's the flow on the Madison River in Montana?"* → Midge listed all 5 Madison gauges by location, asked which one. ✅
2. *"The one near Ennis."* → Model called `get_water_conditions(query='06040050')` — **passed the site number directly**, ONE httpx call straight to the IV endpoint. Returned **1010 CFS, 4.47 ft gauge, 50.7°F water temp**. Real live data. ✅
3. *"What about water temp?"* → Model answered from conversation memory (`tools_used=[]`), said "50.7°F." No re-disambiguation, no fabrication. ✅

Railway log confirmation:
```
MIDGE debug | question='he one near Ennis.' | tools_used=[{'name': 'get_water_conditions', 'query': '06040050'}]
```
That `query='06040050'` (not `'Madison River at Ennis'`) is the moment we knew the prompt fix worked.

### Architecture note — what we DIDN'T build
Considered building a deterministic state-parser in `main.py` to intercept candidate-selection messages and pre-populate sticky-gauge without involving the model. Decided not to — production proved the prompt fix is sufficient. Can revisit if testers find a case where the model still flubs the handoff.

### Bumps to issue list
- **Issue #4 (White River / ambiguous location names):** addressed for flow/temp questions. Still open for Tavily hatch reports.
- **Issue #1 (hallucinations on edge queries):** anti-hallucination rules now cover three layers (books, patterns, live data). Should reduce frequency.

---

## ✅ DONE YESTERDAY (6 hours of work)

### Architecture rebuild — Groq → OpenRouter + tool-calling
- Killed Groq (rate-limited to death on free tier)
- Switched to **OpenRouter** — one API key, any model. Currently using `openai/gpt-4o-mini`. Swap models by changing `MIDGE_MODEL` env var in Railway, no code changes.
- Rewrote `llm.py` with **tool-calling architecture** — Midge decides which tools to call based on the question
- Three tools wired up:
  1. `search_flyvault` — community pattern vault (1,604 patterns)
  2. `search_knowledge_base` — 230+ fly-fishing books via Supabase pgvector
  3. `search_web` — Tavily live web search
- Stripped keyword-based routing from `main.py` — model picks tools, not regex

### Bugs fixed (in order today)
- UTF-8 encoding crash on em-dashes (request body)
- UnicodeEncodeError on em-dash in X-Title header (the real culprit, found by Claude Code)
- Detailed error logging added to OpenRouter calls (visible in Railway logs)
- Lazy tool calling — model was answering from training instead of searching
- KB tool returning too-small snippets (bumped from 600 → 1,100 chars, 5 → 6 results)
- Anti-hallucination prompt rules (must quote literally from search results)

### Ship verification (all green)
- Backend `/health` → 200 OK
- `/ask` endpoint returns real structured answers with vault images
- FlyVault.net "MIDGE AI" nav link points at live Netlify URL
- Frontend → backend wiring verified
- Tested live with real questions (San Juan Worm, Kelson techniques, Madison patterns) — all returned real, grounded content

---

## 🔴 KNOWN ISSUES (NOT BLOCKING — for tomorrow)

### 1. Occasional hallucinations on edge queries — IMPROVED, not solved
GPT-4o-mini still infers/fills gaps when it shouldn't. Today's anti-hallucination rules now cover three categories (books, patterns, live data). The live-data rule was added after catching "mid 50s in fall" in production — the model invented numbers AND a season. New rule explicitly bans both. **Tomorrow's job:** consider dedicated BookSearch subagent that pre-extracts verifiable quotes before the main model writes.

### 2. Multi-turn pronouns still depend on first turn being right
If turn 1 fails to call tools, turn 2 follow-ups inherit the gap. The aggressive tool-calling prompt mitigates this but isn't bulletproof.

### 3. Lazy first-turn tool calls
The model sometimes answers from memory on the first turn (sources_used: 0). New prompt should fix most cases — verify with testers tomorrow.

### 4. White River / ambiguous location names — ADDRESSED (verify with testers)
USGS subagent shipped today handles this for FLOW/TEMP questions: returns multiple candidates labeled by state and the system prompt asks the user which one they mean. Doesn't help Tavily's hatch-report disambiguation yet — still an open problem for non-numeric questions about ambiguous rivers.

---

## 🔲 TOMORROW — IN ORDER

1. **Check Railway logs for tester traffic** — `railway logs --service midge-backend | tail -200`. Look for `MIDGE debug` lines to see what people asked and which tools fired. Specifically watch for: (a) USGS queries that 404'd (the new warning logs include the cleaned siteName + state), (b) follow-up turns where the model re-searched by name instead of passing a SITE_ID, (c) any new hallucination patterns the prompt rules didn't catch.
2. **Test with real users** — testers got the link last night; expect overnight feedback.
3. **Sub-agent architecture decision** — now that we've shipped one and validated the pattern, decide if BookSearch subagent is worth building for hallucination prevention. The USGS subagent took one session of focused work; BookSearch should be similar effort.
4. **Add CJ to GitHub + Railway** — need her GitHub username.
5. **Stripe setup** — postponed; ship to testers free first.
6. **Status file recap** — review what testers asked, prioritize.

---

## 💰 MONTHLY COSTS

| Service | Plan | Cost |
|---------|------|------|
| Netlify | Free | $0 |
| Railway | Pro | $20/mo |
| OpenRouter | Pay-as-you-go | ~$1-5/mo (est) |
| Tavily | Free | $0 |
| Supabase | Free | $0 |
| Groq | CANCELLED | $0 |
| Pickaxe | CANCELLED | $0 |
| **Total** | | **~$21-25/mo** |

---

## 🔑 KEY URLS

- **Live app:** https://dazzling-treacle-f469a4.netlify.app/midge.html
- **FlyVault entry:** https://flyvault.net → click MIDGE AI
- **Backend:** https://midge-backend-production.up.railway.app
- **GitHub frontend:** https://github.com/brenttiesflies-sys/midge-ui
- **GitHub backend:** https://github.com/brenttiesflies-sys/midge-backend
- **Railway dashboard:** https://railway.com/dashboard
- **OpenRouter:** https://openrouter.ai/

---

## 🛠 DEV ENVIRONMENT

- **Railway CLI:** installed, linked to `just-magic` → `production` → `midge-backend`
- **Claude Code:** installed in terminal, logged in as brenttiesflies@gmail.com
  - Restart with `claude --dangerously-skip-permissions` for zero-prompt mode
- **Local repos:**
  - `~/Desktop/midge-backend` — Python FastAPI backend
  - `~/Desktop/MIDGE-2.0-UI` — frontend (HTML/JS, deploys to Netlify on push)

---

## 👥 TEAM

- **Brent** — brenttiesflies@gmail.com — GitHub: brenttiesflies-sys
- **CJ (Cassandra)** — cj22free@gmail.com — GitHub username: TBD (needs to send)
- **Testers** — sent updated link tonight; expect feedback overnight

---

## 🆘 ROLLBACK SAFETY

If anything tomorrow goes sideways, roll back to known-good state:
```
cd ~/Desktop/midge-backend
git reset --hard b6148cc    # CURRENT LIVE — USGS subagent fully verified (prompt fix + INFO logging)
git reset --hard 2343b29    # USGS subagent initial ship (before prompt-fix patch)
git reset --hard fe535aa    # pre-USGS — anti-hallucination + aggressive tool calls (3 tools)
git reset --hard a5feac4    # before anti-hallucination prompt (Kelson fix only)
git reset --hard b143b33    # post-em-dash fix, before aggressive tool rules
git reset --hard 80d7b05    # original OpenRouter swap, conservative tool calling
git reset --hard d22675a    # pre-OpenRouter, on Groq (full rollback)
git push --force origin main
```

---

## 📓 ARCHITECTURE NOTES FOR FRESH CHAT

When picking up tomorrow, the key context is:

**What Midge IS now:**
- Tool-calling agent (model picks vault/books/USGS/web — 4 tools)
- Backed by GPT-4o-mini via OpenRouter (swappable)
- Returns real, grounded answers with vault images
- Pulls Kelson, Marbury, Pritt, and 230+ other authors from the KB
- Authoritative USGS data for flow/temp/gauge on named rivers (fault-isolated subagent, verified live with 1010 CFS on the Madison at Ennis)
- Sticky-gauge per session — once the user picks a Madison gauge, follow-ups don't re-disambiguate
- Three-layer anti-hallucination protection (books, patterns, live data)

**What Midge IS NOT yet:**
- Multi-agent (single model does all reasoning)
- Resilient to all hallucinations (prompt rules help, not solve)
- Charging anyone (Stripe deferred)
- Gated by login (open access)

**The architectural debate to revisit:**
Brent and Sonnet discussed "Midge as harness — BYO model" as a future direction. Tabled for now. Path forward = sharpen single-agent product, gather real user data, decide architecture changes based on what testers actually ask and where Midge actually fails.

---

Good build day. Sleep well.
