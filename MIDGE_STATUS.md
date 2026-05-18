# MIDGE 2.0 — Project Status
Last updated: May 18, 2026 (USGS subagent shipped)

---

## ⚡ STATUS AT END OF DAY

Everything is committed, pushed, and deployed. **Midge 2.0 is live with all of today's fixes including anti-hallucination grounding.**

Latest commit: `fe535aa` (Anti-hallucination grounding rules + bigger KB snippets)

Quick verify tomorrow morning:
```
curl -s https://midge-backend-production.up.railway.app/health
```
Should return `{"status":"ok"}`.

---

## ✅ DONE TODAY (May 18 — USGS subagent)

### USGS Water Conditions subagent (NEW tool, 4th in the chain)
- New module `usgs_water.py` — fault-isolated USGS Water Services client. Any timeout/parse error returns `{"status": "error", ...}` instead of bubbling into the tool loop. **A USGS outage cannot break Midge.**
- Three entry points: `get_conditions(site_id)`, `find_sites(river, state)`, `resolve_and_fetch(query)` — the model can pass either a site number OR a free-text river name.
- **Solves issue #4 (White River disambiguation):** when a river name matches multiple gauges, returns `status="ambiguous"` with candidates labeled by state. The system prompt now treats this like the FlyVault "possible matches (ask for clarification)" path — Midge asks the user before answering.
- **Replaces Tavily for flow/temp/gauge questions.** TOOLS_GUIDANCE updated: "use get_water_conditions for raw numbers, search_web for hatch reports / shop intel only." Tavily was unreliable for live numerics; USGS is the authority.
- 18 unit tests in `tests/test_usgs_water.py` — all mocked, run offline, cover happy path, malformed responses, timeouts, sentinel values (-999999), ambiguous matches, and a fault-isolation guarantee test that confirms no exception can escape the module.
- Live smoke-test script at `tools/smoke_test_usgs.py` — verify against the real API from local before relying on it in production.
- **No env vars needed.** USGS is free and unauthenticated.

### Architecture note — second subagent
This is the second subagent we've discussed (first being the proposed BookSearch subagent for hallucination prevention from yesterday's status). Same fault-isolation pattern: structured returns, never raises into Midge's main loop. If we like how this one behaves under tester load, the BookSearch one becomes much easier to justify.

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

### 1. Occasional hallucinations on edge queries
GPT-4o-mini still infers/fills gaps when it shouldn't. Today's prompt fix tightens this but won't eliminate it. **Tomorrow's job:** consider dedicated BookSearch subagent that pre-extracts verifiable quotes before the main model writes.

### 2. Multi-turn pronouns still depend on first turn being right
If turn 1 fails to call tools, turn 2 follow-ups inherit the gap. The aggressive tool-calling prompt mitigates this but isn't bulletproof.

### 3. Lazy first-turn tool calls
The model sometimes answers from memory on the first turn (sources_used: 0). New prompt should fix most cases — verify with testers tomorrow.

### 4. White River / ambiguous location names — ADDRESSED (verify with testers)
USGS subagent shipped today handles this for FLOW/TEMP questions: returns multiple candidates labeled by state and the system prompt asks the user which one they mean. Doesn't help Tavily's hatch-report disambiguation yet — still an open problem for non-numeric questions about ambiguous rivers.

---

## 🔲 TOMORROW — IN ORDER

1. **Push the pending commit** (one-liner above)
2. **Test with real users** — let testers hammer it overnight, see what breaks
3. **Sub-agent architecture decision** — based on real test data, decide if we build BookSearch sub-agent for hallucination prevention
4. **Add CJ to GitHub + Railway** — need her GitHub username
5. **Stripe setup** — postponed; ship to testers free first
6. **Status file recap** — review what testers asked, prioritize

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
git reset --hard fe535aa    # CURRENT LIVE — anti-hallucination + aggressive tool calls
git reset --hard a5feac4    # before anti-hallucination prompt (Kelson fix only)
git reset --hard b143b33    # post-em-dash fix, before aggressive tool rules
git reset --hard 80d7b05    # original OpenRouter swap, conservative tool calling
git reset --hard d22675a    # pre-OpenRouter, on Groq (full rollback to yesterday)
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
- Authoritative USGS data for flow/temp/gauge on named rivers (fault-isolated subagent)

**What Midge IS NOT yet:**
- Multi-agent (single model does all reasoning)
- Resilient to all hallucinations (prompt rules help, not solve)
- Charging anyone (Stripe deferred)
- Gated by login (open access)

**The architectural debate to revisit:**
Brent and Sonnet discussed "Midge as harness — BYO model" as a future direction. Tabled for now. Path forward = sharpen single-agent product, gather real user data, decide architecture changes based on what testers actually ask and where Midge actually fails.

---

Good build day. Sleep well.
